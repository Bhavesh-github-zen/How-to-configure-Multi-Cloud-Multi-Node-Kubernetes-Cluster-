#! /bin/bash


read -p "Enter Join Command :- "  join_command


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


yum install -y  kubectl kubelet  kubeadm  --disableexcludes=kubernetes


systemctl enable kubelet --now


docker info | grep -i cgroup


cat <<EOF | tee  /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF


systemctl restart docker


docker info | grep -i cgroup


yum install iproute-tc -y


cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF


sysctl --system


yes | kubeadm reset


echo 1 > /proc/sys/net/ipv4/ip_forward


$join_command


echo 3 > /proc/sys/vm/drop_caches
