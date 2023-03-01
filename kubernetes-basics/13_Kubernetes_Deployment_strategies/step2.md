이번에는 Deployment의 **롤백** 방법을 알아보겠습니다.  
먼저 업데이트 **History**는 아래와 같이 확인해볼 수 있습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

> 💻 명령어
>```bash
>kubectl rollout history deployment nginx-deployment
>```

<br><br><br>

최초 생성된 **Revision #1**과 한 번 업데이트 후의 **Revision #2**가 보입니다.  
그 중 하나의 Revision을 콕 집어서 자세히 볼 수도 있습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl rollout history deployment nginx-deployment --revision=1
deployment.apps/nginx-deployment with revision #1
Pod Template:
  Labels:	app=my-nginx
	pod-template-hash=85fc747956
  Containers:
   my-nginx:
    Image:	nginx:1.18
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

> 💻 명령어
>```bash
>kubectl rollout history deployment nginx-deployment --revision=1
>```

<br><br><br>

```bash
ubuntu@ip-172-31-23-60:~$ kubectl rollout history deployment nginx-deployment --revision=2
deployment.apps/nginx-deployment with revision #2
Pod Template:
  Labels:	app=my-nginx
	pod-template-hash=7dd48c557f
  Containers:
   my-nginx:
    Image:	nginx:1.19
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

> 💻 명령어
>```bash
>kubectl rollout history deployment nginx-deployment --revision=2
>```

<br><br><br>

역시 **두 번째 Terminal**에 어떤 변화가 일어날지 확인 할 준비를 하고,
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dd48c557f-6b9b8   1/1     Running   0          4m47s
nginx-deployment-7dd48c557f-k6czs   1/1     Running   0          4m47s
nginx-deployment-7dd48c557f-s9msd   1/1     Running   0          4m47s

```

> 💻 명령어
>```bash
>kubectl get pods --watch
>```
> `--watch` 는 앞의 명령어를 실행한 후 변경(Change)사항을 지속적으로 보여주는 Flag입니다.
> Watch를 멈추려면 Ctrl + c 를 입력합니다.

<br><br><br>

**첫 번째 Terminal**에서 **revision1**으로 롤백 합니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl rollout undo deployment nginx-deployment --to-revision=1
deployment.apps/nginx-deployment rolled back
```

> 💻 명령어
>```bash
>kubectl rollout undo deployment nginx-deployment --to-revision=1
>```

<br><br><br>

**두 번째 Terminal**에는 업데이트 할 때와 비슷한 변경내용을 볼 수 있을겁니다.  
Pod들을 먼저 삭제하고, 새로은 Pod들을 만드는 걸 볼 수 있습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dd48c557f-6b9b8   1/1     Running   0          4m47s
nginx-deployment-7dd48c557f-k6czs   1/1     Running   0          4m47s
nginx-deployment-7dd48c557f-s9msd   1/1     Running   0          4m47s

nginx-deployment-7dd48c557f-s9msd   1/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-k6czs   1/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-6b9b8   1/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-k6czs   0/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-6b9b8   0/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-s9msd   0/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-s9msd   0/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-s9msd   0/1     Terminating   0          5m12s
nginx-deployment-7dd48c557f-k6czs   0/1     Terminating   0          5m13s
nginx-deployment-7dd48c557f-k6czs   0/1     Terminating   0          5m13s
nginx-deployment-7dd48c557f-6b9b8   0/1     Terminating   0          5m13s
nginx-deployment-7dd48c557f-6b9b8   0/1     Terminating   0          5m13s
nginx-deployment-85fc747956-n62jm   0/1     Pending       0          0s
nginx-deployment-85fc747956-n62jm   0/1     Pending       0          0s
nginx-deployment-85fc747956-7vgpg   0/1     Pending       0          0s
nginx-deployment-85fc747956-rkhf8   0/1     Pending       0          0s
nginx-deployment-85fc747956-7vgpg   0/1     Pending       0          0s
nginx-deployment-85fc747956-rkhf8   0/1     Pending       0          0s
nginx-deployment-85fc747956-n62jm   0/1     ContainerCreating   0          1s
nginx-deployment-85fc747956-7vgpg   0/1     ContainerCreating   0          1s
nginx-deployment-85fc747956-rkhf8   0/1     ContainerCreating   0          2s

nginx-deployment-85fc747956-n62jm   1/1     Running             0          2s
nginx-deployment-85fc747956-rkhf8   1/1     Running             0          2s
nginx-deployment-85fc747956-7vgpg   1/1     Running             0          3s
```

<br><br><br>

이전 버젼으로 롤백이 잘 됐는지 아래 명령어로 확인해보세요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85fc747956-7vgpg   1/1     Running   0          92s
nginx-deployment-85fc747956-n62jm   1/1     Running   0          92s
nginx-deployment-85fc747956-rkhf8   1/1     Running   0          92s
ubuntu@ip-172-31-23-60:~$ kubectl describe pod nginx-deployment-85fc747956-7vgpg | grep -i image
    Image:          nginx:1.18
    Image ID:       docker-pullable://nginx@sha256:e90ac5331fe095cea01b121a3627174b2e33e06e83720e9a934c7b8ccc9c55a0
  Normal  Pulled     108s  kubelet            Container image "nginx:1.18" already present on machine
```

> 💻 명령어
>```bash
>kubectl get pods
>```
>```bash
>kubectl describe pod [POD-NAME] | grep -i image
>```
> [POD-NAME] 에는 앞에서 조회된 POD 중 하나의 이름을 넣어주세요.

<br><br><br>

다 해보셨으면 다음 실습을 위해 Object들을 삭제해주세요.  
아시죠? **선언형(Declarative)**...
```bash
ubuntu@ip-172-31-23-60:~$ kubectl delete -f nginx-recreate.yaml
deployment.apps "nginx-deployment" deleted
```

> 💻 명령어
>```bash
>kubectl delete -f nginx-recreate.yaml
>```

이제 두 번째 터미널은 Ctrl + c 를 눌러 Watch를 멈추겠습니다.