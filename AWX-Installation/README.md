# AWX (Ansible Tower Community Edition) Installation on a Kubernetes Cluster

## Prerequisites
- A running Kubernetes cluster (v1.22+ recommended)
- kubectl configured to talk to the cluster
- Helm v3 installed
- Persistent storage support (hostPath below is for lab/development only)
- Sufficient resources (minimum suggestion: 2 CPU / 4GB RAM for AWX, plus PostgreSQL)

## 1. Install AWX Operator (Helm)
```sh
helm repo add awx-operator https://ansible-community.github.io/awx-operator-helm/
helm repo update
helm install ansible-awx-operator awx-operator/awx-operator -n ns-awx --create-namespace
```

Verify operator is running:
```sh
helm list -n ns-awx
kubectl get pods -n ns-awx
```

## 2. Create Storage Resources (Optional – if you do NOT have dynamic provisioning)
The AWX PostgreSQL and application data need storage. Below examples use hostPath (development only).

Apply the StorageClass and PersistentVolume:
```sh
kubectl apply -f awx-storage-class.yaml
kubectl apply -f awx-pv.yaml
```

## 3. Deploy AWX Custom Resource
This instructs the operator to create the AWX components.

```sh
kubectl apply -f awx.yaml
```

Check deployment status:
```sh
kubectl get pods -n ns-awx
kubectl get svc -n ns-awx
```

Wait until the AWX pod is Ready and the service shows a NodePort.

## 4. Retrieve Admin Password
```sh
curl -I http://localhost:32591kubectl get secrets -n ns-awx | grep -i admin-password
kubectl get secret awx-admin-password -n ns-awx -o jsonpath="{.data.password}" | base64 --decode ; echo
```
If the operator used a different secret name, list all secrets and adjust:
```sh
kubectl get secret -n ns-awx | grep admin
```

## 5. Access the AWX Web UI
Find the NodePort:
```sh
kubectl get svc awx-service -n ns-awx
```
Then browse to:
```
http://localhost:<node-port>
OR
http://<node-ip>:<node-port>/
```
Login:
- Username: admin
- Password: (from step 4)

## 6. Cleanup (Optional)
```sh
kubectl delete -f awx.yaml
helm uninstall ansible-awx-operator -n ns-awx
kubectl delete ns ns-awx
```

## Notes
- For production, replace hostPath with a real dynamic provisioner (e.g., CSI driver, NFS, Longhorn).
- You can enable ingress by adding `ingress_type: ingress` and supplying hostname/tls settings in `awx.yaml`.
- Adjust `web_node_port` to avoid conflicts.

## Files in this repository
- `awx-storage-class.yaml` – Static StorageClass (no provisioner)
- `awx-pv.yaml` – HostPath PersistentVolume
- `awx.yaml` – AWX Custom Resource definition

