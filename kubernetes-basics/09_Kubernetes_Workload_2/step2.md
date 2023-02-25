좀 더 나가서, 이번엔 **Deployment** 입니다.  
먼저 YAML파일을 만들어보겠습니다. (~/lab 디렉토리 아래의 파일을 사용하셔도 됩니다.)

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
      - image: nginx:1.19.3
        name: my-nginx
        ports:
        - containerPort: 80
```

> 파일명은 nginx-deployment.yaml로 합니다.

어디서 많이 본 형식인데... 하시면 맞습니다.  
ReplicaSet과 유사해요. (거의 동일)

---

일단 한번 생성해볼게요.

```bash
controlplane $ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```

> **명령어** : `kubectl apply -f nginx-deployment.yaml`{{exec}}  

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `kubectl apply -f ~/lab/nginx-deployment.yaml`{{exec}}

이번엔 새로운 명령어 `kubectl get all`을 해볼까요?

```bash
controlplane $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-56cb9cc9db-fpdc7   1/1     Running   0          21s
pod/nginx-deployment-56cb9cc9db-h8wqn   1/1     Running   0          21s
pod/nginx-deployment-56cb9cc9db-j2l6g   1/1     Running   0          21s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   70d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           21s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-56cb9cc9db   3         3         3       21s
```

> **명령어** : `kubectl get all`{{exec}}  

오~ 다 나오네요... ٩(ˊᗜˋ*)و

---

Pod들을 Label까지 같이 보려면 아래와 같이 하면 됩니다.

```bash
controlplane $ kubectl get po --show-labels
NAME                                READY   STATUS    RESTARTS   AGE   LABELS
nginx-deployment-56cb9cc9db-fpdc7   1/1     Running   0          45s   app=my-nginx,pod-template-hash=56cb9cc9db
nginx-deployment-56cb9cc9db-h8wqn   1/1     Running   0          45s   app=my-nginx,pod-template-hash=56cb9cc9db
nginx-deployment-56cb9cc9db-j2l6g   1/1     Running   0          45s   app=my-nginx,pod-template-hash=56cb9cc9db
```

> **명령어** : `kubectl get po --show-labels`{{exec}}  


이제 Pod 하나를 삭제(delete)해 볼까요?

```bash
controlplane $ kubectl delete po nginx-deployment-56cb9cc9db-fpdc7
pod "nginx-deployment-56cb9cc9db-fpdc7" deleted
```

> **명령어** : `kubectl delete po [POD-NAME]`{{copy}}  
> [POD-NAME] 에는 앞에서 조회된 POD 중 하나의 이름을 넣어주세요.

그리고, 다시 조회를 해보면...

```bash
controlplane $ kubectl get po --show-labels
NAME                                READY   STATUS    RESTARTS   AGE    LABELS
nginx-deployment-56cb9cc9db-h8wqn   1/1     Running   0          106s   app=my-nginx,pod-template-hash=56cb9cc9db
nginx-deployment-56cb9cc9db-j2l6g   1/1     Running   0          106s   app=my-nginx,pod-template-hash=56cb9cc9db
nginx-deployment-56cb9cc9db-lnpql   1/1     Running   0          25s    app=my-nginx,pod-template-hash=56cb9cc9db
```

> **명령어** : `kubectl get po --show-labels`{{exec}}  

새롭게 하나의 POD가 생성된 걸 볼 수 있습니다. ReplicaSet이 자기 역할을 다하고 있는 듯 하네요~ 이제 든든합니다.

---

이번엔 scale in/out 방법을 알아보겠습니다. (replicas를 조정)  
명령형 커맨드 방식으로는 이렇게 할 수 있습니다.

```bash
controlplane $ kubectl scale deployment nginx-deployment --replicas=5
deployment.apps/nginx-deployment scaled
```

> **명령어** : `kubectl scale deployment nginx-deployment --replicas=5`{{exec}}  

조회결과도 보겠습니다.

```bash
controlplane $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-56cb9cc9db-62jjq   1/1     Running   0          18s
pod/nginx-deployment-56cb9cc9db-h8wqn   1/1     Running   0          2m42s
pod/nginx-deployment-56cb9cc9db-j2l6g   1/1     Running   0          2m42s
pod/nginx-deployment-56cb9cc9db-lnpql   1/1     Running   0          81s
pod/nginx-deployment-56cb9cc9db-nc97h   1/1     Running   0          18s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   70d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   5/5     5            5           2m42s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-56cb9cc9db   5         5         5       2m42s
```

> **명령어** : `kubectl get all`{{exec}}  

명령형 커맨드에서 지정한 대로 Pod의 개수가 다섯개가 되었습니다. 새롭게 생성된 두 개의 Pod를 볼 수 있습니다.

---

`kubectl edit deployment nginx-deployment`{{exec}} 명령어로 생성된 리소스를 수정할 수도 있습니다.
마치 vi editor를 이용하여 YAML파일을 수정하는 것과 동일합니다.
한 번 해보세요.

`replicas`를 2로 바꾸고 저장후 빠져나옵니다. (`:wq`)

조회결과는 아래와 같습니다.

```bash
controlplane $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-56cb9cc9db-j2l6g   1/1     Running   0          3m56s
pod/nginx-deployment-56cb9cc9db-lnpql   1/1     Running   0          2m35s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   70d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           3m56s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-56cb9cc9db   2         2         2       3m56s
```

> **명령어** : `kubectl get all`{{exec}}  

---

그리고, 마지막으로 선언형 방법을 적용하려면 처음 사용된 yaml파일을 수정해주시면 됩니다.
vi editor를 이용하여 `.spec.replicas`부분을 수정하면 됩니다. (4로 변경)

그리고, 마법의 주문 `kubectl apply`를 하는거죠.

```bash
controlplane $ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment configured
```

> **명령어** : `kubectl apply -f nginx-deployment.yaml`{{exec}}  

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `kubectl apply -f ~/lab/nginx-deployment.yaml`{{exec}}

조회결과는 아래와 같습니다.

```bash
controlplane $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-56cb9cc9db-dh7rp   1/1     Running   0          18s
pod/nginx-deployment-56cb9cc9db-j2l6g   1/1     Running   0          4m59s
pod/nginx-deployment-56cb9cc9db-lnpql   1/1     Running   0          3m38s
pod/nginx-deployment-56cb9cc9db-s8cvc   1/1     Running   0          18s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   70d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   4/4     4            4           4m59s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-56cb9cc9db   4         4         4       4m59s
```

> **명령어** : `kubectl get all`{{exec}}  

