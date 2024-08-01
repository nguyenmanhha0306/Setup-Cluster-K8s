# Common setup for all servers (Control Plane and Nodes)
```bash
set -euxo pipefail
```

### DNS Setting
```bash
if [ ! -d /etc/systemd/resolved.conf.d ]; then
	sudo mkdir /etc/systemd/resolved.conf.d/
fi
cat <<EOF | sudo tee /etc/systemd/resolved.conf.d/dns_servers.conf
[Resolve]
DNS=8.8.8.8 1.1.1.1
EOF

sudo systemctl restart systemd-resolved
```
### Disable swap
```bash
sudo swapoff -a
```

keeps the swaf off during reboot
```bash
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
sudo apt-get update -y
```
### Create the .conf file to load the modules at bootup
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Sysctl params required by setup, params persist across reboots
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

### Apply sysctl params without reboot

```bash
sudo sysctl --system
```

## Install containerd-installation Runtime
## How to install Containerd on Ubuntu OS

```console
Step 1 : Download & unpack containerd package
Step 2 : Install runc
Step 3 : Download & install CNI plugins
Step 4 : Configure containerd
Step 5 : Start containerd service
```

## Step 1 : Download & unpack containerd package

Containerd versions can be found in this location :  https://github.com/containerd/containerd/releases

### Download :
```bash

  wget https://github.com/containerd/containerd/releases/download/v1.7.19/containerd-1.7.19-linux-amd64.tar.gz
```
### Unpack : 
```bash
  sudo tar Cxzvf /usr/local containerd-1.7.19-linux-amd64.tar.gz
```

## Step 2 : Install runc

Runc is a standardized runtime for spawning and running containers on Linux according to the OCI specification
```bash
  wget https://github.com/opencontainers/runc/releases/download/v1.1.13/runc.amd64
  sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

## Step 3: Download and install CNI plugins :

```bash
  wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
  sudo mkdir -p /opt/cni/bin
  sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

## Step 4: Configure containerd

#### Create a containerd directory for the configuration file
#### config.toml is the default configuration file for containerd
#### Enable systemd group . Use sed command to change the parameter in config.toml instead of using vi editor 
####  Convert containerd into service
```bash
  sudo mkdir /etc/containerd
  containerd config default | sudo tee /etc/containerd/config.toml
  sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
  sudo curl -L https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -o /etc/systemd/system/containerd.service
```

## Step 5: Start containerd service
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd    
```


## kubelet,kubectl,kubeadm installation
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION_SHORT/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION_SHORT/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```bash
sudo apt-get update -y
sudo apt-get install -y kubelet kubectl kubeadm
sudo apt-get update -y
sudo apt-get install -y jq
```
### Disable auto-update services
```bash
sudo apt-mark hold kubelet kubectl kubeadm containerd
```

### Set node ip
```bash
cat > /etc/default/kubelet << EOF
KUBELET_EXTRA_ARGS=--node-ip=$local_ip
${ENVIRONMENT}
EOF
```

Example:
```bash
cat > /etc/default/kubelet << EOF
KUBELET_EXTRA_ARGS=--node-ip=172.18.25.80
${ENVIRONMENT}
EOF
```

## Override hostname
```bash
sudo nano /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
```

--hostname-override=master or --hostname-override=woker-node
Output:
```
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --hostname-override=master
```