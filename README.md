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
+ Now we have to install container runtime. I am using containerd as container runtime. To install containerd visit https://github.com/containerd/containerd/blob/main/docs/getting-started.md  
