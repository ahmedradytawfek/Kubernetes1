# 🚀 Kubernetes API Routing with NGINX Plus 
Multi-API Deployment on Kubernetes with NGINX Plus

This project demonstrates how to deploy two independent NGINX-based APIs into a Kubernetes cluster and expose them via a single NGINX **Ingress Controller**.

API1 → /api1/ → returns Hello from API 1

API2 → /api2/ → returns Hello from API 2

Both applications are accessible through the same Ingress endpoint: **http://mcs.com**.


----------------------

📡 Cluster Architecture

- 3 Ubuntu VMs (1 control-plane, 2 workers)

- Kubernetes v1.30.14 with containerd runtime

- Calico CNI for pod networking and NetworkPolicy support

- NGINX Ingress Controller for external access and routing


* Request Flow
  Client → Ingress Controller"nginx plus" → Service → Pod (nginx container)

----------------

⚙️ Cluster Setup

1. Initialize the Control Plane

On master node
       sudo kubeadm init --pod-network-cidr=10.244.0.0/16

Configure kubectl for your user:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

2. Join Worker Nodes

    On each worker node:

         sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> \
         --discovery-token-ca-cert-hash sha256:<HASH>

3. Install Calico CNI
   
          kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico  


4. Verify:

       kubectl get pods -n kube-system

-------------------------------

🔑 Installing NGINX Plus Ingress

1. Obtain NGINX Plus License

   - Download your nginx-repo.crt and nginx-repo.key from MyF5 portal
   - Copy them to your cluster node (control plane).

2. Create Kubernetes Secret for License

     kubectl create secret generic nginx-repo-secret \
     --from-file=nginx-repo.crt=./nginx-repo.crt \
     --from-file=nginx-repo.key=./nginx-repo.key

3. Add NGINX Plus Helm Repo

     helm repo add nginx-stable https://pkgs.nginx.com/helm/nginx-stable

     helm repo update

-------------------------------

## 📂 Files

| File                            | Purpose                                     |
|---------------------------      |-------------------------------------------- |
| `api1.yaml`                     | Deployment + HTML ConfigMap for API1        |
| `api2.yaml`                     | Deployment + HTML ConfigMap for API2        |
| `configmap1.yaml`               | NGINX config for API1                       |
| `configmap2.yaml`               | NGINX config for API2                       |
| `svc1.yaml`                     | ClusterIP Service for API1                  |
| `svc2.yaml`                     | ClusterIP Service for API2                  |
| `ingress-controller-nginx.yaml` | Ingress controller nginx configuration      |
| `svc-dashboar.yaml`             | nginx-plus dashboard enable from masternode |

------------------------------


⚙️ How It Works

 1. ConfigMaps (`configmap1.yaml`, `configmap2.yaml`)
Provide custom `nginx.conf` files:

- **API1** → Listens on port `8000`, serves at `/api1`
- **API2** → Listens on port `8080`, serves at `/api2`

The `alias` directive ensures `/usr/share/nginx/html/index.html` is served when requesting `/api1/` or `/api2/`.

---

 2. Deployments (`api1.yaml`, `api2.yaml`)
- Use the official `nginx:latest` image
- Mount:
  - **HTML content (`index.html`)** from ConfigMap
  - **Full NGINX config (`nginx.conf`)** from ConfigMap

---

 3. Services (`svc1.yaml`, `svc2.yaml`)
ClusterIP type (only accessible inside the cluster):

- **API1**: forwards port `80 → 8000`
- **API2**: forwards port `80 → 8080`

---

 4. Ingress  (`ingress-controller-nginx.yaml`)
- Uses the **NGINX Ingress Controller**
- Routes traffic:
  - `/api1` → API1 Service
  - `/api2` → API2 Service

--------------

🚀 Deployment

Apply all manifests:

```bash
kubectl apply -f configmap1.yaml
kubectl apply -f configmap2.yaml
kubectl apply -f api1.yaml
kubectl apply -f api2.yaml
kubectl apply -f svc1.yaml
kubectl apply -f svc2.yaml
kubectl apply -f svc-dashboard.yaml
kubectl apply -f ingress-controller-nginx.yaml


Check resources:

kubectl get pods
kubectl get svc
kubectl get ingress

- you have to get these on the namespace nginx-ingress :

test@k8s-master01:~$ kubectl get deploy -n nginx-ingress
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
nic2-nginx-ingress-controller   1/1     1            1           64m


test@k8s-master01:~$ kubectl get pod -n nginx-ingress
NAME                                             READY   STATUS    RESTARTS   AGE
nic2-nginx-ingress-controller-58b8d5f759-f7nf9   1/1     Running   0          53m


test@k8s-master01:~$ kubectl get svc -n nginx-ingress

NAME                            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
nic2-nginx-ingress-controller   NodePort   10.105.214.95   <none>        80:31409/TCP,443:32061/TCP   66m

-------------------------------------

