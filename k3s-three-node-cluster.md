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
       curl -sfL https://get.k3s.io | K3S_TOKEN=<secret_here> sh -s - server --cluster-init
       ```
    2. On other 2 nodes run command:
      ```
    curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --server https://<ip or hostname of server1>:6443
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
    ```

## K3S backup

## 


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

 