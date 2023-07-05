# Configuration

###### TMUX

```bash
# Enable mouse
ctrl-b :set mouse on
# Split window vertically
ctrl-b "
# Sync Panes
ctrl-b :set synchronize-panes
# Zoom on current pane
ctrl-b z
# Detach
ctrl-b d
# Move
ctrl-b <ARROW>
```

###### Env Configuration

```bash
alias k=kubectl                  
# Output yaml: k create pod mypod --image=nginx $do
export do="--dry-run=client -o yaml"
# Delete fast: k delete pod x $now 
export now="--force --grace-period 0"
# Select context to work in   
alias kn='kubectl config set-context --current --namespace '
```

###### ViM Config for YAML

```bash
set expandtab
set tabstop=2
set shiftwidth=2
# or to retab a file
set tabstop=2 shiftwidth=2 expandtab
:retab
```

###### Tips

```bash
# Use custom .kubeconfig
KUBECONFIG=~/.kubeconfig k version
# List all the names of namespaced resources
k api-resources --namespaced -o name
# Get available contexts
### Using kubectl
k config get-contexts -o name
k config view -o jsonpath="{.contexts[*].name}" | tr " " "\n"
### Not using kubectl
cat ~/.kube/config | grep current | sed -e "s/current-context: //"
```

###### Default Namespace

```bash
# Change to a specific namespace
k config set-context --current --namespace=alpha
```

# JSON

###### JsonPath

```bash
# Get available paths
k get nodes -o json | jq -c 'paths'
k get nodes -o jsonpath='.items[0].status' | jq
# Select only a field
k get nodes -o jsonpath='.items[0].status'
# Sort results
k get nodes -o jsonpath='.items[0].status' --sort-by=.metadata.creationTimestamp
# Filtering inside jsonpath
-o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}"
# Show results in columns
-o=custom-columns=<COL_NAME>:<JSONPATH> --sort-by=<SORT_KEY>
# Example (both yield similar results)
-o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}'
-o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
  --sort-by=.status.capacity.cpu
# Get output without headers
k get nodes --no-headers
# Inverse output
k get nodes | tac
# List pods that are candidates for termination (no requests)
k get pod -o jsonpath="{range .items[*]} {.metadata.name}{.spec.containers[*].resources} {'\n'}" -A
```

# Debugging

###### Launch a temporary pod

```shell
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

###### Application Issues

```bash
k get deploy
k describe deploy
k get pods -o wide
k describe pod <podName>
# Obtain logs of pod and specific container(s)
k logs [--previous] <PodName> --all-containers [-c container]
# Get events of k8s
kubectl get events --field-selector involvedObject.name=mypod
k get events -A --sort-by=.metadata.creationTimestamp
```

###### Control Plane Failures

```bash
k get nodes
k get pods
k get pods -n kube-system
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status
service kubelet status
service kube-proxy status
# If kubeadm:
k logs kube-apiserver-master -n kube-system
# If services:
journalctl -u kube-apiserver
```

###### Worker node

```bash
k get nodes
k describe node <name>
# On the node
top
df -h
free -mh
service kubelet status
journalctl -u kubelet
# check the certificate ISSUER, NOTAFTER, O=system:nodes
openssl x509 -in /var/lib/kubelet/worker-1.crt -text
```

###### Debugging Networking

```bash
# CoreDNS
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
proxy . /etc/resolv.conf
# 
```

# Resources

### Authentication

```bash
# Check certificate expiration
kubeadm certs check-expiration
# Renew certificates
kubeadm certs renew apiserver
```

### Volume

###### EmptyDir

The volume lives the times the pod is alive

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```

### Pods

###### Copy files to pod

```bash
kubectl cp file <pod_name>:/path
```

### Deployments

### Services

### NetworkPolicies

https://www.youtube.com/watch?v=3gGpMmYeEO8

[GitHub - ahmetb/kubernetes-network-policy-recipes: Example recipes for Kubernetes Network Policies that you can just copy paste](https://github.com/ahmetb/kubernetes-network-policy-recipes)

### RBAC

1. *Role* + *RoleBinding* (available in single *Namespace*, applied in single *Namespace*)
2. *ClusterRole* + *ClusterRoleBinding* (available cluster-wide, applied cluster-wide)
3. *ClusterRole* + *RoleBinding* (available cluster-wide, applied in single *Namespace*)

###### Create a Role

`k create role developer --verb=create --resource=pods -n development`

`k describe role developer`

###### Create a SA, CR, CRB

```bash
# Create serviceaccount
k create serviceaccount pvviewer
# Create clusterrole
k create clusterrole pvviewer-role 
  --verb=list --resource=persistentvolumes
# Associate SA<->ClusterRoleBinding
k create clusterrolebinding pvviewer-role-binding 
  --clusterrole pvviewer-role --serviceaccount=pvviewer
# Check permissions of SA
k auth can-i list persistentvolumes 
  --as system:serviceaccount:default:pvviewer
```

### Nodes

##### Join a node to a cluster

```bash
# ssh into controlplane
kubeadm token create --print-join-command
# ssh into node and paste the results ^
```

##### Taint & Tolerations

###### Taint

`k taint node foo key1=val1:NoSchedule`

###### Taint (Remove)

`k taint node foo key1=val1:NoSchedule-`

###### Tolerations

To make a pod be scheduled on a node that has a taint you must set a toleration for that pod. Example: the following pod will be scheduled on node **foo**

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "val1"
    effect: "NoSchedule"
```

# Performance Analysis

Be sure you have metrics deployed in your cluster

```bash
k top node
k top pod
```

### Pods

###### From a pod get the name of the node where it is running (Using downward API)

[Downward API | Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/downward-api/)

```yaml
    env:                                                                          # add
    - name: MY_NODE_NAME                                                          # add
      valueFrom:                                                                  # add
        fieldRef:                                                                 # add
          fieldPath: spec.nodeName  
```

###### Schedule a pod in certain node

In this example we will schedule the pod in the controlplane

```bash
# Check the node taints
k describe node cluster1-controlplane | grep Taint -A1
k get node cluster1-controlplane1 --show-labels
# Add this in the yaml of the pod
  tolerations:                                 # add
  - effect: NoSchedule                         # add
    key: node-role.kubernetes.io/control-plane # add
  nodeSelector:                                # add
    node-role.kubernetes.io/control-plane: ""  # add
```



### StatefulSet

Some applications need to persist data in order to work properly, example: databases

###### https://www.youtube.com/watch?v=r_ZEpPTCcPE

```bash
# To scale up or down
k scale sts sts-name --replicas 1
```

### Scheduler

The only thing it does it sets the **nodeName** for a pod

###### Stop scheduler

```bash
ssh controlplane; mv /etc/kubernetes/manifests/sched.yaml /tmp
```

### Deployments

Run a pod where another pod is not running: **podAntiAffinity & topologyKey** or **topologySpreadConstraints**

### Networking

###### What CNI is configured?

`find /etc/cni/net.d`

### Secrets

###### Create secrets

```bash
k create secret generic mysecret --from-literal=user=user1
```

### CRICTL

To look at low level what is going on with containers you can use this tool, is very similar to docker

```bash
crictl rm,ps,inspect,logs
```

-----

**Mock Exam 1**

```bash
# Deploy a nginx pod
k run nginx-pod --image=nginx:alpine
# Depoy a pod with labels
k run messaging --image=redis-alpine --labels="tier=msg"
k get pod messaging --show-labels
k describe pod messaging
# Create ns
k create ns myns
# Get nodes and output in json
k get nodes -o json > /tmp/nodes.json
# Create service (clusterIp)
k expose pod messaging --port 6379 --name messaging-service
k describe svc messaging-service
# Create deployment
k create deployment hr-web-app --image=kodekloud/webapp-color --replicas=2
k describe deploy hr-web-app
# Create static pod
k run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yaml
mv static-busybox.yaml /etc/kubernetes/manifests/
# Deploy pod in ns
k run temp-bus --image=redis:alpine -n finance
# Fix Issue with pod orange
k describe pod orange
k edit pod orange
k replace --force -f /tmp/kubectl-edit-...
k get pods -w
# Expose a deployment through a nodeport
k expose deploy hr-web-app --name=hr-web-app-service --type NodePort --port 8080
k edit svc hr-web-app-service # Change the nodePort to custom port
# Retrieve node Image of the nodes in json (read cheatsheet)
k get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}'
```

```bash
# Create service account, clusterrole and clusterrolebinding
k create serviceaccount pvviewer
k create clusterrole pvviewer-role --verb=list --resource=persistentvolumes
k create clusterrolebinding pvviewer-role-binding 
--clusterrole pvviewer-role --serviceaccount=pvviewer
k auth can-i get pods --as system:serviceaccount:default:pvviewer
```

Exam 2

```bash
# Snapshot of etcd
ETCDCTL_API=3 etcdctl \
--cert=/etc/kubernetes/pki/etcd/server.cr \ 
--key=/etc/kubernetes/pki/etcd/server.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt 
snapshot save snapshotdb
# Create a user, asign permissions given key and csr
# Approve the csr -> create role -> assign perm -> create rolebinding
# Using RBAC
Create a john-csr.yaml
cat jogh.csr | base64 | tr -d "\n" # Save in john-csr.yaml
k apply -f john-csr.yaml
k get csr
k certificate approve john-developer
# Create role
k create role developer --verb=create,get,list,update,delete
  --resource=pods -n development
k describe role -n development
# Get permissions
k auth can-i get pods --namespace=development --as john
# Create rolebinding
k create rolebinding john-developer --role=developer --user=john -n development
# Get permissions
k auth can-i get pods --namespace=development --as john

# Expose port through service
k run nginx-resolver --image=nginx
k expose pod nginx-resolver --name=nginx-resolver-service --port=80

# Check resolution
k run busybox --image=busybox:1.28 -- sleep 4000
k exec busybox -- nslookup nginx-resolver-service
```

Exam 3

```bash
# Get paths json
k get nodes -o json | jq -c 'paths'
k get nodes -o jsonpath='.items[0].status' | jq
# Network Policy
k get pod --show-labels
```

---------------

Deployments

```bash
k set image deployment/myapp-deploy nginx:1.9.1 # Rolling upgrade
k rollout (status|history|undo) deployment/myapp-deploy
```

Etcd

```bash
etcdctl set key1 val1
etcd get key1
```

Generate OpenSSL Certificates

```bash
# 1. Self Sign Certificate
# Generate private key
openssl genrsa -out ca.key 2048
# Generate CSR
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
# Sign Certificate
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt

# 2. Client Certificates
# Generate pk
openssl genrsa -out admin.key 2048
# Generate CSR
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
# Sign certificate with the CA-SelfSignedCertificate
openssl x509 -req -in admin.csr -CA ca.crt -CAKey ca.key -out admin.crt


# Get info about certificate
openssl x509 -in admin.crt -text -noout
```

RBAC

```bash
k get roles
k get rolebindings
k describe role developer
k describe rolebinding devuser-developer-binding
k auth can-i <create deployments>
k auth can-i <create deployments> --as dev-user
k auth can-i <op> --as dev-user --namespace testt
```

Cluster Roles

```bash
k api-resource --namespaced=true 
```

```bash
k create serviceaccount bla
k get serviceaccount
k describe serviceaccount bla
```

```bash
k describe secret secret-name
# Service accounts
k create serviceaccount dashboard-sa
k create token dashboard-sa
```

```bash
k create secret docker-registry \
  --docker-server=xxx \
  --docker-username=xxx \
  --docker-password=xxx \
  --docker-email=xxx
```

```bash
docker run --cap-add \
           --cap-drop \
           --priviledged
# In k8s pod or/and container: securityContext, capabilities
```

### Ingress

```bash
kubectl create ingress <ingress-name> --rule="host/path=service:port"
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

```yaml
# Create a PV bound to the certain hostPath
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
      path: /pv/data-analytics
```

```bash
# Find the node across all clusters that consumes the most CPU 
#and store the result to the file `/opt/high_cpu_node` 
# in the following format `cluster_name,node_name`.
kubectl top node --context cluster1 --no-headers | sort -nr -k2 | head -1
```

```bash
# Can i do a certain action with a specific service account?
k auth can-i delete pods 
  --namespace default 
  --as system:serviceaccount:default:thor-cka24-trb
```

```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: olive-pvc-cka10-str
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: olive-stc-cka10-str
  volumeName: olive-pv-cka10-str
  resources:
    requests:
      storage: 100Mi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: olive-app-cka10-str
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: olive-app-cka10-str
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - cluster1-node01
      containers:
      - name: python
        image: poroko/flask-demo-app
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: python-data
          mountPath: /usr/share/
      - name: busybox
        image: busybox
        command:
          - "bin/sh"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: python-data
            mountPath: "/usr/src"
            readOnly: true
      volumes:
      - name: python-data
        persistentVolumeClaim:
          claimName: olive-pvc-cka10-str
  selector:
    matchLabels:
      app: olive-app-cka10-str

---
apiVersion: v1
kind: Service
metadata:
  name: olive-svc-cka10-str
spec:
  type: NodePort
  ports:
    - port: 5000
      nodePort: 32006
  selector:
    app: olive-app-cka10-str
```

```bash
# Obtain pods matching with labels
k get pod -l mode=exam,type=external                                    

# Display pods with certain info in columns
k get pods 
  -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' 
  --sort-by=.status.podIP
```

```bash
# Add a custom endpoint for a external service in student-node
student-node ~ ➜  export IP_ADDR=$(ifconfig eth0 | grep inet | awk '{print $2}')

student-node ~ ➜ kubectl --context cluster3 apply -f - <<EOF
apiVersion: v1
kind: Endpoints
metadata:
  # the name here should match the name of the Service
  name: external-webserver-cka03-svcn
subsets:
  - addresses:
      - ip: $IP_ADDR
    ports:
      - port: 9999
EOF
```
