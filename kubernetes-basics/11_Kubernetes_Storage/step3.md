도커 실습에서 사용한 **ToDo App**을 PVC를 사용해서 데이터를 저장하는 방법입니다.  
Docker Volumes 실습의 Kubernetes 버젼이라고 보시면 될 것 같아요.

실습에 필요한 파일은 모두 **hands_on_files**아래에 있습니다.  
아래 참고해서 해보세요.
> PVC 생성 > Deployment 생성 > Service 생성 > Ingress 생성

```bash
ubuntu@ip-172-31-23-60:~$ kubectl apply -f todo-pvc.yaml
persistentvolumeclaim/todo-pvc created
ubuntu@ip-172-31-23-60:~$ kubectl apply -f todo-deployment-volume.yaml
deployment.apps/todo-app-deployment created
ubuntu@ip-172-31-23-60:~$ kubectl apply -f todo-clusterip-service.yaml
service/todo-clusterip-service created
ubuntu@ip-172-31-23-60:~$ kubectl apply -f todo-ingress.yaml
ingress.networking.k8s.io/todo-app-ingress created
```
> 💻 명령어
>```bash
>kubectl apply -f todo-pvc.yaml
>kubectl apply -f todo-deployment-volume.yaml
>kubectl apply -f todo-clusterip-service.yaml
>kubectl apply -f todo-ingress.yaml
>
>```

<br><br><br>

ToDo App 접속을 위해서 **hosts**파일에 다음과 같이 하나(***todo-app.info***)를 추가합니다.  
- Windows라면 **C:\Windows\System32\drivers\etc\hosts** 파일에,
- Linux계열은 **/etc/hosts** 파일에 추가하면 됩니다.
```bash
#mspt3
11.22.33.44  my-nginx.info todo-app.info
```
> 11.22.33.44 대신 여러분 EC2 Instance의 **Public IPv4 address**를 써주세요.

이제 브라우저에서 http://todo-app.info/ 로 접속하면, 아래와 같이 접속 가능할거예요.

![h:300](./img/k8s_todo_ingress.png)

<br><br><br>

그리고, 아래처럼 Pod들이 삭제와 생성을 반복해도 데이터는 사라지지 않고 유지될거예요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pod
NAME                                   READY   STATUS    RESTARTS   AGE
todo-app-deployment-55464569cf-2stsv   1/1     Running   0          91s
todo-app-deployment-55464569cf-4jdq8   1/1     Running   0          91s
todo-app-deployment-55464569cf-zwh9z   1/1     Running   0          91s
ubuntu@ip-172-31-23-60:~$ kubectl delete pod --all
pod "todo-app-deployment-55464569cf-2stsv" deleted
pod "todo-app-deployment-55464569cf-4jdq8" deleted
pod "todo-app-deployment-55464569cf-zwh9z" deleted
ubuntu@ip-172-31-23-60:~$ kubectl get pod
NAME                                   READY   STATUS    RESTARTS   AGE
todo-app-deployment-55464569cf-7gn5v   1/1     Running   0          8s
todo-app-deployment-55464569cf-plnfd   1/1     Running   0          8s
todo-app-deployment-55464569cf-x8l6h   1/1     Running   0          8s
```
> 💻 명령어
>```bash
>kubectl get pod
>```
>```bash
>kubectl delete pod --all
>```
>```bash
>kubectl get pod
>```

<br><br><br>

다 해보셨으면, 깨끗이 정리하고 마칠게요.

```bash
ubuntu@ip-172-31-23-60:~$ kubectl delete -f todo-ingress.yaml
ingress.networking.k8s.io "todo-app-ingress" deleted
ubuntu@ip-172-31-23-60:~$ kubectl delete -f todo-clusterip-service.yaml
service "todo-clusterip-service" deleted
ubuntu@ip-172-31-23-60:~$ kubectl delete -f todo-deployment-volume.yaml
deployment.apps "todo-app-deployment" deleted
ubuntu@ip-172-31-23-60:~$ kubectl delete -f todo-pvc.yaml
persistentvolumeclaim "todo-pvc" deleted
```
> 💻 명령어
>```bash
>kubectl delete -f todo-ingress.yaml
>kubectl delete -f todo-clusterip-service.yaml
>kubectl delete -f todo-deployment-volume.yaml
>kubectl delete -f todo-pvc.yaml
>
>```

<br>

이번 실습은 여기까지 입니다.  ˘◡˘  
끝~