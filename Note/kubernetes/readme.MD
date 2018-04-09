# Preparation on All Machines
## Change to root
`sudo passwd root`

`sudo su -`

## Add Machine Host
add line `<machine-ip-address> <hostname>`

## Disable Swap
`swapoff -a`

```
vi /etc/fstab 
**uncomment the last line about swap**
```

## Reboot Machine

## (1/4) Installing kubeadm on your hosts

## Installing Docker on All Machines
```
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
   "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
```

## Installing kubeadm, kubelet and kubectl on All Machines
```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

## Configure cgroup driver used by kubelet on Master Node
Make sure that the cgroup driver used by kubelet is the same as the one used by Docker. Verify that your Docker cgroup driver matches the kubelet config:
```
docker info | grep -i cgroup
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
Manual Add line
`Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"`
To /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
Then restart kubelet:
```
systemctl daemon-reload
systemctl restart kubelet
```

## (2/4) Initializing your master
`kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<master-ip-address>`

`export KUBECONFIG=/etc/kubernetes/admin.conf`

## (3/4) Installing a pod network on Master Node
Calico

`kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml`

Master Isolation

`kubectl taint nodes --all node-role.kubernetes.io/master-`

Once a pod network has been installed, you can confirm that it is working by checking that the kube-dns pod is Running in the output of kubectl get pods --all-namespaces. And once the kube-dns pod is up and running, you can continue by joining your nodes.

## (4/4) Joining cluster from Worker Node

`sudo su -`
```
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```
# Kubernetes Dashboard
## Deploying the Dashboard UI

The Dashboard UI is not deployed by default. To deploy it, run the following command:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

## Accessing the Dashboard UI

透過 `kubectl proxy` 連到 Dashboard 是沒有認證機制的

kubectl proxy 本身提供許多 options 讓該指令的應用更加彈性。例如，若是我們希望別台機器也能透過下 kubectl proxy 指令的機器存取 Dashboard，則我們可以在指令後面加上 --address 與 --accept-hosts 如以下指令
```
kubectl proxy --address='0.0.0.0' --port=8002 --accept-hosts='^*$'
```

目前官方已不推薦使用 http://<master-ip-address>:8002/ui 存取 Dashboard，而是用
```
http://<master-ip-address>:8002/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy/#!/overview?namespace=default
```

# Common Instructions

## Use kubectl on Windows
First, copy the config file from master node to windows.
The config file is located in `/etc/kubernetes/admin.conf`

Usually kubernetes config is put in `%UserProfile%\.kube\admin.conf`

Then in Windows (kubectl.exe is required)

`kubectl --kubeconfig="%UserProfile%\.kube\admin.conf" <kube-command>`

## Show All Pods

`kubectl --kubeconfig="%UserProfile%\.kube\admin.conf" get pods --all-namespaces -o wide`

`kubectl get pods --show-labels`

## Show Detail on Pod by Name

`kubectl describe pod <pod-label-name>`

## Show Detail on Service by Name

`kubectl --kubeconfig="%UserProfile%\.kube\admin.conf" -n kube-system describe services <service-label-name>`
