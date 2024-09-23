# Initiate k8s cluster using kubeadm with AWS EC2 instance

+ Launch EC2 instances for master nodes and worker nodes.
+ After creating EC2 instances, rename all EC2 instance hostname.
  `sudo vi /etc/hostname`
+ After that renaming all the EC2 instances hostname reboot all the intances.
  `sudo init 6`
+ Once instances restart switch user to root user
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
+ Verify that `net.ipv4.ip_forward` is set to 1 with:
+ `sysctl net.ipv4.ip_forward`
+ Disable swap
+ `sudo swapoff -a`
+ Now we have to install container runtime. I am using containerd as container runtime. To install containerd run the below command:
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
+  
