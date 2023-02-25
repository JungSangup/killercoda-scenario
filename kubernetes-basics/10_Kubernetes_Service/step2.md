
이제 **Service**를 생성해서 위의 문제들을 해결해보겠습니다.

먼저 **ClusterIP** 타입의 서비스를 하나 만들어 보겠습니다.  
아래와 같은 파일을 준비해주세요. (~/lab 디렉토리 아래의 파일을 사용하셔도 됩니다.)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip-service
spec:
  type: ClusterIP
  selector:
    app: my-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

> 파일명은 nginx-clusterip-service.yaml로 합니다.

그리고, 생성합니다.

```bash
controlplane $ kubectl apply -f nginx-clusterip-service.yaml
service/nginx-clusterip-service created
```

> **명령어** : `kubectl apply -f nginx-clusterip-service.yaml`{{exec}}  

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `kubectl apply -f ~/lab/nginx-clusterip-service.yaml`{{exec}}

---

생성된걸 조회할 때는 아래와 같이 합니다.

```
controlplane $ kubectl get services
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes                ClusterIP   10.96.0.1      <none>        443/TCP   70d
nginx-clusterip-service   ClusterIP   10.106.200.8   <none>        80/TCP    80s
```

> **명령어** : `kubectl get services`{{exec}}  

생성된 Service의 **CLUSTER-IP**가 보이시나요?
이 아이피로 Pod까지 접근할 수도 있습니다.

```bash
controlplane $ kubectl exec -it curlpod -- curl http://10.106.200.8
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

> **명령어** : `kubectl exec -it curlpod -- curl http://[SVC_IP]`{{copy}}  
> [SVC_IP]는 Service의 CLUSTER-IP

---

IP가 아닌 Name으로도 가능합니다.
이렇게요.

```bash
controlplane $ kubectl exec -it curlpod -- curl nginx-clusterip-service
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

> **명령어** : `kubectl exec -it curlpod -- curl [SVC_NAME]`{{copy}}  
> [SVC_NAME] 는 Service의 NAME

잘 되네요...

이제 Service를 만들면 클러스터 내에서는 서비스의 **이름**(**NAME**)으로도 접근이 가능합니다.