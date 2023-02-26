이번에는 Ingress 리소스를 생성하고 등록된 URL을 이용해서 접속해 보겠습니다.

먼저 [Nginx ingress controller](https://kubernetes.github.io/ingress-nginx/)를 설치할게요.

```bash
controlplane $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

> 💻 명령어 `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml`{{exec}}

<br><br><br>

Nginx ingress controller는 ingress-nginx 네임스페이스에 리소스들이 생성됩니다.  
설치결과는 아래와 같이 조회 가능합니다.
```bash
controlplane $ kubectl get all -n ingress-nginx 
NAME                                           READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-vfvns       0/1     Completed   0          115s
pod/ingress-nginx-admission-patch-r8bgs        0/1     Completed   0          115s
pod/ingress-nginx-controller-c69664497-2hxlp   1/1     Running     0          115s

NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.98.196.58   <pending>     80:30076/TCP,443:31992/TCP   116s
service/ingress-nginx-controller-admission   ClusterIP      10.96.243.97   <none>        443/TCP                      116s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           116s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-c69664497   1         1         1       115s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           12s        116s
job.batch/ingress-nginx-admission-patch    1/1           12s        115s
```

> 💻 명령어 `kubectl get all -n ingress-nginx`{{exec}}

🤔 ingress-nginx-controller pod가 Running이 될 때까지 기다려주세요.

<br><br><br>

이제 Ingress 리소스를 아래와 같이 준비합니다.  

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: nginx
  rules:
    - host: my-nginx.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-clusterip-service
                port:
                  number: 80
```
> 파일명은 **nginx-ingress.yaml**로 합니다.

<br><br><br>

이제 Ingress 리소스를 생성합니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl apply -f nginx-ingress.yaml
ingress.networking.k8s.io/my-nginx-ingress created
```

> 💻 명령어 `kubectl apply -f nginx-ingress.yaml`{{exec}}

<br><br><br>

다음은 로컬 포트를 Ingress controller로 연결합니다.  

```bash
controlplane $ kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 80:80
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
Handling connection for 80
Handling connection for 80
Handling connection for 80
```

> 💻 명령어 `kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 80:80`{{exec}}

👋🏼 위의 명령을 실행하면, 그 탭에서는 다른 명령어는 실행할 수 없습니다.  
탭을 하나 더 실행해서, 아래 명령어들을 실행해주세요.  (탭은 터미널 화면 좌측 상단에 있습니다.)  

마지막으로 한 가지 /etc/hosts 파일에 ingress host를 등록해줍니다.

```bash
controlplane $ echo '127.0.0.1 my-nginx.info' >> /etc/hosts
```

> 💻 명령어 `echo '127.0.0.1 my-nginx.info' >> /etc/hosts`{{exec}}

이제 curl 명령어를 이용해서 연결해볼까요?

```bash
controlplane $ curl http://my-nginx.info
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

> 💻 명령어 `curl http://my-nginx.info`{{exec}}

잘 되네요. (ง˙∇˙)ว

**nginx-ingress.yaml** 파일의 path부분을 `/` 에서 `/test` 처럼 바꾸면 어떻게 될까요?  
한 번 해보세요.

<br>

끝~