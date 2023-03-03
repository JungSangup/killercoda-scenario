도커 실습에서 사용한 **ToDo App**을 PVC를 사용해서 데이터를 저장하는 방법입니다.  
Docker Volumes 실습의 Kubernetes 버젼이라고 보시면 될 것 같아요.

아래 순서대로 생성합니다.
> PV생성 > PVC 생성 > Deployment 생성 > Service 생성 > Ingress 생성

```bash
controlplane $ kubectl apply -f todo-pv.yaml
persistentvolume/todo-pv created
controlplane $ kubectl apply -f todo-pvc.yaml
persistentvolumeclaim/todo-pvc created
controlplane $ kubectl apply -f todo-deployment-volume.yaml
deployment.apps/todo-app-deployment created
controlplane $ kubectl apply -f todo-nodeport-service.yaml
service/todo-clusterip-service created
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

🔗 [ToDo List Manager]({{TRAFFIC_HOST1_30007}})

<br><br><br>

그리고, 아래처럼 Pod들이 삭제와 생성을 반복해도 데이터는 사라지지 않고 유지될거예요.
```bash
controlplane $ kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
todo-app-deployment-d9f967656-bzpz8   1/1     Running   0          35s
todo-app-deployment-d9f967656-lrc7f   1/1     Running   0          35s
todo-app-deployment-d9f967656-qp2fg   1/1     Running   0          35s
controlplane $ kubectl delete pod --all
pod "todo-app-deployment-d9f967656-bzpz8" deleted
pod "todo-app-deployment-d9f967656-lrc7f" deleted
pod "todo-app-deployment-d9f967656-qp2fg" deleted
controlplane $ kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
todo-app-deployment-d9f967656-8dhwg   1/1     Running   0          32s
todo-app-deployment-d9f967656-dsrb5   1/1     Running   0          32s
todo-app-deployment-d9f967656-jhjwx   1/1     Running   0          32s
```
> 💻 명령어 `kubectl get pod`{{exec}}  
> 💻 명령어 `kubectl delete pod --all`{{exec}}  
> 💻 명령어 `kubectl get pod`{{exec}}

위의 명령어들을 실행한 다음에도 데이터들이 유지되는지 확인해보세요.  
이유는... 여러분도 잘 아시다시피, 데이터는 Volume영역에 저장되기 때문입니다.

<br><br><br>

다 해보셨으면, 깨끗이 정리하고 마칠게요.

```bash
controlplane $ kubectl delete -f todo-nodeport-service.yaml
service "todo-clusterip-service" deleted
controlplane $ kubectl delete -f todo-deployment-volume.yaml
deployment.apps "todo-app-deployment" deleted
controlplane $ kubectl delete -f todo-pvc.yaml
persistentvolumeclaim "todo-pvc" deleted
controlplane $ kubectl delete -f todo-pv.yaml
persistentvolume "todo-pv" deleted
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
