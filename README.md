#This project deploys two independent NGINX-based APIs (API1 and API2) into Kubernetes and exposes them via an NGINX Ingress controller.

API1 → serves Hello from API 1 at /api1

API2 → serves Hello from API 2 at /api2

#Both applications are exposed through the same Ingress endpoint.

📂 Files
File	Purpose
api1.yaml	Deployment and HTML ConfigMap for API1
api2.yaml	Deployment and HTML ConfigMap for API2
configmap1.yaml	NGINX config for API1
configmap2.yaml	NGINX config for API2
svc1.yaml	ClusterIP Service for API1
svc2.yaml	ClusterIP Service for API2
test.yml	Ingress configuration
⚙️ How it works
1. ConfigMaps (configmap1.yaml, configmap2.yaml)

Provide custom nginx.conf files for each API:

API1 → listens on port 8000, serves at /api1

API2 → listens on port 8080, serves at /api2

alias is used so that /usr/share/nginx/html/index.html is served when requesting /api1/ or /api2/.

2. Deployments (api1.yaml, api2.yaml)

Use the official nginx:latest image

Mount:

HTML content (index.html) from ConfigMap

Full NGINX config (nginx.conf) from ConfigMap

3. Services (svc1.yaml, svc2.yaml)

ClusterIP type (only accessible inside the cluster)

API1 forwards port 80 → 8000

API2 forwards port 80 → 8080

4. Ingress (test.yml)

Uses the nginx ingress class (the NGINX ingress controller must be installed)

Routes:

/api1 → API1 Service

/api2 → API2 Service

🚀 Deployment

Apply all manifests:

kubectl apply -f configmap1.yaml
kubectl apply -f configmap2.yaml
kubectl apply -f api1.yaml
kubectl apply -f api2.yaml
kubectl apply -f svc1.yaml
kubectl apply -f svc2.yaml
kubectl apply -f test.yml


Check resources:

kubectl get pods
kubectl get svc
kubectl get ingress

🌐 Accessing the services

Your ingress controller is exposed via NodePort (nic-nginx-ingress-controller):

kubectl get svc -n nginx-ingress


Example output:

NAME                           TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
nic-nginx-ingress-controller   NodePort   10.110.200.140   <none>        80:31194/TCP,443:32277/TCP   5d


Here:

NodePort 31194 → HTTP

NodePort 32277 → HTTPS

Test with curl

Add the Host header (Ingress uses mcs.com):

curl -H "Host: mcs.com" http://<NodeIP>:31194/api1/
curl -H "Host: mcs.com" http://<NodeIP>:31194/api2/


Expected responses:

Hello from API 1
Hello from API 2

🛠 Troubleshooting
301 Redirects

Requesting /api1 without a trailing slash may cause a 301 Moved Permanently redirect to /api1/.

Fix: either use /api1/ in requests or adjust nginx.conf to handle both /api1 and /api1/ explicitly.

404 Not Found

Happens if the Ingress path does not match your request.

Ensure:

Services expose the correct ports (targetPort: 8000 / 8080).

Ingress uses Service ports (80), not container ports.

502 Bad Gateway

Usually means Service port mismatch.

Check that Deployment → Service → Ingress port chain is consistent.

✅ Summary

API1 available at: http://mcs.com:31194/api1/

API2 available at: http://mcs.com:31194/api2/

Both served by a single NGINX Ingress controller.
