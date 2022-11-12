# prepare
Launch:
- 1 EC2 installed Ansible
- 1 Ec2 for Kubernetes master
- 1 Ec2 for Kubernetes worker1

then setting SSH key to allow SSH from Ansible to 2 EC2 kubernetes.
on 2 EC2 kubernetes, do:
- turn off swap `sudo swapoff -a`
- sudo sed -i '/swap/ s/^\(.*\)$/#\1/g' /etc/fstab
- change host name master: `sudo hostnamectl set-hostname` masterk8s
- change host name woker1: `sudo hostnamectl set-hostname` worker1
- open port 22, 6443 on Security Group


# install k8s
create 2 file: hosts and playbook.yml (IP is private IP of EC2 master, EC2 worker).

install role ansible
```
ansible-galaxy install geerlingguy.containerd
ansible-galaxy install geerlingguy.kubernetes
```

run `ansible-playbook -i hosts playbook.yml `
if it went wrong, try this command on the master, with IP is private master EC2
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all  --apiserver-advertise-address=172.31.9.61
```
this command will product an URL for worker. Run command that in worker1.
then on the k8s master: Copy file settiing of kubectl to user ubuntu:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get node -o wide
kubectl get pod -A
```

output result would to be:
``` 
ubuntu@k8s-master:~$ kubectl get node -o wide
NAME        STATUS   ROLES                  AGE     VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
masterk8s   Ready    control-plane,master   9m43s   v1.20.15   172.31.9.61    <none>        Ubuntu 20.04.5 LTS   5.15.0-1019-aws   containerd://1.6.9
worker1     Ready    <none>                 2m17s   v1.20.15   172.31.3.100   <none>        Ubuntu 20.04.5 LTS   5.15.0-1019-aws   containerd://1.6.9
ubuntu@k8s-master:~$ kubectl get pod -A
NAMESPACE      NAME                                READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-df56s               1/1     Running   0          2m27s
kube-flannel   kube-flannel-ds-vwqsl               1/1     Running   0          9m36s
kube-system    coredns-74ff55c5b-k7dmt             1/1     Running   0          9m36s
kube-system    coredns-74ff55c5b-mzv87             1/1     Running   0          9m36s
kube-system    etcd-masterk8s                      1/1     Running   0          9m44s
kube-system    kube-apiserver-masterk8s            1/1     Running   0          9m44s
kube-system    kube-controller-manager-masterk8s   1/1     Running   0          9m44s
kube-system    kube-proxy-4hqct                    1/1     Running   0          2m27s
kube-system    kube-proxy-gzbtw                    1/1     Running   0          9m36s
kube-system    kube-scheduler-masterk8s            1/1     Running   0          9m44s
```
