# Setup for Control Plane (Master) servers
```bash
set -euxo pipefail
```
 ```bash
 sudo kubeadm config images pull
 ```


## Kubeadm Init 
 ```bash
 sudo kubeadm init --apiserver-advertise-address=$CONTROL_IP --apiserver-cert-extra-sans=$CONTROL_IP --pod-network-cidr=$POD_CIDR --service-cidr=$SERVICE_CIDR --node-name "$NODENAME" --ignore-preflight-errors Swap
 ```

 Example:
 ```bash
 sudo kubeadm init --kubernetes-version=v1.30.2 --apiserver-advertise-address=172.18.25.80 --apiserver-cert-extra-sans=172.18.25.80 --pod-network-cidr=172.16.1.0/16 --service-cidr=172.17.1.0/18 --node-name "master" --ignore-preflight-errors Swap
 ```


```bash
mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
```

### Install Calico Network Plugin
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```