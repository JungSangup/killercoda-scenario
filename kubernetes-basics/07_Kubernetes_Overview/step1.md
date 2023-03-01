기본적인 쿠버네티스 명령어들에 대해 알아보겠습니다.  
제일 먼저 도움말을 볼까요?
```bash
controlplane $ kubectl --help
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/

Basic Commands (Beginner):
  create          Create a resource from a file or from stdin
  expose          Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run             Run a particular image on the cluster
  set             Set specific features on objects

Basic Commands (Intermediate):
  explain         Get documentation for a resource
  get             Display one or many resources
  edit            Edit a resource on the server
  delete          Delete resources by file names, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout         Manage the rollout of a resource
  scale           Set a new size for a deployment, replica set, or replication controller
  autoscale       Auto-scale a deployment, replica set, stateful set, or replication controller

Cluster Management Commands:
  certificate     Modify certificate resources.
  cluster-info    Display cluster information
  top             Display resource (CPU/memory) usage
  cordon          Mark node as unschedulable
  uncordon        Mark node as schedulable
  drain           Drain node in preparation for maintenance
  taint           Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe        Show details of a specific resource or group of resources
  logs            Print the logs for a container in a pod
  attach          Attach to a running container
  exec            Execute a command in a container
  port-forward    Forward one or more local ports to a pod
  proxy           Run a proxy to the Kubernetes API server
  cp              Copy files and directories to and from containers
  auth            Inspect authorization
  debug           Create debugging sessions for troubleshooting workloads and nodes
  events          List events

Advanced Commands:
  diff            Diff the live version against a would-be applied version
  apply           Apply a configuration to a resource by file name or stdin
  patch           Update fields of a resource
  replace         Replace a resource by file name or stdin
  wait            Experimental: Wait for a specific condition on one or many resources
  kustomize       Build a kustomization target from a directory or URL.

Settings Commands:
  label           Update the labels on a resource
  annotate        Update the annotations on a resource
  completion      Output shell completion code for the specified shell (bash, zsh, fish, or powershell)

Other Commands:
  alpha           Commands for features in alpha
  api-resources   Print the supported API resources on the server
  api-versions    Print the supported API versions on the server, in the form of "group/version"
  config          Modify kubeconfig files
  plugin          Provides utilities for interacting with plugins
  version         Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

> 💻 명령어 `kubectl --help`{{exec}}

여러가지 명령어들을 볼 수 있고, 사용법을 알 수 있습니다.

<br><br><br>

몇 가지 명령어들을 알아볼까요?

버젼을 알아보려면,
```bash
controlplane $ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:58:16Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:51:25Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}

controlplane $ kubectl version --output yaml
clientVersion:
  buildDate: "2023-01-18T15:58:16Z"
  compiler: gc
  gitCommit: 8f94681cd294aa8cfd3407b8191f6c70214973a4
  gitTreeState: clean
  gitVersion: v1.26.1
  goVersion: go1.19.5
  major: "1"
  minor: "26"
  platform: linux/amd64
kustomizeVersion: v4.5.7
serverVersion:
  buildDate: "2023-01-18T15:51:25Z"
  compiler: gc
  gitCommit: 8f94681cd294aa8cfd3407b8191f6c70214973a4
  gitTreeState: clean
  gitVersion: v1.26.1
  goVersion: go1.19.5
  major: "1"
  minor: "26"
  platform: linux/amd64
```

> 💻 명령어 `kubectl version`{{exec}}  
>또는  
> 💻 명령어 `kubectl version --output yaml`{{exec}}

<br><br><br>

쿠버네티스 클러스터 정보를 확인하려면,
```bash
controlplane $ kubectl cluster-info
Kubernetes control plane is running at https://172.30.1.2:6443
CoreDNS is running at https://172.30.1.2:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

> 💻 명령어 `kubectl cluster-info`{{exec}}

<br><br><br>

우리 클러스터의 노드 목록은 아래 명령어로 알아볼 수 있습니다.
```bash
controlplane $ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   5d12h   v1.26.1
node01         Ready    <none>          5d12h   v1.26.1
```

> 💻 명령어 `kubectl get nodes`{{exec}}

<br><br><br>

`--output wide`옵션(또는, `-o wide`)을 사용하면 더 많은 정보를 보여줍니다.
```bash
controlplane $ kubectl get nodes --output wide
NAME           STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
controlplane   Ready    control-plane   5d12h   v1.26.1   172.30.1.2    <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.12
node01         Ready    <none>          5d12h   v1.26.1   172.30.2.2    <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.12
```

> 💻 명령어 `kubectl get nodes --output wide`{{exec}}

<br><br><br>

이제 우리 클러스터에 존재한는 리소스를 조회하는 방법을 알아볼게요.  

먼저 현재 존재하는 **POD**목록은 아래와 같이 조회합니다.
```bash
controlplane $ kubectl get pods
No resources found in default namespace.
```

> 💻 명령어 `kubectl get pods`{{exec}}

<br><br><br>

음. 아무것도 없군요...  
지금은 **default 네임스페이스**에서 조회를 한 경우입니다.

`--namespace`로 네임스페이스를 지정하지 않으면 **default 네임스페이스**를 기본으로 합니다.

다른 네임스페이스는 뭐가 있을까요?  
네임스페이스를 보려면 아래 명령어를 사용하면 됩니다.
```bash
controlplane $ kubectl get namespaces
NAME                 STATUS   AGE
default              Active   5d12h
kube-node-lease      Active   5d12h
kube-public          Active   5d12h
kube-system          Active   5d12h
local-path-storage   Active   5d12h
```

> 💻 명령어 `kubectl get namespaces`{{exec}}

<br><br><br>

이번에는 Pod목록을 조회하는데, `--all-namespaces`옵션을 추가해볼까요?
```bash
controlplane $ kubectl get pods --all-namespaces --output wide
NAMESPACE            NAME                                       READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
kube-system          calico-kube-controllers-5f94594857-9rx4h   1/1     Running   2          5d12h   192.168.0.5   controlplane   <none>           <none>
kube-system          canal-4wd75                                2/2     Running   0          20m     172.30.1.2    controlplane   <none>           <none>
kube-system          canal-mw5k9                                2/2     Running   0          20m     172.30.2.2    node01         <none>           <none>
kube-system          coredns-68dc769db8-66xfw                   1/1     Running   0          5d12h   192.168.0.7   controlplane   <none>           <none>
kube-system          coredns-68dc769db8-bl7xg                   1/1     Running   0          5d12h   192.168.1.2   node01         <none>           <none>
kube-system          etcd-controlplane                          1/1     Running   0          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-system          kube-apiserver-controlplane                1/1     Running   2          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-system          kube-controller-manager-controlplane       1/1     Running   2          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-system          kube-proxy-2cqqp                           1/1     Running   0          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-system          kube-proxy-84r66                           1/1     Running   0          5d12h   172.30.2.2    node01         <none>           <none>
kube-system          kube-scheduler-controlplane                1/1     Running   2          5d12h   172.30.1.2    controlplane   <none>           <none>
local-path-storage   local-path-provisioner-8bc8875b-7f4nb      1/1     Running   0          5d12h   192.168.0.6   controlplane   <none>           <none>
```

> 💻 명령어 `kubectl get pods --all-namespaces --output wide`{{exec}}

<br><br><br>

시스템이 사용하는 Pod들을 보려면 **kube-system** 네임스페이스를 보면 됩니다.
```bash
controlplane $ kubectl get pods --namespace kube-system --output wide
NAME                                       READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
calico-kube-controllers-5f94594857-9rx4h   1/1     Running   2          5d12h   192.168.0.5   controlplane   <none>           <none>
canal-4wd75                                2/2     Running   0          21m     172.30.1.2    controlplane   <none>           <none>
canal-mw5k9                                2/2     Running   0          21m     172.30.2.2    node01         <none>           <none>
coredns-68dc769db8-66xfw                   1/1     Running   0          5d12h   192.168.0.7   controlplane   <none>           <none>
coredns-68dc769db8-bl7xg                   1/1     Running   0          5d12h   192.168.1.2   node01         <none>           <none>
etcd-controlplane                          1/1     Running   0          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-apiserver-controlplane                1/1     Running   2          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-controller-manager-controlplane       1/1     Running   2          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-proxy-2cqqp                           1/1     Running   0          5d12h   172.30.1.2    controlplane   <none>           <none>
kube-proxy-84r66                           1/1     Running   0          5d12h   172.30.2.2    node01         <none>           <none>
kube-scheduler-controlplane                1/1     Running   2          5d12h   172.30.1.2    controlplane   <none>           <none>
```

> 💻 명령어 `kubectl get pods --namespace kube-system --output wide`{{exec}}

<br><br><br>

그 중에 하나, **kube-scheduler**를 좀 더 자세히 볼까요?
정보를 yaml형태로 볼 수 도 있구요. ( `--output yaml` 옵션을 사용 )
```bash
controlplane $ kubectl get pod kube-scheduler-controlplane --namespace kube-system --output yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/config.hash: 4f7b02e4a5e4f9f2bc525ba7b009fc26
    kubernetes.io/config.mirror: 4f7b02e4a5e4f9f2bc525ba7b009fc26
    kubernetes.io/config.seen: "2023-02-23T12:53:36.385849439Z"
    kubernetes.io/config.source: file
  creationTimestamp: "2023-02-23T12:54:08Z"
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler-controlplane
  namespace: kube-system
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node
    name: controlplane
    uid: cb2c5b95-fd92-4813-85cc-6e16bea6d16b
  resourceVersion: "1338"
  uid: dcae7d77-ab4c-40ec-b646-fccb750925a3
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: registry.k8s.io/kube-scheduler:v1.26.1
    imagePullPolicy: IfNotPresent
...생략...
```

> 💻 명령어 `kubectl get pod kube-scheduler-controlplane --namespace kube-system --output yaml`{{exec}}  

<br><br><br>

`kubectl describe` 명령으로 오브젝트의 자세한 정보를 조회할 수도 있습니다.
```bash
controlplane $ kubectl describe pod kube-scheduler-controlplane --namespace kube-system
Name:                 kube-scheduler-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/172.30.1.2
Start Time:           Thu, 23 Feb 2023 12:52:31 +0000
Labels:               component=kube-scheduler
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: 4f7b02e4a5e4f9f2bc525ba7b009fc26
                      kubernetes.io/config.mirror: 4f7b02e4a5e4f9f2bc525ba7b009fc26
                      kubernetes.io/config.seen: 2023-02-23T12:53:36.385849439Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   172.30.1.2
IPs:
  IP:           172.30.1.2
Controlled By:  Node/controlplane
Containers:
  kube-scheduler:
    Container ID:  containerd://586305d1c308b7241812bb31e4def19dacaa0475089a0d31c41d6516fe069e77
    Image:         registry.k8s.io/kube-scheduler:v1.26.1
    Image ID:      registry.k8s.io/kube-scheduler@sha256:af0292c2c4fa6d09ee8544445eef373c1c280113cb6c968398a37da3744c41e4
    ... 생략 ...
```

> 💻 명령어 `kubectl describe pod kube-scheduler-controlplane --namespace kube-system`{{exec}}

<br><br><br>

Pod의 로그를 보려면 아래와 같이 하시면 됩니다.
```bash
controlplane $ kubectl logs -n kube-system kube-scheduler-controlplane
I0223 13:06:26.384785       1 serving.go:348] Generated self-signed cert in-memory
W0301 01:29:11.845313       1 requestheader_controller.go:193] Unable to get configmap/extension-apiserver-authentication in kube-system.  Usually fixed by 'kubectl create rolebinding -n kube-system ROLEBINDING_NAME --role=extension-apiserver-authentication-reader --serviceaccount=YOUR_NS:YOUR_SA'
W0301 01:29:11.845361       1 authentication.go:349] Error looking up in-cluster authentication configuration: configmaps "extension-apiserver-authentication" is forbidden: User "system:kube-scheduler" cannot get resource "configmaps" in API group "" in the namespace "kube-system"
W0301 01:29:11.845371       1 authentication.go:350] Continuing without authentication configuration. This may treat all requests as anonymous.
W0301 01:29:11.845379       1 authentication.go:351] To require authentication configuration lookup to succeed, set --authentication-tolerate-lookup-failure=false
I0301 01:29:12.116972       1 server.go:152] "Starting Kubernetes Scheduler" version="v1.26.1"
I0301 01:29:12.116994       1 server.go:154] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
I0301 01:29:12.147576       1 secure_serving.go:210] Serving securely on 127.0.0.1:10259
I0301 01:29:12.152895       1 configmap_cafile_content.go:202] "Starting controller" name="client-ca::kube-system::extension-apiserver-authentication::client-ca-file"
I0301 01:29:12.152930       1 shared_informer.go:273] Waiting for caches to sync for client-ca::kube-system::extension-apiserver-authentication::client-ca-file
I0301 01:29:12.155811       1 tlsconfig.go:240] "Starting DynamicServingCertificateController"
I0301 01:29:12.452958       1 shared_informer.go:280] Caches are synced for client-ca::kube-system::extension-apiserver-authentication::client-ca-file
I0301 01:29:12.453786       1 leaderelection.go:248] attempting to acquire leader lease kube-system/kube-scheduler...
I0301 01:29:28.751836       1 leaderelection.go:258] successfully acquired lease kube-system/kube-scheduler

... 생략 ...
```

> 💻 명령어 `kubectl logs -n kube-system kube-scheduler-controlplane`{{exec}}

<br>

여기까지, 기본적인 kubectl 명령어들을 알아보았습니다. 더 많은 내용은 차차 알아볼게요~ ٩(ˊᗜˋ*)و