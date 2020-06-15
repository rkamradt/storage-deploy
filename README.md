# storage-deploy
## Deploying a Rook/EdgeFS system to Kubernetes

From the article 'Build Your Own In-Home Cloud Storage Part II'

Kubernetes system requirements: three nodes called node1, node2, and node3 with storage space mounted on /storage in all
three nodes.

```
kubectl apply -f operator.yaml
kubectl apply -f clusterprep.yaml
kubectl apply -f cluster.yaml
```
To expose the EdgeFS GUI:
```
kubectl apply -f issuer.yaml -n rook-edgefs
kubectl apply -f vhost-ingress -n rook-edgefs
```
To use a certificate for HTTPS, assuming you have a ca.crt and ca.unencrypted.key in the .. directory run this
before you create the issuer
```
kubectl create secret tls ca-key-pair -n rook-edgefs --cert=../ca.crt --key=../ca.unencrypted.key
```
Add rook.local to your /etc/hosts to the IP of the ingress, and then go to https://rook.local and login with
username admin and password edgefs

## To use CSI for dynamic provisioning

Set up a namespace

```
kubectl create namespace edgefs-nfs-csi
```
Set up the artifacts in EdgeFS
```
rkamradt@beast:~/storage-deploy$ kubectl exec -it rook-edgefs-mgr-648457b454-j2tcl -n rook-edgefs -- toolbox
Defaulting container name to rook-edgefs-mgr.
Use 'kubectl describe pod/rook-edgefs-mgr-648457b454-j2tcl -n rook-edgefs' to see all of the containers in this pod.
Welcome to EdgeFS Mgmt Toolbox.
Hint: type neadm or efscli to begin
root@rook-edgefs-mgr-648457b454-j2tcl:/opt/nedge# efscli system status
SID                | HOST  |          POD           | USED,% | STATE   
+----------------------------------+-------+------------------------+--------+--------+
  3131644475DD1E38274C22A6E7D14DA8 | node1 | rook-edgefs-target-1-0 |  0.00  | ONLINE  
  40AD4245F79D8924F686E560C2256383 | node2 | rook-edgefs-target-0-0 |  0.00  | ONLINE  
  7A4ED7164285247D68236351AB7CD770 | node3 | rook-edgefs-target-2-0 |  0.00  | ONLINE
root@rook-edgefs-mgr-648457b454-j2tcl:/opt/nedge# efscli cluster create cltest
root@rook-edgefs-mgr-648457b454-j2tcl:/opt/nedge# efscli tenant create cltest/test
root@rook-edgefs-mgr-648457b454-j2tcl:/opt/nedge# efscli service create nfs nfs01
root@rook-edgefs-mgr-648457b454-j2tcl:/opt/nedge# exit
exit
```
Create a secret and start the driver:
```
kubectl create secret generic edgefs-nfs-csi-driver-config \
   -n edgefs-nfs-csi --from-file=./edgefs-nfs-csi-driver-config.yaml
kubectl apply -n edgefs-nfs-csi -f edgefs-nfs-csi-driver.yaml
```
And create a storage class:
```
kubectl apply -f nfs-storage-class.yaml
```
