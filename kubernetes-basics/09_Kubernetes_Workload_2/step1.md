이제 Pod를 관리하는 다른 방법을 알아보겠습니다.

첫 번째는 **ReplicaSet** 입니다. **ReplicaSet** 생성을 위해서 아래 파일을 작성합니다.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: my-nginx
    tier: frontend
spec:
  replicas: 3
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
> 파일명은 **nginx-replicaset.yaml**로 합니다.

<br><br><br>

**spec**부분을 보시면, 우리가 원하는 **Pod**에 대한 **spec**이 보이고, 그 위에 `replicas: 3` 이라는 부분이 보이네요.  
이 부분이 핵심입니다.

나는 Nginx Pod를 세 개 원한다고 선언한 것입니다.

특별한 얘기가 없으면 yaml파일을 이용한 리소스 생성은 `kubectl apply` 명령어를 쓰시면 됩니다.

**ReplicaSet**을 생성해볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl apply -f nginx-replicaset.yaml
replicaset.apps/nginx-replicaset created
```

> 💻 명령어 `kubectl apply -f nginx-replicaset.yaml`{{exec}}

<br><br><br>

조회도 해보시구요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get replicasets -o wide
NAME               DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES         SELECTOR
nginx-replicaset   3         3         3       119s   my-nginx     nginx:1.19.3   app=my-nginx
```

> 💻 명령어 `kubectl get replicasets -o wide`{{exec}}

<br><br><br>

상세조회 결과는 아래와 같습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl describe replicasets nginx-replicaset
Name:         nginx-replicaset
Namespace:    default
Selector:     app=my-nginx
Labels:       app=my-nginx
              tier=frontend
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=my-nginx
  Containers:
   my-nginx:
    Image:        nginx:1.19.3
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  3m57s  replicaset-controller  Created pod: nginx-replicaset-fjjxd
  Normal  SuccessfulCreate  3m57s  replicaset-controller  Created pod: nginx-replicaset-6x5rp
  Normal  SuccessfulCreate  3m57s  replicaset-controller  Created pod: nginx-replicaset-4b52g
```

> 💻 명령어 `kubectl describe replicasets nginx-replicaset`{{exec}}

<br><br><br>

우리는 Pod를 직접 생성하지는 않았지만 Pod도 생성됐습니다.
ReplicaSet이 하는 일이 그런거니까요.

Pod도 조회해볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
nginx-replicaset-4b52g   1/1     Running   0          4m55s   app=my-nginx
nginx-replicaset-6x5rp   1/1     Running   0          4m55s   app=my-nginx
nginx-replicaset-fjjxd   1/1     Running   0          4m55s   app=my-nginx
```

> 💻 명령어 `kubectl get pods --show-labels`{{exec}}

<br><br><br>

이제 뭔가 좀 자동으로 돌아가는 모양새가 나오네요~

이제 생성한 리소스들을 삭제해볼게요.
`apply` 를 `delete` 로 바꿔주시면 됩니다. (ง˙∇˙)ว

```bash
ubuntu@ip-172-31-23-60:~$ kubectl delete -f nginx-replicaset.yaml
replicaset.apps "nginx-replicaset" deleted
```

> 💻 명령어 `kubectl delete -f nginx-replicaset.yaml`{{exec}}