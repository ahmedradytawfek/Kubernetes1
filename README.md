# 🚀 Kubernetes NGINX APIs with Ingress

This project demonstrates how to deploy two independent NGINX-based APIs into a Kubernetes cluster and expose them via a single NGINX **Ingress Controller**.

API1 → /api1/ → returns Hello from API 1

API2 → /api2/ → returns Hello from API 2

Both applications are accessible through the same Ingress endpoint: **http://mcs.com**.


---
📡 Cluster Architecture
*Cluster Setup

- 3 Ubuntu VMs (1 control-plane, 2 workers)

- Kubernetes v1.30.14 with containerd runtime

- NGINX Ingress Controller for routing

* Request Flow
  Client → Ingress Controller → Service → Pod (nginx container)

----
## 📂 Files

| File                      | Purpose                                |
|---------------------------|----------------------------------------|
| `api1.yaml`               | Deployment + HTML ConfigMap for API1   |
| `api2.yaml`               | Deployment + HTML ConfigMap for API2   |
| `configmap1.yaml`         | NGINX config for API1                  |
| `configmap2.yaml`         | NGINX config for API2                  |
| `svc1.yaml`               | ClusterIP Service for API1             |
| `svc2.yaml`               | ClusterIP Service for API2             |
| `ingress-nginx.yml`       | Ingress configuration                  |

---

## ⚙️ How It Works

### 1. ConfigMaps (`configmap1.yaml`, `configmap2.yaml`)
Provide custom `nginx.conf` files:

- **API1** → Listens on port `8000`, serves at `/api1`
- **API2** → Listens on port `8080`, serves at `/api2`

The `alias` directive ensures `/usr/share/nginx/html/index.html` is served when requesting `/api1/` or `/api2/`.

---

### 2. Deployments (`api1.yaml`, `api2.yaml`)
- Use the official `nginx:latest` image
- Mount:
  - **HTML content (`index.html`)** from ConfigMap
  - **Full NGINX config (`nginx.conf`)** from ConfigMap

---

### 3. Services (`svc1.yaml`, `svc2.yaml`)
ClusterIP type (only accessible inside the cluster):

- **API1**: forwards port `80 → 8000`
- **API2**: forwards port `80 → 8080`

---

### 4. Ingress (`ingress-nginx.yml`)
- Uses the **NGINX Ingress Controller**
- Routes traffic:
  - `/api1` → API1 Service
  - `/api2` → API2 Service

---

## 🚀 Deployment

Apply all manifests:

```bash
kubectl apply -f configmap1.yaml
kubectl apply -f configmap2.yaml
kubectl apply -f api1.yaml
kubectl apply -f api2.yaml
kubectl apply -f svc1.yaml
kubectl apply -f svc2.yaml
kubectl apply -f ingrss-nginx.yml


Check resources:

kubectl get pods
kubectl get svc
kubectl get ingress

🌐 Accessing the Services

Your ingress controller is exposed via NodePort (nic-nginx-ingress-controller):

kubectl get svc -n nginx-ingress


Example output:

NAME                           TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
nic-nginx-ingress-controller   NodePort   10.110.200.140   <none>        80:31194/TCP,443:32277/TCP   5d


NodePort 31194 → HTTP

NodePort 32277 → HTTPS

Test with curl

Add the Host header (mcs.com):

curl -H "Host: mcs.com" http://<NodeIP>:31194/api1/
curl -H "Host: mcs.com" http://<NodeIP>:31194/api2/


✅ Expected responses:

Hello from API 1

Hello from API 2

🛠 Troubleshooting
301 Redirects

Requesting /api1 without a trailing slash may cause a redirect to /api1/.

Fix: Use /api1/ or update nginx.conf to handle both paths.

404 Not Found

Path does not match Ingress rules.

Check: Service ports (targetPort: 8000 / 8080) and Ingress paths.

502 Bad Gateway

Usually a Service port mismatch.

Check: Deployment → Service → Ingress port consistency.

✅ Summary

API1 available at: http://mcs.com:31194/api1/

API2 available at: http://mcs.com:31194/api2/
