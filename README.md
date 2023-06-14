#새로운 Ubuntu 22.04 노드에서 안정적인 Kubernetes 1.26 마스터를 구성하는 단계입니다.

1.

    sudo apt-get update
2.

    sudo apt install apt-transport-https curl

containerd 설치(참조: https://docs.docker.com/engine/install/ubuntu/ )

3.

    sudo mkdir -p /etc/apt/keyrings
4.

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
5.

    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
6.

    sudo apt-get update
7.

    sudo apt-get install containerd.io

#containerd 구성 만들기

8.

    sudo mkdir -p /etc/containerd
9.

    sudo containerd config default | sudo tee /etc/containerd/config.toml

#etc/containerd/config.toml 편집 => SystemdCgroup = true로 설정

10.

    sudo vi /etc/containerd/config.toml 
11.

    sudo systemctl restart containerd

#쿠버네티스 설치

12.

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -  

13.

    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    
14.

    sudo apt install kubeadm kubelet kubectl kubernetes-cni
    sudo apt-mark hold kubelet kubeadm kubectl

#스왑 비활성화

15.

    sudo swapoff -a

#존재하는 경우 스왑 항목을 확인하고 제거하십시오.

16.

    sudo vi /etc/fstab

kubeinit에서 "/proc/sys/net/bridge/bridge-nf-call-iptables does not exist" 오류를 방지합니다( https://github.com/kubernetes/kubeadm/issues/1062 참조 ). 

6단계에서 docker도 설치한 경우에는 필요하지 않습니다.

17.

    sudo modprobe br_netfilter

    sysctl -w net.ipv4.ip_forward=1
    
Flannel과 함께 사용하기 위한 kubeinit

18.

    sudo kubeadm init --pod-network-cidr=10.244.0.0/16

kubadm 명령이 말하는 대로 구성에 복사

19.

    mkdir -p $HOME/.kube

20.

    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

21.

    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Flannel 적용( https://github.com/flannel-io/flannel 참조 )

22.

    kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
    
23.

    kubectl get pods --all-namespaces
    
이제 모두 실행 중이어야 합니다.

    NAMESPACE      NAME                                  READY   STATUS    RESTARTS   AGE
    kube-flannel   kube-flannel-ds-mcjmm                 1/1     Running   0          76s
    kube-system    coredns-787d4945fb-fb59g              1/1     Running   0          8m8s
    kube-system    coredns-787d4945fb-t25tj              1/1     Running   0          8m8s
    kube-system    etcd-kube-master                      1/1     Running   0          8m19s
    kube-system    kube-apiserver-kube-master            1/1     Running   0          8m19s
    kube-system    kube-controller-manager-kube-master   1/1     Running   0          8m19s
    kube-system    kube-proxy-2hz29                      1/1     Running   0          8m8s
    kube-system    kube-scheduler-kube-master            1/1     Running   0          8m19s
