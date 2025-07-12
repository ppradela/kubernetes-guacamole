# [Apache Guacamole](https://guacamole.apache.org/) Kubernetes Deployment

This repository contains Kubernetes manifests to deploy Apache Guacamole into your Kubernetes cluster.

## Prerequisites

Before deploying Guacamole, ensure the following components are installed and configured in your Kubernetes cluster:

* **MetalLB:** A bare metal load-balancer for Kubernetes, essential for exposing services externally if you are not using a cloud provider with native LoadBalancer support.
  * [MetalLB Installation Guide](https://metallb.io/installation/)
* **Ingress-Nginx Controller:** An Ingress controller that uses NGINX as a reverse proxy and load balancer. This is used to expose the Guacamole web interface via an Ingress resource.
  * [Ingress-Nginx Installation Guide](        https://kubernetes.github.io/ingress-nginx/deploy/)
* **Persistent Volume Provisioner:** Ensure you have a storage class and a persistent volume provisioner configured that can satisfy `ReadWriteOnce` access mode for the PostgreSQL database. The provided `guacamole.yaml` uses `storageClassName: manual`, implying you might need to manually provision the `postgresql-pv` or adjust the `PersistentVolumeClaim` to use an existing storage class in your environment.

## Deployment

The `guacamole.yaml` file contains all the necessary Kubernetes resources to deploy Apache Guacamole, including:

* `Namespace`: `guacamole`
* `Secret`: For PostgreSQL database password and TLS certificates for Ingress.
* `PersistentVolume` and `PersistentVolumeClaim`: For PostgreSQL data persistence.
* `Deployment`: For PostgreSQL, `guacd` (Guacamole daemon), and `guacamole-client` (web application).
* `Service`: For PostgreSQL, `guacd`, and `guacamole-client` to enable internal communication.
* `Job`: To initialize the Guacamole database schema in PostgreSQL.
* `Ingress`: To expose the Guacamole web interface externally using `nginx.ingress.kubernetes.io` annotations for rewrite rules, SSL redirection, and session affinity.

### Steps to Deploy:

1. **Review and Adjust `guacamole.yaml`:**
    * **TLS Secret:** The `cloudflare-tls` secret in the `guacamole.yaml` contains base64 encoded `tls.crt` and `tls.key`. **You MUST replace these with your own TLS certificate and key for your domain.**
        * To generate base64 encoded values:

            ```bash
            cat your_domain.crt | base64 --wrap=0
            cat your_domain.key | base64 --wrap=0
            ```

        * Update the `data` section of the `cloudflare-tls` secret with your generated base64 strings.
    * **Ingress Host:** Update `guacamole.k8s.lab` in the `Ingress` resource to your desired hostname.
    * **PostgreSQL Password:** The `postgresql-secret` contains a base64 encoded password (`Z3VhY2Ftb2xlCg==` which decodes to `guacamole`). You can change this to a stronger password by base64 encoding your new password.
2. **Apply the Manifest:**

    ```bash
    kubectl apply -f guacamole.yaml
    ```

3. **Verify Deployment:**
    Monitor the deployment status of pods and services:

    ```bash
    kubectl get pods -n guacamole
    kubectl get svc -n guacamole
    kubectl get ingress -n guacamole
    kubectl get job -n guacamole
    ```

    Ensure the `guacamole-init-db` job completes successfully.

### Accessing Guacamole

Once all pods are running and the Ingress is configured, you can access Guacamole via the hostname you specified in the Ingress resource (e.g., `https://guacamole.k8s.lab`).

The default login credentials after initialization are typically `guacadmin`/`guacadmin`. **It is highly recommended to change these immediately after your first login.**

## Customization

* **Replicas:** Adjust the `replicas` count for `guacd` and `guacamole-client` deployments based on your expected load.
* **Guacamole Version:** Update the `image` tags for `guacamole/guacamole` and `guacamole/guacd` to a desired version.
* **PostgreSQL Version:** Update the `image` tag for `postgres` if you wish to use a different PostgreSQL version.
* **Resource Limits:** Consider adding `resources` limits and requests to your deployments for better resource management.
* **Authentication:** Guacamole supports various authentication methods (LDAP, SAML, OpenID Connect, etc.). You can configure these by adding appropriate extensions and environment variables to the `guacamole-client` deployment.
