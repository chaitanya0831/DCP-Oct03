Kube-Master(Controller)
	Kube-Worker1
	Kube-Worker2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
On Both Master and Worker Nodes:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

sudo -i

yum update -y

swapoff -a
#The Kubernetes scheduler determines the best available node on which to deploy newly created pods. If memory swapping is allowed to occur on a host system, this can lead to performance and stability issues within Kubernetes.

setenforce 0
#Disabling the SElinux makes all containers can easily access host filesystem.

yum install docker -y

systemctl enable docker 
systemctl start docker

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF


cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

yum install -y kubeadm-1.21.3 kubelet-1.21.3 kubectl-1.21.3 --disableexcludes=kubernetes 

systemctl enable kubelet 
systemctl start kubelet

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Only on Master Node:
~~~~~~~~~~~~~~~~~~~~~

sudo kubeadm init --apiserver-advertise-address=172.31.11.38 --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


#export KUBECONFIG=/etc/kubernetes/kubelet.conf

#We need to install a flannel network plugin to run coredns to start pod network communication.

sudo kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml 
sudo kubectl apply -f https://docs.projectcalico.org/v3.8/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml 

kubectl get pods --all-namespaces

kubectl get nodes

kubectl describe nodes

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Execute the below commmand in Worker Nodes, to join all the worker nodes with Master :
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#kubeadm join 172.31.39.190:6443 --token 4h1iij.rb53c4jhj2fkl3qm --discovery-token-ca-cert-hash sha256:7c7a55d5559ddc181a63376895f7097fe6a054a9ab1f3f613dadfd06cf167143
#kubeadm join 172.31.45.213:6443 --token 27yrml.4cmz22e0jtkjfzvl --discovery-token-ca-cert-hash sha256:8284680195b668b66ebf37629960bab007f9700f590b48cdb9de60799d4aca1b
#kubeadm join 172.31.36.201:6443 --token 69a43n.i9f5sq4kt383lrik --discovery-token-ca-cert-hash sha256:ef813fb9c38653d61bf0b5739b419542e7ac1f94f4767319a8bea8e3c300cc72
#kubeadm join 172.31.5.173:6443 --token aoztca.d1ol6rk8iv7yekvi --discovery-token-ca-cert-hash sha256:b5ed2a087727283b51dd7b3956226eacf5a592db87cb679db691fde935b4cf39

kubeadm join 172.31.11.38:6443 --token 13zm9f.6r9licbxs3eqwvf2 --discovery-token-ca-cert-hash sha256:60a20111ece6485c5225736bbcdc6a54d54b196340e30b3b1fcc5c4dd3194f60

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#kubeadm join 172.31.11.38:6443 --token 13zm9f.6r9licbxs3eqwvf2 --discovery-token-ca-cert-hash sha256:60a20111ece6485c5225736bbcdc6a54d54b196340e30b3b1fcc5c4dd3194f60

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~




#Enviromments :::

#DEV --- namespace:DEV ==== craete pods 
#QA --- namespace:QA ==== craete pods 
#UAT
#PROD 
