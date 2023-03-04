이제 만들어진 PVC, PV를 사용하는 Pod를 생성해 보겠습니다.  

다음과 같이 Deployment와 Service를 준비해주세요.
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
      volumes:
      - name: nginx-storage
        persistentVolumeClaim:
          claimName: nginx-pvc
      containers:
      - image: nginx:1.19.3
        name: my-nginx
        ports:
        - containerPort: 80
        volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: nginx-storage
```
> 파일명은 **nginx-deployment-volume.yaml**로 합니다.

앞에서 만든 **nginx-pvc**를 사용하고, 컨테이너의 **/usr/share/nginx/html**에 마운트합니다.

<br><br><br>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
```
> 파일명은 **nginx-nodeport-service.yaml**로 합니다.

<br><br><br>

다음은 Deployment와 앞에서 실습한 Service까지 리소스를 생성해주세요.
```bash
controlplane $ kubectl apply -f nginx-deployment-volume.yaml
deployment.apps/my-nginx-deployment created
controlplane $ kubectl apply -f nginx-nodeport-service.yaml
service/nginx-nodeport-service created
```

> 💻 명령어
>```bash
>kubectl apply -f nginx-deployment-volume.yaml
>kubectl apply -f nginx-nodeport-service.yaml
>```{{exec}}

<br><br><br>

아직 한 가지 더 할 일이 남았습니다.  
⚠️ 아래 명령는 pod들이 모두 생성된 후 실행해주세요.
```bash
controlplane $ ssh node01 "echo '<h1>Hello kubernetes</h1>' >> /mnt/data/index.html"
```

> 💻 명령어 `ssh node01 "echo '<h1>Hello kubernetes</h1> <br> <iframe width="1400" height="788" src="https://www.youtube.com/embed/JbHI1yI1Ndk" allowfullscreen></iframe>' >> /mnt/data/index.html"`{{exec}}

<br>

Nginx에서 보여줄 간단한 **index.html**파일을 하나 만들었습니다.  
혹시 PV의 경로가 다르다면 거기에 맞춰서 해주세요.  

Nginx는 (worker) node에서 실행되기 때문에, **node01**의 local(/mnt/data)에 파일(index.html)을 생성했습니다.

<br><br><br>

이제 브라우저에서 어떻게 나오나 볼까요?

🔗 [Nginx]({{TRAFFIC_HOST1_30007}})

![h:200](./img/k8s_nginx_pvc.png)

Pod의 파일시스템에도 위의 내용이 반영되어 있는지도 확인해보세요.
```bash
controlplane $ kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
my-nginx-deployment-d65998955-8lcvk   1/1     Running   0          2m6s
my-nginx-deployment-d65998955-xmbn9   1/1     Running   0          2m6s
my-nginx-deployment-d65998955-zc67r   1/1     Running   0          2m6s
controlplane $ kubectl exec -it my-nginx-deployment-d65998955-8lcvk -- cat /usr/share/nginx/html/index.html
<h1>Hello kubernetes</h1> <br> <iframe width="1400" height="788" src="https://www.youtube.com/embed/JbHI1yI1Ndk" allowfullscreen></iframe>
```

> 💻 명령어 `kubectl get pod`{{exec}}  
> 💻 명령어 `kubectl exec -it [POD-NAME] -- cat /usr/share/nginx/html/index.html`{{copy}}
> [POD-NAME] 에는 앞에서 조회한 POD중 하나의 이름을 넣어주세요.

<br><br><br>

아래와 같이 사용한 리소스들을 정리해주세요.

```bash
controlplane $ kubectl delete -f nginx-nodeport-service.yaml
service "nginx-nodeport-service" deleted
controlplane $ kubectl delete -f nginx-deployment-volume.yaml
deployment.apps "my-nginx-deployment" deleted
controlplane $ kubectl delete -f nginx-pvc.yaml
persistentvolumeclaim "nginx-pvc" deleted
```
> 💻 명령어
>```bash
>kubectl delete -f nginx-nodeport-service.yaml
>kubectl delete -f nginx-deployment-volume.yaml
>kubectl delete -f nginx-pvc.yaml
>```{{exec}}