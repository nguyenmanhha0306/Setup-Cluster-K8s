# Setup for Node servers


Get token join in master-node (Control-plane)
```bash
kubeadm token create --print-join-command 
```
-Output
```
kubeadm join 172.18.25.80:6443 --token 423g423g4.423g4j2h --discovery-token-ca-cert-hash sha256:8423k4bk23h42342e876aff5448b32b3afaa1f8d234hg23hk4g23h4g
```

Copy script output run Worker-node
```bash
sudp kubeadm join 172.18.25.80:6443 --token 423g423g4.423g4j2h --discovery-token-ca-cert-hash sha256:8423k4bk23h42342e876aff5448b32b3afaa1f8d234hg23hk4g23h4g
```


## In case of error
### Woker-node NotRedy
```
$ kubectl get nodes
NAME          STATUS     ROLES    AGE   VERSION
master        Ready      master   10m   v1.21.3
node1         NoteReady  <none>   30s   v1.21.3
node2         Ready      <none>   5m    v1.21.3
```
Check what node is wrong
```bash
kubectl describe node node1
```

Log
```
.....
Conditions:
Ready                False   Mon, 13 Jun 2022 21:42:05 +0000   Mon, 13 Jun 2022 21:32:01 +0000   KubeletNotReady              container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized
```


```
# ls -l /etc/cni/net.d/
total 8
-rw-r--r-- 1 root root  730 Jun 16 14:16 10-calico.conflist
-rw------- 1 root root 3094 Jun 16 14:16 calico-kubeconfig
```
If your woker node does not have the above calico configuration files, then go to master-node and copy those files to woker-node which is in NotRedy status.

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```