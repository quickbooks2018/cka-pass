# Certified kubernetes Administrator (CKA) Exam Preparation

## Below Commands help to prepare for CKA exam

- https://github.com/kodekloudhub/certified-kubernetes-administrator-course
- kubernetes cheat sheet   https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- kubernetes documentation https://kubernetes.io/docs/home/
- kubectl https://kubernetes.io/docs/reference/kubectl/conventions/

- Important Commands

```bash
kubectl get replace --force -f <file-name>
kubectl apply -f <file-name> --force
```

- Schedule a pod on a specific node Method 1 (Label the node)

```bash
kubectl label node <node-name> <label-key>=<label-value>
```
- Schedule a pod on a specific node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
    nodeSelector:
      <label-key>: <label-value>
    containers:
    - name: nginx:latest
```


- Schedule a pod on a specific node Method 2 (Node name)

```kubectl
kubectl -n default run pod-name --image=nginx:latest --restart=Never --node=<node-name>
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
    nodeName: <node-name>
    containers:
    - name: nginx
```

- Schedule a pod on a specific node Method 3 (Taints and Tolerations)

- Taint the node

```bash
kubectl taint node <node-name> <taint-key>=<taint-value>:<taint-effect>
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:latest
  tolerations:
  - key: special-hardware
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

### Label and Selectors

- Important Note: labels mentioned in pod TEMPLATE are actual labels, which must be selected by deployment or service to connect with pod (connection), so deployment and services uses

- kubectl label node --help

- kubectl label pods foo unhealthy=true
```yaml
selector:
  matchLabels:
    app: App1   
```

- filter with kubectl
```bash
kubectl get pods -n default -l tier=db
NAME         READY   STATUS    RESTARTS   AGE
db-1-d2hlf   1/1     Running   0          10m
db-2-hpnnx   1/1     Running   0          10m
db-1-fcr54   1/1     Running   0          10m
db-1-b79wp   1/1     Running   0          10m
db-1-c76z8   1/1     Running   0          10m

kubectl get pods -n default -l env=dev
NAME          READY   STATUS    RESTARTS   AGE
db-1-d2hlf    1/1     Running   0          10m
app-1-7tmj2   1/1     Running   0          10m
db-1-fcr54    1/1     Running   0          10m
app-1-qcr4m   1/1     Running   0          10m
app-1-tz9qx   1/1     Running   0          10m
db-1-b79wp    1/1     Running   0          10m
db-1-c76z8    1/1     Running   0          10m
```
- describe pod to list the labels on a pod
  
```bash
kubectl describe pod/db-1-d2hlf -n default
Name:             db-1-d2hlf
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.2.163.8
Start Time:       Tue, 26 Sep 2023 03:05:48 +0000
Labels:           env=dev
                  tier=db
Annotations:      <none>
Status:           Running
IP:               10.42.0.14
IPs:
  IP:           10.42.0.14
Controlled By:  ReplicaSet/db-1
```

- filter --no-headers
```bash
kubectl get pods -n default -l env=dev --no-headers | wc -l
```
- filter multiple labels
```bash
kubectl get all -A -l env=prod,bu=finance,tier=frontend
```

- Taints & tolerations
```bash
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint nodes node1 key=value:NoExecute
kubectl taint nodes node1 key=value:PreferNoSchedule
```
- Noschedule: No new pods will be scheduled on the node but existing pods will continue to run
- NoExecute: No new pods will be scheduled on the node and existing pods will be terminated (evicted) if they do not tolerate the taint
- PreferNoSchedule: Kubernetes will try to avoid scheduling new pods on the node but there is no guarantee

- Note: Taints tells node to except only those pods which have tolerations for the taints, but taints does not guarantee that pods will be scheduled on the node, it just tells node to except only those pods which have tolerations for the taints

- Note: if you have certain requirement to restrict a pod to certain node, it will achieve by using nodeaffinity.
- Taints & tolerations example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest  # Fixed the indentation here
  tolerations:             # tolerations are used to tolerate the taints
    - key: "key"           # key of the taint
      operator: "Equal"    # Fixed the indentation for the following lines
      value: "value"       # value of the taint
      effect: "NoSchedule" # effect of the taint
```

- kube-scheduler decides where to place pods in the cluster (Note: kube-scheduler is not a pod, it is a process running on the master node & it does not run on the worker node, further it does not responsible for running the pod on the node)
- kublet is responsible for running the pod on the node & it runs on the worker node.
- kubeadm tool does not install kublet, it is the responsibility of the admin to install kublet on the worker node.(manually)

### Sample yaml pod file with kubectl commands
- Note: Must use --dry-run=client
```bash
kubectl run redis --image=redis:latest -n default --dry-run=client -o yaml > pod-definition.yaml
```

### kubectl help
```bash
kubectl run --help
kubectl create --help
kubectl expose --help
kubectl get --help
kubectl describe --help
kubectl delete --help
kubectl edit --help
kubectl label --help
kubectl annotate --help
kubectl scale --help
kubectl autoscale --help
kubectl rollout --help
kubectl rollout history --help
kubectl rollout undo --help
kubectl rollout status --help
kubectl rollout restart --help
kubectl rollout pause --help
kubectl rollout resume --help
```

- ReplicaSet
- Note: ReplicaSet is the next generation of Replication Controller, ReplicaSet is the new way of creating Replication Controller.
- Replicaset yaml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
 ```
    
```bash
kubectl create -f replicaset-definition.yaml
kubectl get replicaset
kubectl get rs
kubectl describe replicaset myapp-replicaset
kubectl delete replicaset myapp-replicaset
kubectl replace -f replicaset-definition.yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl scale --replicas=6 replicaset myapp-replicaset
kubectl scale --replicas=6 rs/myapp-replicaset
kubectl edit replicaset myapp-replicaset
kubectl rollout status replicaset myapp-replicaset
kubectl rollout history replicaset myapp-replicaset
kubectl rollout undo replicaset myapp-replicaset
kubectl rollout undo replicaset myapp-replicaset --to-revision=2
kubectl rollout restart replicaset myapp-replicaset
kubectl rollout pause replicaset myapp-replicaset
kubectl rollout resume replicaset myapp-replicaset
```

- kubectl explain will list the fields of the resource, for example apiVersion, kind, metadata, spec, status etc.
```bash
kubectl explain pods    # list the fields of the pod
kubectl explain pod.spec # list the fields of the spec of pod
kubectl exaplain replicaset # list the fields of the replicaset
kubectl explain replicaset.spec # list the fields of the spec of replicaset
```

## YAML Creation from kubectl command

- Create an NGINX Pod
```bash
kubectl run nginx --image=nginx
```

- Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

- Create a deployment
```bash
kubectl create deployment --image=nginx nginx
```

- Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml 
```

- Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml 
```

- Make necessary changes to the file (for example, adding more replicas) and then create the deployment.
```bash
kubectl create -f nginx-deployment.yaml 
```

- In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.
```bash
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml 
```

#### ETCD – Commands (Optional)
- (Optional) Additional information about ETCDCTL UtilityETCDCTL is the CLI tool used to interact with ETCD.ETCDCTL can interact with ETCD Server using 2 API versions – Version 2 and Version 3.  By default it’s set to use Version 2. Each version has different sets of commands.

- For example, ETCDCTL version 2 supports the following commands:
```bash
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

- Whereas the commands are different in version 3
```bash
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put 
```

- To set the right version of API set the environment variable ETCDCTL_API command
```bash
export ETCDCTL_API=3 
```

- When the API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.
- Apart from that, you must also specify the path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don’t worry if this looks complex.
```bash
--cacert /etc/kubernetes/pki/etcd/ca.crt
--cert /etc/kubernetes/pki/etcd/server.crt
--key /etc/kubernetes/pki/etcd/server.key 
```

- So for the commands, I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form.
```bash
kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"
```

### Kubernetes Yaml fixation tools
- kubeval:
- Validates your Kubernetes configuration files against the official Kubernetes schemas, It can identify misconfigurations but doesn't necessarily automatically fix them.
```bash
 curl -L https://github.com/instrumenta/kubeval/releases/latest/download/kubeval-linux-amd64.tar.gz | tar xvz
sudo cp kubeval /usr/local/bin
kubeval my-k8s-deployment.yaml
```

- kube-score:
- kube-score is a tool that performs static code analysis of your Kubernetes object definitions. The output is a list of recommendations of what you can improve to make your application more secure and resilient.
```bash
curl -L https://github.com/zegl/kube-score/releases/download/v1.13.0/kube-score_1.13.0_linux_amd64.tar.gz | tar xvz
sudo cp kube-score /usr/local/bin
kube-score score my-k8s-deployment.yaml
```

- kube-linter:
- kube-linter is a static analysis tool that checks Kubernetes YAML files and Helm charts to ensure the applications represented in them adhere to best practices.
```bash
curl -L https://github.com/stackrox/kube-linter/releases/download/0.2.5/kube-linter-linux.tar.gz | tar xvz
sudo cp kube-linter /usr/local/bin
kube-linter lint my-k8s-deployment.yaml
```

- pluto:
- Pluto is a CLI tool to help discover deprecated apiVersions in Kubernetes. Pluto will examine Kubernetes manifests, cluster-wide, and report on deprecated apiVersions it finds.
```bash
curl -L https://github.com/FairwindsOps/pluto/releases/download/v5.18.5/pluto_5.18.5_linux_amd64.tar.gz | tar xvz
sudo mv pluto /usr/local/bin/
helm template -f values.yaml ./ | pluto detect -
```

### Kubernetes Service Examples

- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
- Note: (This will automatically use the pod’s labels as selectors)
- Note: Expose has flags --name  (Create has no flags --name)

```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client
kubectl expose pod redis --port=6379 --name redis-service
kubectl -n default expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

- Create service
-  (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)
- https://github.com/kubernetes/kubernetes/issues/46191
```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml

kubectl -n default create service clusterip redis-service --tcp=6379:6379 --dry-run=client -o yaml | sed '/name: 6379-6379/d'
```

- Expose Nginx pod example
- Create a Service named nginx of type NodePort to expose pod nginx’s port 80 on port 30080 on the nodes
- (This will automatically use the pod’s labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)
- https://github.com/kubernetes/kubernetes/issues/25478
- Note: Expose has flags --name  (Create has no flags --name)
```bash
kubectl -n default run nginx --image=nginx:latest
kubectl -n default run nginx --image=nginx --dry-run=client
kubectl -n default run nginx --image=nginx --dry-run=client -o yaml
kubectl -n default run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```
```bash
kubectl -n default expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
kubectl -n default expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml > nginx-service.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: default
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 32000  # Add this line manually
  selector:
    app: nginx
  type: NodePort
```

- Note: Create command (This will not use the pods labels as selectors)
```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml > nginx-service.yaml
```

- Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
- https://kubernetes.io/docs/reference/kubectl/conventions/

- kubectl run --port option https://hub.docker.com/layers/library/nginx/latest/images/sha256-0d60ba9498d4491525334696a736b4c19b56231b972061fab2f536d48ebfd7ce?context=explore

```bash
kubectl run custom-nginx --image=nginx:latest --port=8080 -n default
kubectl run custom-nginx-2 --image=nginx:latest --port=80 -n default
kubectl run custom-nginx-3 --image=nginx:latest -n default
```

- Note: "if you do **NOT** mentioned the port it will run pod on the actual port of container which is actually exposed in Dockerfile"
- "Note: pods custom-nginx-2 & custom-nginx-3 will work but custom-nginx-2 is better approach because LENS kubernetes will detect port for pod"

### Node Affinity and Pod Affinity
- Node affinity is used to place pods on nodes with specific labels.
- Pod affinity is used to place pods on nodes where certain pods are already running, based on labels on those pods.

- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
- Node Affinity
```bash
Node Affinity is conceptually similar to nodeSelector – it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on the node.
```
- Node Affinity Example with pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```
#### **task** Set Node Affinity to the deployment to place the pods on node01 only.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
      containers:
      - name: blue
        image: nginx
        ports:
        - containerPort: 80
```

- Pod Affinity
```bash
Pod affinity is conceptually similar to node affinity – it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on pods that are already running on the node rather than based on labels on nodes.
```
- Pod Affinity Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - other-app
        topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
```

### Operators
- The following are all the logical operators that you can use in the operator field for nodeAffinity and podAffinity mentioned above.

| Operator   | Behavior |
|------------|----------|
| In         | The label value is present in the supplied set of strings |
| NotIn      | The label value is not contained in the supplied set of strings |
| Exists     | A label with this key exists on the object |
| DoesNotExist | No label with this key exists on the object |

### Node Affinity Operators
The following operators can only be used with nodeAffinity.

| Operator | Behaviour |
|----------|-----------|
| Gt       | The supplied value will be parsed as an integer, and that integer is less than the integer that results from parsing the value of a label named by this selector |
| Lt       | The supplied value will be parsed as an integer, and that integer is greater than the integer that results from parsing the value of a label named by this selector |

**Note:** Gt and Lt operators will not work with non-integer values. If the given value doesn't parse as an integer, the pod will fail to get scheduled. Also, Gt and Lt are not available for podAffinity.

## VIM quick yaml edit (to move the text to right)  ---> shift+V & shift+>

- Create a daemonset with kubectl 
- Note: from explanation of daemonset, check the kind type & than update the yaml file, after applying, you will some errors, example replicas is not allowed in daemonset, so remove the replicas from yaml file & apply again, also remove "spec.strategy" from yaml file.
```bash
kubectl explain daemonset
kubectl create deployment --image=nginx nginx-daemonset --dry-run=client -o yaml --dry-run=client -o yaml > nginx-daemonset.yaml
kubectl apply -f nginx-daemonset.yaml
```

### Static Pods

- Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane (for example, a Deployment); instead, the kubelet watches each static Pod (and restarts it if it crashes).

- owner reference is not required for static pods, because static pods are not managed by the API server, they are managed by kubelet daemon on the node.

- owner reference you will find in the metadata section of the pod, it is used to identify the owner of the pod, for example, if you create a deployment, the deployment will be the owner of the pod, so if you delete the deployment, the pod will also be deleted.

- In case of static pod, owner reference will be shown as node, because static pod is managed by kubelet daemon on the node.

- How to check the path of static pod? see the file kubelet config file, it will show the path of static pod.

- location of kubelet config file
```bash
cat /var/lib/kubelet/config.yaml
```

- To find a static pod use a commands below, because lives only on specific node, so first the NODE than ssh on that NODE
```bash
ssh node01-example
cat /var/lib/kubelet/config.yaml | grep staticPodPath
```

- Kubernetes Scheduler, select a specific scheduler for a pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  schedulerName: my-scheduler
  containers:
    - name: nginx
      image: nginx
```

### Configmap Examples
```bash
kubectl create configmap --help
```

- Create a configmap named app-config with the data key1=val1
```bash
kubectl create configmap app-config --from-literal=key1=val1 -n default
kubectl create configmap app-config --from-literal key1=val1 -n default
```

- Create a configmap named app-config with the data key1=val1 and key2=val2
```bash
kubectl create configmap app-config --from-literal=key1=val1 --from-literal=key2=val2 -n default
kubectl create configmap app-config --from-literal key1=val1 --from-literal key2=val2 -n default
```

- Create a configmap named app-config with the data key1=val1 and key2=val2 from a file
```bash
kubectl create configmap app-config --from-file=app_config.properties
```

- Create a configmap named app-config with the data key1=val1 and key2=val2 from all files in a folder
```bash
kubectl create configmap app-config --from-file=.
```

- Create a configmap named app-config with the data key1=val1 and key2=val2 from all files in a folder and all subfolders
```bash
kubectl create configmap app-config --from-file=.. --recursive
```

- Create a configmap named app-config with the data key1=val1 and key2=val2 from all files in a folder and all subfolders with a specific file extension
```bash
kubectl create configmap app-config --from-file=.env --from-file=.. --recursive
```

- Create a configmap named app-config with the data key1=val1 and key2=val2 from all files in a folder and all subfolders with a specific file extension and with a specific namespace
```bash
kubectl create configmap app-config --from-file=.env --from-file=.. --recursive -n default
```

- Create a configmap in kube-system namespace with a file
```bash
kubectl create configmap app-config --from-file=app_config.properties -n kube-system
```
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
- configmap example reference in pod **_envFrom_**
```yaml
apiVersion: v1
kind: pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
 containers:
  - name: nginx-container
    image: nginx
    ports:
      - containerPort: 80
    envFrom:
      - configMapRef:
          name: app-config
```

- configmap example reference in pod with single env
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: default
  labels:
    app: myapp
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: key1
```
- configmap example reference in pod with multiple env
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod-2
  labels:
    app: myapp
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: key1
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: key2
```

### Kubernetes Logging & Monitoring

- metrics-server
```bash
kubectl top node
kubectl top pod
```

- kubectl logs
```bash
kubectl logs -f pod-name -n namespace
kubectl logs -f pod-name -n namespace --tail=10
kubectl logs -f pod-name -n namespace --since=1h
kubectl logs -f pod-name -n namespace --since-time=2021-09-26T03:05:48+0000
```

### Kubernetes Deployment Strategies

- Rolling Update
- Recreate
- Canary
- Blue/Green
- A/B Testing
- Shadow
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy

### Rolling Update
- Rolling Update is the default deployment strategy in Kubernetes. It updates the pods in a rolling fashion, one after the other. It ensures that the application is available throughout the update process. It also ensures that the application is not updated with a broken version of the image.

```bash
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
```

- scale the deployment
```bash
kubectl scale deployment nginx --replicas=4
```

- update the image of the deployment
```bash
kubectl set image deployment nginx nginx=nginx:1.18.0
```

- deployment rollout status
```bash
kubectl rollout status deployment/nginx
```

- rollout history
```bash
kubectl rollout history deployment/nginx
```

- rollout undo
```bash
kubectl rollout undo deployment/nginx
```

- rollout show all revisions
```bash
kubectl rollout history deployment/nginx -n default
```
- rollout show all revisions with images Note: to list images use --revision=1 flag
```bash
kubectl rollout history deployment/nginx -n default --revision=2
kubectl rollout history deployment/nginx -n default --revision=1
```
- rollout undo to specific revision
```bash
kubectl rollout undo deployment/nginx --to-revision=2
```

- rollout restart
```bash
kubectl rollout restart deployment/nginx
```

- rollout pause
```bash
kubectl rollout pause deployment/nginx
```

- rollout resume
```bash
kubectl rollout resume deployment/nginx
```

### JQ
```bash
kubectl get secrets/argocd-initial-admin-secret -o json -n argocd | jq .data.password -r
kubectl get secrets/argocd-initial-admin-secret -o json -n argocd | jq .data.password -r | base64 --decode
kubectl get secrets/argocd-secret -o json -n argocd | jq '.data["admin.password"]' -r | base64 --decode
```

#### ClusterIP to Nodeport Example
```bash
Access the ArgoCD-UI by converting the ArgoCD Server service from type ClusterIP to NodePort.

Use node port 32766 for https port.
```
```bash
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8080
  - name: https
    port: 443
    protocol: TCP
    targetPort: 8080
    nodePort: 32766
  selector:
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: NodePort
```

#### kubernetes pod command and args

- Note: command overrides the entrypoint of the container
- Note: args overides the CMD of the container

- kubectl help
```bash 
 # Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
  kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

  # Start the nginx pod using a different command and custom arguments
  kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>
```
- command example
```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml --command -- sleep 1000
```

- args example
```bash
kubectl run webapp-green --image=kodekloud/webapp-color -n default -o yaml --dry-run=client -- --color=green

kubectl run webapp-green --image=kodekloud/webapp-color -n default -o yaml -- --color=green
```

### kubernetes secrets

- kubernetes secrets example from literal value **_generic_**
```bash
kubectl create secret generic --help
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
kubectl create secret generic db-secret --from-literal DB_Host=sql01 --from-literal DB_User=root --from-literal DB_Password=password123
```

- Use secrets in pod **_envFrom_**
- https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: nginx-container
      image: nginx
      envFrom:
        - secretRef:
            name: db-secret
```

- Note best strategy in exam is to delete a pod and create with yaml, example yaml provided in https://kubernetes.io

- kubernetes mount a volume in pod, example we created a config map and mounted in pod
- kubectl create confimap --help
```bash
 kubectl create configmap app-config --from-literal=key1=val1 --from-literal=key2=val2 -n default
```
```bash
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: default
  labels:
    app: myapp
spec:
    containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
            - name: app-config
              mountPath: /etc/config
              readOnly: true
    volumes:
        - name: app-config
          configMap:
            name: app-config
```

### Init Containers

- Init containers are used to perform initialization logic before the actual application containers are started.
- Init containers always run to completion.
- If the init container fails, Kubernetes will keep restarting the pod until the Init container succeeds.
- Init containers are defined under the initContainers section, at the same level as containers section.
- Init containers are executed in order, one after the other.
- Init containers are useful when you want to perform an action before the actual application container starts.
- Init container example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: default
  labels:
    app: myapp
spec:
  containers:
    - name: nginx-container
      image: nginx
  initContainers:
    - name: init-myservice
      image: busybox
      command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done;"]
```

### Pod not starting troubleshooting, see logs of pod either using describe or logs command
```bash
kubectl describe pod pod-name -n namespace
kubectl logs pod-name -n namespace
```

- check logs of specific container in pod
```bash
kubectl logs pod-name -n namespace -c container-name
```

# kubernetes Self Healing Applications

- Kubernetes supports self-healing applications through (deployment) ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.

# hard restart all pods in default namespace
- Note: this will delete all pods in default namespace & application will be down, if you do not want downtime, you can use rolling update strategy
```bash
kubectl get pods -n default
kubectl delete pod --all -n default
```

- deployment rollout restart all deployments in argocd namespace
```bash
kubectl get deployments -n argocd -o name | xargs -n 1 kubectl rollout restart -n argocd
```

### kubernetes node maintenance

- We need to take node01 out for maintenance. Empty the node of all applications and mark it unscheduled.

```bash
kubectl drain --help
kubectl drain node01
kubectl drain node01 --ignore-daemonsets
```

- After maintenance is done, uncordon the node and make it schedule again.
```bash
kubectl uncordon node01
```

- Note: if a pod is running on a node, which is not of replica set, it will not be recreated, so you need to delete the pod manually & than drain the node.

- Warning: If pod is not part of replicaset, it will not be recreated or self healed.
```bash
kubectl drain node01 --ignore-daemonsets --force
```

- To make node unschedule
```bash
kubectl cordon node01
kubectl get nodes
```

### ETCD backup & restore
- doc https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

- To check the version etcd is running on, run the following command:
```bash
kubectl get pods -n kube-system
kubectl describe pod etcd-controlplane -n kube-system
kubectl describe pod etcd-controlplane -n kube-system | grep -iC5 image
```

- Note: Etcd is a static pod, so you cannot delete it, you need to delete the node, so that etcd will be recreated on another node.
- location of etcd pod will be /etc/kubernetes/manifests/etcd.yaml
- etcd configuration are available at cat /etc/kubernetes/manifests/etcd.yaml
- backup
```bash
export ETCDCTL_API=3
etcdctl --endpoints=https://192.133.162.8:2379 snapshot save /opt/cluster1_backup.db \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt
```
- restore
```bash
  ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore snapshot.db
```  

### kubernetes Cluster Upgrade with kubeadm

- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

- Note: To upgrade the control plane, you must be logged in as a user with sudo privileges on the control plane node.
- Note: To upgrade the kubelet and kubeadm packages, you must be logged in as a user with sudo privileges on each node in the cluster.
- Note: To upgrade worker nodes, you must be logged in as a user with sudo privileges on each node in the cluster.
- Note: To upgrade the kubelet and kubeadm packages, you must be logged in as a user with sudo privileges on each node in the cluster.

### Kubernetes backup & restore

- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

- backup kubernetes cluster means backup etcd database
- To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.
- You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:
- export ETCDCTL_API=3

- backup help
```bash
To see all the options for a specific sub-command, make use of the -h or –help flag.

 

For example, if you want to take a snapshot of etcd, use:

etcdctl snapshot save -h and keep a note of the mandatory global options.

Since our ETCD database is TLS-Enabled, the following options are mandatory:

–cacert                verify certificates of TLS-enabled secure servers using this CA bundle

–cert                    identify secure client using this TLS certificate file

–endpoints=[127.0.0.1:2379] This is the default as ETCD is running on master node and exposed on localhost 2379.

–key                  identify secure client using this TLS key file
```

- Note: stacked ETCD means ETCD is running as static pod, so you need to update the configuration file of static pod, so that it will use the new data directory.
- Note: external ETCD means ETCD is running as service, so you need to update the configuration file of ETCD, so that it will use the new data directory.
- Note: if no pod of etcd verify from kube-api server, because etcd is running as static pod, so you need to update the configuration file of static pod, so that it will use the new data directory.
- Note: Etcd server ip, you can also find by describing the pod of kube-api server, because etcd is running as static pod, so you need to update the configuration file of static pod, so that it will use the new data directory.
- Note: endpoints (etcd server addresses) are configured in the API server settings, and changes to the etcd configuration are made through the static pod manifest file, not directly in the kube-api server's configuration.
- Note: Find etcd service, you can see --data-dir=/var/lib/etcd
- Note: etcd server address is where the API server connects to the etcd database, essential for internal state management, whereas the advertise address is the public interface of the API server for all cluster communications.
```bash
ps -ef | grep etcd
ps aux | grep etcd
```

```bash
kubectl get pods -n kube-system
kubectl describe pod kube-apiserver-controlplane -n kube-system
```

- how can I find the control plane endpoint?
```bash
kubectl cluster-info
kubectl cluster-info
Kubernetes control plane is running at https://cluster1-controlplane:6443
CoreDNS is running at https://cluster1-controlplane:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

- Q. How many nodes are part of etcd server?
- A apply the following command
- Note: endpoint means etcd server address
```bash
ETCDCTL_API=3 etcdctl --endpoints 10.2.0.9:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  member list 
```

- Q. how can I find the control plane endpoint?
- A. kubectl cluster-info or  ps aux | grep etcd or kubectl describe pod kube-apiserver-controlplane -n kube-system

- backup
```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/cluster1_backup.db \
  --endpoints=https://etcd-server:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt
```

- backup status
```bash
ETCDCTL_API=3 etcdctl snapshot status /opt/cluster1_backup.db \
  --endpoints=https://etcd-server:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt
```

- restore
```bash
ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore snapshot.db
```

- Note: To restore a backup in case of non-stacked etcd or external etcd Step-1 copy the back the ETCD server node, Step-2 use this command 
```bash
ETCDCTL_API=3 etcdctl --data-dir /var/lib/etcd-database-backup snapshot restore snapshot.db
```
- Ownership of directory must be etcd:etcd
```bash
chown -R etcd:etcd /var/lib/etcd-from-backup
```
- Step3 update the configuration file of ETCD, so that it will use the new data directory.
- Step4A systemctl daemon-reload
- Step4B restart the ETCD service (ssh on ETCD server node)
- Step5 restart the kube-api server (ssh on control plane node)
- Step6 restart the kube-scheduler (ssh on control plane node)
- Step7 restart the kubelet (ssh on control plane node)
- kubectl get pods -n kube-system
- kubectl delete pod kube-controller-manager-controlplane -n kube-system
- kubectl delete pod kube-scheduler-controlplane -n kube-system
- Do not delete the PODS kube-api-server, kube-proxy, core-dns

- configure --data-dir in etcd config file
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster-from-a-backup

 ![etcd-restore.png](etcd-restore.png)

- etcd configuration file location
```bash
cat /etc/systemd/system/etcd.service
vim /etc/systemd/system/etcd.service
update --data-dir=/var/lib/etcd-from-backup
```

```bash
systemctl daemon-reload
systemctl restart etcd
systemctl restart kube-apiserver
systemctl restart kube-scheduler
systemctl restart kubelet
```

- Note: In our case we have etcd as static pod, we need to update the configuration file of our static pod

```bash
ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore snapshot.db

ETCDCTL_API=3 etcdctl --data-dir /var/lib/etcd-from-backup snapshot restore /backup/snapshot.db
```
- Important: Just update the host path of the static pod, do not change anything else, otherwise you will get error, because etcd is already running on the node, so you need to update the host path of the static pod, so that it will use the new data directory.
```bash
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://172.18.0.3:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://172.18.0.3:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://172.18.0.3:2380
    - --initial-cluster=k8s-control-plane=https://172.18.0.3:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://172.18.0.3:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://172.18.0.3:2380
    - --name=k8s-control-plane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.k8s.io/etcd:3.5.9-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health?serializable=false
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd   # update this path what you have in your backup, this will be /var/lib/etcd-from-backup snapshot
      type: DirectoryOrCreate
    name: etcd-data
status: {}
```

##### Certification Exam Tip!
- Here’s a quick tip. In the exam, you won’t know if what you did is correct or not as in the practice tests in this course. You must verify your work yourself. For example, if the question is to create a pod with a specific image, you must run the the kubectl describe pod command to verify the pod is created with the correct name and correct image.

### kubernetes cluster certificates

- help commands
```bash
kubectl certificate --help
kubectl certificate approve --help
kubectl certificate approve user-request
kubectl certificate approve user-request --kubeconfig=user.kubeconfig
```

- Note: kube-api server is kubernetes
- ETCD Logs
```bash
journalctl -u etcd.service -l
```

- Note in case stacked etcd, you need to update the configuration file of static pod, so that it will use the new certificates.

```bash
crictl ps
crictl ps -a
ctictl logs etcd-controlplane -n kube-system
```

- Note: After updating, use the above commands to verify that the pods are up and running. If a pod is failing, check its logs to identify the exact error. Kubelet will attempt to restart any failing pods, so it's important to monitor the logs for specific error messages.
- Note: if cluster is setup via kubeadm, most components are running as static pod, so you need to update the configuration file of static pod, so that it will use the new certificates.
- Restart Policy: Each pod has a restart policy associated with it, defined in the pod's specification. The most common restart policies are:
```note
Always: Restart the container if it stops. This is the default policy.
OnFailure: Restart the container only if it exits with a non-zero status.
Never: Never restart the container if it stops.
```

### Kubernetes Authentication & CSR (Certificate Signing Request)

- Convert document in to base 64
```bash
cat akshay.csr | base64 | tr -d '\n'
BASE64_ENCODED_CSR=$(cat akshay.csr | base64 | tr -d '\n')
export BASE64_ENCODED_CSR=$(cat akshay.csr | base64 | tr -d '\n')
```
- alternative command to print the value in a single line is below
```bash
cat akshay.csr | base64 -w 0
```
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  request: "${BASE64_ENCODED_CSR}"
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```

- See the CSR
```bash
kubectl get csr
```

- See additional details regarding CSR
```bash
kubectl get csr agent-smith -o yaml
```

- Describe the CSR
```bash
kubectl describe csr akshay
```

- Approve the CSR
```bash
kubectl certificate approve akshay
```

- Reject the CSR
```bash
kubectl certificate deny akshay
```

- Delete the CSR
```bash
kubectl delete csr akshay
```

- Linux tip for cat command
- Note: this command will take user input    cat > test.csr
```bash
cat > test.csr
provide the input
CTRL+C
```

### Kubernetes CONFIG understanding

- kubectl config --help
```bash
kubectl config --help
kubectl config view
kubectl config view --kubeconfig=user.kubeconfig
kubectl config view --kubeconfig=user.kubeconfig --minify
kubectl config view --kubeconfig=user.kubeconfig --minify --raw
kubectl config view --kubeconfig=user.kubeconfig --minify --raw | base64 --decode
```

- To use a specific kubeconfig file
```bash
kubectl --kubeconfig=user.kubeconfig get pods
export KUBECONFIG=user.kubeconfig
```

### Kubernete API Groups

- CORE API GROUP
- List Core API Groups
- kubectl proxy default port is 8001
```bash
kubectl proxy
curl http://localhost:8001 -k
```
- Note: kubectl proxy default port is 8001
```bash
kubectl proxy --port=8080 &
curl http://localhost:8080 -k
```

- NamedAPI Group

##### Role Based Access Control (RBAC)

- Create a role named pod-reader with the following parameters
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

- Create a role binding named pod-reader-binding with the following parameters
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
    kind: Role
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
```
- kubecrtl describe role and role binding
```bash
kubectl describe role pod-reader -n default
kubectl describe rolebinding pod-reader-binding -n default
```

- kubectl auth can-i --help
```bash
kubectl auth can-i --help
kubectl auth can-i create pods
kubectl auth can-i create pods --as john
kubectl auth can-i create pods --as john --namespace dev
kubectl auth can-i create pods --as john --namespace dev --all-namespaces
kubectl auth can-i create pods --as john --namespace dev --all-namespaces --list
kubectl auth can-i create pods --as john --namespace dev --all-namespaces --list --subresource=logs
kubectl auth can-i create pods --as john --namespace dev --all-namespaces --list --subresource=logs --verb=get
kubectl auth can-i create pods --as john --namespace dev --all-namespaces --list --subresource=logs --verb=get --resource=pods
kubectl auth can-i create pods --as john --namespace dev --all-namespaces --list --subresource=logs --verb=get --resource=pods --resource-name=nginx
kubectl auth can-i create deployments
kubectl auth can-i delete deployments
kubectl auth can-i delete nodes
```

- Q Inspect the environment and identify the authorization modes configured on the cluster.
- A kubectl describe pod kube-apiserver-controlplane -n kube-system
- A kubectl describe pod kube-apiserver-controlplane -n kube-system | grep -i authorization-mode
- ps aux | grep -i authorization

- Q.Which account is the kube-proxy role assigned to?
- A kubectl get rolebinding kube-proxy -n kube-system -o yaml
- see authorization section ---> apiGroup: rbac.authorization.k8s.io
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2023-12-04T02:29:51Z"
  name: kube-proxy
  namespace: kube-system
  resourceVersion: "300"
  uid: 5ed9dafd-f5d2-4b90-94b3-2b76ff3cf291
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kube-proxy
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: Group
    name: system:bootstrappers:kubeadm:default-node-token # This is the account 
```

- kubectl create role --help
```bash
kubectl create role --help
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods --dry-run=client -o yaml
```

- kubectl create rolebinding --help
```bash
kubectl create rolebinding --help
kubectl create rolebinding pod-reader-binding --role=pod-reader --user=john
kubectl create rolebinding pod-reader-binding --role=pod-reader --user=john --dry-run=client -o yaml
```

- kubectl create clusterrole --help
```bash
kubectl create clusterrole --help
kubectl create clusterrole pod-reader --verb=get --verb=list --verb=watch --resource=pods
kubectl create clusterrole pod-reader --verb=get --verb=list --verb=watch --resource=pods --dry-run=client -o yaml
```

- kubectl create clusterrolebinding --help
```bash
kubectl create clusterrolebinding --help
kubectl create clusterrolebinding pod-reader-binding --clusterrole=pod-reader --user=john
kubectl create clusterrolebinding pod-reader-binding --clusterrole=pod-reader --user=john --dry-run=client -o yaml
```

##### User Authorization test and check (Very important)
- kubectl get nodes --as michelle
- kubectl auth can-i get nodes --as michelle

#### Kubernetes Service Account
- kubectl create serviceaccount --help
```bash
kubectl create serviceaccount --help
kubectl create serviceaccount dashboard-sa
kubectl create serviceaccount dashboard-sa --dry-run=client -o yaml
```

- Create Service Account Token
```bash
kubectl create serviceaccount dashboard-sa
kubectl create serviceaccount dashboard-sa --dry-run=client -o yaml
```

- Note: kubectl create token dashboard-sa (This token will not stored in kubernetes,  it will be available on command line)

- kubectl private registry secret create https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

- Two Methods a per creation, on the above link
- 1. docker login (This will create a config.json file in .docker folder)
```bash
kubectl create secret generic regcred \
    --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
    --type=kubernetes.io/dockerconfigjson. 
```

- 2. kubectl cli to a create secret
```bash
kubectl create secret docker-registry regcred \
    --docker-server=<your-registry-server> \
    --docker-username=<your-name> \
    --docker-password=<your-pword> \
    --docker-email=<your-email>
```
- kubectl get secret regcred --output=yaml

- Understanding of private registry in image
- Image example
- first part is registry name (docker.io) second part is username (kodekloud) third part is image name (webapp-color) fourth part is tag (blue)
- if the username is not provided, it will take as public image and the default username is library
```bash
docker pull docker.io/nginx:1.14.0
docker pull gcr.io/google-samples/hello-app:1.0
docker pull quay.io/coreos/etcd:v3.3
```

- Note: If question ask for registry of image, you need to provide the registry name, username, image name and tag
- Note: image pull secrets are not part of container spec
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
    imagePullSecrets:
    - name: regcred
    containers:
        - name: private-reg-container
          image: kodekloud/webapp-color
```

- Kubernetes Security Context
- Run the Pod with the securityContext as defined below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]
```

- Note: Above Default Security Context is applied to all containers in the pod, but you can override the default security context at container level.

- Running pod configuration file
```yaml
kubectl get pod pod-name -o yaml
kubectl get pod pod-name -o yaml > pod-name.yaml
```

- Note: From above you will see the security context of the pod, but you will not see the security context of the container, so you need to run the below command to see the security context of the container
```bash
kubectl get pod pod-name -o yaml | grep -i securityContext
```

- By default, pods run with a UID of 0, which is the root user. You can change this to a non-root user for security reasons.

- kubectl delete pod pod-name --force --grace-period=0

##### Kubernetes Network Policies

- Kubernetes network policies allow you to define rules for how groups of pods are allowed to communicate with each other and other network endpoints.

- Kubernetes network policies can be implemented by a network plugin, such as Calico, or by the Kubernetes API server itself. The Kubernetes API server provides basic support for network policies, but it requires a network plugin that implements network policy support.
- Kubernetes network policy can be used with pod selector and labels.

- There are two types of network policies:
- 1. Ingress Network Policy
- 2. Egress Network Policy

- For learning perspective, just think of these as aws NACLs (Network Access Control List) which are stateless, and security groups which are stateful.
- kubectl create networkpolicy --help
```bash
kubectl create networkpolicy --help
kubectl create networkpolicy allow-nginx --dry-run=client -o yaml
```

- How to check the network policy
```bash
kubectl get networkpolicies -n default
kubectl get networkpolicies --all-namespaces
kubectl get networkpolicies --all-namespaces -o yaml
kubectl get networkpolicies --all-namespaces -o yaml | grep -i podSelector
kubectl get networkpolicies --all-namespaces -o yaml | grep -i podSelector -A 5
kubectl get networkpolicies --all-namespaces -o yaml | grep -i podSelector -A 5 | grep -i podSelector
kubectl get networkpolicies --all-namespaces -o yaml | grep -i podSelector -A 5 | grep -i podSelector -B 5
kubectl get networkpolicies --all-namespaces -o yaml | grep -i podSelector -A 5 | grep -i podSelector -B 5 | grep -i podSelector
kubectl get networkpolicies --all-namespaces -o yaml | grep -i podSelector -A 5 | grep -i podSelector -B 5 | grep -i podSelector -B 5
```

- For Network Policy use describe command
```bash
kubectl get networkpolicy/payroll-policy -n default -o yaml
kubectl describe networkpolicy payroll-policy -n default
```

- Network Policies short form
```bash
kubectl get netpol 
```

- Question on which pod network policy is applied from below yaml policy
- Note: podSelector is applied on the pods which have label name=payroll
- kubectl describe netpol payroll-policy -n default
```yaml
Name:         payroll-policy
Namespace:    default
Created on:   2023-12-13 01:59:04 -0500 EST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     name=payroll   # This is applied on the pods which have label name=payroll
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal
  #Not affecting egress traffic
  Policy Types: Ingress
```

- Note: In INGRESS we have (from) and in EGRESS we have (to) (very important)

- To allow multiple ports and pod selector
```yaml
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

# Example of configmap and secret in pod

- Create a configmap from a file, it will be available in kubernetes & it can be mounted in pod
```bash
kubectl create configmap --help
kubectl create configmap app-config --from-file=app_config.properties
kubectl create configmap app-config --from-file=app_config.properties --dry-run=client -o yaml
```
- ConfigMap Example sample
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    user nginx;
    worker_processes auto;
    # ... rest of the nginx configuration ...

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          volumeMounts:
            - name: config-volume
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
      volumes:
        - name: config-volume
          configMap:
            name: nginx-config
```

- Secrets Examples
```bash
kubectl create secret generic --help
```
```bash
echo -n 'your-api-key' | base64
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secrets
type: Opaque
data:
  api-key: <base64-encoded-api-key>
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: secret-volume
          mountPath: /etc/nginx/secrets
          readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
      - name: secret-volume
        secret:
          secretName: nginx-secrets
```

### PVC example with mysql
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: "mysql"
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "yourpassword"
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  type: ClusterIP
```

### Volume Mount in kubernetes

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumes-typed-hostpath

- Note: When you use a below command, you will a mount section in a pod
```bash
kubectl get pod pod-name -o yaml -n default
```

- https://kodekloud.com/topic/practice-test-persistent-volume-claims/
- Create a Persistent Volume with the given specification
- specification is given in the question
```spec
Volume Name: pv-log

Storage: 100Mi

Access Modes: ReadWriteMany

Host Path: /pv/log

Reclaim Policy: Retain 
```

- Answer
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
    capacity:
        storage: 100Mi
    volumeMode: Block
    accessModes:
        - ReadWriteMany
    hostPath:
        path: /pv/log
    persistentVolumeReclaimPolicy: Retain
```

- Tip: search in kubernetes.io create a persistent volume claim, you will see it shows how to create pv and than claim
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes

### kubernetes Networking

- Basic networking switch and routers

- Switch allows to perform communication inside network.

- In order to communicate between two different networks, we need routers, that is where we configure gateway, which is router

- In linux, to see the existing routing, use the command "route"

- In order to connect with other system, we need to add route of other system

- Example in linux, we can add route (this of aws vpc peering)
```bash
ip route add 192.168.1.0/24 via 192.168.2.1
```

- Ip forwarding or packet forwarding
```explaination
IP forwarding, also known as packet forwarding, is a fundamental process in computer network routing where a network device (like a router) forwards an incoming IP packet to its destination. It's essential for the movement of data packets from one network to another, especially in scenarios where networks are interconnected.
```

- Linux /proc/sys/net/ipv4/ip_forward
```explanation
The command cat /proc/sys/net/ipv4/ip_forward in a Linux operating system is used to display the current value of the ip_forward setting. This setting determines whether IP forwarding is enabled or disabled on the system.


cat: This is a standard Unix utility that reads files and outputs their contents. In this context, it's being used to read the contents of a specific system file.
/proc/sys/net/ipv4/ip_forward: This is the path to a special file in the /proc filesystem. The /proc filesystem is a virtual filesystem in Linux that provides a window into the kernel's view of the system. The specific file ip_forward controls the IP forwarding setting.
Use Case of IP Forwarding:

Router or Gateway Functionality: The most common use of IP forwarding is to turn the Linux machine into a router or gateway. This means the machine can forward traffic between multiple networks. For instance, if you have a Linux server that sits between a local network and the internet, enabling IP forwarding allows it to route traffic from the local network to the internet and vice versa.

Virtual Private Network (VPN) and Tunneling: IP forwarding is essential for VPNs and tunneling protocols. When a Linux system is set up as a VPN server, it needs to forward traffic from VPN clients to the appropriate destinations.

Network Address Translation (NAT): In combination with iptables (a tool for configuring firewall rules in Linux), IP forwarding can be used to implement NAT. This is particularly useful in environments where multiple devices share a single public IP address.

Load Balancing and High Availability: IP forwarding can be part of a setup for load balancing and high availability configurations. It allows the system to forward requests to different servers based on load, ensuring better distribution of network traffic.

Development and Testing: Network developers and administrators might enable IP forwarding for testing networking software, protocols, or for setting up complex network scenarios in a controlled environment.

How to View and Modify IP Forwarding Setting:

To view the current setting, you use cat /proc/sys/net/ipv4/ip_forward. A value of 1 indicates that IP forwarding is enabled, while 0 means it is disabled.
To enable IP forwarding temporarily (until the next reboot), you can use the command echo 1 > /proc/sys/net/ipv4/ip_forward.
To make the change permanent, you would modify the corresponding setting in the system's network configuration files, such as /etc/sysctl.conf or /etc/sysctl.d/.
Remember, enabling IP forwarding on a system that's not adequately secured can pose a security risk, as it could potentially expose internal networks to external threats. Therefore, it should be enabled only when necessary and with appropriate security measures in place.
```

- Network Basic

```explaination
ip link: This command displays information about network interfaces, such as Ethernet, WLAN, and loopback interfaces. It shows the state (up or down), MAC address, and other details. Changes made with ip link commands (like bringing an interface up or down) are non-persistent and will be lost after a reboot.

ip addr: This command is used to display and manipulate the IP address assigned to network interfaces. It shows IPv4/IPv6 addresses, their scope, and associated interfaces. Like ip link, changes made using ip addr commands are non-persistent.

ip addr add 192.168.1.10/24 dev eth0: This command assigns the IP address 192.168.1.10 with a subnet mask of 24 bits (255.255.255.0) to the network interface eth0. This change is non-persistent; the added IP address will be lost after a system reboot.

ip route: This command displays and modifies the kernel routing table. It's used for defining how packets should be routed within the network. Alterations made using ip route are also non-persistent.

route: Similar to ip route, this is an older command used for displaying and altering the network routing table. The changes made are non-persistent.

ip addr add 192.168.1.10/24 via 192.168.2.1: This command seems to be a mix-up. For assigning an IP address, via is not used. The via keyword is typically used in routing commands (like ip route add). To assign an IP address, you should use ip addr add 192.168.1.10/24 dev eth0. If setting a route, it would be something like ip route add 192.168.1.0/24 via 192.168.2.1. Either way, the change is non-persistent.

Persistence and Non-Persistence
Non-Persistent Changes: The changes made by the above commands are only effective until the next reboot. They are not saved in the system's permanent network configuration files. This is useful for temporary network adjustments or testing.

Persistent Changes: To make these changes persistent, you would need to add them to the system's network configuration files. The exact location and syntax of these files can vary depending on the Linux distribution. Common places include /etc/network/interfaces for Debian/Ubuntu or /etc/sysconfig/network-scripts/ for CentOS/Red Hat.

Network Managers: In modern Linux distributions, network settings are often managed by dedicated network managers (like NetworkManager or systemd-networkd). These tools have their own configuration files or interfaces for persistent network settings.

Remember, directly using ip commands for network configuration is great for immediate, temporary changes or for troubleshooting, but for a configuration that survives reboots, you should use the appropriate persistent configuration method for your specific Linux distribution.
```

- Note: We can configure linux box as a router as well, that is where, we need to enable ip forward or packet forwarding  /proc/sys/net/ipv4/ip_forward

- Name resolution mean adding entry in /etc/hosts

- Nameserver mean dns server, you make an entry in /etc/resolv.conf

```bash
cat /etc/resolv.conf
nameserver 8.8.8.8
```

- nameserver 8.8.8.8 is google cache only dns

- cat /etc/nsswitch.conf is where the sequence for hostfile and dns is mentioned, by default it is 
```linux
hosts: files dns 
```
- (files) which is /etc/hosts

- route alternative command 
```bash
arp
```

- To list the interfaces on linux box use command
```bash
ip link
```
- Note: In the official exam, all essential CNI deployment details will be provided

- Note: check the ip address of node, the ip assigned to node has interface, example use mentioned below commands
- Note: ip link and ip address are same
```bash
ifconfig
ip -4 a s
ip address
ip link
ip address show eth0
```

- To check the bridge on linux host
```bash
ip address show type bridge
```

- Check the default gateway
```bash
ip route
```

- Check the ports of service
```bash
netstat --help
netstat -aplntu
netstat -npl
```

- Check established connections
```bash
netstat -npa
```

- kubernetes CNI (Container Network Interface)
- location of CNI
```bash
ls /etc/cni/
net.d
root@cloudgeeks-control-plane:/# cat /etc/cni/net.d/10-kindnet.conflist 

{
        "cniVersion": "0.3.1",
        "name": "kindnet",
        "plugins": [
        {
                "type": "ptp",
                "ipMasq": false,
                "ipam": {
                        "type": "host-local",
                        "dataDir": "/run/cni-ipam-state",
                        "routes": [


                                { "dst": "0.0.0.0/0" }
                        ],
                        "ranges": [


                                [ { "subnet": "10.244.0.0/24" } ]
                        ]
                }
                ,
                "mtu": 1500

        },
        {
                "type": "portmap",
                "capabilities": {
                        "portMappings": true
                }
        }
        ]
}
```

- Note: What is kubernetes CNI (Container Network Interface)
- CNI is a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins. CNI concerns itself only with network connectivity of containers and removing allocated resources when the container is deleted. Because of this focus, CNI has a wide range of support and the specification is simple to implement.
- https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#network-plugin-requirements
- CNI is acting as a bridge between pod and network
- CNI runs script to configure the network
- CNI /opt/cni/bin/net-script.sh
- ./net-script.sh ADD <container-id> <namespace> <pod-name> <network-namespace> <interface-name>

- What is Container Runtime?
- A container runtime is a program responsible for running containers. The container runtime is the interface between the operating system kernels and the container. It manages container lifecycle events, and crucially, exposes an API that allows container management tools, such as Kubernetes, to control containers.
- Container runtime must create a network namespace for each container, and configure the network interface inside the container namespace. It must also configure the network bridge on the host, and add the container interface to the bridge.
- Container runtime must also configure the routing table on the host to ensure that traffic is routed to the correct network interface.
- Container runtime must also configure iptables rules to ensure that traffic is routed to the correct network interface.
- Container runtime must identify the network, the container must attach to. This is done using the network name provided in the pod configuration file.
- Container runtime must also identify the IP address and subnet mask to assign to the container interface. This is done using the IP address and subnet mask provided in the pod configuration file.
- Container runtime must also identify the MAC address to assign to the container interface. This is done using the MAC address provided in the pod configuration file.
- Container runtime must also identify the gateway to assign to the container interface. This is done using the gateway provided in the pod configuration file.
- Container runtime must also identify the DNS server to assign to the container interface. This is done using the DNS server provided in the pod configuration file.
- Container runtime must also identify the DNS search domain to assign to the container interface. This is done using the DNS search domain provided in the pod configuration file.
- Container runtime must also identify the port mappings to assign to the container interface. This is done using the port mappings provided in the pod configuration file.
- Container Runtime to invoke the (Network Plugin) bridge, when container is added or deleted

- Check the network plugin, which is used in kubernetes
```bash
ls /etc/cni/net.d
```

- Note CNI Weave Plugin
```note
Note CNI Weave
Important Update: –

Before going to the CNI weave lecture, we have an update for the Weave Net installation link. They have announced the end of service for Weave Cloud.

To know more about this, read the blog from the link below: –

https://www.weave.works/blog/weave-cloud-end-of-service

As an impact, the old weave net installation link won’t work anymore: –

kubectl apply -f “https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d ‘\n’)”

Instead of that, use the below latest link to install the weave net: –

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

Reference links: –

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation
https://github.com/weaveworks/weave/releases 
```

- CNI Binaries lives in /opt/cni/bin
- How we can check, what CNI plugin is used in kubernetes
```bash
ls /etc/cni/net.d
```
- How to check container runtime
```bash
ps aux | grep -i kubelet | grep -i container-runtime
```

- To find the cluster CIDR
```bash
kubectl -n kube-system describe cm kube-proxy | grep -iC5 clustercidr
```

- If you are setting up weave net, you need to provide the cluster CIDR, set IPALLOC_RANGE=10.244.0.0/16
- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation
- Download the weave net yaml file
- Note: do not do direct apply
```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml 
```
- Download the weave net yaml file
```bash
wget https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

- Add variable in the weave net yaml file
```yaml
containers:
  - name: weave
    env:
      - name: IPALLOC_RANGE
        value: 10.0.0.0/16 
```

- Note: it is the responsibility of the CNI plugin, to assign the IP address to the pod.

- Note: See the logs of weave net, this will the IPALLOC_RANGE
```bash
kubectl logs weave-net-xxxxx -n kube-system
ifconfig
```
- Note: ifconfig also shows the IPALLOC_RANGE see the weavenet interface

- Kubernetes DNS
- Note: POD Dns example 10-244-0-2.default.pod.cluster.local is disable by default
- Note: see the cat /etc/coredns or describe the coredns pod ---> configmap (coredns) ---> see the configmap
- Note: if you want to edit this configmap and restart the pod

- Question: What is the IP of the CoreDNS server that should be configured on PODs to resolve services.
- Answer: kubectl get svc -n kube-system the IP address you see, is used to resolve the services

- Question: Identify the DNS solution implemented in this cluster.
- Answer: kubectl get pods -n kube-system | grep -i dns

- How many pods of the DNS server are deployed?
- Answer: kubectl get pods -n kube-system --no-headers | grep -i dns | wc -l

- Question: What is the name of the service created for accessing CoreDNS?
- Answer: kubectl get svc -n kube-system | grep -i dns

- Question: Where is the configuration file located for configuring the CoreDNS service?
- Best Answer: kubectl describe pod/coredns-5d78c9869d-dchcf -n kube-system | grep -i /etc

- Answer: kubectl get cm -n kube-system | grep -i dns
- Answer: kubectl describe cm coredns -n kube-system

- Kubernetes Ingress

- Now, in k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-

- Format - kubectl create ingress <ingress-name> --rule="host/path=service:port"

- Example - kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"

- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-

- https://kubernetes.io/docs/concepts/services-networking/ingress/

- https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types

- Ingress – Annotations and rewrite-target
```text
Ingress – Annotations and rewrite-target
Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen here. I would like to explain one such option that we will use in our labs. The Rewrite target option.

 

Our watch app displays the video streaming webpage at http://<watch-service>:<port>/

Our wear app displays the apparel webpage at http://<wear-service>:<port>/

We must configure Ingress to achieve the below. When user visits the URL on the left, his/her request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forward users to the appropriate application in the backend. The applications don’t have this URL/Path configured on them:

 

http://<ingress-service>:<ingress-port>/watch –> http://<watch-service>:<port>/

http://<ingress-service>:<ingress-port>/wear –> http://<wear-service>:<port>/

 

Without the rewrite-target option, this is what would happen:

http://<ingress-service>:<ingress-port>/watch –> http://<watch-service>:<port>/watch

http://<ingress-service>:<ingress-port>/wear –> http://<wear-service>:<port>/wear

 

Notice watch and wear at the end of the target URLs. The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don’t expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.

 

To fix that we want to “ReWrite” the URL when the request is passed on to the watch or wear applications. We don’t want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

For example: replace(path, rewrite-target)

In our case: replace("/path","/")

 

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
 

In another example given here, this could also be:

replace("/something(/|$)(.*)", "/$2")

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```

- Create Ingress from imperative way
```bash
kubectl -n test create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

- Containerd Run time commands
```bash
ctr -n k8s.io images list
ctr -n k8s.io images list | grep -i nginx
ctr -n k8s.io images rm [IMAGE NAME]
```

- Containerd delete all images (dangerous)
```bash
ctr -n k8s.io images list -q | xargs -n 1 ctr -n k8s.io images rm
```

## kubernetes installation

- Kubernetes the HARD WAY
- https://github.com/mmumshad/kubernetes-the-hard-way
- https://www.youtube.com/watch?v=uUupRagM7m0&list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo

- Kubernetes installation using kubeadm
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

- check the init system
```bash
ps -p 1
```

## Container Runtime Installation is First Step of all, we will use containerd
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd


```explanation
When you install a Kubernetes cluster using kubeadm, it does not automatically install a container runtime for you. You are responsible for installing the container runtime on all nodes before initializing your Kubernetes cluster with kubeadm.

kubeadm supports multiple container runtimes:

Docker: Even though Docker is no longer the default container runtime used by Kubernetes, kubeadm still supports it. If you choose to use Docker, you'll need to install it manually on each node in your cluster.

containerd: This is a popular choice and is used as the default runtime in many Kubernetes installations. If you wish to use containerd, you must install it separately before initializing your Kubernetes cluster with kubeadm.

CRI-O: Another container runtime that is compatible with Kubernetes and can be used with kubeadm. Similar to the others, it must be installed manually.

The installation process generally involves:

Installing the chosen container runtime on each node.
Configuring the Kubernetes kubelet on each node to use the installed runtime.
Remember to follow the specific installation instructions for the container runtime you choose, as each runtime has its own configuration nuances. Once your container runtime is installed and configured on each node, you can then proceed with initializing your Kubernetes cluster using kubeadm
```

### Kubernetes Installation using kubeadm

- Q Install the kubeadm and kubelet packages on the controlplane and node01 

- First Step see the container dcoumentation

- https://kubernetes.io/docs/setup/production-environment/container-runtimes/

- Step 1
- Forwarding IPv4 and letting iptables see bridged traffic (Note: Run on ALL nodes)
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

- Verify that the br_netfilter, overlay modules are loaded by running the following commands:
- Note: Run on ALL nodes
```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

- Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:
- Note: Run on ALL nodes
```bash
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

### containeruntime configuration and installation (containerd)
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd

- First check the containerd binary
- https://github.com/containerd/containerd/blob/main/docs/getting-started.md

- https://github.com/containerd/containerd/tags

- If you intend to start containerd via systemd, you should also download the containerd.service unit file from https://raw.githubusercontent.com/containerd/containerd/main/containerd.service into /usr/local/lib/systemd/system/containerd.service, and run the following commands:

- https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

```bash
systemctl daemon-reload
systemctl enable --now containerd
```

```bash
ls /usr/bin/containerd
```

- Note: remove ... (3 dots) from the file

- To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set

- Make the file blank
```bash
> /etc/containerd/config.toml
```

```conf
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

- updated file
```conf
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

```bash
cat <<EOF > /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
EOF
```

- If you apply this change, make sure to restart containerd:
```bash
systemctl restart containerd
systemctl status containerd
systemctl enable containerd
```


- Step 2 
- Install the kubeadm and kubelet packages on the controlplane and node01 nodes.
- Use the exact version of 1.27.0-2.1 for both.
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

- cat /etc/os-release

- See 4 steps

- Step 1
```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

- Step 2 Note: this must according to your required package example 1.27.0-*
```bash
# If the folder `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

- Step 3 Note: this must according to your required package example 1.27.0-*
- Note: for kubeadm repo link is updated https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
```bash
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

- Step 4 Note: For specific version, use VERSION=1.27.0-*
```bash
sudo apt-get update
sudo apt-get install -y kubelet=1.27.0-* kubeadm=1.27.0-* kubectl=1.27.0-*
sudo apt-mark hold kubelet=1.27.0-* kubeadm=1.27.0-* kubectl=1.27.0-*
```

- Step 5 check and verify the version
```bash
kubeadm version
kubectl version
kubelet --version
```

- What is kubelet?
- The kubelet is the primary “node agent” that runs on each node. It can register the node with the apiserver using one of: the hostname; a flag to override the hostname; or specific logic for a cloud provider.
```explanation
The kubelet is indeed an essential component in the Kubernetes architecture, typically running on each node in the cluster. Its primary role is as a "node agent," which means it ensures that the containers are running in a Pod. Here's a breakdown of its functions:

Node Registration: The kubelet is responsible for registering the node with the Kubernetes API server. This registration can be done using the node's hostname, an override flag for the hostname, or through specific logic tailored for cloud providers.

Pod Management: It manages the lifecycle of Pods and their containers based on the specifications in Pod manifests. This includes starting, stopping, and maintaining application containers organized into Pods as needed.

Node Resource Management: Kubelet also monitors the health of nodes and containers. It reports back to the control plane, ensuring that the scheduler can make decisions about where to place new Pods based on resource availability and constraints.

Execute Container Commands: The kubelet can execute commands inside containers at the request of the API server.

Communication with API Server: It regularly communicates with the Kubernetes API server, both to receive instructions and to send updates about the node's status.

Overall, the kubelet is a critical part of the Kubernetes system, enabling containerized applications to run and be managed effectively across a cluster of machines.
```

- Q Initialize the cluster using kubeadm 
```question
Initialize Control Plane Node (Master Node). Use the following options:


apiserver-advertise-address - Use the IP address allocated to eth0 on the controlplane node

apiserver-cert-extra-sans - Set it to controlplane

pod-network-cidr - Set to 10.244.0.0/16

Once done, set up the default kubeconfig file and wait for node to be part of the cluster.
```
- A Use below command to initialize the cluster on Control Plane Node (Master Node)

Note: eth0 is the interface name on controlplane node which ip address is 192.22.194.6

```bash
ip add
ifconfig
kubeadm init --apiserver-advertise-address=192.22.194.6 --apiserver-cert-extra-sans=controlplane --pod-network-cidr=10.244.0.0/16
```

- Note: Copy the commands from output and run on the controlplane node
```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Note: to join the worker node, run the command on worker node
```bash
kubeadm join 192.22.194.6:6443 --token 00yh1c.2c5mrazy5aqk6y9u \
        --discovery-token-ca-cert-hash sha256:a9a792c40ecae9a3de73bf0cde2a563ec2c39da1b78d6b8dc6bf1abe917f248f
```

- Note: The above output also provide the network plugin document link
- https://kubernetes.io/docs/concepts/cluster-administration/addons/
```explain
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
```

- Last Step installation of network plugin, we will use flannel

- https://github.com/flannel-io/flannel#deploying-flannel-manually

- Flannel installation (Do not use this)
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

- Use this
```bash
wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```
```yaml
---
kind: Namespace
apiVersion: v1
metadata:
  name: kube-flannel
  labels:
    k8s-app: flannel
    pod-security.kubernetes.io/enforce: privileged
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: flannel
  name: flannel
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - networking.k8s.io
  resources:
  - clustercidrs
  verbs:
  - list
  - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: flannel
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-flannel
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: flannel
  name: flannel
  namespace: kube-flannel
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-flannel
  labels:
    tier: node
    k8s-app: flannel
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-flannel
  labels:
    tier: node
    app: flannel
    k8s-app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni-plugin
        image: docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1
        command:
        - cp
        args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        volumeMounts:
        - name: cni-plugin
          mountPath: /opt/cni/bin
      - name: install-cni
        image: docker.io/flannel/flannel:v0.24.2
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: docker.io/flannel/flannel:v0.24.2
        command:
        - /opt/bin/flanneld
        args:
        # https://stackoverflow.com/questions/47845739/configuring-flannel-to-use-a-non-default-interface-in-kubernetes
        - --iface=eth0 # updated
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: EVENT_QUEUE_DEPTH
          value: "5000"
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
        - name: xtables-lock
          mountPath: /run/xtables.lock
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni-plugin
        hostPath:
          path: /opt/cni/bin
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
      - name: xtables-lock
        hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
```

### Kubernetes trouble shooting

- Main commands

```bash
kubectl get pods -n beta
kubectl get deploy -n beta
kubectl get svc -n beta
kubectl get endpoints -n beta
```

- Note: kubectl get svc -n beta will the port number of service
- Note: kubectl get endpoints -n beta will the ip address of (pod ip address) and port number of pod (target port) which is container port

### Control Plane Failures

- check the status of Nodes
```bash
kubectl get nodes
```

- check the status of pods
```bash
kubectl get pods -n kube-system
```

- if control plane is setup via kubeadm, then check the status of control plane
```bash
kubectl get pods -n kube-system | grep -i control
```

- If control plane components are deployed as a service, then check the status of control plane
```bash
service kube-apiserver status
service kube-controller-manager status
service kubelet status
service kube-proxy status
service kube-scheduler status
```

- Check the logs of control plane
```bash
journalctl -u kube-apiserver
journalctl -u kube-controller-manager
journalctl -u kubelet
journalctl -u kube-proxy
journalctl -u kube-scheduler
```

- Check the logs of control plane if deployed as pod
```bash
kubectl logs kube-apiserver-controlplane -n kube-system
kubectl logs kube-controller-manager-controlplane -n kube-system
kubectl logs kubelet-controlplane -n kube-system
kubectl logs kube-proxy-controlplane -n kube-system
kubectl logs kube-scheduler-controlplane -n kube-system
```

- Note: Assigning a pod to a node, is the responsibility of kube-scheduler

- Note: All controllers are managed by kube-controller-manager eg. node controller, replication controller, endpoints controller, service account and token controller

### Worker Node Failures

- check the status of Nodes
```bash
kubectl get nodes
kubectl get nodes -o wide
```

- check the status of nodes in detail
```bash
kubectl describe nodes
```

- check the status of kubelet
```bash
systemctl status kubelet
```

- check the logs of kubelet
```bash
journalctl -u kubelet
```

- In-case kubelet is running, as a pod
```bash
kubectl logs kubelet-worker -n kube-system
```

- Check the kubelet certificate, make those are not expired
```bash
openssl x509 -in /var/lib/kubelet/pki/worker.crt -text
```

- Role of kubelet explained
```explain
The kubelet plays a crucial role in the Kubernetes architecture as the primary "node agent" running on every node in the cluster. Here's a detailed explanation of its responsibilities and functions:

Node Registration: One of the primary responsibilities of the kubelet is to register the node it is running on with the Kubernetes API server. This registration process is essential for the Kubernetes control plane to be aware of the node's existence and its resources. The kubelet can use the hostname of the node for registration, but it also provides flexibility. It allows the override of the hostname with a specific flag or employs cloud provider-specific logic to determine the node's identity. This flexibility is particularly useful in environments where the node's hostname might not be unique or in cloud environments where nodes are dynamically created and managed.

Pod Lifecycle Management: The kubelet is responsible for ensuring that the containers described in PodSpecs (Pod specifications) are running and healthy. It manages the lifecycle of containers from their creation, starting, stopping, and restarting, based on the state desired by the Kubernetes control plane. The kubelet continuously monitors the state of pods and containers on its node against the desired state defined in the PodSpecs and takes action to reconcile any differences.

Resource Management: It monitors the resources available on the node (such as CPU, memory, and storage) and allocates these resources to the containers as required. The kubelet ensures that each container has the resources it needs to operate as specified in its configuration.

Health Checking: The kubelet performs health checks on containers to ensure they are operating correctly. If a container fails a health check, the kubelet can restart it automatically, according to the policy defined in the PodSpec. This ensures high availability and reliability of the applications running in Kubernetes.

Node Communication: The kubelet communicates with the Kubernetes API server to receive commands and manifest files (which define the desired state for the pods and containers on the node) and to report back the status of those pods and containers. This communication is vital for the orchestration and management of the containerized applications across the cluster.

Log and Metrics Collection: Although not its primary role, the kubelet also facilitates the collection of logs and metrics from containers, making them available for troubleshooting and monitoring purposes.

In summary, the kubelet is a critical component in the Kubernetes ecosystem, acting as the agent on each node that communicates with the Kubernetes API server. It ensures that the containers are running as expected according to the configurations specified by the users and manages the node's resources to keep applications running smoothly.
```

- kubelet debug
```bash
journalctl -u kubelet | tail -n 10
```

- kubectl auto completion
- https://kubernetes.io/docs/reference/kubectl/quick-reference/
```bash
source <(kubectl completion zsh)  # set up autocomplete in zsh into the current shell
echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc # add autocomplete permanently to your zsh shell
```

- Note the advantage of using kubectl auto completion
```note
The kubectl command-line tool can be configured to provide command suggestions and auto-completion for kubectl commands and their arguments. This can be a huge time-saver, especially when working with complex commands or when you're not sure of the exact syntax for a particular command.
in case of troubleshooting, you can quickly access the commands and their arguments without having to remember the exact syntax. This can help you to quickly identify and resolve issues in your Kubernetes environment.
```

### Network Troubleshooting

```explanaion
Network Plugin in kubernetes
——————–

There are several plugins available and these are some.

1. Weave Net:

To install,

kubectl apply -f
https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

You can find details about the network plugins in the following documentation :

https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

2. Flannel :

To install,

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml

 

Note: As of now flannel does not support kubernetes network policies.

3. Calico :

 

To install,

curl https://docs.projectcalico.org/manifests/calico.yaml -O

Apply the manifest using the following command.

kubectl apply -f calico.yaml

Calico is said to have most advanced cni network plugin.

In CKA and CKAD exam, you won’t be asked to install the cni plugin. But if asked you will be provided with the exact url to install it.

Note: If there are multiple CNI configuration files in the directory, the kubelet uses the configuration file that comes first by name in lexicographic order.

DNS in Kubernetes
—————–
Kubernetes uses CoreDNS. CoreDNS is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS.

Memory and Pods

In large scale Kubernetes clusters, CoreDNS’s memory usage is predominantly affected by the number of Pods and Services in the cluster. Other factors include the size of the filled DNS answer cache, and the rate of queries received (QPS) per CoreDNS instance.

Kubernetes resources for coreDNS are:

a service account named coredns,
cluster-roles named coredns and kube-dns
clusterrolebindings named coredns and kube-dns, 
a deployment named coredns,
a configmap named coredns and a
service named kube-dns.
While analyzing the coreDNS deployment you can see that the the Corefile plugin consists of important configuration which is defined as a configmap.

Port 53 is used for for DNS resolution.

    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
This is the backend to k8s for cluster.local and reverse domains.

proxy . /etc/resolv.conf

Forward out of cluster domains directly to right authoritative DNS server.

Troubleshooting issues related to coreDNS
1. If you find CoreDNS pods in pending state first check network plugin is installed.

2. coredns pods have CrashLoopBackOff or Error state

If you have nodes that are running SELinux with an older version of Docker you might experience a scenario where the coredns pods are not starting. To solve that you can try one of the following options:

a)Upgrade to a newer version of Docker.

b)Disable SELinux.

c)Modify the coredns deployment to set allowPrivilegeEscalation to true:

kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
d)Another cause for CoreDNS to have CrashLoopBackOff is when a CoreDNS Pod deployed in Kubernetes detects a loop.

There are many ways to work around this issue, some are listed here:

Add the following to your kubelet config yaml: resolvConf: <path-to-your-real-resolv-conf-file> This flag tells kubelet to pass an alternate resolv.conf to Pods. For systems using systemd-resolved, /run/systemd/resolve/resolv.conf is typically the location of the “real” resolv.conf, although this can be different depending on your distribution.
Disable the local DNS cache on host nodes, and restore /etc/resolv.conf to the original.
A quick fix is to edit your Corefile, replacing forward . /etc/resolv.conf with the IP address of your upstream DNS, for example forward . 8.8.8.8. But this only fixes the issue for CoreDNS, kubelet will continue to forward the invalid resolv.conf to all default dnsPolicy Pods, leaving them unable to resolve DNS.
3. If CoreDNS pods and the kube-dns service is working fine, check the kube-dns service has valid endpoints.

kubectl -n kube-system get ep kube-dns

If there are no endpoints for the service, inspect the service and make sure it uses the correct selectors and ports.

Kube-Proxy
———
kube-proxy is a network proxy that runs on each node in the cluster. kube-proxy maintains network rules on nodes. These network rules allow network communication to the Pods from network sessions inside or outside of the cluster.

In a cluster configured with kubeadm, you can find kube-proxy as a daemonset.

kubeproxy is responsible for watching services and endpoint associated with each service. When the client is going to connect to the service using the virtual IP the kubeproxy is responsible for sending traffic to actual pods.

If you run a kubectl describe ds kube-proxy -n kube-system you can see that the kube-proxy binary runs with following command inside the kube-proxy container.

    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
 

So it fetches the configuration from a configuration file ie, /var/lib/kube-proxy/config.conf and we can override the hostname with the node name of at which the pod is running.

 

In the config file we define the clusterCIDR, kubeproxy mode, ipvs, iptables, bindaddress, kube-config etc.

 

Troubleshooting issues related to kube-proxy
1. Check kube-proxy pod in the kube-system namespace is running.

2. Check kube-proxy logs.

3. Check configmap is correctly defined and the config file for running kube-proxy binary is correct.

4. kube-config is defined in the config map.

5. check kube-proxy is running inside the container

# netstat -plan | grep kube-proxy
tcp        0      0 0.0.0.0:30081           0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 172.17.0.12:33706       172.17.0.12:6443        ESTABLISHED 1/kube-proxy
tcp6       0      0 :::10256                :::*                    LISTEN      1/kube-proxy
References:

Debug Service issues:

https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

DNS Troubleshooting:

https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
```

- Note: If logs says specific plugin is not found, then check the plugin is installed or not
- Example weave net plugin
```bash
kubectl get pods -n kube-system | grep -i weave
```
- Always use kubernetes.io and follow instructions https://kubernetes.io/search/?q=weave%20net%20installation

- Network plugin location
```bash
ls /opt/cni/bin
```

- kube-proxy configuration
```bash
kubectl get ds -n kube-system
kubectl get cm -n kube-system
kubectl describe cm kube-proxy -n kube-system
kubectl edit ds/kube-proxy -n kube-system
--config=/var/lib/kube-proxy/config.conf
```

### Upgrade Kubernetes Cluster

- Search in kubernetes.io for upgrade
- https://v1-27.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
```search
kubernetes upgrade
kubeadm upgrade
```

- Step 1 Control Plane Node
```bash
kubectl drain controlplane --ignore-daemonsets
```

- Step 2 Control Plane Upgrade with specific version
```bash
# replace x in 1.27.x-* with the latest patch version example 1.27.0-00 must use same version mentioned in the question
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm='1.27.0-00' && \
apt-mark hold kubeadm
```

- Step 3 Upgrade the control plane
```bash
kubeadm version
```

- Step 4 Upgrade the control plane
```bash
kubeadm upgrade plan
```

- Step 5 Upgrade the control plane see from commands shown on console
```bash
kubeadm upgrade apply v1.27.0
```

- Step 6 Upgrade the control plane 1.27.0-00
```bash
# replace x in 1.27.x-* with the latest patch version
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet='1.27.0-00' kubectl='1.27.0-00' && \
apt-mark hold kubelet kubectl
```

- Step 7 Upgrade the control plane
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

- Step 8 Control Plane Node Uncordon
```bash
kubectl uncordon controlplane
```

### Upgrade Worker Node

- Step 1 Drain the worker node
```bash
kubectl drain node01 --ignore-daemonsets
```

- Step 2 Upgrade the worker node by doing ssh node01
```bash
# replace x in 1.27.x-* with the latest patch version
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet='1.27.0-00' kubectl='1.27.0-00' && \
apt-mark hold kubelet kubectl
```

- Step 3 Upgrade the worker node
```bash
kubeadm upgrade node
```

- Step 4 Upgrade the worker node
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

- Step 5 Uncordon the worker node
```bash
kubectl uncordon node01
```

- Note: Always check the kubernetes.io for upgrade

- Jsonpath
```bash
Print the names of all deployments in the admin2406 namespace in the following format:

DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE

<deployment name>   <container image used>   <ready replica count>   <Namespace>
. The data should be sorted by the increasing order of the deployment name.


Example:

DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy0   nginx:alpine   1   admin2406
Write the result to the file /opt/admin2406_data.
```

- Solution 1 with jq
```bash
kubectl get deployments -n admin2406 -o json | jq -r '.items | sort_by(.metadata.name) | .[] | "\(.metadata.name)\t\(.spec.template.spec.containers[0].image)\t\(.status.readyReplicas)\t\(.metadata.namespace)"' | (echo -e "DEPLOYMENT\tCONTAINER_IMAGE\tREADY_REPLICAS\tNAMESPACE" && cat) > /opt/admin2406_data
```

- Solution 2
```bash
kubectl get deployments -n admin2406 -o jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.spec.template.spec.containers[0].image}{'\t'}{.status.readyReplicas}{'\t'}{.metadata.namespace}{'\n'}{end}" | sort | awk 'BEGIN {print "DEPLOYMENT\tCONTAINER_IMAGE\tREADY_REPLICAS\tNAMESPACE"} {print $0}' > /opt/admin2406_data
```

### Create Users in kubernetes

- https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
- Scroll down the document till end. you will all commands

```explain
To add a user like John to a Kubernetes cluster and grant them specific permissions, you don't directly add users to the cluster in the way traditional systems manage user accounts. Instead, Kubernetes uses certificates and role-based access control (RBAC) to manage access.

The user's identity is represented by a certificate issued by a trusted Certificate Authority (CA) that the Kubernetes API server recognizes. You manage what users can do by defining roles and role bindings within the cluster.

Here's a high-level overview of the process to add a user and grant them access
```

- Step-1: Generate Key and Certificate Signing Request (CSR) for the User
- https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
```bash
openssl genrsa -out john.key 2048 # Even these commands are in document do copy paste
```

- Step-2: openssl req -new -key john.key -out john.csr -subj "/CN=john/O=development"

```bash
openssl req -new -key john.key -out john.csr -subj "/CN=john/O=development"
```

- Step-3: Submit and Approve the CSR in Kubernetes

- Tip: create a bash script ---> do copy paste

```bash
#!/bin/bash
CSR=`cat myuser.csr | base64 | tr -d "\n"` # This command is alo in document do copy paste https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-csr
spec:
  request: "$CSR"
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF
```

- Step-4: Approve the CSR (Approve the CertificateSigningRequest)
```bash
kubectl get csr
```

- Step-5: Approve the CSR (Approve the CertificateSigningRequest)
```bash
kubectl certificate approve john-csr
```

- Step-6: Create Role and RoleBinding
```bash
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
```

- Question:
```bash
Create a nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod 
```

- Solution
```bash
k run busybox --image=busybox:1.28 --command sleep 1000000000000000
k exec -it pod/busybox -- nslookup nginx-resolver-service.default.svc.cluster.local
k exec -it pod/busybox -- nslookup nginx-resolver-service.default.svc.cluster.local > /root/CKA/nginx.svc
k exec -it pod/busybox -- nslookup 10-244-192-1.default.pod.cluster.local > /root/CKA/nginx.pod
kubectl -n default run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup 10-244-192-1.default.pod.cluster.local > /root/CKA/nginx.pod
k delete pod busybox -n default --force
```

- Learning tip
- nodeName is the name of the node where the pod is running
- volumeName is the name of the volume that is used to store the data, you can use this in pvc to get the chunk of volume
- static pod is a pod that is managed directly by the kubelet on a node, not by the control plane
- static example, if question to launch a pod on node01, pod.yaml will be created on node01 at /etc/kubernetes/manifests
- Note: PVCs by default expect a volumeMode of Filesystem unless explicitly specified otherwise

- Create pv and pvc
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
    capacity:
        storage: 1Gi
    volumeMode: Filesystem
    accessModes:
        - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: manual
    hostPath:
        path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-vol1
spec:
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
        storage: 1Gi
    storageClassName: manual
 ```

Note: In-case of Block storage, Note: volumeMode: Block must be defined in both PV and PVC
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /pv-1
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
--- 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: "" # Empty string must be explicitly set otherwise default StorageClass will be set
  volumeName: pv-1
  volumeMode: Block
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/data"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc
```

- Note: A pod definition file is created at /root/CKA/use-pv.yaml. Make use of this manifest file
- Note: as per instructions in exam, always use the file provided in the question, create a backup of that file, because that file has labels and names which are used in the question, at the end result will look for those values, if not found, then it will not be marked as correct.

- Run Pod for debugging
```bash
k run busybox --image=busybox:latest -n default -- sleep infinity
```

### VIM for quick indentation fix <---> use Visual mode
```bash
shift + > (select the lines)
shift + < to move left
```

### In-case of ClusterRole and ClusterRoleBinding always use kubectl create clusterrole --help and kubectl create clusterrolebinding --help
```bash
kubectl create clusterrole --help
kubectl create clusterrolebinding --help
```

- If question ask to bind the service account to the clusterrole, then use below command
```bash
kubectl create clusterrolebinding <name> --clusterrole=<role> --serviceaccount=<namespace>:<serviceaccount>
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: pod-reader
  namespace: default
```

- Note: Subjects can be User, Group, ServiceAccount for example in-case of User
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
    kind: ClusterRole
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
```

- Note: In-case of Group
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods
subjects:
- kind: Group
  name: dev-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
    kind: ClusterRole
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
```

- Note: In-case of ServiceAccount
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: pod-reader
  namespace: default
roleRef:
    kind: ClusterRole
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
```

- Note: In-case of RoleBinding, use kubectl create rolebinding --help
```bash
kubectl create rolebinding --help
```

- Note: In-case of RoleBinding, if subject is ServiceAccount, then use below command
```bash
kubectl create rolebinding <name> --role=<role> --serviceaccount=<namespace>:<serviceaccount>
```

- Note: In-case of RoleBinding, if subject is User, then use below command
```bash
kubectl create rolebinding <name> --role=<role> --user=<username>
```

- Note: In-case of RoleBinding, if subject is Group, then use below command
```bash
kubectl create rolebinding <name> --role=<role> --group=<groupname>
```

- Note: to replace item from a file, use linux command sed
```bash
sed -i 's/old-text/new-text/g' file.txt
```

### JQ how can I list all paths | jq -c paths | grep and grep ignore (example we ignore condition)
```bash
jq -c paths
kubectl get nodes -o json | jq -c paths | grep -i type | grep -iv condition
```

- Type on ---> kubernetes.io ---> cheatsheet
- https://kubernetes.io/docs/reference/kubectl/quick-reference/
- Tip | jq -c paths | grep -i type | grep -iv condition (Note: this will give all the paths of json file)
- kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'
- kubectl get nodes -o json | jq '.items[].status.addresses[] | select(.type == "InternalIP") | {type, address}'

### Label with kuebctl
```bash
kubectl label nodes node01 disktype=ssd
kubectl run nginx --image=nginx --restart=Never --labels="env=prod,app=nginx"
kubectl label pod mypod app=v1
```

### Instal etcd and etdcctl
```info
https://etcd.io/
```

- From above url https://etcd.io/ ---> install ---> click releases ---> https://github.com/etcd-io/etcd/releases/

- Linux bash script to install etcd
```bash
ETCD_VER=v3.5.12

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version
/tmp/etcd-download-test/etcdutl version
```

- Double check the version
```bash
/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version
/tmp/etcd-download-test/etcdutl version
```

### Network Example
- Network Policy to deny all ingress traffic to the pods in the test namespace
```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: test
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: test
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

- Network Policy to Allow Ingress Traffic from a Specific Pod
```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: test
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: test
spec:
  podSelector: {}
  policyTypes:
  - Egress
---


### Sidecar Container Example 2 containers in a pod shares a log directory
```question
A pod called elastic-app-cka02-arch is running in the default namespace. The YAML file for this pod is available at /root/elastic-app-cka02-arch.yaml on the student-node. The single application container in this pod writes logs to the file /var/log/elastic-app.log.


One of our logging mechanisms needs to read these logs to send them to an upstream logging server but we don't want to increase the read overhead for our main application container so recreate this POD with an additional sidecar container that will run along with the application container and print to the STDOUT by running the command tail -f /var/log/elastic-app.log. You can use busybox image for this sidecar container.
```

- Solution (Note: Volume mount is used to share the logs)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: elastic-app-cka02-arch
spec:
  containers:
  - name: elastic-app
    image: busybox:1.28 # Assuming this is your main app's correct image
    args:
    - /bin/sh
    - -c
    - >
      if [ ! -d /var/log ]; then
        mkdir /var/log;
      fi;
      i=0;
      while true;
      do
        echo "$(date) INFO $i" >> /var/log/elastic-app.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: sidecar
    image: busybox
    args:
    - /bin/sh
    - -c
    - tail -f /var/log/elastic-app.log
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

- Note: Cka Bash script commands use this way as mentioned below
```bash
#!/bin/bash
kubectl --context=kind-cloudgeeks get pods
```

### configmap as files mounting, use nginx.conf as example (subPath: nginx.conf  # Assume your ConfigMap has a file named nginx.conf)
- kubectl create configmap --help
- kubectl create configmap nginx-config --from-file=nginx.conf

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: shared-files
        - mountPath: /etc/nginx/nginx.conf
          name: nginx-config
          subPath: nginx.conf  # Correctly assumes your ConfigMap has a file named nginx.conf
  volumes:
    - name: shared-files
      emptyDir: {}
    - name: nginx-config
      configMap:
        name: nginx-config
 ```

### ServiceAccount issue with less permissions quick solution
```bash
k get clusterrolebinding -o yaml | grep -iC5 thor-cka24-trb
```
- Note: You will see the below format which will show clusterrole associated with the service account
```yaml
apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: asim
  subjects:
  - kind: ServiceAccount
    name: thor-cka24-trb
    namespace: default
- apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
 ```
- Note: to fix simply edit the clusterrole mentioned which is asim
- Note: kubectl edit clusterrole asim

### node affinity

- Note: In case of node affinity use labels ---> mean check labels on nodes
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

- Note: Utilizing node affinity requires labeling nodes accordingly. This means checking for specific labels on nodes to ensure pods are scheduled on nodes that meet the criteria defined by the affinity rules.
- Definition: Node affinity allows you to specify rules that limit which nodes a pod can be scheduled on, based on labels on nodes. This feature enables the enforcement of constraints that must be met for a pod to be placed on a node, making it possible to ensure that pods are hosted on intended nodes.
- Conceptual Comparison: While node affinity is conceptually similar to nodeSelector, it provides a more expressive syntax that allows defining more complex rules and conditions for node selection.
- Mechanics: It is a property of Pods that attracts them to a set of nodes, either as a soft preference or a hard requirement

##### Types of Node Affinity:
- requiredDuringSchedulingIgnoredDuringExecution: The scheduler must satisfy the rule for the pod to be scheduled on a node. However, if the labels on a node change after pod placement, the pod will not be moved.
- preferredDuringSchedulingIgnoredDuringExecution: The scheduler tries to find a node that satisfies the rule, but the pod can be scheduled on a node that does not satisfy these conditions if necessary.
- requiredDuringSchedulingRequiredDuringExecution: This type ensures that not only must the scheduling constraints be met, but they must continue to be met for the pod to remain on the node. If the node no longer satisfies the pod's requirements (due to a change in node labels, for example), the pod will be evicted.
### Pod Affinity

- Pod Affinity: This concept allows you to specify rules that co-locate pods within the same node or topology domain (e.g., same node, same rack) based on labels. It's used to ensure that a set of pods are scheduled together on the same domain.
- Complexity and Use: Similar to node affinity but targets pod placement relative to other pods rather than node labels. Allows for specifying complex inter-pod relationships and dependencies.
- Pod Anti-Affinity: Contrary to pod affinity, pod anti-affinity ensures that pods are not placed on nodes where certain conditions are met or certain pods are running. This is useful for spreading pods of a given application across different nodes or domains to ensure high availability and failure tolerance.

- Conclusion: Both node and pod affinity/anti-affinity features are crucial for optimizing pod placement according to specific policies, ensuring high availability, and facilitating more efficient resource utilization across a Kubernetes cluster.


### Tip After restore backup always restart kubelet on controlplane
```bash
systemctl daemon-reload
systemctl restart kubelet
systemctl status kubelet
```

### Reason in troubleshooting
```bash
kubectl describe pod/green-deployment-cka15-trb-5fd5d7b6b9-vrzp2 | grep -iC2 reason
```

### kubernetes Secrets & ConfigMap Rule if application read secrets from environment variabes
- (Simple Approach)
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webapp-color-wl10
  name: webapp-color-wl10
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp-color-wl10
  template:
    metadata:
      labels:
        app: webapp-color-wl10
    spec:
      containers:
      - image: kodekloud/webapp-color
        name: webapp-color-wl10
        envFrom:
        - configMapRef: 
            name: webapp-wl10-config-map
```

- Secrets Example use as environment variables

- Complex approach (In this approach, you can use specific key from secret)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secret
type: Opaque
data:
  DATABASE_PASSWORD: base64-encoded-password
  API_KEY: base64-encoded-api-key
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-color-wl10
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp-color-wl10
  template:
    metadata:
      labels:
        app: webapp-color-wl10
    spec:
      containers:
      - name: webapp-color-wl10
        image: kodekloud/webapp-color
        env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: webapp-secret
              key: DATABASE_PASSWORD
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: webapp-secret
              key: API_KEY
```

- Note: (Simple Approach) In below format all key:vales will be exported as environment variables on the container
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-color-wl10
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp-color-wl10
  template:
    metadata:
      labels:
        app: webapp-color-wl10
    spec:
      containers:
      - name: webapp-color-wl10
        image: kodekloud/webapp-color
        envFrom:
        - secretRef:
            name: webapp-secret
```

## Kubernetes Installation

- Note: Always use kubernetes.io for installation

- Step 1: Container Runtime (Docker, Containerd, CRI-O) we will use Containerd
- Step 2: Install kubeadm, kubelet, kubectl
- Step 3: kubeadm init
- Step 4: kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml


### Containerd Installation
- https://github.com/containerd/containerd/blob/main/docs/getting-started.md
- https://github.com/containerd/containerd/releases
- https://github.com/containerd/containerd/releases/tag/v1.7.13

- Pre-requisite

- Forwarding IPv4 and letting iptables see bridged traffic
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system


lsmod | grep br_netfilter
lsmod | grep overlay


sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

- Cgroups we will use systemd
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/
```bash
ps -p 1 
```

```bash
curl -# -LO https://github.com/containerd/containerd/releases/download/v1.7.13/containerd-1.7.13-linux-amd64.tar.gz
tar -xzvf containerd-1.7.13-linux-amd64.tar.gz
sudo cp bin/* /usr/local/bin/
mkdir -p /etc/containerd
```

- containerd config
- https://v1-27.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
```conf
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]

  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

```bash
cat <<EOF > /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]

  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
EOF
```

- Create systemd service link is given in same document
- https://github.com/containerd/containerd/blob/main/docs/getting-started.md
- https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
```bash
cat <<EOF > /etc/systemd/system/containerd.service
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
EOF
```

- Forwarding IPv4 and letting iptables see bridged traffic
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system


lsmod | grep br_netfilter
lsmod | grep overlay


sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

- Note: Containerd can be installed via apt 
```bash
apt update -y
apt install -y containerd 
mkdir -p /etc/containerd

cat <<EOF > /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]

  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
EOF

systemctl restart containerd
systemctl enable containerd
systemctl status containerd
```

```bash
systemctl daemon-reload
systemctl restart containerd
systemctl enable containerd
```

- check the status
```bash
systemctl status containerd
```

### kubeadm kubectl kubelet installation

- https://kodekloud.com/topic/demo-deployment-with-kubeadm/

- we are looking to install 1.27.0-00
- https://v1-27.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

- Note: Old repo is deprecated, use new package repo
- https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
```bash
sudo apt-get update
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo apt-get update
```

- kubeadm kubelet kubectl installation
```bash
sudo apt-get update
sudo apt update -y && sudo apt-get install -y kubelet=1.27.* kubeadm=1.27.* kubectl=1.27.*
sudo apt-mark hold kubelet kubeadm kubectl
```

- Swap must be off https://v1-27.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
```bash
sudo swapoff -a
```

### Note Package kubeadm is KubeAdmin is used to initialize the kubernetes cluster
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

- must do only on master node
- ip -4 a s
- eth0 interface address will be the address of apiserver
```bash
sudo kubeadm init --pod-network-cidr=20.20.0.0/16 --apiserver-advertise-address=10.0.1.101
```

- Note: Must save the output of kubeadm join command (Do not close the terminal)

- save this information
```info
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.1.101:6443 --token 6cqla8.v24zu1ifcos65nos \
        --discovery-token-ca-cert-hash sha256:7988042e333818ce7a14c2386c3bc86bee6549e04636ae14cd25e5a1eff0f00a
```

- click this link from info https://kubernetes.io/docs/concepts/cluster-administration/addons/
- Install Flannel (Note: We are using flannel as pod network)

- https://github.com/flannel-io/flannel#deploying-flannel-manually

- kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
- If you use custom podCIDR (not 10.244.0.0/16) you first need to download the above manifest and modify the network to match your one.
- Now check kubectl get nodes, you will see controlplane as master node READY

### SecretKey Ref just like configmap exports all keys and values as environment variables
- https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-single-secret
spec:
  containers:
  - name: envars-test-container
    image: nginx
    envFrom:
    - secretRef:
        name: mysecret
```

- configmap example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-single-configmap
spec:
    containers:
    - name: envars-test-container
        image: nginx
        envFrom:
        - configMapRef:
            name: myconfigmap
 ``` 
    
### kubelet config file location and logs 

```bash
journalctl -u kubelet --since 30m
cat /var/lib/kubelet/config.yaml
```

# CKA Exam tips for pods creation via bash script

- bash pod
```bash
#!/bin/bash

POD_NAME='dns-resolver'
NAMESPACE='default'
IMAGE="nginx"
LABELS="run="$POD_NAME""
ENV='environment=dev'

kubectl -n "$NAMESPACE" run "$POD_NAME" --image="$IMAGE" -l "$LABELS" --env "$ENV" -o yaml --dry-run=client > pod.yaml

#End
```

- bash deployment
```bash
#!/bin/bash

DEPLOYMENT_NAME='dns-resolver'
NAMESPACE='default'
IMAGE="nginx"
REPLICAS=1

kubectl -n "$NAMESPACE" create deployment "$DEPLOYMENT_NAME" --image="$IMAGE" --replicas="$REPLICAS" -o yaml --dry-run=client > deploy.yaml 
```

### Expose a deployment as service, type can be NodePort, LoadBalancer, ClusterIP, default is ClusterIP
```bash
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort --name=nginx-service
```

### Linux find directory paths of a package
```bash
dpkg -L kubelet 
rpm -qc kubelet
```
- Example above command on debian base system dpkg -L kubelet will give all the paths of kubelet
- dpkg -L kubelet | find / -type f | grep -i "config.yaml"

### Add columns in a file with awk
```bash
kubectl get pods -o wide -n kube-system --no-headers | awk '{print $1,$8}' > pod.txt
```


### Troubleshooting tips and tricks
- if service is exposed see the PROTOCOL as well, is it TCP or UDP
- if dns resolution is not working, check the dns pod and service example core-dns ---> k get deploy -n kube-system

- Service Expose
```explaination
When you expose a deployment as a service in Kubernetes without specifying a targetPort, the behavior can vary based on the specifics of how you're exposing the service and the characteristics of your deployment.

By default, if you do not specify a targetPort when creating a Service, Kubernetes will use the same port value for targetPort as the one you specify for the port field of the Service. The port field specifies the port on which the Service is exposed internally within the cluster. The targetPort is the port on the Pod where the application is running and to which the Service will forward the traffic. 
```

- to check the ingress use below command
```bash
kubectl get ingressclasses
```

- PVC in read-only mode
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: mycontainer
      image: myimage
      volumeMounts:
        - name: myvolume
          mountPath: "/path/to/mount"
          readOnly: true
  volumes:
    - name: myvolume
      persistentVolumeClaim:
        claimName: mypvc
```

- Ingress Rule Host
- https://kubernetes.io/docs/concepts/services-networking/ingress/
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```

- Kubernetes Networking Important Policy Example with -

- Question
```question
There are existing Pods in Namespace space1 and space2 .

We need a new NetworkPolicy named np that restricts all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2 . Incoming traffic not affected.

We also need a new NetworkPolicy named np that restricts all Pods in Namespace space2 to only have incoming traffic from Pods in Namespace space1 . Outgoing traffic not affected.

The NetworkPolicies should still allow outgoing DNS traffic on port 53 TCP and UDP. 
```

- Solution 1 will not work because of -
- "-" here mean it is under ports
- Note: Only Use for communication between namespaces as this will allow ports, use k descibe netpol policyname (do not use, if you want to allow --->podselector (AND) Rule
```yaml
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
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: space2
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

- Solution 1 Correct
- - "-" here mean it is under to and dns is living in kube-system which is coredns so 53 port is allowed
```yaml
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
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: space2
   
    - ports:
      - protocol: TCP
        port: 53
      - protocol: UDP
        port: 53
```

- Solution 2
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space2
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
   - from:
     - namespaceSelector:
        matchLabels:
         kubernetes.io/metadata.name: space1
```
- Important incase of dns not wokring remove the ports from ingress example below
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

- example remove ports section from ingress which is 6379

  - Crictl is CLI for containerd
- etcd.io
- Tip: alias docker=criclt
- docker ps -a
- docker logs container-id

### Two containers listing on same port in pod will not work, make sure to use different ports

- example below pod will be crashed
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: myapp
spec:
  containers:
  - name: web-container
    image: my-web-app:latest
    ports:
    - containerPort: 8080
  - name: api-container
    image: my-api-service:latest
    ports:
    - containerPort: 8080
```

- Solution: you can adjust the listing port of application to make sure it runs, on a different port.

### RBAC Service Accounts verification
```bash
k auth can-i get pods --as system:serviceaccount:ns1:pipeline
k get pod --as system:serviceaccount:ns1:pipeline
```

- Question
```question
There is existing Namespace applications .

User smoke should be allowed to create and delete Pods, Deployments and StatefulSets in Namespace applications.
User smoke should have view permissions (like the permissions of the default ClusterRole named view ) in all Namespaces but not in kube-system .
Verify everything using kubectl auth can-i .
```
- Answer
```rbac
Because of this there are 4 different RBAC combinations and 3 valid ones:

Role + RoleBinding (available in single Namespace, applied in single Namespace)
ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)
ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)
Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)
```

- As of now it’s not possible to create deny-RBAC in K8s

- Solution attach ---> rolebinding ---> with clusterrole
- Create rolebinding in all namespaces except kube-system and attach with clusterrole


### PV Persistent Volume must create on specific Node

- Question
```question
A storage class called coconut-stc-cka01-str was created earlier.


Use this storage class to create a persistent volume called coconut-pv-cka01-str as per below requirements:


- Capacity should be 100Mi.

- The volume type should be hostpath and the path should be /opt/coconut-stc-cka01-str.

- Use coconut-stc-cka01-str storage class.

- This volume must be created on cluster1-node01 (the /opt/coconut-stc-cka01-str directory already exists on this node).

- It must have a label with key: storage-tier with value: gold.


Also create a persistent volume claim with the name coconut-pvc-cka01-str as per below specs:


- Request 50Mi of storage from coconut-pv-cka01-str PV, it must use matchLabels to use the PV.

- Use coconut-stc-cka01-str storage class.

- The access mode must be ReadWriteMany.

is below anwer is correct?
```

- Ans

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: coconut-pv-cka01-str
  labels: 
    storage-tier: gold
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /opt/coconut-stc-cka01-str
  storageClassName: coconut-stc-cka01-str
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - cluster1-node01
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: coconut-pvc-cka01-str
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
  storageClassName: coconut-stc-cka01-str
  selector: 
    matchLabels:
      storage-tier: gold
```

- Tip: search nodeaffinity in kubernetes.io https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/  this is removed "requiredDuringSchedulingIgnoredDuringExecution:"

### Ingress SSL
- Note: for annotation use quotes "" example below
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress-cka04-svcn
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service-cka04-svcn
            port:
              number: 80
```

### kubelet

- /etc/kubernetes/kubelet.conf
```detail
The /etc/kubernetes/kubelet.conf file is a configuration file for the kubelet, which is a key component of Kubernetes running on each node in the cluster. The kubelet works as an agent that communicates with the Kubernetes API server to manage the containers running on a node. Specifically, the kubelet takes a set of PodSpecs (which are YAML or JSON descriptions of pods) and ensures that the containers described in those PodSpecs are running and healthy. 
```
- /var/lib/kubelet/config.yaml

### Change default namespace
```bash
kubectl config -h
kubectl config set-context --current --namespace=<namespace-name>
```

### OpenSSL

- Server certificate expiration date
```bash
openssl help
openssl x509 -help
openssl x509  -noout -text -in /etc/kubernetes/pki/etcd/server.crt | grep Validity -A2
```

### Kubectl get all pods,deploy,svc in all namespaces
```bash
k -n project-hamster get pod,svc,deploy
```

- Question
```question
Create a Pod named check-ip in Namespace default using image httpd:2.4.41-alpine. Expose it on port 80 as a ClusterIP Service named check-ip-service. Remember/output the IP of that Service.

Change the Service CIDR to 11.96.0.0/12 for the cluster.

Then create a second Service named check-ip-service2 pointing to the same Pod to check if your settings did take effect. Finally check if the IP of the first Service has changed.
```

- Solution
- Recursively grep the service-cluster-ip-range in /etc/kubernetes/manifests
- grep -irn "service-cluster-ip-range" /etc/kubernetes/manifests
```bash
ssh controlplane
cd /etc/kubernetes/manifests
grep -irn "service-cluster-ip-range" .
vi kube-apiserver.yaml
# change the service-cluster-ip-range to
# --service-cluster-ip-range=11.96.0.0/12
vi /etc/kubernetes/manifests/kube-controller-manager.yaml
# change the service-cluster-ip-range to
# --service-cluster-ip-range=11.96.0.0/12
systemctl restart kubelet
```

- Note: If service IP of existing service is not changing, then you need to delete the service and recreate it.

### kubelet service check is it run as a systemd service
```bash
systemctl status kubelet
```

- quick check of kubelet configuration files
```bash
ps aux | grep -i kubelet
```

### Network Policy Describe will tell us how the policy will behave must see that
- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- read behavior of network policy
```read
Behavior of to and from selectors
There are four kinds of selectors that can be specified in an ingress from section or egress to section:

podSelector: This selects particular Pods in the same namespace as the NetworkPolicy which should be allowed as ingress sources or egress destinations.

namespaceSelector: This selects particular namespaces for which all Pods should be allowed as ingress sources or egress destinations.

namespaceSelector and podSelector: A single to/from entry that specifies both namespaceSelector and podSelector selects particular Pods within particular namespaces. Be careful to use correct YAML syntax. For example:
```
```bash
k describe networkpolicy default-deny-ingress -n cloudgeeks



Name:         dns-network-policy
Namespace:    cloudgeeks
Created on:   2024-03-10 13:02:41 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      NamespaceSelector: kubernetes.io/metadata.name=cloudgeeks
    From:
      PodSelector: <none>
    ----------
    To Port: 53/TCP
    To Port: 53/UDP
    From: <any> (traffic not restricted by source)
  Allowing egress traffic:
    To Port: <any> (traffic allowed to all ports)
    To:
      NamespaceSelector: kubernetes.io/metadata.name=cloudgeeks
    To:
      PodSelector: <none>
    ----------
    To Port: 53/TCP
    To Port: 53/UDP
    To: <any> (traffic not restricted by destination)
  Policy Types: Ingress, Egress


Name:         test-network-policy
Namespace:    cloudgeeks
Created on:   2024-03-10 12:23:55 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: 80/TCP
    To Port: 53/TCP
    To Port: 53/UDP
    From:
      NamespaceSelector: kubernetes.io/metadata.name=cloudgeeks
    From:
      PodSelector: <none>
  Allowing egress traffic:
    To Port: 80/TCP
    To Port: 53/TCP
    To Port: 53/UDP
    To:
      NamespaceSelector: kubernetes.io/metadata.name=cloudgeeks
    To:
      PodSelector: <none>
  Policy Types: Ingress, Egress
```

- nw policy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: cloudgeeks
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: cloudgeeks
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: cloudgeeks
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 80
```

- 2nd policy dns-resolver
- "-" here mean it is under to and dns is living in kube-system which is coredns so 53 port is allowed

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dns-network-policy
  namespace: cloudgeeks
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: cloudgeeks
    - podSelector: {}
  - ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: cloudgeeks
    - podSelector: {}
  - ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

- describe dns-network-policy in cloudgeeks namespace
```bash
k describe networkpolicy dns-network-policy -n cloudgeeks

Name:         dns-network-policy
Namespace:    cloudgeeks
Created on:   2024-03-10 13:02:41 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      NamespaceSelector: kubernetes.io/metadata.name=cloudgeeks
    From:
      PodSelector: <none>
    ----------
    To Port: 53/TCP
    To Port: 53/UDP
    From: <any> (traffic not restricted by source)
  Allowing egress traffic:
    To Port: <any> (traffic allowed to all ports)
    To:
      NamespaceSelector: kubernetes.io/metadata.name=cloudgeeks
    To:
      PodSelector: <none>
    ----------
    To Port: 53/TCP
    To Port: 53/UDP
    To: <any> (traffic not restricted by destination)
  Policy Types: Ingress, Egress
```

- Note: Effect of above policies are dns resolution is allowed from all namespaces but connectivity to port is only allowed from cloudgeeks namespace

### Network Policy
- netpol
- k get netpol -n cloudgeeks

- Solution only allowed ingress traffic from cloudgeeks namespace
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: cloudgeeks
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: cloudgeeks
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 80
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dns-network-policy
  namespace: cloudgeeks
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: cloudgeeks
    - podSelector: {}
  - ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: cloudgeeks
    - podSelector: {}
  - ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

- kubernetes network policy rules are stateful, meaning that if you allow traffic in one direction, it is not automatically allowed in the other direction. You must explicitly allow the return traffic.
```explaination
Kubernetes NetworkPolicy rules are stateful in terms of connection tracking but not in the way traditional stateful firewalls work. When you define a NetworkPolicy, Kubernetes uses this definition to determine whether to allow or block traffic based on the labels and conditions specified. However, the enforcement of these policies and the statefulness aspect depend on the network plugin (CNI) that implements the NetworkPolicy specification.

Most network plugins that support NetworkPolicy use iptables or similar mechanisms under the hood, which track the state of a connection (NEW, ESTABLISHED, RELATED, etc.). This means that if an ingress or egress rule allows traffic, the returning traffic (in the case of TCP, the SYN-ACK packets following the initial SYN packet) is automatically allowed based on the connection tracking, even if there isn't a specific rule allowing the returning traffic. This behavior is akin to how stateful firewalls work.

For example, in the NetworkPolicy you've provided, if a pod within the cloudgeeks namespace establishes a connection to another pod that matches the podSelector on TCP port 80, the return traffic from that pod back to the originating pod is automatically allowed by the network policy enforcement mechanism, thanks to connection tracking. This stateful behavior is crucial for the practical functioning of network communications within Kubernetes, ensuring that once a connection is established by following the defined rules, the packets belonging to this connection are allowed to flow freely in both directions.

It's important to note, though, that the exact implementation details can vary between different CNI plugins, but the general principle of leveraging connection tracking for enforcing NetworkPolicy rules remains consistent across implementations that support such policies.
```

### Network Policy Rule Understanding

- Network Policy with rule to allow a specific pod from a default namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: cloudgeeks
spec:
  podSelector:
    matchLabels:
      run: cloudgeeks
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
      podSelector:
        matchLabels:
          run: white
    ports:
    - protocol: TCP
      port: 80
```

- Note: In above policy "podSelector" is under Single Rule to understand describe the policy
- k describe netpol/test-network-policy -n cloudgeeks
```describe
Name:         test-network-policy
Namespace:    cloudgeeks
Created on:   2024-03-12 05:26:13 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     run=cloudgeeks
  Allowing ingress traffic:
    To Port: 80/TCP
    From:
      NamespaceSelector: kubernetes.io/metadata.name=default
      PodSelector: run=white
  Not affecting egress traffic
  Policy Types: Ingress
controlplane $ cat chatgpt.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: cloudgeeks
spec:
  podSelector:
    matchLabels:
      run: cloudgeeks
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
      podSelector:
        matchLabels:
          run: white
    ports:
    - protocol: TCP
      port: 80
```


- Q which network plugin is installed
```bash
k get pods -n kube-system
k get cm -n kube-system
k describe cm kube-proxy -n kube-system
```

### Network Policies Deny specific port is not possible
```explain
NetworkPolicy objects define port ranges. Kubernetes NetworkPolicy API does not support the direct specification of port ranges in the format "1-9089" or "9091-65535". Instead, the port field in a NetworkPolicy rule expects a single integer value or a named port as a string. This limitation means you can't specify a range of ports directly within a single rule; you can only specify individual ports explicitly.

Given your requirement to allow traffic on all ports except 9090, both for egress and ingress, from the default namespace, you can't achieve this by specifying port ranges directly in the policy due to Kubernetes API limitations. Instead, you would typically allow traffic broadly and then restrict or manage access to the specific port (9090) through other controls or by not running services on that port.
```

- Important directories
- CNI stands for Container Network Interface
```bash
ls /etc/cni/net.d/
ps -ef | grep kubelet
ls /opt/cni/bin/
```

### Create a temporary pod and remove it after use
```bash
k run output-pod --image=busybox:latest --rm -it --restart=Never -- echo "Congratulations! you have passed the CKA Exam" > output.txt
```

### Port Name
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:1.0
        ports:
        - containerPort: 80
          name: http
```

```explain
To create a ClusterIP service using an imperative kubectl command and reference a port by name, you would typically specify the port and target port numbers directly in the command. However, when wanting to reference the port by its name as defined in the deployment, the imperative approach is limited because kubectl expose does not allow you to specify the target port by name directly in the command.

That being said, you can still use an imperative command to create a service and then manually edit it to use the port name, or you can specify the port number and target port number, assuming you know them. Here's how you can do it by specifying the port and target port numbers:

kubectl expose deployment my-app-deployment --type=ClusterIP --name=my-app-service --port=80 --target-port=80

kubectl edit service my-app-service

In the editor that opens, change the targetPort from the numeric value to the name you've used in your Deployment (e.g., http).
```

### Storage Preparations

Note: when you define a PersistentVolume (PV), the volumeType is indeed a must.
```explain

In Kubernetes, when you define a PersistentVolume (PV), the volumeType is indeed a must because it specifies the underlying storage system being used. Each volumeType has its own set of parameters that define the characteristics and access mode of the storage. Common volume types include hostPath, nfs, iscsi, persistentVolumeClaim (for dynamic provisioning), and others.

However, emptyDir is not used as a volumeType for PersistentVolumes. Instead, emptyDir volumes are used directly in Pods, under the volumes section of a Pod specification. The emptyDir volume type is designed to provide a temporary space that is erased when a Pod is removed or restarted. It is typically used for temporary storage of data that a Pod needs to share with other containers in the same Pod.
```

- If question asked for label the pv it means pvc must use matchLabels
```question
For this question, please set this context (In exam, diff cluster name)

kubectl config use-context kubernetes-admin@kubernetes


Create a PersistentVolume (PV) and a PersistentVolumeClaim (PVC) using an existing storage class named gold-stc-cka to meet the following requirements:

Step 1: Create a Persistent Volume (PV)

Name the PV as gold-pv-cka .
Set the capacity to 50Mi .
Use the volume type hostpath with the path /opt/gold-stc-cka .
Assign the storage class as gold-stc-cka .
Ensure that the PV is created on node01 , where the /opt/gold-stc-cka directory already exists.
Apply a label to the PV with key tier and value white .
Step 2: Create a Persistent Volume Claim (PVC)

Name the PVC as gold-pvc-cka .
Request 30Mi of storage from the PV gold-pv-cka using the matchLabels criterion.
Use the gold-stc-cka storage class.
Set the access mode to ReadWriteMany .
```

- Solution
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gold-pv-cka
  labels:
    tier: white
spec:
  nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - node01
  capacity:
    storage: 50Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
 
  storageClassName: gold-stc-cka
  hostPath:
    path: /opt/gold-stc-cka
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gold-pvc-cka
spec:
  selector:
    matchLabels:
      tier: "white"
  volumeName: gold-pv-cka
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 30Mi
  storageClassName: gold-stc-cka
```

### Shared Volume for logging, shared directories
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-busybox-sidecar
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}

  containers:
    - name: nginx
      image: nginx:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx

    - name: busybox
      image: busybox:latest
      args:
        - /bin/sh
        - -c
        - 'while true; do sleep 30; echo "$(date): Processing Nginx logs"; ls /var/log/nginx; done'
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
```

### Shared Volume for logging, read only
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-busybox-specific-log
spec:
  volumes:
    - name: shared-log
      emptyDir: {}

  containers:
    - name: nginx
      image: nginx:latest
      volumeMounts:
        - name: shared-log
          mountPath: /var/log/nginx
          # No subPath used here as we're mounting the whole log directory

    - name: busybox-log-processor
      image: busybox:latest
      volumeMounts:
        - name: shared-log
          mountPath: /var/log/nginx/access.log
          subPath: access.log  # Mounting only the specific access.log file
      args:
        - /bin/sh
        - -c
        - >
          while true; do
            sleep 30;
            echo "$(date): Processing specific Nginx log file";
            cat /var/log/nginx/access.log;
          done
```

- Use While loop with sleep to keep the container running
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: busybox
    name: sidecar
    command: ["/bin/sh","-c","while true; do sleep 3; cat /var/log/nginx/access.log; done"]
    volumeMounts:
    - mountPath: /var/log/nginx
      readOnly: true
      name: shared-volume
      
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /var/log/nginx
      name: shared-volume
      
  volumes:
  - name: shared-volume
    emptyDir: {}
```

- Note: When a PVC is bound to a PV, the CAPACITY shown by kubectl get pvc command reflects the capacity of the bound PV, not the requested amount in the PVC specification.

### PATH must be set for example for kubernetes backup edctl must be in /usr/local/bin ---> which edcdctl 
- Note: if you take a backup by this path /usr/local/edctl it will be considered incorrect

### Volume Expansion

```question
For this question, please set this context (In exam, diff cluster name)

kubectl config use-context kubernetes-admin@kubernetes


Modify the size of the existing Persistent Volume Claim (PVC) named yellow-pvc-cka to request 60Mi of storage from the yellow-pv-cka volume. Ensure that the PVC successfully resizes to the new size and remains in the Bound state.
```

- Solution

- Important kubectl commands
```bash
kubectl get sc,pv,pvc

k get sc,pv,pvc
NAME                                               PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/local-path (default)   rancher.io/local-path          Delete          WaitForFirstConsumer   false                  13d
storageclass.storage.k8s.io/yellow-stc-cka         kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   true                   4m2s

NAME                             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS     VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/yellow-pv-cka   100Mi      RWO            Retain           Bound    default/yellow-pvc-cka   yellow-stc-cka   <unset>                          4m2s

NAME                                   STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS     VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/yellow-pvc-cka   Bound    yellow-pv-cka   100Mi      RWO            yellow-stc-cka   <unset>                 4m2s
```
- Note: Volume storage expansion must be set to ALLOWVOLUMEEXPANSION by default it is false

- other important commands

```bash
k describe persistentvolume/yellow-pv-cka
k describe persistentvolumeclaim/yellow-pvc-cka
```

- Note: If you use command
```bash
k get pv,pvc
```

- Notice: you will notice the pvc number shows the same as pv number (storage).
- Note: to see actual storage requested by pvc use below command
```bash
k get persistentvolumeclaim/yellow-pvc-cka -o yaml | grep -iC5 request
```

- To expand simply do
```bash
k edit persistentvolumeclaim/yellow-pvc-cka

k get persistentvolumeclaim/yellow-pvc-cka -o yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"yellow-pvc-cka","namespace":"default"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"40Mi"}},"storageClassName":"yellow-stc-cka","volumeName":"yellow-pv-cka"}}
    pv.kubernetes.io/bind-completed: "yes"
  creationTimestamp: "2024-03-17T00:58:30Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: yellow-pvc-cka
  namespace: default
  resourceVersion: "8151"
  uid: 0e48de8a-4eed-4048-9b1d-5bb3cadeded0
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 60Mi
  storageClassName: yellow-stc-cka
  volumeMode: Filesystem
  volumeName: yellow-pv-cka
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 100Mi
  phase: Bound
```

### Storage path local
```question
Your task involves setting up storage components in a Kubernetes cluster. Follow these steps:

Step 1: Create a Storage Class named blue-stc-cka with the following properties:

Provisioner: kubernetes.io/no-provisioner
Volume binding mode: WaitForFirstConsumer
Step 2: Create a Persistent Volume (PV) named blue-pv-cka with the following properties:

Capacity: 100Mi
Access mode: ReadWriteOnce
Reclaim policy: Retain
Storage class: blue-stc-cka
Local path: /opt/blue-data-cka
Node affinity: Set node affinity to create this PV on controlplane .
Step 3: Create a Persistent Volume Claim (PVC) named blue-pvc-cka with the following properties:

Access mode: ReadWriteOnce
Storage class: blue-stc-cka
Storage request: 50Mi
The volume should be bound to blue-pv-cka
```

- Solution
```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: blue-stc-cka
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true

volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: blue-pv-cka
spec:
  nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - controlplane
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "blue-stc-cka"
  local:
    path: /opt/blue-data-cka
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: blue-pvc-cka
spec:
  volumeName: blue-pv-cka
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 50Mi
  storageClassName: blue-stc-cka
```

### Network Policy to Deny a specific port 
- policy check https://editor.networkpolicy.io/
- Q Allow all traffic from dev namespace except port 9090
- kubectl create ns dev;kubectl create ns app
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-except-9090-from-dev
  namespace: app
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: dev
      ports:
        - protocol: TCP
          port: 1
          endPort: 9089
        - protocol: TCP
          port: 9091
          endPort: 65535
        - protocol: UDP
          port: 1
          endPort: 9089
        - protocol: UDP
          port: 9091
          endPort: 65535
```

- Sample test deploy in app namespace
- app-9090-deployment
```yaml
# app-deployment-for-9090.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment-9090
  namespace: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-app
  template:
    metadata:
      labels:
        app: app-app
    spec:
      containers:
      - name: netcat
        image: subfuzion/netcat
        args:
          - "-lk"
          - "9090"

---
# app-service-on-9090.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service-9090
  namespace: app
spec:
  ports:
  - port: 9090
    targetPort: 9090
  selector:
    app: app-app
```

- app app-deployment-80.yaml
```yaml
# app-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: app
---
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-app
  template:
    metadata:
      labels:
        app: app-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
# app-service-on-9090.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service-80
  namespace: app
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: app-app
```

- Sample test deploy in dev namespace
```yaml
# dev-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
# dev-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dev-deployment
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dev-app
  template:
    metadata:
      labels:
        app: dev-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
# dev-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: dev-service
  namespace: dev
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: dev-app
```

- test
```bash
kubectl -n dev exec -it dev-deployment-557c477957-6rqgc -- curl app-deployment-80.app.svc.cluster.local:80
kubectl -n dev exec -it dev-deployment-557c477957-6rqgc -- curl app-service-9090.app.svc.cluster.local:9090
kubectl -n dev exec -it dev-deployment-557c477957-6rqgc -- sh
curl -v telnet://app-deployment-80.app.svc.cluster.local:80
curl -v telnet://app-service-9090.app.svc.cluster.local:9090
```

### Ingress

- Question
```question
Test the ingress on internal ip
```

- Solution
```answer
It means you should use the node ip to test the ingress
ssh controlplane
curl node-ip/path

Check the NodePort Service for the Nginx Ingress Controller to see the ports

k get pods -A

k -n ingress-nginx get svc ingress-nginx-controller

k -n ingress-nginx get svc ingress-nginx-controller
NAME                       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   NodePort   10.103.25.170   <none>        80:30080/TCP,443:30443/TCP   11m


curl 172.30.1.2:30080/asia
hello, you reached ASIA

curl 172.30.1.2:30080/europe
hello, you reached EUROPE
```

### Kubeconfig not found on controlplane node
```explain
The kubeconfig file is not found through the env command because it is not set as an environment variable in your session. By default, Kubernetes uses the kubeconfig file located at $HOME/.kube/config for a regular user or /etc/kubernetes/admin.conf on the master/control plane node for administrative tasks.

export KUBECONFIG=/etc/kubernetes/admin.conf

kubectl get nodes
kubectl --kubeconfig=/etc/kubernetes/admin.conf cluster-info
```

### tail keep the pod alive example busybox alpine
```bash
command: ["/bin/sh","-c","tail -f /dev/null"]
```

- Other option to keep the pod alive
```bash
command: ["/bin/sh","-c","while true; do sleep 30; echo 'Hello'; done"]
```
```bash
command: ["/bin/sh","-c","while true; do sleep 1;cat /var/log/access.log; done"]
```

### mountPath and SubPath
- Correct
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine-pod-pod
  name: alpine-pod-pod
spec:
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: config-volume
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: log-configmap
  containers:
  - image: alpine:latest
    name: alpine-pod-pod
    command: ["/bin/sh","-c","tail -f /config/log.txt"] # Note: if mentioned args than replace command with args
    volumeMounts:
      - name: config-volume
        mountPath: "/config/log.txt"
        subPath: log.txt
        readOnly: false
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

- Correct Use below
- For logs subPath is not required but nginx.conf is required
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine-pod-pod
  name: alpine-pod-pod
spec:
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: config-volume
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: log-configmap
  containers:
  - image: alpine:latest
    name: alpine-pod-pod
    command: ["/bin/sh","-c","tail -f /config/log.txt"]
    volumeMounts:
      - name: config-volume
        mountPath: "/config"
        readOnly: true
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

### Upgrade Cluster to a new version

- Main Commands
```bash
# see current versions
kubectl get node
kubectl version
kubeadm upgrade -h
apt-cache show kubeadm

apt-cache show kubeadm will be show the available versions

apt-cache show kubeadm | grep -i version
Version: 1.29.3-1.1
Version: 1.29.2-1.1
Version: 1.29.1-1.1
Version: 1.29.0-1.1

kubeadm upgrade plan

kubeadm upgrade apply v1.29.1
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Mounting & SSH

- Daemonset Example
- Foucus just like docker hostmount and container mount

```explain
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: configurator
  namespace: configurator
  labels:
    k8s-app: configurator
spec:
  selector:
    matchLabels:
      name: configurator
  template:
    metadata:
      labels:
        name: configurator
    spec:
      containers:
      - name: configurator
        image: bash
        command:
          - sh
          - -c
          - 'echo aba997ac-1c89-4d64 > /mount/config && sleep 1d'
        volumeMounts:
        - name: vol
          mountPath: /mount
      volumes:
      - name: vol
        hostPath:
          path: /configurator
```

- SSH stay on the same node and run command on other nodes

```ssh
ssh controlplane -- cat /configurator/config | grep aba997ac-1c89-4d64
ssh node01 -- cat /configurator/config | grep aba997ac-1c89-4d64
```

### --sort-by

- Q See the highest cpu and memory consumed by pods

```bash
k top pods -A --sort-by=cpu
k top pods -A --sort-by=memory
kubectl get pods -A -o wide --sort-by=.status.podIP
```

- Q See the highest cpu and memory consumed by pods with label name=overloaded-cpu
```bash
k top pods -A -l name=overloaded-cpu --sort-by=cpu
k top pods -A -l name=overloaded-cpu --sort-by=memory
```

### kubelet service
- Note: Incase of failure just do not restart but must enable it
```bash
systemctl restart kubelet
systemctl enable kubelet
systemctl enable --now kubelet

--now: This is a convenient option that allows you to combine enabling a service and starting it immediately in one command. Without --now, you would have to run systemctl start kubelet separately after enabling it.
```

### Ingress Path
```explain
pathType: Exact
When you use pathType: Exact, the path must match the exact URL path requested by the client. This means the path will only route requests that match the entire path component exactly as specified. For example, if your Ingress rule specifies path: /hi with pathType: Exact, it will only route requests that are made to http://yourdomain.com/hi and not to http://yourdomain.com/hi/there or http://yourdomain.com/hi?query=string.

pathType: Prefix
Conversely, pathType: Prefix is more flexible and matches the initial part of the URL path. It effectively routes any requests whose paths start with the specified prefix. If your Ingress rule specifies path: /hi with pathType: Prefix, it will route requests not only to http://yourdomain.com/hi but also to http://yourdomain.com/hi/there, http://yourdomain.com/hi/, and any other path that begins with /hi.
```

### Label Namespace
```bash
kubectl label namespace cloudgeeks env=dev
```

### Node Selector
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disk: ssd
```

### Log shared volume

```explain
If you're interested in viewing output from the big-corp-app container via kubectl logs, you would need it to generate console output. Since your current setup is designed to write to a file, consider adding logging to both the file (for the sidecar to follow) and standard output

Docker and Kubernetes, when you run a command or an application within a container that prints output, the default behavior is to send this output to /dev/stdout and /dev/stderr. These are special file descriptors that essentially represent the standard output (stdout) and standard error (stderr) streams of the container. Kubernetes kubectl logs command displays the logs that are sent to these streams.

So, if your application or command within the container writes messages directly to the console (i.e., it prints to stdout or stderr), you'll be able to see these messages using kubectl logs. This behavior mirrors how Unix-like systems work, where many commands and programs write their output and errors to these standard streams, and tools like shell redirection and pipes can be used to manage this output.
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: big-corp-app
  name: big-corp-app
spec:
  volumes:
  - name: logger
    emptyDir: {}
  containers:
  - image: busybox:1.28
    name: big-corp-app
    command: ["/bin/sh","-c","while true;do sleep 10 && date >> /var/log/big-corp-app.log;done"]
    volumeMounts:
    - name: logger
      mountPath: /var/log
  - image: busybox
    name: sidecar
    command: ["/bin/sh","-c","while [ ! -f /var/log/big-corp-app.log ];do sleep 3;done;tail -n+1 -f /var/log/big-corp-app.log"]
    volumeMounts:
    - name: logger
      mountPath: /var/log
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### Endpoints link your kubernetes service with internal or external IP
- https://kubernetes.io/docs/reference/kubernetes-api/service-resources/endpoints-v1/
- Routing to Internal Pods Without a Selector
- Internal Ip
```yaml
apiVersion: v1
kind: Service
metadata:
  name: manual-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Endpoints
metadata:
  name: manual-service
subsets:
  - addresses:
      - ip: 10.1.2.3
    ports:
      - port: 9376
```

- Exposing External Services
- External IP
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - protocol: TCP
    port: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 80
```


### For loop Iteration over clusters context
```bash
for i in cluster1 cluster2 cluster3 cluster4;do k config use-context "$i"; k top nodes --sort-by=cpu;done
```

### Events help in troubleshoot

- namespace quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: myquota
  namespace: default
spec:
  hard:
    pods: "1"
    services: "5"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi 
```

- See Resource Quota in default namespace
```bash
k get resourcequota -n default
```

- Create a deployment in default namespace
```bash
k create deploy nginx --image=nginx --replicas=3 -n default
```

- troubleshoot
```bash
k get events -n default
k get events -n default | grep nginx
```

- Fix by editing the resource quota

### Show Labels
```bash
k get pods,svc --show-labels
```


### Sample Mock Exam Questions and Solutions

- Q1 Question 1 | Contexts
```question
You have access to multiple clusters from your main terminal through kubectl contexts. Write all those context names into /opt/course/1/contexts.

Next write a command to display the current context into /opt/course/1/context_default_kubectl.sh, the command should use kubectl.

Finally write a second command doing the same thing into /opt/course/1/context_default_no_kubectl.sh, but without the use of kubectl
```

```answer
1a. k config get-contexts # copy manually

1a. k config get-contexts -o name > /opt/course/1/contexts

1b. kubectl config current-context

1b. echo 'kubectl config current-context' /opt/course/1/context_default_kubectl.sh

1c. cat ~/.kube/config | grep current-context | awk '{print $2}'

1c. echo 'cat ~/.kube/config | grep current-context | awk "{print $2}"' > /opt/course/1/context_default_no_kubectl.sh
```

- Q2 Question 2 | Schedule Pod on Controlplane Nodes
```question
Use context: kubectl config use-context k8s-c1-H

 

Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container. This Pod should only be scheduled on controlplane nodes. Do not add new labels to any nodes
```
- Solution
```answer
k get node # find controlplane node

k describe node cluster1-controlplane1 | grep Taint -A1 # get controlplane node taints

k get node cluster1-controlplane1 --show-labels # get controlplane node labels
```

```yaml
# 3.yaml
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: pod1
name: pod1
spec:
nodeName: cluster1-controlplane1  
containers:
- image: httpd:2.4.41-alpine
  name: pod1-container                       # change
  resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:                                 # add
- effect: NoSchedule                         # add
  key: node-role.kubernetes.io/control-plane # add
#  nodeSelector:                                # add or nodeName will work
#    node-role.kubernetes.io/control-plane: ""  # add
  status: {}
```

- Q3 Question 3 | Scale down StatefulSet
```question
Use context: kubectl config use-context k8s-c1-H

 

There are two Pods named o3db-* in Namespace project-c13. C13 management asked you to scale the Pods down to one replica to save resources.
```

- Solution
```answer
➜ k -n project-c13 get pod | grep o3db
o3db-0                                  1/1     Running   0          52s
o3db-1                                  1/1     Running   0          42s

If we check the Pods we see two replicas:

➜ k -n project-c13 get pod | grep o3db
o3db-0                                  1/1     Running   0          52s
o3db-1                                  1/1     Running   0          42s
From their name it looks like these are managed by a StatefulSet. But if we're not sure we could also check for the most common resources which manage Pods:

➜ k -n project-c13 get deploy,ds,sts | grep o3db
statefulset.apps/o3db   2/2     2m56s
Confirmed, we have to work with a StatefulSet. To find this out we could also look at the Pod labels:

➜ k -n project-c13 get pod --show-labels | grep o3db
o3db-0                                  1/1     Running   0          3m29s   app=nginx,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-0
o3db-1                                  1/1     Running   0          3m19s   app=nginx,controller-revision-hash=o3db-5fbd4bb9cc,statefulset.kubernetes.io/pod-name=o3db-1
To fulfil the task we simply run:

➜ k -n project-c13 scale sts o3db --replicas 1
statefulset.apps/o3db scaled

or k -n project-c13 edit sts/o3db and change replicas to 1


➜ k -n project-c13 get sts o3db
NAME   READY   AGE
o3db   1/1     4m39s
```

- Q4 Question 4 | Pod Ready if Service is reachable
```question
Use context: kubectl config use-context k8s-c1-H

 

Do the following in Namespace default. Create a single Pod named ready-if-service-ready of image nginx:1.16.1-alpine. Configure a LivenessProbe which simply executes command true. Also configure a ReadinessProbe which does check if the url http://service-am-i-ready:80 is reachable, you can use wget -T2 -O- http://service-am-i-ready:80 for this. Start the Pod and confirm it isn't ready because of the ReadinessProbe.

Create a second Pod named am-i-ready of image nginx:1.16.1-alpine with label id: cross-server-ready. The already existing Service service-am-i-ready should now have that second Pod as endpoint.

Now the first Pod should be in ready state, confirm that.
```

- Solution
```answer
It's a bit of an anti-pattern for one Pod to check another Pod for being ready using probes, hence the normally available readinessProbe.httpGet doesn't work for absolute remote urls. Still the workaround requested in this task should show how probes and Pod<->Service communication works.

First we create the first Pod:

k run ready-if-service-ready --image=nginx:1.16.1-alpine $do > 4_pod1.yaml

vim 4_pod1.yaml
Next perform the necessary additions manually:

# 4_pod1.yaml
```

- livenessProbe readinessProbe
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ready-if-service-ready
  name: ready-if-service-ready
spec:
  containers:
  - image: nginx:1.16.1-alpine
    name: ready-if-service-ready
    resources: {}
    livenessProbe:                                      # add from here
      exec:
        command:
        - 'true'
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - 'wget -T2 -O- http://service-am-i-ready:80'   # to here
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- kubectl apply -f 4_pod1.yaml
- For other pod use
```bash
k run am-i-ready --image=nginx:1.16.1-alpine --labels="id=cross-server-ready"

k run am-i-ready --image=nginx:1.16.1-alpine -l id=cross-server-ready 
```

- Q5 Question 5 | Create a Pod with a specific label
```question
Question 5 | Kubectl sorting
 

Use context: kubectl config use-context k8s-c1-H

 

There are various Pods in all namespaces. Write a command into /opt/course/5/find_pods.sh which lists all Pods sorted by their AGE (metadata.creationTimestamp).

Write a second command into /opt/course/5/find_pods_uid.sh which lists all Pods sorted by field metadata.uid. Use kubectl sorting for both commands.
```

- Solution
```answer
echo 'kubectl get pod -A --sort-by=.metadata.creationTimestamp' >> /opt/course/5/find_pods.sh
echo 'kubectl get pod -A --sort-by=.metadata.uid' >> /opt/course/5/find_pods_uid.sh
```

- Q6 Question 6 | Storage, PV, PVC, Pod volume
- Note: pvc is namespace specific
```question
Use context: kubectl config use-context k8s-c1-H

 

Create a new PersistentVolume named safari-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

Next create a new PersistentVolumeClaim in Namespace project-tiger named safari-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment safari in Namespace project-
```

- Solution
```answer
kind: PersistentVolume
apiVersion: v1
metadata:
 name: safari-pv
spec:
 capacity:
  storage: 2Gi
 accessModes:
  - ReadWriteOnce
 hostPath:
  path: "/Volumes/Data"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: safari-pvc
  namespace: project-tiger
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
     storage: 2Gi
---
```

- bash
```bash
➜ k -n project-tiger get pv,pvc
NAME                         CAPACITY  ... STATUS   CLAIM                    ...
persistentvolume/safari-pv   2Gi       ... Bound    project-tiger/safari-pvc ...

NAME                               STATUS   VOLUME      CAPACITY ...
persistentvolumeclaim/safari-pvc   Bound    safari-pv   2Gi      ... 
```

- yaml
```yaml
# 6_dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: safari
  name: safari
  namespace: project-tiger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: safari
    spec:
      volumes:                                      # add
      - name: data                                  # add
        persistentVolumeClaim:                      # add
          claimName: safari-pvc                     # add
      containers:
      - image: httpd:2.4.41-alpine
        name: container
        volumeMounts:                               # add
        - name: data                                # add
          mountPath: /tmp/safari-data               # add
```

- Q Question 7 | Node and Pod Resource Usage
```question
Use context: kubectl config use-context k8s-c1-H

 

The metrics-server has been installed in the cluster. Your college would like to know the kubectl commands to:

show Nodes resource usage

show Pods and their containers resource usage

Please write the commands into /opt/course/7/node.sh and /opt/course/7/pod.sh.
```

- Solution
```answer
➜ k top -h
Display Resource (CPU/Memory/Storage) usage.

 The top command allows you to see the resource consumption for nodes or pods.

 This command requires Metrics Server to be correctly configured and working on the server.

Available Commands:
  node        Display Resource (CPU/Memory/Storage) usage of nodes
  pod         Display Resource (CPU/Memory/Storage) usage of pods


➜ k top node
NAME               CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
cluster1-controlplane1   178m         8%     1091Mi          57%       
cluster1-node1   66m          6%     834Mi           44%       
cluster1-node2   91m          9%     791Mi           41% 

echo 'kubectl top node' >> /opt/course/7/node.sh


➜ k top pod -A --containers=true

echo 'kubectl top pod -A --containers=true' >> /opt/course/7/pod.sh
```

- Q Question 8 | Get Controlplane Information
```question
Use context: kubectl config use-context k8s-c1-H

 

Ssh into the controlplane node with ssh cluster1-controlplane1. Check how the controlplane components kubelet, kube-apiserver, kube-scheduler, kube-controller-manager and etcd are started/installed on the controlplane node. Also find out the name of the DNS application and how it's started/installed on the controlplane node.

Write your findings into file /opt/course/8/controlplane-components.txt. The file should be structured like:

# /opt/course/8/controlplane-components.txt
kubelet: [TYPE]
kube-apiserver: [TYPE]
kube-scheduler: [TYPE]
kube-controller-manager: [TYPE]
etcd: [TYPE]
dns: [TYPE] [NAME]
```

- Solution
- Note: techinicallly kubelet is a systemd service but also all pod are run as a process
```answer
# /opt/course/8/controlplane-components.txt
kubelet: process
kube-apiserver: static-pod
kube-scheduler: static-pod
kube-controller-manager: static-pod
etcd: static-pod
dns: pod coredns
```

- Q Question 9 | Kill Scheduler, Manual Scheduling
```question
Use context: kubectl config use-context k8s-c2-AC

 

Ssh into the controlplane node with ssh cluster2-controlplane1. Temporarily stop the kube-scheduler, this means in a way that you can start it again afterwards.

Create a single Pod named manual-schedule of image httpd:2.4-alpine, confirm it's created but not scheduled on any node.

Now you're the scheduler and have all its power, manually schedule that Pod on node cluster2-controlplane1. Make sure it's running.

Start the kube-scheduler again and confirm it's running correctly by creating a second Pod named manual-schedule2 of image httpd:2.4-alpine and check if it's running on cluster2-node1.
```

- Solution
```answer
Stop the Scheduler
First we find the controlplane node:

➜ k get node
NAME                     STATUS   ROLES           AGE   VERSION
cluster2-controlplane1   Ready    control-plane   26h   v1.29.0
cluster2-node1           Ready    <none>          26h   v1.29.0
Then we connect and check if the scheduler is running:

➜ ssh cluster2-controlplane1

➜ root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule
kube-scheduler-cluster2-controlplane1            1/1     Running   0          6s
Kill the Scheduler (temporarily):

➜ root@cluster2-controlplane1:~# cd /etc/kubernetes/manifests/

➜ root@cluster2-controlplane1:~# mv kube-scheduler.yaml ..
And it should be stopped:

➜ root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule

➜ root@cluster2-controlplane1:~# 
 

Create a Pod
Now we create the Pod:

k run manual-schedule --image=httpd:2.4-alpine
And confirm it has no node assigned:

➜ k get pod manual-schedule -o wide
NAME              READY   STATUS    ...   NODE     NOMINATED NODE
manual-schedule   0/1     Pending   ...   <none>   <none>        
 

Manually schedule the Pod
Let's play the scheduler now:

k get pod manual-schedule -o yaml > 9.yaml
# 9.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-09-04T15:51:02Z"
  labels:
    run: manual-schedule
  managedFields:
...
    manager: kubectl-run
    operation: Update
    time: "2020-09-04T15:51:02Z"
  name: manual-schedule
  namespace: default
  resourceVersion: "3515"
  selfLink: /api/v1/namespaces/default/pods/manual-schedule
  uid: 8e9d2532-4779-4e63-b5af-feb82c74a935
spec:
  nodeName: cluster2-controlplane1        # add the controlplane node name
  containers:
  - image: httpd:2.4-alpine
    imagePullPolicy: IfNotPresent
    name: manual-schedule
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-nxnc7
      readOnly: true
  dnsPolicy: ClusterFirst
...

The only thing a scheduler does, is that it sets the nodeName for a Pod declaration. How it finds the correct node to schedule on, that's a very much complicated matter and takes many variables into account.

As we cannot kubectl apply or kubectl edit , in this case we need to delete and create or replace:

k -f 9.yaml replace --force
How does it look?

➜ k get pod manual-schedule -o wide
NAME              READY   STATUS    ...   NODE            
manual-schedule   1/1     Running   ...   cluster2-controlplane1
It looks like our Pod is running on the controlplane now as requested, although no tolerations were specified. Only the scheduler takes tains/tolerations/affinity into account when finding the correct node name. That's why it's still possible to assign Pods manually directly to a controlplane node and skip the scheduler.

 

Start the scheduler again
➜ ssh cluster2-controlplane1

➜ root@cluster2-controlplane1:~# cd /etc/kubernetes/manifests/

➜ root@cluster2-controlplane1:~# mv ../kube-scheduler.yaml .
Checks it's running:

➜ root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule
kube-scheduler-cluster2-controlplane1            1/1     Running   0          16s
Schedule a second test Pod:

k run manual-schedule2 --image=httpd:2.4-alpine
➜ k get pod -o wide | grep schedule
manual-schedule    1/1     Running   ...   cluster2-controlplane1
manual-schedule2   1/1     Running   ...   cluster2-node1
Back to normal.
```

- Q Question 10 | RBAC ServiceAccount Role RoleBinding
```question
Use context: kubectl config use-context k8s-c1-H

 

Create a new ServiceAccount processor in Namespace project-hamster. Create a Role and RoleBinding, both named processor as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

 

Answer:
Let's talk a little about RBAC resources
A ClusterRole|Role defines a set of permissions and where it is available, in the whole cluster or just a single Namespace.

A ClusterRoleBinding|RoleBinding connects a set of permissions with an account and defines where it is applied, in the whole cluster or just a single Namespace.

Because of this there are 4 different RBAC combinations and 3 valid ones:

Role + RoleBinding (available in single Namespace, applied in single Namespace)

ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)

ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)

Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)
```

- Solution
```answer
We first create the ServiceAccount:

➜ k -n project-hamster create sa processor
serviceaccount/processor created
Then for the Role:

k -n project-hamster create role -h # examples
So we execute:

k -n project-hamster create role processor \
  --verb=create \
  --resource=secret \
  --resource=configmap
Which will create a Role like:

# kubectl -n project-hamster create role processor --verb=create --resource=secret --resource=configmap
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: processor
  namespace: project-hamster
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  - configmaps
  verbs:
  - create
Now we bind the Role to the ServiceAccount:

k -n project-hamster create rolebinding -h # examples
So we create it:

k -n project-hamster create rolebinding processor \
  --role processor \
  --serviceaccount project-hamster:processor
This will create a RoleBinding like:

# kubectl -n project-hamster create rolebinding processor --role processor --serviceaccount project-hamster:processor
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: processor
  namespace: project-hamster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: processor
subjects:
- kind: ServiceAccount
  name: processor
  namespace: project-hamster
To test our RBAC setup we can use kubectl auth can-i:

k auth can-i -h # examples
Like this:

➜ k -n project-hamster auth can-i create secret \
  --as system:serviceaccount:project-hamster:processor
yes

➜ k -n project-hamster auth can-i create configmap \
  --as system:serviceaccount:project-hamster:processor
yes

➜ k -n project-hamster auth can-i create pod \
  --as system:serviceaccount:project-hamster:processor
no

➜ k -n project-hamster auth can-i delete secret \
  --as system:serviceaccount:project-hamster:processor
no

➜ k -n project-hamster auth can-i get configmap \
  --as system:serviceaccount:project-hamster:processor
no
Done.
```

- Q Question 11 | DaemonSet on all Nodes
```question
Use context: kubectl config use-context k8s-c1-H

 

Use Namespace project-tiger for the following. Create a DaemonSet named ds-important with image httpd:2.4-alpine and labels id=ds-important and uuid=18426a0b-5f59-4e10-923f-c0e078e82462. The Pods it creates should request 10 millicore cpu and 10 mebibyte memory. The Pods of that DaemonSet should run on all nodes, also controlplanes.
```

- Solution
```answer
As of now we aren't able to create a DaemonSet directly using kubectl, so we create a Deployment and just change it up:

k -n project-tiger create deployment --image=httpd:2.4-alpine ds-important $do > 11.yaml

vim 11.yaml
(Sure you could also search for a DaemonSet example yaml in the Kubernetes docs and alter it.)

 

Then we adjust the yaml to:

# 11.yaml
apiVersion: apps/v1
kind: DaemonSet                                     # change from Deployment to Daemonset
metadata:
  creationTimestamp: null
  labels:                                           # add
    id: ds-important                                # add
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462      # add
  name: ds-important
  namespace: project-tiger                          # important
spec:
  #replicas: 1                                      # remove
  selector:
    matchLabels:
      id: ds-important                              # add
      uuid: 18426a0b-5f59-4e10-923f-c0e078e82462    # add
  #strategy: {}                                     # remove
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: ds-important                            # add
        uuid: 18426a0b-5f59-4e10-923f-c0e078e82462  # add
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: ds-important
        resources:
          requests:                                 # add
            cpu: 10m                                # add
            memory: 10Mi                            # add
      tolerations:                                  # add
      - effect: NoSchedule                          # add
        key: node-role.kubernetes.io/control-plane  # add
#status: {}                                         # remove
It was requested that the DaemonSet runs on all nodes, so we need to specify the toleration for this.

Let's confirm:

k -f 11.yaml create
➜ k -n project-tiger get ds
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-important   3         3         3       3            3           <none>          8s
➜ k -n project-tiger get pod -l id=ds-important -o wide
NAME                      READY   STATUS          NODE
ds-important-6pvgm        1/1     Running   ...   cluster1-node1
ds-important-lh5ts        1/1     Running   ...   cluster1-controlplane1
ds-important-qhjcq        1/1     Running   ...   cluster1-node2
```

- Q Question 12 | Deployment on all Nodes
```question
Question 12 | Deployment on all Nodes
 

Use context: kubectl config use-context k8s-c1-H

 

Use Namespace project-tiger for the following. Create a Deployment named deploy-important with label id=very-important (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image nginx:1.17.6-alpine and the second one named container2 with image google/pause.

There should be only ever one Pod of that Deployment running on one worker node. We have two worker nodes: cluster1-node1 and cluster1-node2. Because the Deployment has three replicas the result should be that on both nodes one Pod is running. The third Pod won't be scheduled, unless a new worker node will be added. Use topologyKey: kubernetes.io/hostname for this.

In a way we kind of simulate the behaviour of a DaemonSet here, but using a Deployment and a fixed number of replicas.
```

- Solution
- Note: Simply solution is pod topology spread is equal to (podantiaffinity) ---> search in kubernetes.io

```search
    topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: foo
        
```
```answer
Answer:
There are two possible ways, one using podAntiAffinity and one using topologySpreadConstraint.

 

PodAntiAffinity
The idea here is that we create a "Inter-pod anti-affinity" which allows us to say a Pod should only be scheduled on a node where another Pod of a specific label (here the same label) is not already running.

Let's begin by creating the Deployment template:

k -n project-tiger create deployment \
  --image=nginx:1.17.6-alpine deploy-important $do > 12.yaml

vim 12.yaml
Then change the yaml to:

# 12.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important                  # change
  name: deploy-important
  namespace: project-tiger              # important
spec:
  replicas: 3                           # change
  selector:
    matchLabels:
      id: very-important                # change
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important              # change
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: container1                # change
        resources: {}
      - image: google/pause             # add
        name: container2                # add
      affinity:                                             # add
        podAntiAffinity:                                    # add
          requiredDuringSchedulingIgnoredDuringExecution:   # add
          - labelSelector:                                  # add
              matchExpressions:                             # add
              - key: id                                     # add
                operator: In                                # add
                values:                                     # add
                - very-important                            # add
            topologyKey: kubernetes.io/hostname             # add
status: {}
Specify a topologyKey, which is a pre-populated Kubernetes label, you can find this by describing a node.

 

TopologySpreadConstraints
We can achieve the same with topologySpreadConstraints. Best to try out and play with both.

# 12.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important                  # change
  name: deploy-important
  namespace: project-tiger              # important
spec:
  replicas: 3                           # change
  selector:
    matchLabels:
      id: very-important                # change
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important              # change
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: container1                # change
        resources: {}
      - image: google/pause             # add
        name: container2                # add
      topologySpreadConstraints:                 # add
      - maxSkew: 1                               # add
        topologyKey: kubernetes.io/hostname      # add
        whenUnsatisfiable: DoNotSchedule         # add
        labelSelector:                           # add
          matchLabels:                           # add
            id: very-important                   # add
status: {}
 

Apply and Run
Let's run it:

k -f 12.yaml create
Then we check the Deployment status where it shows 2/3 ready count:

➜ k -n project-tiger get deploy -l id=very-important
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deploy-important   2/3     3            2           2m35s
And running the following we see one Pod on each worker node and one not scheduled.

➜ k -n project-tiger get pod -o wide -l id=very-important
NAME                                READY   STATUS    ...   NODE             
deploy-important-58db9db6fc-9ljpw   2/2     Running   ...   cluster1-node1
deploy-important-58db9db6fc-lnxdb   0/2     Pending   ...   <none>          
deploy-important-58db9db6fc-p2rz8   2/2     Running   ...   cluster1-node2
If we kubectl describe the Pod deploy-important-58db9db6fc-lnxdb it will show us the reason for not scheduling is our implemented podAntiAffinity ruling:

Warning  FailedScheduling  63s (x3 over 65s)  default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/control-plane: }, that the pod didn't tolerate, 2 node(s) didn't match pod affinity/anti-affinity, 2 node(s) didn't satisfy existing pods anti-affinity rules.
Or our topologySpreadConstraints:

Warning  FailedScheduling  16s   default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/control-plane: }, that the pod didn't tolerate, 2 node(s) didn't match pod topology spread constraints.
```

- Q Question 13 | Multi Containers and Pod shared Volume
```question
Use context: kubectl config use-context k8s-c1-H

 

Create a Pod named multi-container-playground in Namespace default with three containers, named c1, c2 and c3. There should be a volume attached to that Pod and mounted into every container, but the volume shouldn't be persisted or shared with other Pods.

Container c1 should be of image nginx:1.17.6-alpine and have the name of the node where its Pod is running available as environment variable MY_NODE_NAME.

Container c2 should be of image busybox:1.31.1 and write the output of the date command every second in the shared volume into file date.log. You can use while true; do date >> /your/vol/path/date.log; sleep 1; done for this.

Container c3 should be of image busybox:1.31.1 and constantly send the content of file date.log from the shared volume to stdout. You can use tail -f /your/vol/path/date.log for this.

Check the logs of container c3 to confirm correct setup.
```

- Solution
```answer
First we create the Pod template:

k run multi-container-playground --image=nginx:1.17.6-alpine $do > 13.yaml

vim 13.yaml
And add the other containers and the commands they should execute:

# 13.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-container-playground
  name: multi-container-playground
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: c1                                                                      # change
    resources: {}
    env:                                                                          # add
    - name: MY_NODE_NAME                                                          # add
      valueFrom:                                                                  # add
        fieldRef:                                                                 # add
          fieldPath: spec.nodeName                                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c2                                                                      # add
    command: ["sh", "-c", "while true; do date >> /vol/date.log; sleep 1; done"]  # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c3                                                                      # add
    command: ["sh", "-c", "tail -f /vol/date.log"]                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                                                        # add
    - name: vol                                                                   # add
      emptyDir: {}                                                                # add
status: {}
k -f 13.yaml create
Oh boy, lot's of requested things. We check if everything is good with the Pod:

➜ k get pod multi-container-playground
NAME                         READY   STATUS    RESTARTS   AGE
multi-container-playground   3/3     Running   0          95s
Good, then we check if container c1 has the requested node name as env variable:

➜ k exec multi-container-playground -c c1 -- env | grep MY
MY_NODE_NAME=cluster1-node2
And finally we check the logging:

➜ k logs multi-container-playground -c c3
Sat Dec  7 16:05:10 UTC 2077
Sat Dec  7 16:05:11 UTC 2077
Sat Dec  7 16:05:12 UTC 2077
Sat Dec  7 16:05:13 UTC 2077
Sat Dec  7 16:05:14 UTC 2077
Sat Dec  7 16:05:15 UTC 2077
Sat Dec  7 16:05:16 UTC 2077
```

- Q Question 14 | Find out Cluster Information
```question
Use context: kubectl config use-context k8s-c1-H

 

You're ask to find out following information about the cluster k8s-c1-H:

How many controlplane nodes are available?

How many worker nodes are available?

What is the Service CIDR?

Which Networking (or CNI Plugin) is configured and where is its config file?

Which suffix will static pods have that run on cluster1-node1?

Write your answers into file /opt/course/14/cluster-info, structured like this:

# /opt/course/14/cluster-info
1: [ANSWER]
2: [ANSWER]
3: [ANSWER]
4: [ANSWER]
5: [ANSWER]
```

- Solution
```answer
Answer:
How many controlplane and worker nodes are available?
➜ k get node
NAME                    STATUS   ROLES          AGE   VERSION
cluster1-controlplane1  Ready    control-plane  27h   v1.29.0
cluster1-node1          Ready    <none>         27h   v1.29.0
cluster1-node2          Ready    <none>         27h   v1.29.0
We see one controlplane and two workers.

 

What is the Service CIDR?
➜ ssh cluster1-controlplane1

➜ root@cluster1-controlplane1:~# cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep range
    - --service-cluster-ip-range=10.96.0.0/12
 

Which Networking (or CNI Plugin) is configured and where is its config file?
➜ root@cluster1-controlplane1:~# find /etc/cni/net.d/
/etc/cni/net.d/
/etc/cni/net.d/10-weave.conflist

➜ root@cluster1-controlplane1:~# cat /etc/cni/net.d/10-weave.conflist
{
    "cniVersion": "0.3.0",
    "name": "weave",
...
By default the kubelet looks into /etc/cni/net.d to discover the CNI plugins. This will be the same on every controlplane and worker nodes.

 

Which suffix will static pods have that run on cluster1-node1?
The suffix is the node hostname with a leading hyphen. It used to be -static in earlier Kubernetes versions.

 

Result
The resulting /opt/course/14/cluster-info could look like:

# /opt/course/14/cluster-info

# How many controlplane nodes are available?
1: 1

# How many worker nodes are available?
2: 2

# What is the Service CIDR?
3: 10.96.0.0/12

# Which Networking (or CNI Plugin) is configured and where is its config file?
4: Weave, /etc/cni/net.d/10-weave.conflist

# Which suffix will static pods have that run on cluster1-node1?
5: -cluster1-node1
```

- Q Question 15 | Cluster Event Logging
```question
Use context: kubectl config use-context k8s-c2-AC

 

Write a command into /opt/course/15/cluster_events.sh which shows the latest events in the whole cluster, ordered by time (metadata.creationTimestamp). Use kubectl for it.

Now delete the kube-proxy Pod running on node cluster2-node1 and write the events this caused into /opt/course/15/pod_kill.log.

Finally kill the containerd container of the kube-proxy Pod on node cluster2-node1 and write the events into /opt/course/15/container_kill.log.

Do you notice differences in the events both actions caused?
```

- Solution
```answer
# /opt/course/15/cluster_events.sh
kubectl get events -A --sort-by=.metadata.creationTimestamp
Now we delete the kube-proxy Pod:

k -n kube-system get pod -o wide | grep proxy # find pod running on cluster2-node1

k -n kube-system delete pod kube-proxy-z64cg
Now check the events:

sh /opt/course/15/cluster_events.sh
Write the events the killing caused into /opt/course/15/pod_kill.log:

# /opt/course/15/pod_kill.log
kube-system   9s          Normal    Killing           pod/kube-proxy-jsv7t   ...
kube-system   3s          Normal    SuccessfulCreate  daemonset/kube-proxy   ...
kube-system   <unknown>   Normal    Scheduled         pod/kube-proxy-m52sx   ...
default       2s          Normal    Starting          node/cluster2-node1  ...
kube-system   2s          Normal    Created           pod/kube-proxy-m52sx   ...
kube-system   2s          Normal    Pulled            pod/kube-proxy-m52sx   ...
kube-system   2s          Normal    Started           pod/kube-proxy-m52sx   ...
Finally we will try to provoke events by killing the container belonging to the container of the kube-proxy Pod:

➜ ssh cluster2-node1

➜ root@cluster2-node1:~# crictl ps | grep kube-proxy
1e020b43c4423   36c4ebbc9d979   About an hour ago   Running   kube-proxy     ...

➜ root@cluster2-node1:~# crictl rm 1e020b43c4423
1e020b43c4423

➜ root@cluster2-node1:~# crictl ps | grep kube-proxy
0ae4245707910   36c4ebbc9d979   17 seconds ago      Running   kube-proxy     ...     
We killed the main container (1e020b43c4423), but also noticed that a new container (0ae4245707910) was directly created. Thanks Kubernetes!

Now we see if this caused events again and we write those into the second file:

sh /opt/course/15/cluster_events.sh
# /opt/course/15/container_kill.log
kube-system   13s         Normal    Created      pod/kube-proxy-m52sx    ...
kube-system   13s         Normal    Pulled       pod/kube-proxy-m52sx    ...
kube-system   13s         Normal    Started      pod/kube-proxy-m52sx    ...
Comparing the events we see that when we deleted the whole Pod there were more things to be done, hence more events. For example was the DaemonSet in the game to re-create the missing Pod. Where when we manually killed the main container of the Pod, the Pod would still exist but only its container needed to be re-created, hence less events.
```

- Q Question 16 | Namespaces and Api Resources
```question
Use context: kubectl config use-context k8s-c1-H

 

Write the names of all namespaced Kubernetes resources (like Pod, Secret, ConfigMap...) into /opt/course/16/resources.txt.

Find the project-* Namespace with the highest number of Roles defined in it and write its name and amount of Roles into /opt/course/16/crowded-namespace.txt.
```

- Solution
```answer
Answer:
Namespace and Namespaces Resources
Now we can get a list of all resources like:

k api-resources    # shows all

k api-resources -h # help always good

k api-resources --namespaced -o name > /opt/course/16/resources.txt
Which results in the file:

# /opt/course/16/resources.txt
bindings
configmaps
endpoints
events
limitranges
persistentvolumeclaims
pods
podtemplates
replicationcontrollers
resourcequotas
secrets
serviceaccounts
services
controllerrevisions.apps
daemonsets.apps
deployments.apps
replicasets.apps
statefulsets.apps
localsubjectaccessreviews.authorization.k8s.io
horizontalpodautoscalers.autoscaling
cronjobs.batch
jobs.batch
leases.coordination.k8s.io
events.events.k8s.io
ingresses.extensions
ingresses.networking.k8s.io
networkpolicies.networking.k8s.io
poddisruptionbudgets.policy
rolebindings.rbac.authorization.k8s.io
roles.rbac.authorization.k8s.io
 

Namespace with most Roles
➜ k -n project-c13 get role --no-headers | wc -l
No resources found in project-c13 namespace.
0

➜ k -n project-c14 get role --no-headers | wc -l
300

➜ k -n project-hamster get role --no-headers | wc -l
No resources found in project-hamster namespace.
0

➜ k -n project-snake get role --no-headers | wc -l
No resources found in project-snake namespace.
0

➜ k -n project-tiger get role --no-headers | wc -l
No resources found in project-tiger namespace.
0
Finally we write the name and amount into the file:

# /opt/course/16/crowded-namespace.txt
project-c14 with 300 resources


# Quick option is to use for loop
k get ns
for i in project-c13 project-c14 project-hamster project-snake project-tiger; do k -n $i get role --no-headers | wc -l; done
```

- Q Question 17 | Find Container of Pod and check info
```question
Use context: kubectl config use-context k8s-c1-H

 

In Namespace project-tiger create a Pod named tigers-reunite of image httpd:2.4.41-alpine with labels pod=container and container=pod. Find out on which node the Pod is scheduled. Ssh into that node and find the containerd container belonging to that Pod.

Using command crictl:

Write the ID of the container and the info.runtimeType into /opt/course/17/pod-container.txt

Write the logs of the container into /opt/course/17/pod-container.log
```

- Solution
```answer
Answer:
First we create the Pod:

k -n project-tiger run tigers-reunite \
  --image=httpd:2.4.41-alpine \
  --labels "pod=container,container=pod"
Next we find out the node it's scheduled on:

k -n project-tiger get pod -o wide

# or fancy:
k -n project-tiger get pod tigers-reunite -o jsonpath="{.spec.nodeName}"
Then we ssh into that node and and check the container info:

➜ ssh cluster1-node2

➜ root@cluster1-node2:~# crictl ps | grep tigers-reunite
b01edbe6f89ed    54b0995a63052    5 seconds ago    Running        tigers-reunite ...

➜ root@cluster1-node2:~# crictl inspect b01edbe6f89ed | grep runtimeType
    "runtimeType": "io.containerd.runc.v2",
Then we fill the requested file (on the main terminal):

# /opt/course/17/pod-container.txt
b01edbe6f89ed io.containerd.runc.v2
Finally we write the container logs in the second file:

ssh cluster1-node2 'crictl logs b01edbe6f89ed' &> /opt/course/17/pod-container.log
The &> in above's command redirects both the standard output and standard error.

You could also simply run crictl logs on the node and copy the content manually, if it's not a lot. The file should look like:

# /opt/course/17/pod-container.log
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.44.0.37. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.44.0.37. Set the 'ServerName' directive globally to suppress this message
[Mon Sep 13 13:32:18.555280 2021] [mpm_event:notice] [pid 1:tid 139929534545224] AH00489: Apache/2.4.41 (Unix) configured -- resuming normal operations
[Mon Sep 13 13:32:18.555610 2021] [core:notice] [pid 1:tid 139929534545224] AH00094: Command line: 'httpd -D FOREGROUND'
```

- Q Question 18 | Fix Kubelet
```question
Use context: kubectl config use-context k8s-c3-CCC

 

There seems to be an issue with the kubelet not running on cluster3-node1. Fix it and confirm that cluster has node cluster3-node1 available in Ready state afterwards. You should be able to schedule a Pod on cluster3-node1 afterwards.

Write the reason of the issue into /opt/course/18/reason.txt
```

- Solution
```answer
Answer:
The procedure on tasks like these should be to check if the kubelet is running, if not start it, then check its logs and correct errors if there are some.

Always helpful to check if other clusters already have some of the components defined and running, so you can copy and use existing config files. Though in this case it might not need to be necessary.

Check node status:

➜ k get node
NAME                     STATUS     ROLES           AGE   VERSION
cluster3-controlplane1   Ready      control-plane   14d   v1.29.0
cluster3-node1           NotReady   <none>          14d   v1.29.0
First we check if the kubelet is running:

➜ ssh cluster3-node1

➜ root@cluster3-node1:~# ps aux | grep kubelet
root     29294  0.0  0.2  14856  1016 pts/0    S+   11:30   0:00 grep --color=auto kubelet
Nope, so we check if it's configured using systemd as service:

➜ root@cluster3-node1:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead) (Result: exit-code) since Thu 2024-01-04 13:12:54 UTC; 1h 23min ago
       Docs: https://kubernetes.io/docs/
    Process: 27577 ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=>
   Main PID: 27577 (code=exited, status=203/EXEC)

Jan 04 13:12:52 cluster3-node1 systemd[1]: kubelet.service: Main process exited, code=exited, status=203/EXEC
Jan 04 13:12:52 cluster3-node1 systemd[1]: kubelet.service: Failed with result 'exit-code'.
Jan 04 13:12:54 cluster3-node1 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
Yes, it's configured as a service with config at /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf, but we see it's inactive. Let's try to start it:

➜ root@cluster3-node1:~# service kubelet start

➜ root@cluster3-node1:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Thu 2024-01-04 14:37:02 UTC; 6s ago
       Docs: https://kubernetes.io/docs/
    Process: 27935 ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=>
   Main PID: 27935 (code=exited, status=203/EXEC)

Jan 04 14:37:02 cluster3-node1 systemd[1]: kubelet.service: Main process exited, code=exited, status=203/EXEC
Jan 04 14:37:02 cluster3-node1 systemd[1]: kubelet.service: Failed with result 'exit-code'.
We see it's trying to execute /usr/local/bin/kubelet with some parameters defined in its service config file. A good way to find errors and get more logs is to run the command manually (usually also with its parameters).

➜ root@cluster3-node1:~# /usr/local/bin/kubelet
-bash: /usr/local/bin/kubelet: No such file or directory

➜ root@cluster3-node1:~# whereis kubelet
kubelet: /usr/bin/kubelet
Another way would be to see the extended logging of a service like using journalctl -u kubelet.

Well, there we have it, wrong path specified. Correct the path in file /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf and run:

vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf # fix binary path

systemctl daemon-reload

service kubelet restart

service kubelet status  # should now show running
Also the node should be available for the api server, give it a bit of time though:

➜ k get node
NAME                     STATUS   ROLES           AGE   VERSION
cluster3-controlplane1   Ready    control-plane   14d   v1.29.0
cluster3-node1           Ready    <none>          14d   v1.29.0
Finally we write the reason into the file:

# /opt/course/18/reason.txt
wrong path to kubelet binary specified in service config
```

- Q Question 19 | Create Secret and mount into Pod
- Note: Secret ---> secretKeyRef and secretRef ---> search in kubernetes.io
- Note: Configmap ---> configmapKeyRef and configmapRef ---> search in kubernetes.io


```question
NOTE: This task can only be solved if questions 18 or 20 have been successfully implemented and the k8s-c3-CCC cluster has a functioning worker node

 

Use context: kubectl config use-context k8s-c3-CCC

 

Do the following in a new Namespace secret. Create a Pod named secret-pod of image busybox:1.31.1 which should keep running for some time.

There is an existing Secret located at /opt/course/19/secret1.yaml, create it in the Namespace secret and mount it readonly into the Pod at /tmp/secret1.

Create a new Secret in Namespace secret called secret2 which should contain user=user1 and pass=1234. These entries should be available inside the Pod's container as environment variables APP_USER and APP_PASS.

Confirm everything is working.
```

- Solution
```answer
First we create the Namespace and the requested Secrets in it:

k create ns secret

cp /opt/course/19/secret1.yaml 19_secret1.yaml

vim 19_secret1.yaml
We need to adjust the Namespace for that Secret:

# 19_secret1.yaml
apiVersion: v1
data:
  halt: IyEgL2Jpbi9zaAo...
kind: Secret
metadata:
  creationTimestamp: null
  name: secret1
  namespace: secret           # change
k -f 19_secret1.yaml create
Next we create the second Secret:

k -n secret create secret generic secret2 --from-literal=user=user1 --from-literal=pass=1234
Now we create the Pod template:

k -n secret run secret-pod --image=busybox:1.31.1 $do -- sh -c "sleep 5d" > 19.yaml

vim 19.yaml
Then make the necessary changes:

# 19.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
  namespace: secret                       # add
spec:
  containers:
  - args:
    - sh
    - -c
    - sleep 1d
    image: busybox:1.31.1
    name: secret-pod
    resources: {}
    env:                                  # add
    - name: APP_USER                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: user                       # add
    - name: APP_PASS                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: pass                       # add
    volumeMounts:                         # add
    - name: secret1                       # add
      mountPath: /tmp/secret1             # add
      readOnly: true                      # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                # add
  - name: secret1                         # add
    secret:                               # add
      secretName: secret1                 # add
status: {}
It might not be necessary in current K8s versions to specify the readOnly: true because it's the default setting anyways.

And execute:

k -f 19.yaml create
Finally we check if all is correct:

➜ k -n secret exec secret-pod -- env | grep APP
APP_PASS=1234
APP_USER=user1
➜ k -n secret exec secret-pod -- find /tmp/secret1
/tmp/secret1
/tmp/secret1/..data
/tmp/secret1/halt
/tmp/secret1/..2019_12_08_12_15_39.463036797
/tmp/secret1/..2019_12_08_12_15_39.463036797/halt
➜ k -n secret exec secret-pod -- cat /tmp/secret1/halt
#! /bin/sh
### BEGIN INIT INFO
# Provides:          halt
# Required-Start:
# Required-Stop:
# Default-Start:
# Default-Stop:      0
# Short-Description: Execute the halt command.
# Description:
...
All is good.
```

- Q Question 20 | Update Kubernetes Version and join cluster
```question
Use context: kubectl config use-context k8s-c3-CCC

 

Your coworker said node cluster3-node2 is running an older Kubernetes version and is not even part of the cluster. Update Kubernetes on that node to the exact version that's running on cluster3-controlplane1. Then add this node to the cluster. Use kubeadm for this.
```

- Note: Main command & Upgrade Cluster
```bash
kubeadm token create --print-join-command 

sudo apt update -y
sudo apt-cache madison kubeadm
sudo apt-cache show kubeadm

kubeadm upgrade node
systemctl deamon-reload
systemctl restart kubelet
```
- Solution
```answer
Upgrade Kubernetes to cluster3-controlplane1 version
Search in the docs for kubeadm upgrade: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade

➜ k get node
NAME                     STATUS   ROLES           AGE     VERSION
cluster3-controlplane1   Ready    control-plane   3h28m   v1.29.0
cluster3-node1           Ready    <none>          3h23m   v1.29.0
Controlplane node seems to be running Kubernetes 1.29.0. Node cluster3-node1 might not yet be part of the cluster depending on the completion of a previous task.

➜ ssh cluster3-node2

➜ root@cluster3-node2:~# kubectl version --short
Client Version: v1.28.5
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
The connection to the server localhost:8080 was refused - did you specify the right host or port?

➜ root@cluster3-node2:~# kubelet --version
Kubernetes v1.28.5

➜ root@cluster3-node2:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"29", GitVersion:"v1.29.0", GitCommit:"3f7a50f38688eb332e2a1b013678c6435d539ae6", GitTreeState:"clean", BuildDate:"2023-12-13T08:50:10Z", GoVersion:"go1.21.5", Compiler:"gc", Platform:"linux/amd64"}
Above we can see that kubeadm is already installed in the wanted version, so we don't need to install it. Hence we can run:

➜ root@cluster3-node2:~# kubeadm upgrade node
couldn't create a Kubernetes client from file "/etc/kubernetes/kubelet.conf": failed to load admin kubeconfig: open /etc/kubernetes/kubelet.conf: no such file or directory
To see the stack trace of this error execute with --v=5 or higher
This is usually the proper command to upgrade a node. But this error means that this node was never even initialised, so nothing to update here. This will be done later using kubeadm join. For now we can continue with kubelet and kubectl:

➜ root@cluster3-node2:~# apt update
Hit:1 http://ppa.launchpad.net/rmescandon/yq/ubuntu focal InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu focal InRelease                                                                                             
Hit:4 http://security.ubuntu.com/ubuntu focal-security InRelease    
Hit:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.28/deb  InRelease
Hit:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.29/deb  InRelease
Get:6 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Hit:7 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease
Get:8 http://us.archive.ubuntu.com/ubuntu focal-updates/main i386 Packages [919 kB]
Get:9 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [3,023 kB]
Get:10 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1,141 kB]
Get:11 http://us.archive.ubuntu.com/ubuntu focal-updates/universe i386 Packages [762 kB]
Fetched 5,959 kB in 3s (2,049 kB/s)                   
Reading package lists... Done
Building dependency tree       
Reading state information... Done
222 packages can be upgraded. Run 'apt list --upgradable' to see them.

➜ root@cluster3-node2:~# apt show kubectl -a | grep 1.29
Version: 1.29.0-1.1
APT-Sources: https://pkgs.k8s.io/core:/stable:/v1.29/deb  Packages

➜ root@cluster3-node2:~# apt install kubectl=1.29.0-1.1 kubelet=1.29.0-1.1
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 220 not upgraded.
Need to get 30.3 MB of archives.
After this operation, 782 kB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.29/deb  kubectl 1.29.0-1.1 [10.5 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.29/deb  kubelet 1.29.0-1.1 [19.8 MB]
Fetched 30.3 MB in 1s (40.8 MB/s)
(Reading database ... 112588 files and directories currently installed.)
Preparing to unpack .../kubectl_1.29.0-1.1_amd64.deb ...
Unpacking kubectl (1.29.0-1.1) over (1.28.5-1.1) ...
Preparing to unpack .../kubelet_1.29.0-1.1_amd64.deb ...
Unpacking kubelet (1.29.0-1.1) over (1.28.5-1.1) ...
Setting up kubectl (1.29.0-1.1) ...
Setting up kubelet (1.29.0-1.1) ...

➜ root@cluster3-node2:~# kubelet --version
Kubernetes v1.29.0
Now we're up to date with kubeadm, kubectl and kubelet. Restart the kubelet:

➜ root@cluster3-node2:~# service kubelet restart

➜ root@cluster3-node2:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Thu 2024-01-04 14:01:11 UTC; 2s ago
       Docs: https://kubernetes.io/docs/
    Process: 43818 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAIL>
   Main PID: 43818 (code=exited, status=1/FAILURE)

Jan 04 14:01:11 cluster3-node2 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
Jan 04 14:01:11 cluster3-node2 systemd[1]: kubelet.service: Failed with result 'exit-code'.
These errors occur because we still need to run kubeadm join to join the node into the cluster. Let's do this in the next step.

 

Add cluster3-node2 to cluster
First we log into the controlplane1 and generate a new TLS bootstrap token, also printing out the join command:

➜ ssh cluster3-controlplane1

➜ root@cluster3-controlplane1:~# kubeadm token create --print-join-command
kubeadm join 192.168.100.31:6443 --token pbuqzw.83kz9uju8talblrl --discovery-token-ca-cert-hash sha256:eae975465f73f316f322bcdd5eb6a5a53f08662ecb407586561cdc06f74bf7b2 

➜ root@cluster3-controlplane1:~# kubeadm token list
TOKEN                     TTL         EXPIRES                ...
dm3ws5.hga8xkwpp0f2lk4q   20h         2024-01-05T10:28:19Z
pbuqzw.83kz9uju8talblrl   23h         2024-01-05T14:01:38Z
rhjon6.qra3to1sjf2xnn0l   <forever>   <never>
We see the expiration of 23h for our token, we could adjust this by passing the ttl argument.

Next we connect again to cluster3-node2 and simply execute the join command:

➜ ssh cluster3-node2

➜ root@cluster3-node2:~# kubeadm join 192.168.100.31:6443 --token pbuqzw.83kz9uju8talblrl --discovery-token-ca-cert-hash sha256:eae975465f73f316f322bcdd5eb6a5a53f08662ecb407586561cdc06f74bf7b2

[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.


➜ root@cluster3-node2:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Thu 2024-01-04 14:02:45 UTC; 13s ago
       Docs: https://kubernetes.io/docs/
   Main PID: 44103 (kubelet)
      Tasks: 10 (limit: 462)
     Memory: 55.5M
     CGroup: /system.slice/kubelet.service
             └─44103 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/k>
If you have troubles with kubeadm join you might need to run kubeadm reset.

This looks great though for us. Finally we head back to the main terminal and check the node status:

➜ k get node
NAME                     STATUS     ROLES           AGE     VERSION
cluster3-controlplane1   Ready      control-plane   3h34m   v1.29.0
cluster3-node1           Ready      <none>          3h29m   v1.29.0
cluster3-node2           NotReady   <none>          20s     v1.29.0
Give it a bit of time till the node is ready.

➜ k get node
NAME                     STATUS   ROLES           AGE     VERSION
cluster3-controlplane1   Ready    control-plane   3h34m   v1.29.0
cluster3-node1           Ready    <none>          3h29m   v1.29.0
cluster3-node2           Ready    <none>          27s     v1.29.0
We see cluster3-node2 is now available and up to date.
```

- Q Question 21 | Create a Static Pod and Service
- Note: if question asked for request there is not to define the limits
```question
Use context: kubectl config use-context k8s-c3-CCC

 

Create a Static Pod named my-static-pod in Namespace default on cluster3-controlplane1. It should be of image nginx:1.16-alpine and have resource requests for 10m CPU and 20Mi memory.

Then create a NodePort Service named static-pod-service which exposes that static Pod on port 80 and check if it has Endpoints and if it's reachable through the cluster3-controlplane1 internal IP address. You can connect to the internal node IPs from your main terminal.
```

- Solution
```answer
➜ ssh cluster3-controlplane1

➜ root@cluster1-controlplane1:~# cd /etc/kubernetes/manifests/

➜ root@cluster1-controlplane1:~# kubectl run my-static-pod \
    --image=nginx:1.16-alpine \
    -o yaml --dry-run=client > my-static-pod.yaml
Then edit the my-static-pod.yaml to add the requested resource requests:

# /etc/kubernetes/manifests/my-static-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-static-pod
  name: my-static-pod
spec:
  containers:
  - image: nginx:1.16-alpine
    name: my-static-pod
    resources:
      requests:
        cpu: 10m
        memory: 20Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
 

And make sure it's running:

➜ k get pod -A | grep my-static
NAMESPACE     NAME                                   READY   STATUS   ...   AGE
default       my-static-pod-cluster3-controlplane1   1/1     Running  ...   22s
Now we expose that static Pod:

k expose pod my-static-pod-cluster3-controlplane1 \
  --name static-pod-service \
  --type=NodePort \
  --port 80
This would generate a Service like:

# kubectl expose pod my-static-pod-cluster3-controlplane1 --name static-pod-service --type=NodePort --port 80
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: my-static-pod
  name: static-pod-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: my-static-pod
  type: NodePort
status:
  loadBalancer: {}
Then run and test:

➜ k get svc,ep -l run=my-static-pod
NAME                         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/static-pod-service   NodePort   10.99.168.252   <none>        80:30352/TCP   30s

NAME                           ENDPOINTS      AGE
endpoints/static-pod-service   10.32.0.4:80   30s
Looking good.
```

- Q Question 22 | Check how long certificates are valid
```question
Use context: kubectl config use-context k8s-c2-AC

 

Check how long the kube-apiserver server certificate is valid on cluster2-controlplane1. Do this with openssl or cfssl. Write the exipiration date into /opt/course/22/expiration.

Also run the correct kubeadm command to list the expiration dates and confirm both methods show the same date.

Write the correct kubeadm command that would renew the apiserver server certificate into /opt/course/22/kubeadm-renew-certs.sh.
```

- Solution
- Main command
```bash
openssl x509 -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2

kubeadm certs -h

kubeadm certs check-expiration | grep apiserver

kubeadm certs renew apiserver
```

```answer
First let's find that certificate:

➜ ssh cluster2-controlplane1

➜ root@cluster2-controlplane1:~# find /etc/kubernetes/pki | grep apiserver
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/apiserver-etcd-client.key
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/apiserver-kubelet-client.key
Next we use openssl to find out the expiration date:

➜ root@cluster2-controlplane1:~# openssl x509  -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2
        Validity
            Not Before: Dec 20 18:05:20 2022 GMT
            Not After : Dec 20 18:05:20 2023 GMT
There we have it, so we write it in the required location on our main terminal:

# /opt/course/22/expiration
Dec 20 18:05:20 2023 GMT
And we use the feature from kubeadm to get the expiration too:

➜ root@cluster2-controlplane1:~# kubeadm certs check-expiration | grep apiserver
apiserver                Jan 14, 2022 18:49 UTC   363d        ca               no      
apiserver-etcd-client    Jan 14, 2022 18:49 UTC   363d        etcd-ca          no      
apiserver-kubelet-client Jan 14, 2022 18:49 UTC   363d        ca               no 
Looking good. And finally we write the command that would renew all certificates into the requested location:

# /opt/course/22/kubeadm-renew-certs.sh
kubeadm certs renew apiserver
```

- Q Question 23 | Kubelet client/server cert info
```question
Use context: kubectl config use-context k8s-c2-AC

 

Node cluster2-node1 has been added to the cluster using kubeadm and TLS bootstrapping.

Find the "Issuer" and "Extended Key Usage" values of the cluster2-node1:

kubelet client certificate, the one used for outgoing connections to the kube-apiserver.

kubelet server certificate, the one used for incoming connections from the kube-apiserver.

Write the information into file /opt/course/23/certificate-info.txt.

Compare the "Issuer" and "Extended Key Usage" fields of both certificates and make sense of these.
```

- Solution
```answer
To find the correct kubelet certificate directory, we can look for the default value of the --cert-dir parameter for the kubelet. For this search for "kubelet" in the Kubernetes docs which will lead to: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet. We can check if another certificate directory has been configured using ps aux or in /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf.

First we check the kubelet client certificate:

➜ ssh cluster2-node1

➜ root@cluster2-node1:~# openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep Issuer
        Issuer: CN = kubernetes
        
➜ root@cluster2-node1:~# openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep "Extended Key Usage" -A1
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
Next we check the kubelet server certificate:

➜ root@cluster2-node1:~# openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep Issuer
          Issuer: CN = cluster2-node1-ca@1588186506

➜ root@cluster2-node1:~# openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep "Extended Key Usage" -A1
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
We see that the server certificate was generated on the worker node itself and the client certificate was issued by the Kubernetes api. The "Extended Key Usage" also shows if it's for client or server authentication.

More about this: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping
```

- Q Question 24 | NetworkPolicy
```question
Use context: kubectl config use-context k8s-c1-H

 

There was a security incident where an intruder was able to access the whole cluster from a single hacked backend Pod.

To prevent this create a NetworkPolicy called np-backend in Namespace project-snake. It should allow the backend-* Pods only to:

connect to db1-* Pods on port 1111

connect to db2-* Pods on port 2222

Use the app label of Pods in your policy.

After implementation, connections from backend-* Pods to vault-* Pods on port 3333 should for example no longer work.
```

- Question 24 | NetworkPolicy
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: project-snake
---
apiVersion: v1
kind: Pod
metadata:
  name: backend-example
  namespace: project-snake
  labels:
    app: backend
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: db1-example
  namespace: project-snake
  labels:
    app: db1
spec:
  containers:
    - name: python-server
      image: python:alpine
      command: ["python"]
      args: ["-m", "http.server", "1111"]
      ports:
        - containerPort: 1111
---
apiVersion: v1
kind: Pod
metadata:
  name: db2-example
  namespace: project-snake
  labels:
    app: db2
spec:
  containers:
    - name: python-server
      image: python:alpine
      command: ["python"]
      args: ["-m", "http.server", "2222"]
      ports:
        - containerPort: 2222
---
apiVersion: v1
kind: Pod
metadata:
  name: vault-example
  namespace: project-snake
  labels:
    app: vault
spec:
  containers:
    - name: python-server
      image: python:alpine
      command: ["python"]
      args: ["-m", "http.server", "3333"]
      ports:
        - containerPort: 3333
```

- Solution
```answer

Understanding "OR" and "AND" in NetworkPolicies
In the context of NetworkPolicies, "AND" and "OR" are logical operators that determine how traffic is allowed or denied based on combined conditions. These conditions are based on labels, ports, and other selectors.

AND: When you use "AND," all conditions specified must be met for the rule to apply. It's like saying, "Only allow traffic if it matches condition A AND condition B."

OR: In contrast, "OR" means that if any of the specified conditions are met, the rule will apply. It's like saying, "Allow traffic if it matches condition A OR condition B."

Note: After creating the policy describe the policy, you will find "AND" the conditions will be understand same rule.
First we look at the existing Pods and their labels:

➜ k -n project-snake get pod
NAME        READY   STATUS    RESTARTS   AGE
backend-0   1/1     Running   0          8s
db1-0       1/1     Running   0          8s
db2-0       1/1     Running   0          10s
vault-0     1/1     Running   0          10s

➜ k -n project-snake get pod -L app
NAME        READY   STATUS    RESTARTS   AGE     APP
backend-0   1/1     Running   0          3m15s   backend
db1-0       1/1     Running   0          3m15s   db1
db2-0       1/1     Running   0          3m17s   db2
vault-0     1/1     Running   0          3m17s   vault
We test the current connection situation and see nothing is restricted:

➜ k -n project-snake get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP          ...
backend-0   1/1     Running   0          4m14s   10.44.0.24  ...
db1-0       1/1     Running   0          4m14s   10.44.0.25  ...
db2-0       1/1     Running   0          4m16s   10.44.0.23  ...
vault-0     1/1     Running   0          4m16s   10.44.0.22  ...

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.25:1111
database one

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.23:2222
database two

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.22:3333
vault secret storage
Now we create the NP by copying and chaning an example from the k8s docs:

vim 24_np.yaml
# 24_np.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress                    # policy is only about Egress
  egress:
    -                           # first rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db1
      ports:                        # second condition "port"
      - protocol: TCP
        port: 1111
    -                           # second rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db2
      ports:                        # second condition "port"
      - protocol: TCP
        port: 2222
The NP above has two rules with two conditions each, it can be read as:

allow outgoing traffic if:
  (destination pod has label app=db1 AND port is 1111)
  OR
  (destination pod has label app=db2 AND port is 2222)
 

Wrong example
Now let's shortly look at a wrong example:

# WRONG
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
    -                           # first rule
      to:                           # first condition "to"
      - podSelector:                    # first "to" possibility
          matchLabels:
            app: db1
      - podSelector:                    # second "to" possibility
          matchLabels:
            app: db2
      ports:                        # second condition "ports"
      - protocol: TCP                   # first "ports" possibility
        port: 1111
      - protocol: TCP                   # second "ports" possibility
        port: 2222
The NP above has one rule with two conditions and two condition-entries each, it can be read as:

allow outgoing traffic if:
  (destination pod has label app=db1 OR destination pod has label app=db2)
  AND
  (destination port is 1111 OR destination port is 2222)
Using this NP it would still be possible for backend-* Pods to connect to db2-* Pods on port 1111 for example which should be forbidden.

 If there are no other restrictive network policies in place, a new pod without the specified label should be able to connect, assuming the default behavior of allowing traffic when no policies are applied.
 

Create NetworkPolicy
We create the correct NP:

k -f 24_np.yaml create
And test again:

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.25:1111
database one

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.23:2222
database two

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.22:3333
^C
Also helpful to use kubectl describe on the NP to see how k8s has interpreted the policy.

Great, looking more secure. Task done.
```

- Q Question 25 | Etcd Snapshot Save and Restore
```question
Use context: kubectl config use-context k8s-c3-CCC

 

Make a backup of etcd running on cluster3-controlplane1 and save it on the controlplane node at /tmp/etcd-backup.db.

Then create any kind of Pod in the cluster.

Finally restore the backup, confirm the cluster is still working and that the created Pod is no longer with us.
```

- Solution
```answer
Etcd Backup
First we log into the controlplane and try to create a snapshop of etcd:

➜ ssh cluster3-controlplane1

➜ root@cluster3-controlplane1:~# ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db
Error:  rpc error: code = Unavailable desc = transport is closing
But it fails because we need to authenticate ourselves. For the necessary information we can check the etc manifest:

➜ root@cluster3-controlplane1:~# vim /etc/kubernetes/manifests/etcd.yaml
We only check the etcd.yaml for necessary information we don't change it.

# /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.100.31:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt                           # use
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.100.31:2380
    - --initial-cluster=cluster3-controlplane1=https://192.168.100.31:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key                            # use
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.100.31:2379   # use
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.100.31:2380
    - --name=cluster3-controlplane1
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt                    # use
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: k8s.gcr.io/etcd:3.3.15-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: etcd
    resources: {}
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd                                                     # important
      type: DirectoryOrCreate
    name: etcd-data
status: {}
But we also know that the api-server is connecting to etcd, so we can check how its manifest is configured:

➜ root@cluster3-controlplane1:~# cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
We use the authentication information and pass it to etcdctl:

➜ root@cluster3-controlplane1:~# ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key

Snapshot saved at /tmp/etcd-backup.db
 

NOTE: Dont use snapshot status because it can alter the snapshot file and render it invalid

 

Etcd restore
Now create a Pod in the cluster and wait for it to be running:

➜ root@cluster3-controlplane1:~# kubectl run test --image=nginx
pod/test created

➜ root@cluster3-controlplane1:~# kubectl get pod -l run=test -w
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          60s
 

NOTE: If you didn't solve questions 18 or 20 and cluster3 doesn't have a ready worker node then the created pod might stay in a Pending state. This is still ok for this task.

 

Next we stop all controlplane components:

root@cluster3-controlplane1:~# cd /etc/kubernetes/manifests/

root@cluster3-controlplane1:/etc/kubernetes/manifests# mv * ..

root@cluster3-controlplane1:/etc/kubernetes/manifests# watch crictl ps
Now we restore the snapshot into a specific directory:

➜ root@cluster3-controlplane1:~# ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
--data-dir /var/lib/etcd-backup \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key

2020-09-04 16:50:19.650804 I | mvcc: restore compact to 9935
2020-09-04 16:50:19.659095 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
We could specify another host to make the backup from by using etcdctl --endpoints http://IP, but here we just use the default value which is: http://127.0.0.1:2379,http://127.0.0.1:4001.

The restored files are located at the new folder /var/lib/etcd-backup, now we have to tell etcd to use that directory:

➜ root@cluster3-controlplane1:~# vim /etc/kubernetes/etcd.yaml
# /etc/kubernetes/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
...
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-backup                # change
      type: DirectoryOrCreate
    name: etcd-data
status: {}
Now we move all controlplane yaml again into the manifest directory. Give it some time (up to several minutes) for etcd to restart and for the api-server to be reachable again:

root@cluster3-controlplane1:/etc/kubernetes/manifests# mv ../*.yaml .

root@cluster3-controlplane1:/etc/kubernetes/manifests# watch crictl ps
Then we check again for the Pod:

➜ root@cluster3-controlplane1:~# kubectl get pod -l run=test
No resources found in default namespace.
Awesome, backup and restore worked as our pod is gone.
```

- Optional Setup

```bash
Fast dry-run output

export dry="--dry-run=client -o yaml"
This way you can just run k run pod1 --image=nginx $do. Short for "dry output", but use whatever name you like.

Fast pod delete

export now="--force --grace-period 0"
```


### ETCD Backup and Restore in non stacked environment or (systemd service)

```explain
# find etcd servers
 cat kube-apiserver.yaml | grep -i etcd
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem
    - --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem
    - --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem
    - --etcd-servers=https://192.1.115.22:2379
    
    
 # find etcd servers   
 ps aux | grep -i etcd
 
 
 # find etcd servers
  k describe pod/kube-apiserver-cluster2-controlplane -n kube-system | grep -i etcd
 
 Update ETCD Data Directory Guide
 
🕵️‍♂️ Finding ETCD Server's Address
Identify ETCD Processes: Begin by identifying running ETCD processes and their configurations on your Kubernetes control plane node. This helps in finding the ETCD server's address.


step1: ps aux | grep -i etcd
Look for the --listen-client-urls or --advertise-client-urls arguments to find the ETCD server's address.

SSH into the ETCD Server: Once you've identified the ETCD server's IP address from the previous step, SSH into the ETCD server.


step2: ssh <etcd-server-ip>
🔄 Updating the ETCD Data Directory
Prepare the New Data Directory: Create the new ETCD data directory and set appropriate permissions.


step3: sudo mkdir -p /root/etcd-asim && sudo chown -R etcd:etcd /root/etcd-asim
Stop the ETCD Service: It's important to stop ETCD before making changes to prevent data corruption.


step4: sudo systemctl stop etcd.service
Backup ETCD Data Using etcdctl: Utilize the etcdctl utility to back up your ETCD data safely.


step5: etcdctl snapshot save /path/to/your/backup/snapshot.db
Make sure to replace /path/to/your/backup/snapshot.db with your actual backup file path.

Edit the ETCD Service File: Update the ETCD systemd unit file to point to the new data directory.

Open the file in a text editor:

step6: sudo nano /etc/systemd/system/etcd.service
Locate the --data-dir flag and update its value to the new directory path (chown -R etcd:etcd /var/lib/etcd-data-asim).
Apply Configuration Changes: Reload the systemd manager configuration and restart the ETCD service.

#########################################
Note: must use this path /var/lib
chown -R etcd:etcd /var/lib/etcd-data-asim
##########################################

step7: sudo systemctl daemon-reload && sudo systemctl start etcd.service
Verify the Configuration Update: Check the ETCD service to ensure it's running correctly with the new data directory.

step8: sudo systemctl status etcd.service
🛠 Troubleshooting
Review Logs: If the ETCD service encounters issues, review the logs for diagnostics.

step9: sudo journalctl -u etcd.service
Permission Checks: Confirm that the ETCD user has necessary permissions for the new data directory.

Step 10: Restart Control Plane Components
After making changes to the ETCD configuration and ensuring ETCD is up and functioning correctly, you need to restart the Kubernetes control plane components. This usually includes the kube-apiserver, kube-controller-manager, and kube-scheduler. If you're running these as pods managed by the kubelet (which is common in setups initiated by kubeadm), you can restart the controller-manager and scheduler pods directly. The kubelet itself may be managed by your system's service manager (like systemd).

Delete Controller-Manager and Scheduler Pods
Kubernetes control plane components such as the kube-controller-manager and kube-scheduler run as static pods on the control plane node. Their manifests are typically located in /etc/kubernetes/manifests, and the kubelet is responsible for their lifecycle.

To "restart" these components, you can delete their pods. The kubelet will automatically recreate them using the manifests:

kubectl delete pod -n kube-system kube-controller-manager-<node_name>
kubectl delete pod -n kube-system kube-scheduler-<node_name>
Replace <node_name> with the name of your control plane node. You can find the exact pod names with kubectl get pods -n kube-system.

# quick ---> k delete pod --all -n kube-system
Restart Kubelet
The kubelet may need to be restarted to pick up changes in its configuration or to ensure it refreshes its state. You can restart the kubelet using your system's service manager. For systems using systemd:

sudo systemctl restart kubelet
This command restarts the kubelet service, which in turn ensures it reloads its configuration and refreshes its state regarding the pods it manages.

Considerations
Cluster Downtime: Restarting control plane components might lead to temporary unavailability of the Kubernetes API and other control plane functionalities. Plan to perform these steps during a maintenance window if possible.

High Availability: If your Kubernetes cluster has a High Availability (HA) setup with multiple control plane nodes, you might want to perform these restarts one node at a time to minimize downtime and impact on cluster operations.

Backup: Ensure you have a recent backup of ETCD and your Kubernetes cluster's state before performing these operations, especially if the changes involve critical components like ETCD.

Following these steps ensures that your Kubernetes control plane components are correctly restarted and are in sync with the updated ETCD configuration or any significant changes made to the ETCD cluster.

This guide assumes that you have the required access rights and prerequisites installed, such as the etcdctl utility for backing up ETCD data. Adjust <etcd-server-ip> with the actual IP address of your ETCD server identified in step 1.
```


### Learning Pod Affinity think of pod labels/Node Affinity think of node labels

### Pod Affinity (attract) tightly coupled
- search topology
```yaml
affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
```


- Q
```question
#That Pod should be REQUIRED to be only scheduled on Nodes where Pods with label level=restricted are running.
#That Pod should be PREFERRED to be only scheduled on Nodes where Pods with label level=restricted are running.
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    level: hobby
  name: hobby-project
spec:
  containers:
  - image: nginx:alpine
    name: c
```

- Answer
```yaml
# kubectl get pod --show-labels
apiVersion: v1
kind: Pod
metadata:
  labels:
    level: hobby
  name: hobby-project
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: level
            operator: In
            values:
            - restricted
        topologyKey: kubernetes.io/hostname
  containers:
  - image: nginx:alpine
    name: c
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    level: hobby
  name: hobby-project
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: level
              operator: In
              values:
              - restricted
          topologyKey: kubernetes.io/hostname
  containers:
  - image: nginx:alpine
    name: c
```



### Pod Anti Affinity (repel) HA high availibility
```yaml
affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
```

### emptyDir and InitContainers
- Note: initContainers are executed before the main container starts and separate from the main container
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      volumes:
        - name: html
          emptyDir: {}
      initContainers:
        - name: init-myservice
          image: busybox:1.28
          command: ["/bin/sh","-c","echo 'hello world' > /html/index.html"]
          volumeMounts:
            - mountPath: /html
              name: html
      containers:
        - image: nginx
          name: nginx
          volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: html
          resources: {}
status: {}
```

### Networking question
```question
For this question, please set this context (In exam, diff cluster name)

kubectl config use-context kubernetes-admin@kubernetes


my-app-deployment and cache-deployment deployed, and my-app-deployment deployment exposed through a service named my-app-service . Create a NetworkPolicy named my-app-network-policy to restrict incoming and outgoing traffic to my-app-deployment pods with the following specifications:

Allow incoming traffic only from pods within the same namespace.
Allow incoming traffic from a specific pod with the label app=trusted
Allow outgoing traffic to pods within the same namespace.
Deny all other incoming and outgoing traffic.
```
- Note: same namespace means where the policy is applied (default namespace)
```describe
When implementing Kubernetes NetworkPolicies, it's important to understand that by default, if no policies are applied to a set of pods, all traffic is allowed to and from those pods. However, as soon as you apply a NetworkPolicy to a set of pods, it restricts the traffic according to the specified rules, and all other traffic not explicitly allowed by any NetworkPolicy is denied. 
```
- Solution
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-app-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector: {}  # Allows incoming traffic from any pod within the same namespace
        - podSelector:
            matchLabels:
              app: trusted  # Specifically allows incoming traffic from pods labeled app=trusted
  egress:
    - to:
        - podSelector: {}  # Allows outgoing traffic to any pod within the same namespace
```


### Quick Search 

```search
pvc select option 3 for persistent volumes ---> ctlf + F ---> nfs

network select option 1 for network policies

secretkeyref select option 2 for secrets ---> ctlf + F ---> secretkeyref

secretref select option 1 for secrets ---> ctlf + F ---> secretref

secret as a volume search option 2 ---> search kind

configmapkeyref select option 2 for configmaps ---> ctlf + F ---> configmapkeyref

configmapref select option 1 for configmaps ---> ctlf + F ---> configmapref

configmap as a volume search option 2 ---> search kind

nodeaffinity option 1

podtopology or pod anti-affinity option 1

kubernetes backup select option 2 ---> search cacert

kubernetes upgrade select option 1 ---> search madison

network select option 1 ---> search namespaceselector
```

### PV PVC
```question
Task -
Create a new PersistentVolumeClaim:
✑ Name: pv-volume
✑ Class: csi-hostpath-sc
✑ Capacity: 10Mi
Create a new Pod which mounts the PersistentVolumeClaim as a volume:
✑ Name: web-server
✑ Image: nginx
✑ Mount path: /usr/share/nginx/html
Configure the new Pod to have ReadWriteOnce access on the volume.
Finally, using kubectl edit or kubectl patch expand the PersistentVolumeClaim to a capacity of 70Mi and record that change.
```

- Solution
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 70Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: csi-hostpath-sc
  hostPath:
    path: /mnt/data
    type: DirectoryOrCreate
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  volumeName: pv-volume
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
  storageClassName: csi-hostpath-sc
---
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: html-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html-volume
    persistentVolumeClaim:
      claimName: pv-volume
```
- afterwards update the yaml for pvc 70Mi with --record command
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 70Mi
  storageClassName: csi-hostpath-sc
```
```bash
kubectl apply -f pvc.yaml --record
```

### Important commands

```bash
jobs
kill %1
kill -9 %1
bq %1 #resume the background job, here job number is 1

fg

k top pods --containers=true

openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem 

kubeadm -h

kubeadm certs -h

kubeadm certs check-expiration | grep apiserver

kubeadm certs renew apiserver

kubeadm token create -h

kubeadm token create --print-join-command 

apt-cache madison kubeadm

apt-cache show kubeadm

k api-resources    # shows all

k api-resources -h # help always good

k api-resources --namespaced -o name 

k scale -h

k scale deploy nginx --replicas=3 -n default

k get pods -o name

k top pods --sort-by=cpu

k top -l app=nginx --sort-by=cpu

k top --selector app=nginx --sort-by=cpu

k top pods --sort-by=memory

k get pods --sort-by=.metadata.creationTimestamp

grep -irn /etc/kubernetes/pki/etcd/ca.crt

k get svc/redis-service -o json | jq -c paths | grep -iC5 target

k get svc/redis-service -o jsonpath='{.spec.ports[].targetPort}'

k create clusterrole -h | head -n10 

k create clusterrolebinding -h | head -n10

k expose -h | head -n20

k expose -h | tail -n10

k config get-contexts

for i in cluster1 cluster2 cluster3 cluster4;do k --context="$i" top nodes --sort-by=cpu;done

k get pods,deploy,svc -n cyan-ns-cka28-trb

k get netpol -n cyan-np-cka28-trb

k get netpol -A

kubectl set image deployment deplpoymentname containename=image:tag

k rollout -h

k rollout history -h

k rollout history deploy ocean-tv-wl09

k rollout history deploy ocean-tv-wl09 --revision=2

kubectl rollout undo -h

k rollout undo deploy ocean-tv-wl09 --to-revision=1

k set -h

k set image -h

# Note: Run below commands from the same place (node)
ssh node01 -- ls /etc/kubernetes/manifests

ssh controlplane -- ls /etc/kubernetes/manifests

k drain -h

k drain node01 --ignore-daemonsets

kubectl get pod --show-labels

k get pods -L level

k config get-contexts -o name > contexts-name.txt

for i in `cat contexts-name.txt`;do k --context=$i top nodes --sort-by=memory;done

cat ~/.kube/config | grep current-context | awk '{print $2}'

k get pods -A --sort-by=metadata.creationTimestamp

k top pod -h

kubectl top pod POD_NAME --containers

k top pod -A --containers=true

sed -h 

sed -i 's/scheduler.config/scheduler.conf/' kube-scheduler.yaml

k get ns -o name | awk -F '/' '{print $2}'

 &>
 The &> in above's command redirects both the standard output and standard error.
 ssh cluster1-node2 -- 'crictl logs b01edbe6f89ed' &> /opt/course/17/pod-container.log
 
 # Run the command in the background
 k -n default port-forward svc/nginx-service --address 0.0.0.0 80:80 &
 bg to list background jobs
fg to bring the background job to the foreground
ctrl+ z to pause the foreground job
bg %1 to resume the paused job in the background

k explain pod

k explain ingress

# trouble shooting
k describe node01
journalctl -u kubelet ---> shift g # this will take to the end of the logs

k get priorityclass
k describe pod podname | grep -iC5 prior

k get ingressclasses

k get ingressclass
```

- bash script for quick commands, simply source it
- Note: bash script is not required, even create a file k8.txt and below variables and simply source it
```bash
export dry='-o yaml --dry-run=client'
export del='kubectl delete --force --grace-period=0 -f'
export apply='kubectl apply --force --grace-period-0 -f'
```

- Usage
```bash
k run test --image=nginx $dry
k create deploy test --image=buxybox:1.28 $dry
```

- https://killer.sh/killercoda-access

### Take a shell on a node 
```bash
kubectl debug node/gke-cluster-1-default-pool-456e988f-czg5 -it --image=ubuntu

# use chroot /host in your debug container to access the host's root filesystem
chroot /host
```

### Kubernetes Troubleshooting
```bash
# Timestamps
k logs pod/podname -n default --timestamps --since=1h

# Timestamps follow
k logs pod/podname -n default --timestamps -f

# Search for a specific string in pod logs
k logs pod/podname -n default | grep "error"

# Search for a string with context (3 lines before and after)
k logs pod/podname -n default | grep -A 3 -B 3 -C5 "error"

# Case insensitive search
k logs pod/podname -n default | grep -i "error"

# Count occurrences of a string
k logs pod/podname -n default | grep -c "error"

# Show logs for previous container instance
k logs pod/podname -n default --previous | grep "error"

# Combine with timestamps and search
k logs pod/podname -n default --timestamps | grep "error"

# Multiple string patterns (OR condition)
k logs pod/podname -n default | grep -E "error|warning|failed"

# Search in multiple pods matching a label
k logs -l app=myapp -n default | grep "error"

# Save matching logs to a file
k logs pod/podname -n default | grep "error" > error_logs.txt

# Show only matched string with line numbers
k logs pod/podname -n default | grep -n "error"
```

### Debug pod
```bash
k debug pod/POD-NAME -n default -it --image=google/cloud-sdk:latest -- bash
```
