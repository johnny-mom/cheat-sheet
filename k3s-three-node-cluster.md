# k3s setup three-node-cluster (step-by-step)

1. Install Ubuntu Server on three nodes (HP ProDesk 600 G2)
    1. Make sure OpenSSH Server is installed
2. Setup SSH Key Login
    1. Update `~/.ssh/authorized_keys` to include `id_ed25519.pub`
    2. Update /etc/ssh/sshd_config and uncomment:
        ```
        #PubkeyAuthentication yes
        ```
    3. Test accessing to host with ssh key:
        ```
        ssh 192.168.0.xx -i ~/.ssh/id_ed25519
        ```
3. Install K3s 
    1. Run on node 1 run command:
       ```
       export K3S_TOKEN=K3SSecret1
       curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --cluster-init --disable traefik --disable servicelb" sh
       ```
    2. On other 2 nodes run command:
      ```
      export K3S_TOKEN=K3SSecret1
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --server https://192.168.0.31:6443 --disable traefik --disable servicelb" sh

      ```
    3. Validate nodes are showing and connected
      ```
    sudo kubectl get nodes
      ```
4. Setup k9s or openlens to manage
    1. Copy config in `/etc/rancher/k3s/k3s.yaml` to `~/.kube/config`
    2. Update server url from 
    ```
        server: https://127.0.0.1:6443
        to
        server: https://<ip_of_master_node>:6443
5. Add MetalLB (loadbalancer)
    ```
    # First add metallb repository to your helm
    helm repo add metallb https://metallb.github.io/metallb
    
    # Check if it was found
    helm search repo metallb
    
    # Install metallb
    helm upgrade --install metallb metallb/metallb --create-namespace \
    --namespace metallb-system --wait
    ```
6. Configure MetalLB
    ```
    cat << 'EOF' | kubectl apply -f -
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
    name: default-pool
    namespace: metallb-system
    spec:
    addresses:
    - 192.168.0.40-192.168.0.80
    ---
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
    name: default
    namespace: metallb-system
    spec:
    ipAddressPools:
    - default-pool
    EOF
    ```
7. Install nginx-ingress
    ```
    helm install ingress-nginx ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --namespace ingress-nginx --create-namespace
    ```
8. Validate nginx-ingress loadbalancer service has an external IP Address allocated from the MetalLB IP Address Pool (192.168.0.xx-192.168.0.xx)    
```
kubectl -n ingress-nginx get svc
```
9. Deploy a sample app:
```
apiVersion: v1
kind: Namespace
metadata:
  name: sampleapp
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app-deployment
  labels:
    app: sampleapp
  namespace: sampleapp
spec:
  replicas: 1
  minReadySeconds: 10
  selector:
    matchLabels:
      app: sampleapp
  template:
    metadata:
      labels:
        app: sampleapp
    spec:
      containers:
        - name: sample-app-container
          image: nginx
          ports:
            - containerPort: 80
          resources:
            limits:
              memory: "128Mi" #128 MB
              cpu: "200m" #200 millicpu (.2 cpu or 20% of the cpu)
---
apiVersion: v1
kind: Service
metadata:
  name: sample-app-service
  namespace: sampleapp
spec:
 type: NodePort
 selector:
  app: sampleapp
 ports:
  - name: "sampleapp-tcp"
    port: 8000
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-app-ingress
  labels:
    app: sampleapp
  namespace: sampleapp
spec:
  ingressClassName: nginx
  rules:
    - host: test.johnnymom.com
      http:
        paths:
          - backend:
              service:
                name: sample-app-service
                port:
                  number: 80
            path: /
            pathType: Exact
```
10. Validate site works by assigning IP Address of ingress-nginx loadbalancer IP to a DNS name test.johnnymom.com.

## K3S certmanager

## K3S Storage

## K3S Deployment (ARGOCD)


## K3S backup


## Using the SSH Config File (~/.ssh/config)
1. Create ~/.ssh/config:
    ``` 
    sudo touch ~/.ssh/config 
    sudo vim ~/.ssh/config
    ```
2. Add hosts:
    ```   
    Host <host_name>
    HostName 192.168.0.xx
    Port 22
    IdentityFile ~/.ssh/<private_ssh_key>  
    ```
3. Test if alias works
    ```
    ssh <host_name>
    ```

 