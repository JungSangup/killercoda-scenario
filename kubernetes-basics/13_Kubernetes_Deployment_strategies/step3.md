이번엔 **RollingUpdate** 입니다.  
기존에 서비스되고 있던 Pod들을 새로운 Pod로 조금씩(N개씩) 업데이트 하는 방식입니다.

Deployment 의 `.spec.strategy`를 아래와 같이 지정하면 됩니다.
```yaml
spec:
  strategy:
    type: RollingUpdate
```

<br>

아래는 **RollingUpdate** 방식을 적용한 **Deployment** 예제파일입니다.  
실습을 위해 아래 파일을 만들어주세요.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: my-nginx
    tier: frontend
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
      name: my-nginx
    spec:
      containers:
      - image: nginx:1.18
        name: my-nginx
        ports:
        - containerPort: 80
```
> 파일명은 **nginx-rollingupdate.yaml**로 합니다.

<br><br><br>

다음과 같이 Deployment를 생성합니다.
```bash
controlplane $ kubectl apply -f nginx-rollingupdate.yaml
deployment.apps/nginx-deployment created
```

> 💻 명령어(Tab1) `kubectl apply -f nginx-rollingupdate.yaml`{{exec}}

<br><br><br>

그리고, 생성된 Object들도 확인해 보겠습니다.
```bash
controlplane $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-6859c5bd49-8ngpf   1/1     Running   0          3m3s
pod/nginx-deployment-6859c5bd49-xclhc   1/1     Running   0          3m3s
pod/nginx-deployment-6859c5bd49-zxw4f   1/1     Running   0          3m3s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5d23h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3m3s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-6859c5bd49   3         3         3       3m3s
```

> 💻 명령어(Tab1) `kubectl get all`{{exec}}

<br><br><br>

생성된 Deployment의 정보를 보고 현재 실행된 이미지를 확인해봅니다.
```bash
controlplane $ kubectl describe deployment nginx-deployment | grep -i image
    Image:        nginx:1.18
```

> 💻 명령어(Tab1) `kubectl describe deployment nginx-deployment | grep -i image`{{exec}}

사용된 Image는 `nginx:1.18` 입니다.

<br><br><br>

업데이트를 위해서 Deployment yaml파일에서 버젼을 변경하구요.
```bash
controlplane $ sed -i 's/image: nginx:1.18/image: nginx:1.19/g' nginx-rollingupdate.yaml
```

> 💻 명령어(Tab1) `sed -i 's/image: nginx:1.18/image: nginx:1.19/g' nginx-rollingupdate.yaml`{{exec}}

<br><br><br>

**두 번째 Terminal**에는 확인 할 명령어를 실행한 후에
```bash
controlplane $ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6859c5bd49-8ngpf   1/1     Running   0          4m7s
nginx-deployment-6859c5bd49-xclhc   1/1     Running   0          4m7s
nginx-deployment-6859c5bd49-zxw4f   1/1     Running   0          4m7s

```

> 💻 명령어(Tab2) `kubectl get pods --watch`{{exec}}

<br><br><br>

**첫 번재 Terminal**에서 업데이트를 합니다.
```bash
controlplane $ kubectl apply -f nginx-rollingupdate.yaml
deployment.apps/nginx-deployment configured
```

> 💻 명령어(Tab1) `kubectl apply -f nginx-rollingupdate.yaml`{{exec}}

<br><br><br>

**두 번째 Terminal**은 아래와 비슷한 걸 볼 수 있을겁니다. **Recreate**때와는 달리 Pod들이 순차적으로 변경되는 걸 볼 수 있습니다.
```bash
controlplane $ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6859c5bd49-8ngpf   1/1     Running   0          4m7s
nginx-deployment-6859c5bd49-xclhc   1/1     Running   0          4m7s
nginx-deployment-6859c5bd49-zxw4f   1/1     Running   0          4m7s

nginx-deployment-76c6c66746-pmndf   0/1     Pending   0          0s
nginx-deployment-76c6c66746-pmndf   0/1     Pending   0          0s
nginx-deployment-76c6c66746-pmndf   0/1     ContainerCreating   0          0s
nginx-deployment-76c6c66746-pmndf   0/1     ContainerCreating   0          0s
nginx-deployment-76c6c66746-pmndf   1/1     Running             0          0s
nginx-deployment-6859c5bd49-zxw4f   1/1     Terminating         0          4m27s
nginx-deployment-76c6c66746-9qrnh   0/1     Pending             0          0s
nginx-deployment-76c6c66746-9qrnh   0/1     Pending             0          1s
nginx-deployment-76c6c66746-9qrnh   0/1     ContainerCreating   0          1s
nginx-deployment-6859c5bd49-zxw4f   1/1     Terminating         0          4m28s
nginx-deployment-76c6c66746-9qrnh   0/1     ContainerCreating   0          1s
nginx-deployment-76c6c66746-9qrnh   1/1     Running             0          1s
nginx-deployment-6859c5bd49-xclhc   1/1     Terminating         0          4m28s
nginx-deployment-6859c5bd49-zxw4f   0/1     Terminating         0          4m28s
nginx-deployment-76c6c66746-k4n5q   0/1     Pending             0          0s
nginx-deployment-76c6c66746-k4n5q   0/1     Pending             0          0s
nginx-deployment-6859c5bd49-zxw4f   0/1     Terminating         0          4m29s
nginx-deployment-6859c5bd49-zxw4f   0/1     Terminating         0          4m29s
nginx-deployment-76c6c66746-k4n5q   0/1     ContainerCreating   0          1s
nginx-deployment-6859c5bd49-xclhc   1/1     Terminating         0          4m29s
nginx-deployment-76c6c66746-k4n5q   0/1     ContainerCreating   0          1s
nginx-deployment-6859c5bd49-xclhc   0/1     Terminating         0          4m29s
nginx-deployment-6859c5bd49-xclhc   0/1     Terminating         0          4m29s
nginx-deployment-6859c5bd49-xclhc   0/1     Terminating         0          4m29s
nginx-deployment-76c6c66746-k4n5q   1/1     Running             0          2s
nginx-deployment-6859c5bd49-8ngpf   1/1     Terminating         0          4m30s
nginx-deployment-6859c5bd49-8ngpf   1/1     Terminating         0          4m30s
nginx-deployment-6859c5bd49-8ngpf   0/1     Terminating         0          4m30s
nginx-deployment-6859c5bd49-8ngpf   0/1     Terminating         0          4m31s
nginx-deployment-6859c5bd49-8ngpf   0/1     Terminating         0          4m31s
```

<br><br><br>

첫 번째 Terminal에서 아래와 같이 업데이트 이후의 변경사항도 확인해보세요.
```bash
controlplane $ kubectl describe deployment nginx-deployment | grep -i image
    Image:        nginx:1.19
```

> 💻 명령어(Tab1) `kubectl describe deployment nginx-deployment | grep -i image`{{exec}}

<br><br><br>

새로 생성된 Pod의 정보도 확인해봅니다.
```bash
controlplane $ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-76c6c66746-9qrnh   1/1     Running   0          59s
nginx-deployment-76c6c66746-k4n5q   1/1     Running   0          58s
nginx-deployment-76c6c66746-pmndf   1/1     Running   0          59s
controlplane $ kubectl describe pod nginx-deployment-76c6c66746-9qrnh | grep -i image
    Image:          nginx:1.19
    Image ID:       docker.io/library/nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
  Normal  Pulled     77s   kubelet            Container image "nginx:1.19" already present on machine
```

> 💻 명령어(Tab1) `kubectl get pods`{{exec}}  
> 💻 명령어(Tab1) `kubectl describe pod [POD-NAME] | grep -i image`{{copy}}  
> [POD-NAME] 에는 앞에서 조회된 POD 중 하나의 이름을 넣어주세요.

<br><br><br>

앞에서와 마찬가지로 롤백도 해보세요.  
자세한 설명은 생략하고 명령어들만 알려드릴게요.

> 💻 명령어(Tab1) `kubectl rollout history deployment nginx-deployment`{{exec}}  
> 💻 명령어(Tab1) `kubectl rollout undo deployment nginx-deployment --to-revision=1`{{exec}}  
> 💻 명령어(Tab1) `kubectl get pods`{{exec}}  
> 💻 명령어(Tab1) `kubectl describe pod [POD-NAME] | grep -i image`{{copy}}  

<br><br><br>

다 해보셨으면 다음 실습을 위해 Object들을 삭제해주세요.  
```bash
controlplane $ kubectl delete -f nginx-rollingupdate.yaml
deployment.apps "nginx-deployment" deleted
```

> 💻 명령어(Tab1) `kubectl delete -f nginx-rollingupdate.yaml`{{exec}}

두 번째 터미널은 Ctrl + c 를 눌러 Watch를 멈추겠습니다.

끝~