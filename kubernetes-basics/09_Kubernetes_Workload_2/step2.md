좀 더 나가서, 이번엔 **Deployment** 입니다. 먼저 YAML파일을 만들어보겠습니다.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
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
    spec:
      containers:
      - image: nginx:1.19.3
        name: my-nginx
        ports:
        - containerPort: 80
```
> 파일명은 **nginx-deployment.yaml**로 합니다.

<br><br><br>

일단 한번 생성해볼게요.
```bash
ubuntu@ip-172-31-23-60:~/mspt3/hands_on_files$ kubectl apply -f nginx-deployment.yaml
deployment.apps/my-nginx-deployment created
```

> 💻 명령어 `kubectl apply -f nginx-deployment.yaml`{{exec}}

<br><br><br>

이번엔 새로운 명령어 `kubectl get all`을 해볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-55985c7fcf-2b2r4   1/1     Running   0          33s
pod/my-nginx-deployment-55985c7fcf-czs5p   1/1     Running   0          32s
pod/my-nginx-deployment-55985c7fcf-jwmbt   1/1     Running   0          33s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d11h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   3/3     3            3           33s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-55985c7fcf   3         3         3       33s
```

> 💻 명령어 `kubectl get all`{{exec}}

오~ 다 나오네요... ٩(ˊᗜˋ*)و

<br><br><br>

Pod들을 Label까지 같이 보려면 아래와 같이 하면 됩니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods --show-labels
NAME                                   READY   STATUS    RESTARTS   AGE   LABELS
my-nginx-deployment-55985c7fcf-2b2r4   1/1     Running   0          82s   app=my-nginx,pod-template-hash=55985c7fcf
my-nginx-deployment-55985c7fcf-czs5p   1/1     Running   0          81s   app=my-nginx,pod-template-hash=55985c7fcf
my-nginx-deployment-55985c7fcf-jwmbt   1/1     Running   0          82s   app=my-nginx,pod-template-hash=55985c7fcf
```

> 💻 명령어 `kubectl get pods --show-labels`{{exec}}

<br><br><br>

이제 Pod 하나를 삭제(**delete**)해 볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl delete pods my-nginx-deployment-55985c7fcf-2b2r4
pod "my-nginx-deployment-55985c7fcf-2b2r4" deleted
```

> 💻 명령어 `kubectl delete pods [POD-NAME]`{{copy}}  
> [POD-NAME] 에는 앞에서 조회된 POD 중 하나의 이름을 넣어주세요.

<br><br><br>

그리고, 다시 조회를 해보면...
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods --show-labels
NAME                                   READY   STATUS    RESTARTS   AGE     LABELS
my-nginx-deployment-55985c7fcf-7tkdt   1/1     Running   0          44s     app=my-nginx,pod-template-hash=55985c7fcf
my-nginx-deployment-55985c7fcf-czs5p   1/1     Running   0          4m34s   app=my-nginx,pod-template-hash=55985c7fcf
my-nginx-deployment-55985c7fcf-jwmbt   1/1     Running   0          4m35s   app=my-nginx,pod-template-hash=55985c7fcf
```

> 💻 명령어 `kubectl get pods --show-labels`{{exec}}

새롭게 하나의 POD가 생성된 걸 볼 수 있습니다.  
ReplicaSet이 자기 역할을 다하고 있는 듯 하네요~  
이제 든든합니다.

<br><br><br>

이번엔 scale in/out 방법을 알아보겠습니다. (**replicas**를 조정)  
**명령형 커맨드** 방식으로는 이렇게 할 수 있습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl scale deployment my-nginx-deployment --replicas=5
deployment.apps/my-nginx-deployment scaled
```

> 💻 명령어 `kubectl scale deployment my-nginx-deployment --replicas=5`{{exec}}

<br><br><br>

조회결과도 보겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-55985c7fcf-7tkdt   1/1     Running   0          2m54s
pod/my-nginx-deployment-55985c7fcf-88jwb   1/1     Running   0          52s
pod/my-nginx-deployment-55985c7fcf-czs5p   1/1     Running   0          6m44s
pod/my-nginx-deployment-55985c7fcf-jwmbt   1/1     Running   0          6m45s
pod/my-nginx-deployment-55985c7fcf-jx8hx   1/1     Running   0          52s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d11h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   5/5     5            5           6m45s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-55985c7fcf   5         5         5       6m45s
```

> 💻 명령어 `kubectl get all`{{exec}}

명령형 커맨드에서 지정한 대로 Pod의 개수가 다섯개가 되었습니다.  
새롭게 생성된 두 개의 Pod를 볼 수 있습니다.

Deployment와 ReplicaSet의 내용도 변경된게 보이네요. (Pod 개수에 대한 정보들)

<br><br><br>

`kubectl edit deployment my-nginx-deployment` 명령어로 생성된 리소스를 수정할 수도 있습니다.  
마치 vi editor를 이용하여 YAML파일을 수정하는 것과 동일합니다.

> 💻 명령어 `kubectl edit deployment my-nginx-deployment`{{exec}}

위의 명령어를 실행한 후, `replicas`를 2로 바꾸고 저장후 빠져나옵니다. (숫자 변경 후 ESC를 누르고 `:wq` 입력 후 엔터)

<br><br><br>

조회결과는 아래와 같습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-55985c7fcf-czs5p   1/1     Running   0          10m
pod/my-nginx-deployment-55985c7fcf-jwmbt   1/1     Running   0          10m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d11h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   2/2     2            2           10m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-55985c7fcf   2         2         2       10m
```

> 💻 명령어 `kubectl get all`{{exec}}

<br><br><br>

그리고, 마지막으로 **선언형 방법**을 적용하려면 처음 사용된 yaml파일을 수정해주시면 됩니다.  
editor(e.g. VI Editor)를 이용하여 `.spec.replicas`부분을 수정하면 됩니다. (**4**로 변경)

그리고, 마법의 주문 `kubectl apply`를 하는거죠.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/my-nginx-deployment configured
```

> 💻 명령어 `kubectl apply -f nginx-deployment.yaml`{{exec}}

<br><br><br>

조회결과는 아래와 같습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-55985c7fcf-bqv7g   1/1     Running   0          48s
pod/my-nginx-deployment-55985c7fcf-czs5p   1/1     Running   0          14m
pod/my-nginx-deployment-55985c7fcf-dvl2w   1/1     Running   0          48s
pod/my-nginx-deployment-55985c7fcf-jwmbt   1/1     Running   0          14m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d11h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   4/4     4            4           14m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-55985c7fcf   4         4         4       14m
```

> 💻 명령어 `kubectl get all`{{exec}}

<br><br><br>

마지막으로 생성한 리소스들을 삭제하고 마치겠습니다.  ˘◡˘  

```bash
ubuntu@ip-172-31-23-60:~$ kubectl delete -f nginx-deployment.yaml
deployment.apps "my-nginx-deployment" deleted
```

> 💻 명령어 `kubectl delete -f nginx-deployment.yaml`{{exec}}