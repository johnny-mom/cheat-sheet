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

 