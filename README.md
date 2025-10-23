# 🥑 Apache Guacamole on Kubernetes

This guide provides step-by-step instructions to deploy **[Apache Guacamole](https://guacamole.apache.org/)** in a Kubernetes cluster.

---

## 📑 Table of Contents
1. [Introduction](#-introduction)  
2. [Prerequisites](#-prerequisites)  
3. [Deployment](#-deployment)  
4. [Accessing Guacamole](#-accessing-guacamole)  
5. [Customization](#-customization)  
6. [Author](#-author)

---

## 📝 Introduction
This repository contains Kubernetes manifests to deploy Apache Guacamole into your Kubernetes cluster. The deployment includes PostgreSQL, Guacamole Daemon (`guacd`), and the Guacamole client (web application).

---

## ⚙️ Prerequisites
Before deploying Guacamole, make sure your cluster has:

- **MetalLB:** A bare-metal load balancer for Kubernetes (required if not using a cloud provider).
  - [MetalLB Installation Guide](https://metallb.io/installation/)
- **Ingress-Nginx Controller:** Used to expose the Guacamole web interface.
  - [Ingress-Nginx Installation Guide](https://kubernetes.github.io/ingress-nginx/deploy/)
- **Persistent Volume Provisioner:** Ensure a `ReadWriteOnce` compatible provisioner is configured. The provided manifests use `storageClassName: manual`.

---

## 🚀 Deployment
The file `guacamole.yaml` contains all resources:
- `Namespace`: `guacamole`
- `Secrets`: PostgreSQL password & TLS certificates for Ingress
- `PersistentVolume` & `PersistentVolumeClaim`: PostgreSQL storage
- `Deployments`: PostgreSQL, `guacd`, `guacamole-client`
- `Services`: For internal communication
- `Job`: Initializes PostgreSQL schema
- `Ingress`: Exposes the web interface

### Steps to Deploy

1. **Review and Adjust `guacamole.yaml`:**
   - **TLS Secret:** Replace `tls.crt` and `tls.key` in `guacamole-tls` with your own, base64 encoded.
   - **Ingress Host:** Update `guacamole.k8s.lab` to your domain.
   - **PostgreSQL Password:** Update the `postgresql-secret` with a strong password (base64 encoded).
   - **Storage:** Adjust `postgresql-pv` hostPath or configure shared storage for multi-node clusters.

2. **Apply the Manifest:**
   ```bash
   kubectl apply -f guacamole.yaml
   ```

3. **Verify Deployment:**
   ```bash
   kubectl get pods -n guacamole
   kubectl get svc -n guacamole
   kubectl get ingress -n guacamole
   kubectl get job -n guacamole
   ```
   Ensure `guacamole-init-db` completes successfully.

---

## 🌐 Accessing Guacamole
Once all pods are running and Ingress is configured, access the web UI at:
```
https://<your-domain>
```

Default credentials:
- **Username:** `guacadmin`
- **Password:** `guacadmin`

⚠️ **Change the default password immediately after login.**

---

## 🎛️ Customization
- **Replicas:** Scale `guacd` and `guacamole-client` deployments.
- **Versions:** Update container image tags.
- **Resources:** Add CPU/memory requests and limits.
- **Authentication:** Extend with LDAP, SAML, OIDC, etc. via Guacamole extensions.

---

## 👤 Author
**Przemyslaw Pradela**  
- 💼 GitHub: [@ppradela](https://github.com/ppradela)  
- ✉️ Email: [przemyslaw.pradela@pradela.ovh](mailto:przemyslaw.pradela@pradela.ovh?subject=Apache%20Guacamole%20Kubernetes%20Deployment)  
- 🔗 LinkedIn: [linkedin.com/in/przemyslaw-pradela](https://www.linkedin.com/in/przemyslaw-pradela)
