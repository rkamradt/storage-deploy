# storage-deploy
Deploying a Rook/EdgeFS system to Kubernetes

From the article 'Build Your Own In-Home Cloud Storage Part II'

Kubernetes system requirements: three nodes called node1, node2, and node3 with storage space mounted on /storage

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
To use a certificat for HTTPS, assuming you have a ca.crt and ca.unencrypted.key in the .. directory run this
before you create the issuer
```
kubectl create secret tls ca-key-pair -n rook-edgefs --cert=../ca.crt --key=../ca.unencrypted.key
```
Add rook.local to you /etc/hosts to the IP of the ingress, and then go to https://rook.local and login with
username admin and password edgefs
