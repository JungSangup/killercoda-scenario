이번 실습은 Deployment의 업데이트 방법 두 가지를 비교해보는 실습입니다.

그리고, 이번 실습은 Terminal이 두 개 필요합니다.  
미리 준비해주세요.  

Tab을 추가할 수 있는 툴(e.g. Windows Terminal, iTerm, MobaXTerm etc.)은 탭을 하나 더 열어서 준비하고, 아닌경우 [tmux](https://github.com/tmux/tmux/wiki)등을 활용할 수도 있습니다.

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
ubuntu@ip-172-31-23-60:~$ kubectl apply -f nginx-recreate.yaml
deployment.apps/nginx-deployment created
```

> 💻 명령어
>```bash
>kubectl apply -f nginx-recreate.yaml
>```

<br><br><br>

그리고, 생성된 Object들도 확인해 보겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-85fc747956-4jgxq   1/1     Running   0          15s
pod/nginx-deployment-85fc747956-vzsbg   1/1     Running   0          15s
pod/nginx-deployment-85fc747956-x6zmj   1/1     Running   0          15s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d15h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           17s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-85fc747956   3         3         3       16s
```

> 💻 명령어
>```bash
>kubectl get all
>```

Spec에 정의된 대로 세 개의 Nginx Pod가 생성되어 있습니다.

<br><br><br>

우리가 생성한 nginx 버젼을 알아볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl describe deployment nginx-deployment | grep -i image
    Image:        nginx:1.18
```

> 💻 명령어
>```bash
>kubectl describe deployment nginx-deployment | grep -i image
>```

사용된 Image는 `nginx:1.18` 입니다.


<br><br><br>

이제 버젼을 변경하려고 합니다.  
앞에서 배웠으니 **선언형**(**Declarative**)으로 해볼게요.

yaml파일의 버젼부분을 수정합니다. (`image: nginx:1.18` -> `image: nginx:1.19` , [sed](https://www.gnu.org/software/sed/) 명령어 사용)
```bash
ubuntu@ip-172-31-23-60:~$ sed -i 's/image: nginx:1.18/image: nginx:1.19/g' nginx-recreate.yaml
```

> 💻 명령어
>```bash
>sed -i 's/image: nginx:1.18/image: nginx:1.19/g' nginx-recreate.yaml
>```

<br><br><br>

그리고, Pod들이 어떻게 변하는지 살펴보기 위해서 다음 명령어를 실행해주세요.  
이 명령어는 **두 번째 Terminal**에서 실행해주세요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85fc747956-4jgxq   1/1     Running   0          4m23s
nginx-deployment-85fc747956-vzsbg   1/1     Running   0          4m23s
nginx-deployment-85fc747956-x6zmj   1/1     Running   0          4m23s

```

> 💻 명령어
>```bash
>kubectl get pods --watch
>```
> `--watch` 는 앞의 명령어를 실행한 후 변경(Change)사항을 지속적으로 보여주는 Flag입니다.
> Watch를 멈추려면 Ctrl + c 를 입력합니다.

<br><br><br>

이제 다시 첫 번째 Terminal에서 아래와 같이 변경사항을 적용합니다.

그리고 변경된 yaml파일을 이용해서 업데이트를 합니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl apply -f nginx-recreate.yaml
deployment.apps/nginx-deployment configured
```

> 💻 명령어
>```bash
>kubectl apply -f nginx-recreate.yaml
>```

<br><br><br>

두 번째 Terminal에서 어떤 일이 일어나는지 유심히 보세요. 아마도, 있던 Pod들이 모두 삭제되고 새로운 Pod들이 생길거예요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85fc747956-4jgxq   1/1     Running   0          4m23s
nginx-deployment-85fc747956-vzsbg   1/1     Running   0          4m23s
nginx-deployment-85fc747956-x6zmj   1/1     Running   0          4m23s

nginx-deployment-85fc747956-vzsbg   1/1     Terminating   0          4m58s
nginx-deployment-85fc747956-4jgxq   1/1     Terminating   0          4m58s
nginx-deployment-85fc747956-x6zmj   1/1     Terminating   0          4m58s
nginx-deployment-85fc747956-4jgxq   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-vzsbg   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-x6zmj   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-4jgxq   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-4jgxq   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-vzsbg   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-vzsbg   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-x6zmj   0/1     Terminating   0          4m59s
nginx-deployment-85fc747956-x6zmj   0/1     Terminating   0          4m59s
nginx-deployment-7dd48c557f-6b9b8   0/1     Pending       0          0s
nginx-deployment-7dd48c557f-6b9b8   0/1     Pending       0          0s
nginx-deployment-7dd48c557f-k6czs   0/1     Pending       0          0s
nginx-deployment-7dd48c557f-s9msd   0/1     Pending       0          0s
nginx-deployment-7dd48c557f-k6czs   0/1     Pending       0          0s
nginx-deployment-7dd48c557f-s9msd   0/1     Pending       0          0s
nginx-deployment-7dd48c557f-6b9b8   0/1     ContainerCreating   0          2s
nginx-deployment-7dd48c557f-k6czs   0/1     ContainerCreating   0          2s
nginx-deployment-7dd48c557f-s9msd   0/1     ContainerCreating   0          2s

nginx-deployment-7dd48c557f-6b9b8   1/1     Running             0          6s
nginx-deployment-7dd48c557f-s9msd   1/1     Running             0          6s
nginx-deployment-7dd48c557f-k6czs   1/1     Running             0          6s
```

<br><br><br>

Deployment에 어떤 변화가 생겼나 볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl describe deployment nginx-deployment | grep -i image
    Image:        nginx:1.19
```

> 💻 명령어
>```bash
>kubectl describe deployment nginx-deployment | grep -i image
>```

<br><br><br>

그리고, 새로 생성된 Pod도 한번 보구요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dd48c557f-6b9b8   1/1     Running   0          92s
nginx-deployment-7dd48c557f-k6czs   1/1     Running   0          92s
nginx-deployment-7dd48c557f-s9msd   1/1     Running   0          92s
ubuntu@ip-172-31-23-60:~$ kubectl describe pod nginx-deployment-7dd48c557f-6b9b8 | grep -i image
    Image:          nginx:1.19
    Image ID:       docker-pullable://nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
  Normal  Pulling    115s  kubelet            Pulling image "nginx:1.19"
  Normal  Pulled     112s  kubelet            Successfully pulled image "nginx:1.19" in 3.453818458s
```

> 💻 명령어
>```bash
>kubectl get pods
>```
>```bash
>kubectl describe pod [POD-NAME] | grep -i image
>```
> [POD-NAME] 에는 앞에서 조회된 POD 중 하나의 이름을 넣어주세요.

어떤가요? 업데이트가 잘 이루어졌나요?

이제 두 번째 터미널은 Ctrl + c 를 눌러 Watch를 멈추겠습니다.