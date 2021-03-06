#### Two nodes k8s cluster and access from Windows host machine using WSL

##### Install WSL and SSH client on host machine 
- Enable WSL in Windows features and install *Ubuntu*
- Install any ssh client


##### Create two Ubuntu VMs in VirtualBox. Remember the folloing points when creating the VMs
- Master node should have minimum 2 CPU cores and 2 GB RAM
- Worker node should have minimum 2 CPU cores and 1 GB RAM
- Select ***bidirectional*** for ***clipboard*** setting
- Select *Bridged network adapter*

##### Make the following changes once VMs are up and running (on both nodes)
- Change IP address allocation from Automatic to Manual (IPV4) along with DNS. Make sure they get the same IP subnet as host. 
- Make local DNS entry of these nodes in */etc/hosts*
- Run the command ```swapoff -a``` in both the VMs
- Edit */etc/fstab* and comment swap line
- Create *kubernetes.list* file in */etc/apt/sources.list.d/* and add ```deb https://apt.kubernetes.io/ kubernetes-xenial main```
- Run ```sudo apt-get update && apt-get upgrade -y```
- Run ```sudo apt-get install -y vim open-ssh server```
- Access both the VMs from host machines using ssh client
- Switch to root user and run ```apt-get install -y apt-transport-https curl```
- Run ```curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -```
- Run apt-get install -y kubectl kubeadm kubelet
- Run ```apt-get install docker.io```
- Run ```sudo apt-mark hold kubectl kubeadm kubelet```
- Edit */lib/systemd/system/docker.service* and add ```--exec-opt native.cgroupdriver=systemd``` to this line 
```ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock```
- Run ```systemctl daemon-reload``` and ```systemctl restart docker```

##### On Master node
- Run ```kubeadm init --apiserver-advertise-address=<Master_node_IP> --pod-network-cidr=10.244.0.0/16```   # For Flannel CNI
- Run ```kubeadm init --apiserver-advertise-address=<Master_node_IP> --pod-network-cidr=192.168.0.0/16```  # For Calico CNI
- Run ```export KUBECONFIG=/etc/kubernetes/admin.conf``` # For root user to access the cluster. 
- Switch to normal user and run ```mkdir -p $HOME/.kube``` ```sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config```                                                       ```sudo chown $(id -u):$(id -g) -     $HOME/.kube/config```
- Run ```kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml``` #For Flannel
- Run ```kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml``` #For calico

##### On worker node
- Run the command that is displayed on the screen ```kubeadm join <master_IP>:6443 --token...``` # Run ```kubeadm token create --print-join-command``` if you didn't make a note

##### On WSL
- In home directory, run ```mkdir .kube``` and ```scp <master_user>@<IP>:/home/<master_user/.kube/config .```
- Run ```kubectl get po -A``` to confirm that the cluster can be accessed


