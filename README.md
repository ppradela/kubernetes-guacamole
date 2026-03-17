# Apache Guacamole on Kubernetes — Production-Ready Deployment

> A complete Kubernetes deployment for Apache Guacamole with PostgreSQL backend, TLS termination via nginx Ingress, and idempotent schema initialization. Built and tested on a bare-metal homelab cluster.

![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28%2B-blue?logo=kubernetes)
![Guacamole](https://img.shields.io/badge/Guacamole-1.6.0-orange)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?logo=postgresql)
![License](https://img.shields.io/badge/License-MIT-green)

---

### Features

- **Single-apply deployment** — one `kubectl apply -f guacamole.yaml` brings up the entire stack
- **PostgreSQL backend** — persistent storage with a `hostPath` PV; swap for any `ReadWriteOnce` provisioner
- **Idempotent schema init** — a Kubernetes Job generates and applies the Guacamole schema only if it does not already exist, safe to re-run
- **nginx Ingress with TLS** — HTTPS termination with cookie-based session affinity for stable WebSocket connections
- **Real client IP forwarding** — `REMOTE_IP_VALVE_ENABLED` passes the original client IP through the nginx proxy
- **Readiness sequencing** — `initContainers` ensure PostgreSQL is up before the init Job runs, and `guacd` is up before the Guacamole client starts
- **Modular manifests** — individual files under `manifests/` for per-component management alongside the all-in-one file

---

### Architecture

```
Browser (HTTPS :443)
    │
    └── nginx Ingress  (guacamole.k8s.lab)
          │  rewrite: /(.*)  →  /guacamole/$1
          │  TLS: guacamole-tls Secret
          │
          └── Service: guacamole :8080
                │
                └── Deployment: guacamole-client  (guacamole/guacamole:1.6.0)
                      │  env: GUACD_HOSTNAME, POSTGRESQL_HOSTNAME, ...
                      │
                      ├── Service: guacd :4822
                      │     └── Deployment: guacd  (guacamole/guacd:1.6.0)
                      │
                      └── Service: postgresql :5432
                            └── Deployment: postgresql  (postgres:16)
                                  └── PVC: postgresql-pvc  →  PV: postgresql-pv (hostPath)

Job: guacamole-init-db  (runs once, idempotent)
  initContainer: wait-for-postgresql  (busybox, nc probe)
  initContainer: generate-guacamole-schema  (guacamole image, initdb.sh --postgresql)
  container:     apply-guacamole-schema  (postgres:16, psql)
```

| Component | Image | Port |
|---|---|---|
| guacamole-client | `guacamole/guacamole:1.6.0` | 8080 |
| guacd | `guacamole/guacd:1.6.0` | 4822 |
| postgresql | `postgres:16` | 5432 |

---

### File Structure

```
kubernetes-guacamole/
├── manifests/
│   ├── 00-namespace.yaml       # Namespace: guacamole
│   ├── 01-secrets.yaml         # PostgreSQL password Secret
│   ├── 02-postgresql.yaml      # PV, PVC, Deployment, Service
│   ├── 03-init-db.yaml         # Schema initialization Job
│   ├── 04-guacd.yaml           # guacd Deployment and Service
│   ├── 05-guacamole.yaml       # Guacamole client Deployment and Service
│   └── 06-ingress.yaml         # TLS Secret and nginx Ingress
└── guacamole.yaml              # All-in-one manifest (all of the above combined)
```

---

### Prerequisites

- **Ingress-Nginx Controller** — exposes the Guacamole web interface over HTTPS
  - [Ingress-Nginx Installation Guide](https://kubernetes.github.io/ingress-nginx/deploy/)
- **MetalLB** (bare-metal only) — provides a `LoadBalancer` IP for the Ingress controller
  - [MetalLB Installation Guide](https://metallb.io/installation/)
- **Persistent Volume provisioner** — a `ReadWriteOnce`-compatible provisioner. The manifests use `storageClassName: manual` with a `hostPath` PV; replace with your cluster's provisioner for multi-node setups.

---

### Deployment

#### Option A — All-in-one

```bash
kubectl apply -f guacamole.yaml
```

#### Option B — Per-component

```bash
kubectl apply -f manifests/
```

#### Verify

```bash
kubectl get pods    -n guacamole
kubectl get svc     -n guacamole
kubectl get ingress -n guacamole
kubectl get job     -n guacamole
```

Wait for the `guacamole-init-db` Job to reach `Completed` before logging in.

---

### Before You Deploy — Required Changes

| What | Where | How |
|---|---|---|
| PostgreSQL password | `01-secrets.yaml` → `postgresql-password` | `echo -n 'your-password' \| base64` |
| TLS certificate | `06-ingress.yaml` → `tls.crt` / `tls.key` | `base64 -w 0 < tls.crt` |
| Ingress hostname | `06-ingress.yaml` → `host` / `tls.hosts` | Replace `guacamole.k8s.lab` |
| PV host path | `02-postgresql.yaml` → `hostPath.path` | Set to an existing directory on your node |

For the all-in-one file, the same fields apply in `guacamole.yaml`.

---

### Accessing Guacamole

Once all pods are running and the Ingress is configured, open:

```
https://<your-domain>
```

Default credentials:

| Username | Password |
|---|---|
| `guacadmin` | `guacadmin` |

> **Change the default password immediately after first login.**

---

### Customization

- **Scale** — increase `replicas` on `guacd` and `guacamole-client` for higher availability
- **Images** — update image tags to a newer Guacamole release
- **Resources** — add `resources.requests` / `resources.limits` to all containers
- **Storage** — replace the `hostPath` PV with a shared provisioner (NFS, Longhorn, Ceph) for multi-node clusters
- **Authentication** — extend Guacamole with LDAP, SAML, or OIDC via [Guacamole extensions](https://guacamole.apache.org/doc/gug/guacamole-ext.html)
- **Session timeout** — adjust `session-cookie-expires` and `session-cookie-max-age` on the Ingress annotations

---

### License

MIT

---

### Author

**Przemysław Pradela**
- GitHub: [@ppradela](https://github.com/ppradela)
- LinkedIn: [linkedin.com/in/przemyslaw-pradela](https://www.linkedin.com/in/przemyslaw-pradela)
- Website: [pradela.ovh](https://pradela.ovh)
