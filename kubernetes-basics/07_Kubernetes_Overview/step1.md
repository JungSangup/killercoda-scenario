기본적인 쿠버네티스 명령어들에 대해 알아보겠습니다.  
제일먼저 도움말을 볼까요?

```bash
controlplane $ kubectl --help
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create        Create a resource from a file or from stdin
  expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run           Run a particular image on the cluster
  set           Set specific features on objects

Basic Commands (Intermediate):
  explain       Get documentation for a resource
  get           Display one or many resources
  edit          Edit a resource on the server
  delete        Delete resources by file names, stdin, resources and names, or by resources and label selector
... 생략 ...
Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

> **명령어** : `kubectl --help`{{exec}}  

여러가지 명령어들을 볼 수 있고, 사용법을 알 수 있습니다.

---

몇 가지 명령어들을 알아볼까요?

버젼을 알아보려면,

```bash
controlplane $ kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:25:17Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:19:12Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/amd64"}
```

> **명령어** : `kubectl version`{{exec}}  

쿠버네티스 클러스터 정보를 확인하려면,

```bash
controlplane $ kubectl cluster-info
Kubernetes control plane is running at https://172.30.1.2:6443
CoreDNS is running at https://172.30.1.2:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

> **명령어** : `kubectl cluster-info`{{exec}}  

우리 클러스터의 노드목록은 아래 명령어로 알아볼 수 있습니다.

```bash
controlplane $ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   70d   v1.24.0
node01         Ready    <none>          70d   v1.24.0
```

> **명령어** : `kubectl get nodes`{{exec}}  

---

`--output wide`옵션(또는, `-o wide`)을 주면 더 많은 정보를 보여줍니다.

```bash
controlplane $ kubectl get nodes --output wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   70d   v1.24.0   172.30.1.2    <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic   containerd://1.5.9
node01         Ready    <none>          70d   v1.24.0   172.30.2.2    <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic   containerd://1.5.9
```

> **명령어** : `kubectl get nodes --output wide`{{exec}}  

help와 유사하게 쿠버네티스 리소스들의 정의와 설명을 보려면 `kubectl explain` 명령을 사용하면 됩니다.

예를들어 POD에 대해 알아보려면 아래와 같이 실행하면 됩니다.

```bash
controlplane $ kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
   ... 생략 ...
```

> **명령어** : `kubectl explain pod`{{exec}}  

---

이제 우리 클러스터에 존재한는 리소스들 중 Pod에 대해 좀 더 자세히 알아볼까요?

먼저 현재 존재하는 POD 목록은 아래와 같이 조회합니다.

```bash
controlplane $ kubectl get pods
No resources found in default namespace.
```

> **명령어** : `kubectl get pods`{{exec}}  

음. 아무것도 없군요...  
지금은 **default 네임스페이스**에서 조회를 한 경우입니다.

`--namespace`로 네임스페이스를 지정하지 않으면 **default 네임스페이스**를 기본으로 합니다.

다른 네임스페이스는 뭐가 있을까요?  
네임스페이스를 보려면 아래 명령어를 사용하면 됩니다.

```bash
controlplane $ kubectl get namespaces
NAME              STATUS   AGE
default           Active   70d
kube-node-lease   Active   70d
kube-public       Active   70d
kube-system       Active   70d
```

> **명령어** : `kubectl get namespaces`{{exec}}  

---

이번에는 Pod목록을 조회하는데, --all-namespaces옵션을 추가해볼까요?

```bash
controlplane $ kubectl get pods --all-namespaces --output wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS      AGE   IP            NODE           NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-54589b89dc-kfbb5   1/1     Running   0             70d   192.168.1.3   node01         <none>           <none>
kube-system   canal-6p28v                                2/2     Running   1 (13m ago)   70d   172.30.2.2    node01         <none>           <none>
kube-system   canal-8hb2n                                2/2     Running   0             70d   172.30.1.2    controlplane   <none>           <none>
kube-system   coredns-7f6d6547b-2rgsz                    1/1     Running   0             70d   192.168.1.2   node01         <none>           <none>
kube-system   coredns-7f6d6547b-rjr4g                    1/1     Running   0             70d   192.168.0.5   controlplane   <none>           <none>
kube-system   etcd-controlplane                          1/1     Running   0             70d   172.30.1.2    controlplane   <none>           <none>
kube-system   kube-apiserver-controlplane                1/1     Running   2             70d   172.30.1.2    controlplane   <none>           <none>
kube-system   kube-controller-manager-controlplane       1/1     Running   2             70d   172.30.1.2    controlplane   <none>           <none>
kube-system   kube-proxy-vllmz                           1/1     Running   0             70d   172.30.2.2    node01         <none>           <none>
kube-system   kube-proxy-xskb9                           1/1     Running   0             70d   172.30.1.2    controlplane   <none>           <none>
kube-system   kube-scheduler-controlplane                1/1     Running   2             70d   172.30.1.2    controlplane   <none>           <none>
```

> **명령어** : `kubectl get pods --all-namespaces --output wide`{{exec}}  

시스템이 사용하는 Pod들을 보려면 kube-system 네임스페이스를 보면 됩니다.

```bash
controlplane $ kubectl get pods --namespace kube-system --output wide
NAME                                       READY   STATUS    RESTARTS      AGE   IP            NODE           NOMINATED NODE   READINESS GATES
calico-kube-controllers-54589b89dc-kfbb5   1/1     Running   0             70d   192.168.1.3   node01         <none>           <none>
canal-6p28v                                2/2     Running   1 (14m ago)   70d   172.30.2.2    node01         <none>           <none>
canal-8hb2n                                2/2     Running   0             70d   172.30.1.2    controlplane   <none>           <none>
coredns-7f6d6547b-2rgsz                    1/1     Running   0             70d   192.168.1.2   node01         <none>           <none>
coredns-7f6d6547b-rjr4g                    1/1     Running   0             70d   192.168.0.5   controlplane   <none>           <none>
etcd-controlplane                          1/1     Running   0             70d   172.30.1.2    controlplane   <none>           <none>
kube-apiserver-controlplane                1/1     Running   2             70d   172.30.1.2    controlplane   <none>           <none>
kube-controller-manager-controlplane       1/1     Running   2             70d   172.30.1.2    controlplane   <none>           <none>
kube-proxy-vllmz                           1/1     Running   0             70d   172.30.2.2    node01         <none>           <none>
kube-proxy-xskb9                           1/1     Running   0             70d   172.30.1.2    controlplane   <none>           <none>
kube-scheduler-controlplane                1/1     Running   2             70d   172.30.1.2    controlplane   <none>           <none>
```

> **명령어** : `kubectl get pods --namespace kube-system --output wide`{{exec}}  

---

그 중에 하나, kube-scheduler를 좀 더 자세히 볼까요?  
정보를 yaml형태로 볼 수 도 있구요.

```bash
controlplane $ kubectl get pod kube-scheduler-controlplane --namespace kube-system --output yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/config.hash: 6cf82a20eaca89f94f02af803510cf74
    kubernetes.io/config.mirror: 6cf82a20eaca89f94f02af803510cf74
    kubernetes.io/config.seen: "2022-05-08T19:33:37.832380102Z"
    kubernetes.io/config.source: file
    seccomp.security.alpha.kubernetes.io/pod: runtime/default
  creationTimestamp: "2022-05-08T19:34:08Z"
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
    uid: 3f64b139-f902-4afe-ad6a-548a52b64313
  resourceVersion: "1085"
  uid: 64f9e859-1683-409f-8e5a-d38bbe169a65
spec:
  containers:
  ... 생략 ...
```

> **명령어** : `kubectl get pod kube-scheduler-controlplane --namespace kube-system --output yaml`{{exec}}  

---

describe명령으로 자세한 정보를 조회할 수도 있습니다.

```bash
controlplane $ kubectl describe pod kube-scheduler-controlplane --namespace kube-system
Name:                 kube-scheduler-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/172.30.1.2
Start Time:           Sun, 08 May 2022 19:32:32 +0000
Labels:               component=kube-scheduler
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: 6cf82a20eaca89f94f02af803510cf74
                      kubernetes.io/config.mirror: 6cf82a20eaca89f94f02af803510cf74
                      kubernetes.io/config.seen: 2022-05-08T19:33:37.832380102Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   172.30.1.2
IPs:
  IP:           172.30.1.2
Controlled By:  Node/controlplane
Containers:
  kube-scheduler:
    Container ID:  containerd://da924d974bcde2110e676b2e8281971fc6de20b51b3dd59c9cc64ad7143008d8
    Image:         k8s.gcr.io/kube-scheduler:v1.24.0
    Image ID:      k8s.gcr.io/kube-scheduler@sha256:db842a7c431fd51db7e1911f6d1df27a7b6b6963ceda24852b654d2cd535b776
    Port:          <none>
    Host Port:     <none>
    ... 생략 ...
```

> **명령어** : `kubectl describe pod kube-scheduler-controlplane --namespace kube-system`{{exec}}  

---

Pod의 로그를 보려면 아래와 같이 하시면 됩니다.

```bash
controlplane $ kubectl logs -n kube-system kube-scheduler-controlplane
I0718 04:28:50.141123       1 serving.go:348] Generated self-signed cert in-memory
W0718 04:28:53.900885       1 requestheader_controller.go:193] Unable to get configmap/extension-apiserver-authentication in kube-system.  Usually fixed by 'kubectl create rolebinding -n kube-system ROLEBINDING_NAME --role=extension-apiserver-authentication-reader --serviceaccount=YOUR_NS:YOUR_SA'
W0718 04:28:53.900918       1 authentication.go:346] Error looking up in-cluster authentication configuration: configmaps "extension-apiserver-authentication" is forbidden: User "system:kube-scheduler" cannot get resource "configmaps" in API group "" in the namespace "kube-system"
W0718 04:28:53.900925       1 authentication.go:347] Continuing without authentication configuration. This may treat all requests as anonymous.
W0718 04:28:53.900930       1 authentication.go:348] To require authentication configuration lookup to succeed, set --authentication-tolerate-lookup-failure=false
I0718 04:28:54.058506       1 server.go:147] "Starting Kubernetes Scheduler" version="v1.24.0"
I0718 04:28:54.058525       1 server.go:149] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
I0718 04:28:54.059449       1 secure_serving.go:210] Serving securely on 127.0.0.1:10259
I0718 04:28:54.059518       1 configmap_cafile_content.go:202] "Starting controller" name="client-ca::kube-system::extension-apiserver-authentication::client-ca-file"
I0718 04:28:54.059530       1 shared_informer.go:255] Waiting for caches to sync for client-ca::kube-system::extension-apiserver-authentication::client-ca-file
I0718 04:28:54.059540       1 tlsconfig.go:240] "Starting DynamicServingCertificateController"
W0718 04:28:54.127434       1 reflector.go:324] vendor/k8s.io/client-go/informers/factory.go:134: failed to list *v1.Namespace: namespaces is forbidden: User "system:kube-scheduler" cannot list resource "namespaces" in API group "" at the cluster scope
E0718 04:28:54.127465       1 reflector.go:138] vendor/k8s.io/client-go/informers/factory.go:134: Failed to watch *v1.Namespace: failed to list *v1.Namespace: namespaces is forbidden: User "system:kube-scheduler" cannot list resource "namespaces" in API group "" at the cluster scope
W0718 04:28:54.127556       1 reflector.go:324] vendor/k8s.io/client-go/informers/factory.go:134: failed to list *v1.StorageClass: storageclasses.storage.k8s.io is forbidden: User "system:kube-scheduler" cannot list resource "storageclasses" in API group "storage.k8s.io" at the cluster scope

... 생략 ...
```

> **명령어** : `kubectl logs -n kube-system kube-scheduler-controlplane`{{exec}}  

<br>

여기까지, 기본적인 kubectl 명령어들을 알아보았습니다. 더 많은 내용은 차차 알아볼게요~ ٩(ˊᗜˋ*)و

