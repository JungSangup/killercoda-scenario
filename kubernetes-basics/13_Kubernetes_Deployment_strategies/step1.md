이번 실습은 Deployment의 업데이트 방법 두 가지를 비교해보는 실습입니다.

그리고, 이번 실습은 Terminal이 두 개 필요합니다.  
미리 준비해주세요.  

탭을 하나 더 열어서 준비하거나 [tmux](https://github.com/tmux/tmux/wiki)등을 활용할 수도 있습니다.
> 탭 추가 방법 : 터미널 화면 좌측 상단의 **Tab 1** 옆의 **+**를 클릭하세요.

<br>

첫 번째는 **Recreate** 입니다.  
말 그대로 **다시 생성**하는 방법입니다.  
기존에 서비스되고 있던 Pod들을 모두 정지하고, 새로운 Pod를 실행하는거죠.

Deployment 의 `.spec.strategy`를 아래와 같이 지정하면 됩니다.
```yaml
spec:
  strategy:
    type: Recreate
```

<br><br><br>

아래는 **Recreate** 방식을 적용한 **Deployment** 예제파일입니다.
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
    type: Recreate
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
> 파일명은 **nginx-recreate.yaml**로 합니다.

<br><br><br>

그리고, 다음과 같이 **Deployment**를 생성합니다.
```bash
controlplane $ kubectl apply -f nginx-recreate.yaml
deployment.apps/nginx-deployment created
```

> 💻 명령어(Tab1) `kubectl apply -f nginx-recreate.yaml`{{exec}}

<br><br><br>

그리고, 생성된 Object들도 확인해 보겠습니다.
```bash
controlplane $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-6859c5bd49-56dhk   1/1     Running   0          4m10s
pod/nginx-deployment-6859c5bd49-cp97v   1/1     Running   0          4m10s
pod/nginx-deployment-6859c5bd49-mk7fk   1/1     Running   0          4m10s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5d22h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           4m10s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-6859c5bd49   3         3         3       4m10s
```

> 💻 명령어(Tab1) `kubectl get all`{{exec}}

Spec에 정의된 대로 세 개의 Nginx Pod가 생성되어 있습니다.

<br><br><br>

우리가 생성한 nginx 버젼을 알아볼까요?
```bash
controlplane $ kubectl describe deployment nginx-deployment | grep -i image
    Image:        nginx:1.18
```

> 💻 명령어(Tab1) `kubectl describe deployment nginx-deployment | grep -i image`{{exec}}

사용된 Image는 `nginx:1.18` 입니다.


<br><br><br>

이제 버젼을 변경하려고 합니다.  
앞에서 배웠으니 **선언형**(**Declarative**)으로 해볼게요.

yaml파일의 버젼부분을 수정합니다. (`image: nginx:1.18` -> `image: nginx:1.19` , [sed](https://www.gnu.org/software/sed/) 명령어 사용)
```bash
controlplane $ sed -i 's/image: nginx:1.18/image: nginx:1.19/g' nginx-recreate.yaml
```

> 💻 명령어(Tab1) `sed -i 's/image: nginx:1.18/image: nginx:1.19/g' nginx-recreate.yaml`{{exec}}

<br><br><br>

그리고, Pod들이 어떻게 변하는지 살펴보기 위해서 다음 명령어를 실행해주세요.  
이 명령어는 **두 번째 Terminal**에서 실행해주세요.
```bash
controlplane $ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6859c5bd49-56dhk   1/1     Running   0          5m39s
nginx-deployment-6859c5bd49-cp97v   1/1     Running   0          5m39s
nginx-deployment-6859c5bd49-mk7fk   1/1     Running   0          5m39s

```

> 💻 명령어(Tab2) `kubectl get pods --watch`{{exec}}  
> `--watch` 는 앞의 명령어를 실행한 후 변경(Change)사항을 지속적으로 보여주는 Flag입니다.  
> Watch를 멈추려면 Ctrl + c 를 입력합니다.

<br><br><br>

이제 다시 첫 번째 Terminal에서 아래와 같이 변경사항을 적용합니다.

그리고 변경된 yaml파일을 이용해서 업데이트를 합니다.
```bash
controlplane $ kubectl apply -f nginx-recreate.yaml
deployment.apps/nginx-deployment configured
```

> 💻 명령어(Tab1) `kubectl apply -f nginx-recreate.yaml`{{exec}}

<br><br><br>

두 번째 Terminal에서 어떤 일이 일어나는지 유심히 보세요. 아마도, 있던 Pod들이 모두 삭제되고 새로운 Pod들이 생길거예요.
```bash
controlplane $ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6859c5bd49-56dhk   1/1     Running   0          5m39s
nginx-deployment-6859c5bd49-cp97v   1/1     Running   0          5m39s
nginx-deployment-6859c5bd49-mk7fk   1/1     Running   0          5m39s

nginx-deployment-6859c5bd49-mk7fk   1/1     Terminating   0          6m4s
nginx-deployment-6859c5bd49-56dhk   1/1     Terminating   0          6m4s
nginx-deployment-6859c5bd49-cp97v   1/1     Terminating   0          6m4s
nginx-deployment-6859c5bd49-mk7fk   1/1     Terminating   0          6m5s
nginx-deployment-6859c5bd49-cp97v   1/1     Terminating   0          6m5s
nginx-deployment-6859c5bd49-56dhk   1/1     Terminating   0          6m5s
nginx-deployment-6859c5bd49-56dhk   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-56dhk   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-56dhk   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-mk7fk   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-mk7fk   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-mk7fk   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-cp97v   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-cp97v   0/1     Terminating   0          6m6s
nginx-deployment-6859c5bd49-cp97v   0/1     Terminating   0          6m6s
nginx-deployment-76c6c66746-4lrfz   0/1     Pending       0          0s
nginx-deployment-76c6c66746-4lrfz   0/1     Pending       0          0s
nginx-deployment-76c6c66746-24mvf   0/1     Pending       0          0s
nginx-deployment-76c6c66746-554kk   0/1     Pending       0          0s
nginx-deployment-76c6c66746-554kk   0/1     Pending       0          0s
nginx-deployment-76c6c66746-24mvf   0/1     Pending       0          0s
nginx-deployment-76c6c66746-4lrfz   0/1     Pending       0          1s
nginx-deployment-76c6c66746-4lrfz   0/1     ContainerCreating   0          1s
nginx-deployment-76c6c66746-554kk   0/1     Pending             0          1s
nginx-deployment-76c6c66746-24mvf   0/1     Pending             0          1s
nginx-deployment-76c6c66746-554kk   0/1     ContainerCreating   0          1s
nginx-deployment-76c6c66746-24mvf   0/1     ContainerCreating   0          2s
nginx-deployment-76c6c66746-4lrfz   1/1     Running             0          6s
nginx-deployment-76c6c66746-554kk   1/1     Running             0          7s
nginx-deployment-76c6c66746-24mvf   1/1     Running             0          7s
```

<br><br><br>

Deployment에 어떤 변화가 생겼나 볼까요?  
첫 번째 터미널에서 실행해주세요.
```bash
controlplane $ kubectl describe deployment nginx-deployment | grep -i image
    Image:        nginx:1.19
```

> 💻 명령어(Tab1) `kubectl describe deployment nginx-deployment | grep -i image`{{exec}}

<br><br><br>

그리고, 새로 생성된 Pod도 한번 보구요.
```bash
controlplane $ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-76c6c66746-24mvf   1/1     Running   0          2m5s
nginx-deployment-76c6c66746-4lrfz   1/1     Running   0          2m5s
nginx-deployment-76c6c66746-554kk   1/1     Running   0          2m5s
controlplane $ kubectl describe pod nginx-deployment-76c6c66746-24mvf | grep -i image
    Image:          nginx:1.19
    Image ID:       docker.io/library/nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
  Normal  Pulling    2m58s  kubelet            Pulling image "nginx:1.19"
  Normal  Pulled     2m53s  kubelet            Successfully pulled image "nginx:1.19" in 398.759679ms (4.714616498s including waiting)
```

> 💻 명령어(Tab1) `kubectl get pods`{{exec}}  
> 💻 명령어(Tab1) `kubectl describe pod [POD-NAME] | grep -i image`{{copy}}  
> [POD-NAME] 에는 앞에서 조회된 POD 중 하나의 이름을 넣어주세요.

어떤가요? 업데이트가 잘 이루어졌나요?

이제 두 번째 터미널은 Ctrl + c 를 눌러 Watch를 멈추겠습니다.