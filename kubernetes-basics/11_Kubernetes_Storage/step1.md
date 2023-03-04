도커에서와 마찬가지로 컨테이너의 데이터 저장을 위해서 Volume을 생성해 보겠습니다.  
이번 실습은 다양한 방법 중에서

- Storage class를 이용한 [Dynamic Volume Provisioning](https://kubernetes.io/ko/docs/concepts/storage/dynamic-provisioning/) 을 적용하고,
- [local](https://kubernetes.io/ko/docs/concepts/storage/volumes/#local) 유형의 Volume을 사용  

해서 진행해 보겠습니다.

먼저 우리 환경이 준비가 되어있는지 확인해볼게요.  
우리 Cluster에서 사용가능한 Storageclass가 있는지 확인합니다.
```bash
controlplane $ kubectl get storageclasses
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  8d
```

> 💻 명령어 `kubectl get storageclasses`{{exec}}

<br>

Killercoda는 기본적으로 위와같은 StorageClass가 있습니다.  
간단히 테스트해볼 수 있도록, local 타입의 Volume을 만들 수 있습니다.

<br><br><br>

이제 **PersistentVolumeClaim**(**PVC**)를 만들어볼게요.  
아래와 같은 파일을 준비합니다.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
> 파일명은 **nginx-pvc.yaml**로 합니다.

<br>

그리고, 아래와 같이 적용합니다.
```bash
controlplane $ kubectl apply -f nginx-pvc.yaml
persistentvolumeclaim/nginx-pvc created
```

> 💻 명령어 `kubectl apply -f nginx-pvc.yaml`{{exec}}

<br><br><br>

만들어진 K8s 리소스들을 볼까요?  
먼저 PVC를 확인해볼게요.
```bash
controlplane $ kubectl get persistentvolumeclaims
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-pvc   Pending                                      local-path     29s
```
> 💻 명령어 `kubectl get persistentvolumeclaims`{{exec}}  
>또는  
> 💻 명령어 `kubectl get pvc`{{exec}}

결과를 보니 STATUS는 **Pending**이네요.  

<br><br><br>

그럼, 이번에는 `kubectl describe` 명령어를 실행해 볼까요?
```bash
controlplane $ kubectl describe pvc nginx-pvc
Name:          nginx-pvc
Namespace:     default
StorageClass:  local-path
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age                  From                         Message
  ----    ------                ----                 ----                         -------
  Normal  WaitForFirstConsumer  4s (x17 over 3m59s)  persistentvolume-controller  waiting for first consumer to be created before binding
```

> 💻 명령어 `kubectl describe pvc nginx-pvc`{{exec}}  

**Events**를 보니 **consumer**를 기다리고 있네요.  
그럼, consumer를 생성하고 다시 볼까요?