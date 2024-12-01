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
    - 192.168.0.60-192.168.0.80
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
# deploys nginx
kubectl create deploy nginx --image nginx

# exposes nginx deployment
kubectl expose deploy nginx --port 80 --type LoadBalancer

## Cleanup deployment
kubectl delete deployment nginx
kubectl delete svc nginx
```
10. Grab IP Address of the service and validate site works via IP Address (metalLB will assign an IP Address to the service from the IP Address Pool)

```
# Get the IP Address
kubectl get svc nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

11. Update coredns config map ``` coredns ``` 
```
# add dns ips (8.8.8.8, 8.8.4.4) so pods can resolve external dns names (by default it resolves with search domain):

# Stackoverflow: https://stackoverflow.com/questions/62664701/resolving-external-domains-from-within-pods-does-not-work

prometheus :9153
        forward . /etc/resolv.conf 8.8.8.8 8.8.4.4
        cache 30
        loop
        reload
```
12. Test doing nslookup on test dnstools pod can resolve DNS Names to IP Addresses 
```
# Spin up a test pod
kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools
```
## Deploy test nginx app (metallb + nginx + certmanager)
1. 
```

```
## K3S certmanager
1. Add cert-manager helm repo
```
helm repo add jetstack https://charts.jetstack.io
```
2. Update helm chart repo cache
```
helm repo update
```
3. Install cert-manager CRDs
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml
```
4. Install cert-manager
```
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.11.0 \
  # --set installCRDs=true
```
5. Install staging + prod issuer (HTTP01)
```
# This method requires http access to nginx ingress loadbalancer (DNS name points to this NAT'd IP Address. ex. nginx.johnnymom.com > 192.168.0.40 via port 80) for ACME to validate they can access our loadbalancer

kubectl -f staging-issuer.yml
kubectl -f prod-issuer.yml

# Need to validate the the issuer is registered to ACME
kubectl get clusterissuers -A
kubectl describe clusterissuer

```


## K3S Storage

## K3S Deployment (ARGOCD)
Create argocd namespace:
```
kubectl create namespace argocd
```
Install ArgoCD Helm chart:
```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd
```
Deploy ArgoCD ingress 
```
kubectl apply -f argocd-deployment/argocd-ingress.yaml
```
Update nginx-ingress controller deployment to include command:
```
--enable-ssl-passthrough
```
#### need to do above due to issues with multiple redirect issue w/ nginx-ingress
#### source: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/

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

 
