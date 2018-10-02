
## kubernetes 설치하기 

### 환경 구성(VM)
Oracle VM VirtualBox 구성
- 파일 > 환경 설정 > 입력 > 가상 머신(tab) > 호스트 키 조합 : Ctrl + Alt
- 파일 > 환경 설정 > 네트워크 > 추가(icon) 
  . 네트워크 이름 : docker
  . 네트워크 CIDR : 10.0.0.0/24 
  . 네트워크 옵션 : DHCP 지원 (checked)
- VM 생성 > CentOS 최신버전(7.4)
- VM 설정 > 네트워크
  . 어댑터1 > NAT 네트워크 , docker
  . 어댑터2 > 호스트 전용 어댑터 , VirtualBox Host-Only Ethernet Adapter
  

### 설치 스크립트

```bash
ifup enp0s3
ifup enp0s8

hostnamectl set-hostname master / node1
vi /etc/hosts 
=> hosts 파일에 master,node1 IP설정 (10.0.0.x)

systemctl stop firewalld
systemctl disable firewalld

swapon -s 
swapoff `swapon -s  | grep dev | awk {'print $1'}`
swapoff /dev/dm-1
vi /etc/fstab 
=> swap 라인 주석처리

sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux

-------------------------------------------------------------------------------------
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

wget https://download.docker.com/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

yum install -y docker-ce 

systemctl start docker && systemctl enable docker

-------------------------------------------------------------------------------------

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

( sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux )

setenforce 0

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet

[확인]
yum clean all
yum repolist

getenforce
-------------------------------------------------------------------------------------

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

-----------------------------------------------------------------------------------------

systemctl daemon-reload
systemctl restart kubelet

--- MASTER Only
kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors all
export KUBECONFIG=/etc/kubernetes/admin.conf
kubeadm reset (init이 잘못된 경우 reset)
-> 변경
kubeadm init --pod-network-cidr=192.168.0.0/16
export KUBECONFIG=/etc/kubernetes/admin.conf

--- minion node only
(삭제)curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/linux/amd64/kubectl
(삭제)curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/linux/amd64/kubelet

(삭제)chmod +x kube*
(삭제)mv kube* /usr/bin/

(삭제)rm -f /etc/kubernetes/bootstrap-kubelet.conf
(삭제)rm -f /etc/kubernetes/pki/ca.crt

(삭제) kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml

master init 시 출력되는 명령어 node에서 실행
(vi join_kubu)
kubeadm join 10.0.0.4:6443 --token 1o6sbo.sg4tyjgu0d5k4u0w --discovery-token-ca-cert-hash sha256:e6c9b20aaf4212ea65edb3d6f51dd42921cb7779a78f741850e159e953c6010f

// create-cluster-kubeadm
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

### deploy 스크립트

```bash
wget https://k8s.io/examples/application/deployment.yaml
kubectl apply -f deployment.yaml
kubectl describe deployment nginx-deployment
kubectl get pods -l app=nginx

wget https://k8s.io/examples/application/deployment-update.yaml
kubectl apply -f deployment-update.yaml
kubectl describe deployment nginx-deployment
kubectl get pods -l app=nginx

wget https://k8s.io/examples/application/deployment-scale.yaml
kubectl apply -f deployment-update.yaml
kubectl describe deployment nginx-deployment
kubectl get pods -l app=nginx
```
