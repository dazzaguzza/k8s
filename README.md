#새로운 Ubuntu 22.04 노드에서 안정적인 Kubernetes 1.26 마스터를 구성하는 단계입니다.


    sudo apt-get update

    sudo apt install apt-transport-https curl

containerd 설치(참조: https://docs.docker.com/engine/install/ubuntu/ )

    sudo mkdir -p /etc/apt/keyrings

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    sudo apt-get update

    sudo apt-get install containerd.io

#containerd 구성 만들기

    sudo mkdir -p /etc/containerd

    sudo containerd config default | sudo tee /etc/containerd/config.toml

#etc/containerd/config.toml 편집 => SystemdCgroup = true로 설정

    sudo vi /etc/containerd/config.toml 

    sudo systemctl restart containerd

#쿠버네티스 설치

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

    sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

    sudo apt install kubeadm kubelet kubectl kubernetes-cni

#스왑 비활성화

    sudo swapoff -a

#존재하는 경우 스왑 항목을 확인하고 제거하십시오.

    sudo nano /etc/fstab

kubeinit에서 "/proc/sys/net/bridge/bridge-nf-call-iptables does not exist" 오류를 방지합니다( https://github.com/kubernetes/kubeadm/issues/1062 참조 ). 6단계에서 docker도 설치한 경우에는 필요하지 않습니다.

    sudo modprobe br_netfilter

sudo nano /proc/sys/net/ipv4/ip_forward ip_forward 파일의 항목을 편집하고 1로 변경합니다. (또는 사용 sysctl -w net.ipv4.ip_forward=1- @dpjanes 덕분에 주석 참조)

Flannel과 함께 사용하기 위한 kubeinit

    sudo kubeadm init --pod-network-cidr=10.244.0.0/16

kubadm 명령이 말하는 대로 구성에 복사

    mkdir -p $HOME/.kube

    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Flannel 적용( https://github.com/flannel-io/flannel 참조 )

    kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml

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
