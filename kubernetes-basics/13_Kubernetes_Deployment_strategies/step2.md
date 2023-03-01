이번에는 Deployment의 **롤백** 방법을 알아보겠습니다.  
먼저 업데이트 **History**는 아래와 같이 확인해볼 수 있습니다.
```bash
controlplane $ kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

> 💻 명령어(Tab1) `kubectl rollout history deployment nginx-deployment`{{exec}}

<br><br><br>

최초 생성된 **Revision #1**과 한 번 업데이트 후의 **Revision #2**가 보입니다.  
그 중 하나의 Revision을 콕 집어서 자세히 볼 수도 있습니다.
```bash
controlplane $ kubectl rollout history deployment nginx-deployment --revision=1
deployment.apps/nginx-deployment with revision #1
Pod Template:
  Labels:       app=my-nginx
        pod-template-hash=6859c5bd49
  Containers:
   my-nginx:
    Image:      nginx:1.18
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

> 💻 명령어(Tab1) `kubectl rollout history deployment nginx-deployment --revision=1`{{exec}}

<br><br><br>

```bash
controlplane $ kubectl rollout history deployment nginx-deployment --revision=2
deployment.apps/nginx-deployment with revision #2
Pod Template:
  Labels:       app=my-nginx
        pod-template-hash=76c6c66746
  Containers:
   my-nginx:
    Image:      nginx:1.19
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

> 💻 명령어(Tab1) `kubectl rollout history deployment nginx-deployment --revision=2`{{exec}}

<br><br><br>

역시 **두 번째 Terminal**에 어떤 변화가 일어날지 확인 할 준비를 하고,
```bash
controlplane $ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-76c6c66746-24mvf   1/1     Running   0          6m57s
nginx-deployment-76c6c66746-4lrfz   1/1     Running   0          6m57s
nginx-deployment-76c6c66746-554kk   1/1     Running   0          6m57s

```

> 💻 명령어(Tab2) `kubectl get pods --watch`{{exec}}  
> `--watch` 는 앞의 명령어를 실행한 후 변경(Change)사항을 지속적으로 보여주는 Flag입니다.
> Watch를 멈추려면 Ctrl + c 를 입력합니다.

<br><br><br>

**첫 번째 Terminal**에서 **revision1**으로 롤백 합니다.
```bash
controlplane $ kubectl rollout undo deployment nginx-deployment --to-revision=1
deployment.apps/nginx-deployment rolled back
```

> 💻 명령어(Tab1) `kubectl rollout undo deployment nginx-deployment --to-revision=1`{{exec}}

<br><br><br>

**두 번째 Terminal**에는 업데이트 할 때와 비슷한 변경내용을 볼 수 있을겁니다.  
Pod들을 먼저 삭제하고, 새로은 Pod들을 만드는 걸 볼 수 있습니다.
```bash
controlplane $ kubectl get pods --watch
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-76c6c66746-24mvf   1/1     Running   0          6m57s
nginx-deployment-76c6c66746-4lrfz   1/1     Running   0          6m57s
nginx-deployment-76c6c66746-554kk   1/1     Running   0          6m57s

nginx-deployment-76c6c66746-24mvf   1/1     Terminating   0          7m36s
nginx-deployment-76c6c66746-554kk   1/1     Terminating   0          7m36s
nginx-deployment-76c6c66746-4lrfz   1/1     Terminating   0          7m36s
nginx-deployment-76c6c66746-4lrfz   1/1     Terminating   0          7m39s
nginx-deployment-76c6c66746-554kk   1/1     Terminating   0          7m39s
nginx-deployment-76c6c66746-24mvf   1/1     Terminating   0          7m39s
nginx-deployment-76c6c66746-24mvf   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-24mvf   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-24mvf   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-4lrfz   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-4lrfz   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-4lrfz   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-554kk   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-554kk   0/1     Terminating   0          7m40s
nginx-deployment-76c6c66746-554kk   0/1     Terminating   0          7m40s
nginx-deployment-6859c5bd49-bhh7w   0/1     Pending       0          0s
nginx-deployment-6859c5bd49-bhh7w   0/1     Pending       0          0s
nginx-deployment-6859c5bd49-ptrzt   0/1     Pending       0          0s
nginx-deployment-6859c5bd49-2gzrv   0/1     Pending       0          0s
nginx-deployment-6859c5bd49-2gzrv   0/1     Pending       0          0s
nginx-deployment-6859c5bd49-ptrzt   0/1     Pending       0          0s
nginx-deployment-6859c5bd49-bhh7w   0/1     Pending       0          1s
nginx-deployment-6859c5bd49-ptrzt   0/1     Pending       0          1s
nginx-deployment-6859c5bd49-bhh7w   0/1     ContainerCreating   0          1s
nginx-deployment-6859c5bd49-2gzrv   0/1     Pending             0          1s
nginx-deployment-6859c5bd49-2gzrv   0/1     ContainerCreating   0          2s
nginx-deployment-6859c5bd49-ptrzt   0/1     ContainerCreating   0          2s
nginx-deployment-6859c5bd49-ptrzt   1/1     Running             0          3s
nginx-deployment-6859c5bd49-bhh7w   1/1     Running             0          3s
nginx-deployment-6859c5bd49-2gzrv   1/1     Running             0          3s
```

<br><br><br>

이전 버젼으로 롤백이 잘 됐는지 아래 명령어로 확인해보세요.
```bash
controlplane $ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6859c5bd49-2gzrv   1/1     Running   0          36s
nginx-deployment-6859c5bd49-bhh7w   1/1     Running   0          36s
nginx-deployment-6859c5bd49-ptrzt   1/1     Running   0          36s
controlplane $ kubectl describe pod nginx-deployment-6859c5bd49-2gzrv | grep -i image
    Image:          nginx:1.18
    Image ID:       docker.io/library/nginx@sha256:e90ac5331fe095cea01b121a3627174b2e33e06e83720e9a934c7b8ccc9c55a0
  Normal  Pulled     52s   kubelet            Container image "nginx:1.18" already present on machine
```

> 💻 명령어(Tab1) `kubectl get pods`{{exec}}  
> 💻 명령어(Tab1) `kubectl describe pod [POD-NAME] | grep -i image`{{copy}}  
> [POD-NAME] 에는 앞에서 조회된 POD 중 하나의 이름을 넣어주세요.

<br><br><br>

다 해보셨으면 다음 실습을 위해 Object들을 삭제해주세요.  
아시죠? **선언형(Declarative)**...
```bash
controlplane $ kubectl delete -f nginx-recreate.yaml
deployment.apps "nginx-deployment" deleted
```

> 💻 명령어(Tab1) `kubectl delete -f nginx-recreate.yaml`{{exec}}

이제 두 번째 터미널은 Ctrl + c 를 눌러 Watch를 멈추겠습니다.