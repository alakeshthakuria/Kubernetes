# Initiate k8s cluster using kubeadm v1.31 with AWS EC2 instance

+ Launch EC2 instances for master nodes and worker nodes.
+ After creating EC2 instances, rename all EC2 instance hostname.
  `sudo vi /etc/hostname`
+ After that renaming all the EC2 instances hostname reboot all the intances.
  `sudo init 6`
+ Once all the instances restart switch user to root user in all the instances
  `sudo -i` or `sudo su -`
+ Enable IPv4 packet forwarding in all the nodes. To manually enable IPv4 packet forwarding, run the below command in all the nodes:
+ ````
  # sysctl params required by setup, params persist across reboots
  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.ipv4.ip_forward = 1
  EOF

  # Apply sysctl params without reboot
  sudo sysctl --system
  ````
+ Verify that `net.ipv4.ip_forward` is set to 1, run below command in all the nodes to verify:
+ `sysctl net.ipv4.ip_forward`
+ Next disable swap in all the nodes from this command: `sudo swapoff -a`
+ Now we have to install container runtime in all the nodes. I am using containerd as container runtime. To install containerd run the below command in all the nodes:
+ ````
  # Add Docker's official GPG key:
  sudo apt-get update
  sudo apt-get install ca-certificates curl
  sudo install -m 0755 -d /etc/apt/keyrings
  sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  sudo chmod a+r /etc/apt/keyrings/docker.asc

  # Add the repository to Apt sources:
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] 
  https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

  sudo apt-get update

  sudo apt-get install containerd.io
  ````
+  I have got the above block of command to install containerd from this link https://docs.docker.com/engine/install/ubuntu/  where I have modified this command `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` to `sudo apt-get install containerd.io` to install only containerd.
+  Next we have to configure systemd cgroup driver in all the nodes. To configure systemd cgroup driver, run the below command:
+  `containerd config default > /etc/containerd/config.toml`
+  Next we have to open the config.toml file in all the nodes, to open use this command `sudo vi /etc/containerd/config.toml`
+  Once config.toml file open, look for SytemdCgroup which is set to false `SystemdCgroup = false` by default, so we need to change to it true.  `SytemdCgroup = true` in all the nodes.
+  After configuring SystemdCgroup we have to restart the containerd. To restart containerd run the below command:
+  `sudo systemctl restart containerd`
+  It takes hardly 2 sec to restart containerd. To check the status whether containerd is running or not run the below command:
+  `sudo systemctl status containerd`
+  Next we have to install kubeadm, kubelet and kubectl. To install kubeadm, kubelet and kubectl run the below command in all the nodes.
+  ````
   sudo apt-get update

   sudo apt-get install -y apt-transport-https ca-certificates curl gpg

   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

   sudo apt-get update

   sudo apt-get install -y kubelet kubeadm kubectl

   sudo apt-mark hold kubelet kubeadm kubectl
   ````
+ Now we have to initialize kubeadm in controlplane node only. To initialize control plane run this command `kubeadm init <arguments>`in controlplane or master nodes only. Do not run this command in worker nodes.
+ From the command `kubeadm init <arguments>` we have to replace arguments with `--apiserver-advertise-address 172.31.88.125`, `--pod-network-cidr 10.244.0.0/16`, `--cri-socket unix:///var/run/containerd/containerd.sock` where `172.31.88.125` is private IP address of my contolplane node instance and `10.244.0.0/16` is a cidr range.
+ So in my case the complete kubeadm initialize command will be
+ `kubeadm init --apiserver-advertise-address 172.31.88.125  --pod-network-cidr 10.244.0.0/16  --cri-socket unix:///var/run/containerd/containerd.sock`
+ After running the above command our kubernetes control-plane will be initialized and then we have to exit the root user to go back to regualar user to run kubernets command.
+ Now as a regular user we have to initiate below command which has generated at the time of `kubead init <arguments>` command.
+ ````
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ````
+ Next we have to add worker node to control plane, to do this we can use the kubeadm join command which has generated by `kubeadm init <arguments>` command. For now it is looks like this `kubeadm join 172.31.88.125:6443 --token 2kcks8.2uapy0olnxsb3u4p \
        --discovery-token-ca-cert-hash sha256:09894bd98f578aa2a39564ebe836abe4a9768d110c7eab445cd7bec9531ed970`. We need to run this command in all the worker nodes which need to join this control plane.
+ Next we need to install container network interface (CNI) plugin in controlplane node only. In this case we are using weave net. Below is the command. 
+ `kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.31/net.yaml`
+ From the above command `v1.31` is the kubeadm version we are using. This will be change according to the kubeadm version we will use. 
