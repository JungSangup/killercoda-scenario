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

> 💻 명령어
>```bash
>kubectl get storageclasses
>```

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
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-pvc   Bound    nginx-pv   3Gi        RWO            manual         24s
```
> 💻 명령어 `kubectl get persistentvolumeclaims`{{exec}}  
>또는  
> 💻 명령어 `kubectl get pvc`{{exec}}

결과를 보니 **VOLUME(nginx-pv)** 도 보이고, STATUS는 **Bound**네요.

<br><br><br>

그럼, 이번에는 **PersistentVolume**(**PV**)을 볼까요?
```bash
controlplane $ kubectl get persistentvolume
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
nginx-pv   3Gi        RWO            Retain           Bound    default/nginx-pvc   manual                  41s
```

> 💻 명령어 `kubectl get persistentvolume`{{exec}}  
>또는  
> 💻 명령어 `kubectl get pv`{{exec}}

<br><br><br>

PV를 좀 더 자세히 볼까요?
```bash
controlplane $ kubectl describe persistentvolume nginx-pv
Name:            nginx-pv
Labels:          type=local
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Bound
Claim:           default/nginx-pvc
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        3Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt/data
    HostPathType:  
Events:            <none>
```
> 💻 명령어 `kubectl describe persistentvolume [PV-NAME]`{{copy}}  
>또는  
> 💻 명령어 `kubectl describe pv [PV-NAME]`{{exec}}  
> [PV-NAME] 에는 앞에서 만들어진 PV의 Name을 넣어주세요.

**Source**아래 내용을 보시면 어디에 Volume영역이 할당되었는지 알 수 있습니다.  
위의 경우는 **HostPath**타입을 이용했고, **/mnt/data**를 Volume의 위치로 사용하고 있습니다.