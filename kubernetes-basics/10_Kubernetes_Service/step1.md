이번 실습에서는 **Service**를 이용하는 방법을 알아보겠습니다.  
먼저 **Deployment**를 이용해서 Pod를 몇 개 생성해 보겠습니다. (~/lab 디렉토리 아래의 파일을 사용하셔도 됩니다.)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: my-nginx
    tier: frontend
spec:
  replicas: 2
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

---

그리고, 아래와 같이 생성한 다음, 생성된 Pod을 조회합니다.

```bash
controlplane $ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

controlplane $ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
nginx-deployment-56cb9cc9db-9rgsc   1/1     Running   0          34s   192.168.1.4   node01   <none>           <none>
nginx-deployment-56cb9cc9db-bl652   1/1     Running   0          34s   192.168.1.5   node01   <none>           <none>
```

> **명령어** : `kubectl apply -f nginx-deployment.yaml`{{exec}} , `kubectl get pods -o wide`{{exec}}  

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `kubectl apply -f ~/lab/nginx-deployment.yaml`{{exec}} , `kubectl get pods -o wide`{{exec}}
 
<br><br>

이제 위에서 생성한 Pod들을 사용하는 또다른 Pod를 하나 만들겠습니다.

```bash
controlplane $ kubectl run curlpod --image=radial/busyboxplus:curl --command -- /bin/sh -c "while true; do echo hi; sleep 10; done"
pod/curlpod created
```

> **명령어** : `kubectl run curlpod --image=radial/busyboxplus:curl --command -- /bin/sh -c "while true; do echo hi; sleep 10; done"`{{exec}}  

---

그리고,앞에서 만들어진 Nginx Pod의 IP를 이용해서 접속해보겠습니다.

```bash
controlplane $ kubectl exec -it curlpod -- curl http://192.168.1.4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

> **명령어** : `kubectl exec -it curlpod -- curl http://[POD_IP]`{{copy}}  
> [POD_IP]는 Nginx Pod 중 하나의 IP

---

잘 동작하네요.

하지만, 이렇게는 쓰기 어렵습니다.  
Pod의 IP가 어떻게 주어질지 우리는 알 수가 없고, Scaling되는 환경이라면 개별 Pod에 대한 Loadbalancing 문제도 있습니다.

그리고, Cluster IP는 내부에서만 사용되는 IP이기 때문에 클러스터 외부에서 접근이 필요한 경우에는 사용할 수 없습니다.