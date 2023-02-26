도커 실습에서 사용한 **ToDo App**을 PVC를 사용해서 데이터를 저장하는 방법입니다.  
Docker Volumes 실습의 Kubernetes 버젼이라고 보시면 될 것 같아요.

아래 순서대로 생성합니다.
> PV생성 > PVC 생성 > Deployment 생성 > Service 생성 > Ingress 생성

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
>kubectl apply -f todo-pv.yaml
>kubectl apply -f todo-pvc.yaml
>kubectl apply -f todo-deployment-volume.yaml
>kubectl apply -f todo-nodeport-service.yaml
>```{{exec}}

<br><br><br>

이제 브라우저에서 어떻게 나오나 볼까요?

🔗 [Nginx]({{TRAFFIC_HOST1_30008}})

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
> 💻 명령어 `kubectl get pod`{{exec}}  
> 💻 명령어 `kubectl delete pod --all`{{exec}}  
> 💻 명령어 `kubectl get pod`{{exec}}

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
>kubectl delete -f todo-nodeport-service.yaml
>kubectl delete -f todo-deployment-volume.yaml
>kubectl delete -f todo-pvc.yaml
>kubectl delete -f todo-pv.yaml
>```{{exec}}

<br>

이번 실습은 여기까지 입니다.  ˘◡˘  
끝~