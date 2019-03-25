- This script will do the most of things, just complete the action to exercise

sudo su

*******Ensure that***********
- SELinux is disabled.
cat /etc/sysconfig/selinux

- Enable the br_netfilter module for cluster communication.
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

- Disable swap to prevent memory allocation issues.
 swapoff -a
 vim /etc/fstab  ->  Comment out the swap line

Reboot.
----------------------------------------------------------------------------
Enable and start Kubernetes.

systemctl enable kubelet
systemctl start kubelet

- Check the group Docker is running in.
docker info | grep -i cgroup

- Set Kubernetes to run in the same group.
sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

- Reload systemd for the changes to take effect, and then restart Kubernetes.
systemctl daemon-reload
systemctl restart kubelet


*Note: Complete the following section on the MASTER ONLY!

- Initialize the cluster using the IP range for Flannel.
kubeadm init --pod-network-cidr=10.244.0.0/16

**************Copy the kubeadmin join command.*******************

- Exit sudo and run the following:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

- Deploy Flannel.
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


- Check the cluster state.
kubectl get pods —all-namespaces


**Note: Complete the following steps on the NODES ONLY!

- Run the join command that you copied earlier (this command needs to be run as sudo), then check your nodes from the master.

kubectl get nodes
--------------------------------------------------------------------------------------------------------------------------------

kubeadm join 10.0.2.15:6443 --token ok4qr6.212s39k6mhl0vdfc --discovery-token-ca-cert-hash sha256:49278735f04571f2e78b25fad7998a57b9125c3f15f938e7551059db5982bd1c

