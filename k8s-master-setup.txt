#! /bin/bash

read -p "Enter Control Plane Endpoint :- "  endpoint


cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF


yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes


systemctl enable --now kubelet


kubeadm config images pull


docker info | grep -i cgroup


cat <<EOF | tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF


systemctl restart docker


docker info | grep -i cgroup


yum install iproute-tc -y


kubeadm init --pod-network-cidr=10.240.0.0/16 --control-plane-endpoint=$endpoint:6443 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem


mkdir -p $HOME/.kube


cp -i /etc/kubernetes/admin.conf $HOME/.kube/config


chown $(id -u):$(id -g) $HOME/.kube/config


echo 3 > /proc/sys/vm/drop_caches


kubectl apply  -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


echo  -e  "\n\nThis is your join-command :- \n "


kubeadm token create  --print-join-command

