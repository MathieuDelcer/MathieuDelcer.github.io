---
layout: post
title: Kubernetes CKA Certification Study Notes
date: 01-11-2023
categories: [study-notes]
tag: [kubernetes, orchestration, linux, certification]
---

## Context
I prepared for the Certified Kubernetes Administrator (CKA) certification primarily using the [KodeKloud course](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/?srsltid=AfmBOooQ_HP2nCpwTQPz7rRPilclKdxyp6vLB1SaPTGJPbeYjsLmptYq&couponCode=24T5MT071025) with hands-on labs + [killer.sh](https://killer.sh/cka). Here are the notes I took during my exam preparation.

## DNS
DNS for Services and Pods
- Instead of using an IP address, a service can be contacted via its DNS name.
- The DNS format: `service_name.namespace.service.domain`
- Example:
  ```
  db-service.dev.svc.cluster.local
  ```

---

## Checking Component Configuration
Example: kube-apiserver
- **Using its manifest file:**
  ```
  cat /etc/kubernetes/manifests/kube-apiserver.yaml
  ```
- **Checking the existing pod:**
  ```
  kubectl get po -n kube-system kube-apiserver -o yaml
  ```
- **Checking the process (on the control plane):**
  ```
  ps -aux
  ```

---

## Kubectl Configuration
- View current configuration:
  ```
  kubectl config view
  ```
- Switch to an existing context:
  ```
  kubectl config use-context contextName
  ```

---

## Scheduling
### Assigning Pods to Nodes
- **Using `nodeName`** (ignores taints):
  
  ```yaml
  spec:
    nodeName: node01
    containers:
    - image: nginx
      name: nginx
  ```
- **Using `nodeSelector`** (respects taints):
  
  ```yaml
  spec:
    containers:
    - image: nginx
      name: nginx
    nodeSelector:
      size: Large
  ```

### Node Affinity
Node affinity can be combined with Taints/Tolerations. There are different types:
- `requiredDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`
- `requiredDuringSchedulingRequiredDuringExecution`

The requiredDuringSchedulingIgnoredDuringExecution option ensures that a pod is only scheduled on a node that meets the specified node affinity rules. If no matching node is found, the pod will not be scheduled. However, once the pod is running, any changes to node labels do not affect it, meaning the pod will remain on that node even if the labels change.

The preferredDuringSchedulingIgnoredDuringExecution option is softer—it attempts to place the pod on a node that meets the affinity rules, but if no such node is available, it will schedule the pod on any available node. Like the required version, once the pod is running, changes to node labels do not impact it.

Example:

```yaml
spec:
  containers:
  - image: nginx
    name: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```
If we just want the label to exist without any specific value:

```yaml
    spec:
      affinity:   
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane  
                operator: Exists
```

---

## Custom Scheduler
- Assigning a pod to a custom scheduler:
  
  ```yaml
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-scheduler
  ```
- Creating a custom scheduler:
  
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      run: my-scheduler
    name: my-scheduler
    namespace: kube-system
  spec:
    serviceAccountName: my-scheduler
    containers:
    - command:
      - /usr/local/bin/kube-scheduler
      - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
      image: <use-correct-image>
      name: kube-second-scheduler
  ```

---

## DaemonSets
- Used to ensure a copy of a pod runs on each node.
- Useful for monitoring agents deployed on all nodes.
- Similar to Deployments, but `kind: DaemonSet` is used, and `replicas` and `strategy` fields are removed.

Example:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - image: k8s.gcr.io/fluentd-elasticsearch:1.20
        name: fluentd-elasticsearch
        resources: {}
```

Check DaemonSets:
```sh
kubectl get daemonsets
```

---

## Static Pods
- YAML files in `/etc/kubernetes/manifests` are automatically created and updated.
- Created by the kubelet directly without going through the API Server.
- Only pods can be static; no Deployments or ReplicaSets.
- Easy to find out if it's a static pod: its name ends with the node's name (e.g: kubescheduler-
controlplane).

If the default path has been changed, you can find the current one using:
```sh
cat /var/lib/kubelet/config.yaml | grep staticPodPath
```

---

## Metrics and Logs
### Sorting Nodes/Pods by Resource Usage
```sh
kubectl top nodes
kubectl top pods
kubectl top pods --sort-by=cpu
```

### Viewing Logs
```sh
journalctl -u etcd.service -l
kubectl logs etcd-master
```

---

## Kubernetes Upgrades
### Order of Upgrade
1. **Upgrade the control plane first.**
   - During downtime, management functions will be unavailable, but worker pods will continue running.
2. **Upgrade worker nodes.**
   - Can be done all at once (causing downtime) or one by one.

### Drain Nodes Before Maintenance
```sh
kubectl drain node01
```
- Moves pods to other nodes.
- Also applies `cordon`, preventing new pods from scheduling on the node.

### Uncordon After Maintenance
```sh
kubectl uncordon node01
```

### Check version
```sh
kubectl get nodes
```

### Kubernetes doc for upgrades
[Upgrade kubeadm cluster: controlplane  + worker node](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

### Example: upgrade controlplane
```sh
kubectl drain controlplane

apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm='1.27.0-00' && \
apt-mark hold kubeadm

sudo kubeadm upgrade apply v1.27.0

apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet='1.27.0-00' kubectl='1.27.0-00' && \
apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon controlplane
```

### Example 2: upgrade worker node
```sh
kubectl drain node01 --ignore-daemonsets #Depuis controlplane

ssh node01

apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm='1.27.0-00' && \
apt-mark hold kubeadm

sudo kubeadm upgrade node

apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet='1.27.0-00' kubectl='1.27.0-00' && \
apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon node01 #Depuis controlplane
```

---

## Backup and Restore

### ETCD: Verify Configuration

Check the ETCD used by the API-Server:
```bash
kubectl get po -n kube-system kube-apiserver -o yaml
```
If you see `127.0.0.1` (localhost), it's stacked. If it's another IP, it means the ETCD instance is external.

View the ETCD Configuration:
```bash
kubectl describe po -n kube-system etcd-controlplane
```
This command will show the configuration of the ETCD pod, including the location of its data directory:
```yaml
volumes:
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
```
If you are logged into the server where ETCD is running:

View the ETCD process configuration
```bash
ps -ef | grep -i etcd
```

View the ETCD pod configuration file
```bash
cat /etc/kubernetes/manifests/etcd.yml
```

Filter the output to show only certificate files
```bash
cat /etc/kubernetes/manifests/etcd.yml | grep file
```

Alternative: view the ETCD pod configuration in YAML format
```bash
kubectl get po -n kube-system etcd-controlplane -o yaml
```

List the nodes that are members of the ETCD cluster:
```bash
ETCDCTL_API=3 etcdctl member list
```

To create a backup of ETCD, find the data directory location in the `etcd.service` file:
```bash
systemctl list-unit-files # Find the name of the "etcd.service" file
systemctl cat etcd.service 
--data-dir=/var/lib/etcd-data
```
Check the location of ETCD certificate files:
```bash
ls /etc/kubernetes/pki/etcd
```

### ETCD Backup

ETCDCTL commands (except restore) requires parameters. E.g:
`--endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.crt --cert=/etc/etcd/etcdserver.
crt --key=/etc/etcd/etcd-server.key`

Backup:

```sh
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

Check the backup:

```sh
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

### Restore ETCD
Indicate the new path for the restored backup
```sh
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
```

Then update the ETCD manifest file to use the new backup path:
```yaml
volumes:
- hostPath:
    path: /var/lib/etcd-from-backup
    type: DirectoryOrCreate
  name: etcd-data
```

Restart ETCD:
```sh
systemctl daemon-reload
service etcd restart
service kube-apiserver start
```

---

## Certificate Management
### Generating and signing a Certificate Authority (CA)
```sh
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

### Verifying an Existing Certificate
```sh
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

### Using a key/certificate to access the API-Server
You can use a key/certificate to access the API-Server instead of logging in. Here is an example command:
```bash
curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt
```
### Viewing details about a certificate
You can use `openssl` to view the details about a certificate that is already in place. Here's an example command:
```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

### Certificate Signing Request (CSR)
Creating a new CSR:
If you want to add a second admin instead of using the CA server, you can generate your key and request for signing the certificate (.csr). Here is an example YAML object:
```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: admin2
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request:
    # Replace with the base64 encoded CSR content
    contenu_certificat_csr_encodé_en_base64
```
Note: to get the base64-encoded CSR content, you can use a command like this:
```bash
cat admin2.csr | base64 -w 0
```
After applying the object, verify the pending CSRs using `kubectl`:
```bash
kubectl get csr
```
Approve the CSR:
```bash
kubectl certificate approve admin2
```

---

## RBAC (Role-Based Access Control)

To manage access to resources based on user roles.

Note that if you want to control access to cluster-wide resources, not just within a specific namespace (NS), you need to create:
- ClusterRole: defines the permissions for a role at the cluster level
- ClusterRoleBinding: binds a role to a user or group at the cluster level

### Creating a Role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### Creating a RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-developer-binding
  namespace: default
subjects:
- kind: User
  name: dev-user
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

Can be done using imperative commands:
```sh
kubectl create role developer --verb=list,create,delete --resource=pods
kubectl create rolebinding dev-user-binding --role=developer --user=dev-user
```

### Verifying user access
```sh
kubectl auth can-i create deployments
kubectl auth can-i delete nodes
kubectl auth can-i create deployments --as dev-user
kubectl get pod -n blue my-pod --as username
```
---

## Kubeconfig

### Authenticating with the Kube API Server using CURL
```sh
curl https://my-kube-playground:6443/api/v1/pods \
  --key admin.key \
  --cert admin.cert \
  --cacert ca.crt
```

### Authenticating with the Kube API Server using KUBECTL
```sh
kubectl get pods \
  --server my-kube-playground:6443 \
  --client-key admin.key \
  --client-certificate admin.crt \
  --certificate-authority ca.crt
```

### Using a KubeConfig File

To avoid specifying credentials every time, use the `KUBECONFIG` file.

Default Configuration Location

The `kubectl` tool will automatically use the configuration stored in the following location:

```sh
$HOME/.kube/config
```

### Config YAML Structure
```yaml
apiVersion: v1
kind: Config

current-context: dev-user@google

clusters:
- name: my-kube-playground
- name: development
- name: production
- name: google

contexts:
- name: my-kube-admin@my-kube-playground
- name: dev-user@google
- name: prod-user@production

users:
- name: my-kube-admin
- name: admin
- name: dev-user
- name: prod-user
```

### Working with Contexts

- Default context
By default, you will be using the `current-context`.

- Switching to a different context  
```sh
kubectl config use-context prod-user@production
```

- Using a custom config File  
```sh
kubectl config use-context --kubeconfig=/root/my-kube-config research
```

- Verifying the used configuration
   
```sh
kubectl config view
kubectl config view --kubeconfig=my-custom-config
```

- Setting a custom KUBECONFIG File as Default

To set a custom `KUBECONFIG` file as default, move it to the following location:

```sh
mv /root/my-kube-config /root/.kube/config
```

---

## Debugging

### If components deployed by kubeadm
```sh
kubectl get nodes
kubectl get pods -n kube-system
kubectl logs kube-apiserver-master -n kube-system #check controlplane logs components
```

### If deployed by services
Check if kubelet running & kube-apiserver logs
```sh
ps aux | grep kubelet 
sudo journalctl -u kube-apiserver 
```
On Master node:
```sh
service kube-apiserver status 
service kube-controller-manager status 
service kube-scheduler status 
```
On Worker node:
```sh
service kubelet status 
service kube-proxy status
```
Debugging node with crictl:
```sh
crictl ps -a
crictl logs container-id
```

---

## JSON Queries in Kubernetes

- Get the first container's image:
  ```sh
  kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
  ```
- Get CPU capacities of all nodes:
  ```sh
  kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'
  ```
- Combine 2 requests:
```sh
kubectl get nodes -o=jsonpath=.'{.items[*].status.nodeInfo.architecture}
{.items[*].status.capacity.cpu}'
```

---

## Miscellaneous
Kubelet config folder
`/var/lib/kubelet`

Test a kubeconfig file:
```sh
kubectl get pods --kubeconfig /path/to/the/file
```

Viewing Cluster Events:
```sh
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

Finding Service CIDR:
```sh
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep range
```

Checking Container Runtime Info:
```sh
crictl ps | grep pod_name
crictl inspect container_id | grep runtimeType
crictl logs container_id
```

Find CNI (Container Network Interface)
```sh
ls /etc/cni/net. d
```

Check kubeadm version and possible upgrades
```sh
kubeadm version
kubeadm upgrade plan
```

Update worker node and join the cluster
```sh
ssh worker-node
apt install kubectl=1.28.2-00 kubelet=1.28.2-00
service kubelet restart
ssh controlplane
kubeadm token create --print-join-command
<output>
kubeadm token list
ssh worker-node
<copy_output>
```

Backup all resources yaml
```sh
kubectl get all -A -o yaml > all-deploy-services.yml
```