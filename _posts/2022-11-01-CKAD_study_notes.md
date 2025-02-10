---
layout: post
title: Kubernetes CKAD Certification Study Notes
date: 01-11-2022
categories: [study-notes]
tag: [kubernetes, orchestration, linux, certification]
---

## Context
I prepared for the Kubernetes Certified Application Developer (CKAD) certification primarily using the [KodeKloud course](https://www.udemy.com/course/certified-kubernetes-application-developer/?srsltid=AfmBOoor5Dvoq2Nlo7QF8atbBD4qPKliplubQPY-ojSthI2T8jO1E6ET) with hands-on labs + [killer.sh](https://killer.sh/ckad). Here are the notes I took during my exam preparation.

## EXPLAIN / HELP / OPTIONS
- Get a list of all API versions 
`kubectl api-resources`
- View documentation for a resource
`kubectl explain pod --recursive`
- View documentation for a resource and a specific field (e.g., ResourceQuota.spec.hard)
`kubectl explain ResourceQuota.spec.hard`
- Get help on imperative commands and their usage
`kubectl create --help`
- Get all available command options
`kubectl options`

<br/>

## TIPS
- Test a command without creating the resource
`--dry-run=client`
- Export output to a YAML file
`-o yaml > fichier.yml`
- Create an alias for kubectl
`alias k=kubectl`
- Create a variable (do) for exporting in YAML, then use it with $do
`export do="--dry-run=client -o yaml"`
- List all resources across all namespaces
`kubectl get all -A`
- Display detailed information (wide output)
`kubectl get pods -o wide`
- Run a command as a specific user
`kubectl get nodes --as nom_utilisateur`
- Create a temporary pod that deletes itself after running a command
`kubectl run exemple --image=nginx --rm -it --restart=Never -- whoami`
- Create a pod just to test a connection
`k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl 10.0.0.67`

`k run tmp --restart=Never --rm -i --image=busybox -i -- wget -O- frontend:80`

<br/>

## PODS
- Create directly
`kubectl run nginx --image=nginx`
- Create a template
`kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yml`
- Create a pod using the template
`kubectl apply -f pod.yml`
- Forcefully replace a pod with a new YAML file
`kubectl replace -f simple-webapp-2.yaml --force`
- Export the configuration of an existing pod to YAML
`kubectl get pod nginx -o yaml > nginx-pod.yaml`
- Edit a pod
`kubectl edit pod nginx`
- Change the image of a pod
`kubectl set image pod/nom_pod CONTAINER_NAME=IMAGE_NAME:TAG`
- Create a pod that allows traffic on port 8080
`kubectl run custom-nginx --image=nginx --port=8080`
- Create a pod and a service (ClusterIP type)
`kubectl run custom-nginx --image=nginx --port=8080 --expose`
- List pods
`kubectl get pods`

`kubectl get pod <name>`
- Check pod status 
`kubectl describe pods`

`kubectl describe pod <name>`
- View pod logs 
`kubectl logs <nom_pod>`
- View logs from the previous instance of a pod
`kubectl logs nginx -p`
- Delete a pod 
`kubectl delete pod <name>`

`kubectl delete pods <pod> --grace-period=0 --force` #To force deletion

- Delete all pods
`kubectl delete pods --all`
- Execute a command inside an existing pod
`kubectl exec [POD] -- [COMMAND]`
- Execute multiple commands inside an existing pod
`kubectl exec exemple -- /bin/bash -c 'whoami;pwd'`
- Create a pod and directly execute a command in it
`kubectl run exemple --image=nginx -it --restart=Never -- whoami`
- Run commands and arguments inside a pod
`kubectl run nginx --image=nginx --command -- <cmd1> <arg1> <arg n>` #command + args if needed

`kubectl run nginx --image=nginx -- <arg1> <arg n>` #only args
- Access a pod to run commands (-c to specify a particular container)
`kubectl exec -it pod_name -c nom_container -- /bin/sh`

`kubectl exec --stdin --tty nom_pod -- /bin/bash`
- Schedule a pod on a specific node

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: foo-node # schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```
- Schedule a pod on a node with a specific label

```
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

<br/>

## VOLUMES
- PV
  
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
- PVC
  
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
- Use PVC as Volume in a Pod
  
```
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: busybox
      volumeMounts:
        - mountPath: "/usr/share"
          name: task-pv-storage
```

<br/>

## REPLICASETS
- Scale 
`kubectl scale replicaset --replicas=5 <name>`

<br/>

## DEPLOYMENTS
- Create 
`kubectl create deploy --image=nginx nginx --replicas=3`
- Scale 
`kubectl scale deploy --replicas=5 <name>`
- Change the image of a deployment
`kubectl set image deployment/my-deployment mycontainer=myimage:1.15`

<br/>

## NAMESPACE
- Create a namespace
`kubectl create ns <name>`
- Total number of namespaces
`kubectl get ns --no-headers | wc -l`
- List the pods in a namespace
`kubectl get pods --namespace=<name>`
- Specify a namespace when creating a pod
`kubectl run nginx --image=nginx --namespace=<name>`
- List pods in all namespaces
`kubectl get pods --all-namespaces`
- Find the namespace of a pod
`kubectl get pods --all-namespaces | grep <nom_pod>`

<br/>

## SERVICE
- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
`kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml > redis-service.yml`
- Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml > nginx-svc.yml` #then add nodePort: 30080 in the template

<br/>

## CONFIG MAPS
- Create a ConfigMap with variables from a file
`kubectl create cm nom_cm --from-file=nom_variable=fichier.txt`
- Create a ConfigMap that contains "TIME=60"
`kubectl create cm nom_cm --from-literal=TIME=60`
- Use the entire ConfigMap as environment variable(s)

```
spec:
  containers:
  - envFrom:
    - configMapRef:
         name: webapp-config-map
    image: kodekloud/webapp-color
    name: webapp-color
```
- If you need a specific variable from the ConfigMap

```
    env:
      - name: COLOR
        valueFrom:
          configMapKeyRef:
            key: color
            name: webapp-config-map
```
- Mount the ConfigMap in a volume and use it as a file in a pod

```
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```

<br/>

## SECRETS
- Create and pass password directly
`kubectl create secret generic <nom_secret> --from-literal=DB_Host=sql --from-literal=id2=mdp2`
- Create secret with certificate and key
`kubectl create secret tls <nom_secret> --cert "/root/keys/server-tls.crt --key "/root/keys/server-tls.key`
- Use the secret as an environment variable

```
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: db-secret
```
- If you need a specific variable from the secret

```
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
```
- Mount the secret in a volume and use it as a file in a pod

```
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

<br/>

## SECURITY CONTEXT
- Who is executing the pod
`kubectl exec nom_pod -- whoami`

- Choose the user that executes the process in a pod

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
```
- Add a capability

```
securityContext:
      capabilities:
        add: ["SYS_TIME"]
```
<br/>

## NETWORK POLICY
- In "policyTypes", if neither "Ingress" nor "Egress" is specified, neither is affected, and all traffic is allowed by default. If only "Egress" is specified, "Ingress" is not affected, and all incoming traffic is allowed.

- When inbound traffic (Ingress) is allowed, the response to that traffic is permitted by default. There's no need to create a second rule for the response. For example, one rule is sufficient for an API pod to query a database and receive responses. When creating the rule, you only need to consider the direction of the incoming request. However, the database will not be able to initiate communication with the API pod unless an additional Egress rule is added for the database pod.

- When creating a rule, be cautious with the use of "-", as it changes the behavior of the rule.

- Here, the rule applies if the pod meets both conditions: ns=prod and name=api-pod:


```
- from 
  - podSelector:
      matchLabels:
        name: api-pod
    namespaceSelector:
      matchLabels:
        name: prod
```   
Here, the rule applies if the pod meets either of the two conditions: ns=prod OR name=api-pod.

```
- from 
  - podSelector:
      matchLabels:
        name: api-pod
  - namespaceSelector:
      matchLabels:
        name: prod
```   

- EXAMPLE 1:
We need a new NetworkPolicy named np that restricts all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2 . Incoming traffic not affected. The NetworkPolicy should still allow outgoing DNS traffic on port 53 TCP and UDP.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
  - to:
     - namespaceSelector:
        matchLabels:
         kubernetes.io/metadata.name: space2
```


- EXAMPLE 2:

Create a network policy to allow traffic from the Internal application only to the payroll-service and db-service.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: internal
  policyTypes:
    - Egress
  egress:
    - to:
      - podSelector:
          matchLabels:
            name: payroll
      ports:  
      - protocol: TCP
        port: 8080
    - to:
      - podSelector:
          matchLabels:
            name: mysql
      ports:  
      - protocol: TCP
        port: 3306
```

<br/>

## INGRESS RESOURCE
- EXAMPLE 1:
Create a new Ingress Resource for the service: my-video-service to be made available at the URL: http://ckad-mock-exam-solution.com:30093/video. Create an ingress resource with host: ckad-mock-exam-solution.com path: /video http://ckad-mock-exam-solution.com:30093/video should be accessible.

`k create ingress video-ingress --rule=ckad-mock-exam-solution.com/video*=my-video-service:30093`
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: video-ingress
spec:
  rules:
  - host: ckad-mock-exam-solution.com
    http:
      paths:
      - backend:
          service:
            name: my-video-service
            port:
              number: 8080
        path: /video
        pathType: Prefix
status:
  loadBalancer: {}
```

<br/>
 
## SERVICE ACCOUNT
A User account is used by humans (Admin, Developer...)
While a Service account is used by machines (Prometheus, Jenkins...)
The service account also creates a token, then a secret that uses this token, and finally links it to the service account

- Add serviceaccountname

```
spec:
      serviceAccountName: dashboard-sa
```

<br/>

## TAINTS & TOLERATIONS
- TAINTS are set on NODES, and TOLERATIONS are set on PODS
- List taints (for a node)
`kubectl describe node <name> | grep -i taints`
- Create a taint
`kubectl taint nodes node01 <name>=<value>:<effect>`

`kubectl taint nodes node1 key1=value1:NoSchedule`
- Remove a taint
`kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule-`
- Add a toleration to a pod

```
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: spray
    value: mortein
    effect: NoSchedule
    operator: Equal
```

<br/>

## LABELS
- Add a label to a node
`kubectl label node <name> color=blue`
- List labels of a node
`kubectl get node <name> --show-labels`
- List labels of pods
`kubectl get po --show-labels`
- List pods with the 'app' label
`kubectl get po -L app`
- List only the pods with app=v2
`kubectl get po --selector app=v2`
- List all objects with app=v2 and app=v1
`kubectl get all --selector app=v2,app=v1`
- Change the label of a pod directly
`kubectl label po nginx2 app=v2 --overwrite`
- Remove the 'app' label from a pod
`kubectl label po nginx1 app-`
- Add the label lab=test to all pods already labeled app=v2
`k label pod -l app=v2 lab=test`

<br/>

## ANNOTATIONS
- Add an annotation to a pod
`kubectl annotate pod <name> description='the_description'`
- Remove an annotation from a pod
`kubectl annotate pod <name> description-`
- Read annotations
`kubectl annotate pod <name> --list`

<br/>

## SORTING
- Sort by memory, CPU usage, etc.
`kubectl top node`

`kubectl top pod`

`kubectl top node --sort-by='memory'`

- Sort using a selector
`kubectl get pod --selector env=prod`

`kubectl get all --selector env=prod,bu=finance,tier=frontend`

<br/>

## JOBS
- Example: Create a Job with BusyBox that runs the command 'echo hello; sleep 30; echo world' 
`kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'`
- Important specifications that can be set in a Job:

`completions: 5` #5 iterations

`parallelism: 5`  #5 iterations in parallel

`activeDeadlineSeconds: 30` #Terminate if runtime exceeds 30 second

<br/>

## CRON JOBS
- Example: Create a CronJob
`kubectl create cronjob <nom_cron_job> --image=<img> --schedule "* * * * *" --dry-run=client -o yaml > cronjob.yml` 
- Specify the limit of successful and failed jobs to keep in history (default = 3 successes / 1 failure)
```
spec:
  successfulJobsHistoryLimit: 10
  failedJobsHistoryLimit: 0
```

<br/>

## KUBECONFIG AND CONTEXTS
- Default kubeconfig path
`$HOME/.kube/config`
- View kubeconfig file
`kubectl config view` 
- Show current context
`kubectl config current-context` 
- List all contexts
`kubectl config get-contexts`
- Switch to a specific context
`kubectl config use-context developer`

`kubectl config --kubeconfig=/path/config use-context <name>` 
- Set a default namespace for the current context
`kubectl config set-context --current --namespace=NAMESPACE`
- Create a context specifying a namespace, cluster, and user
`kubectl config set-context developer --namespace=development --user=martin --cluster=kubernetes`
- Configure a user with certificates
`kubectl config set-credentials martin --client-certificate ./martin.crt --client-key ./martin.key`

<br/>

## CLUSTER ROLE AND CLUSTER ROLE BINDING
- Create a ClusterRole
`kubectl create clusterrole nom_role --verb=get,list,watch --ressources=nodes`
- Create a ClusterRoleBinding
`kubectl create clusterrolebinding name-role-binding --clusterrole=name-role  --user=nom_user`

<br/>

## ROLLOUT
- Roll back a modification of a deployment
`kubectl rollout undo deploy nginx`
- Roll back to a specific version
`kubectl rollout undo deploy nginx --to-revision=2`
- View the rollout history (you can specify a desired version)
`kubectl rollout history deploy nginx --revision=4`

<br/>

## RESOURCE QUOTA
- Create a quota to limit CPU, memory, pod usage...
`kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2`

<br/>

## ADMISSION CONTROLLER
- check admission controller plugins
`cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep admission-plugins`
- enable admission controller plugin
`vim /etc/kubernetes/manifests/kube-apiserver.yaml`

```
- --enable-admission-plugins=NodeRestriction,LimitRanger,Priority,MutatingAdmissionWebhook
```
- disable admission controller plugin

```
--disable-admission-plugins=NamespaceLifecycle
```

<br/>

## API DEPRECIATIONS
- Show installed Kubernetes version
`k version --short`
- Find the API group of a resource
`k explain deploy`
```
This will show VERSION: apps/v1 .
The version is displayed as VERSION: {group}/{version}.
```
- Change the version in a YAML file on the first line
```
apiVersion: batch/v1
kind: CronJob
```

<br/>

## HELM
- Command help
`helm --help`
- Search for a chart (e.g: Wordpress)
`helm search hub wordpress`
- Add another repository to Helm (e.g: bitnami)
`helm repo add bitnami https://charts.bitnami.com/bitnami`
- View all repositories
`helm repo list`
- Search for a chart in the added repository 
`helm search repo wordpress`
- List installed charts 
`helm install release_name bitnami/wordpress`
- List installed charts 
`helm list`
- List all charts, including pending ones (by default, they are excluded)
`helm list -a`
- Uninstall a chart
`helm uninstall release-1`
- Download a chart without installing it (and extract it)
`helm pull --untar bitnami/wordpress`
- Install a chart from a file 
`helm install release_name ./wordpress`
- View the list of configurable values and use them during installation
`helm show values bitnami/apache`

`helm show values bitnami/apache | yq e #parse yaml and show with colors`

<br/>

## DOCKER
- help commands
`docker --help`
- build image ( . = if we are in the folder where the dockerfile is)
`docker build .`
- build image with a tag name => image_name:tag
`docker build -t webserver:latest .`
- list images
`docker images`
- push image to the registry
`docker push webserver:latest`
- create container (create fresh container, but doesn't run it immediately)
`docker container create <image> `
- start container 
`docker container start <id>`
- create and start a container (-d : container runs in background --name to give it a name)
`docker run -d --name mycontainer <image>`
- list container (-a to list all, including non-running containers)
`docker container ls -a`

`docker ps`
- logs
`docker container ls -a`