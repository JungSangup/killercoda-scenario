이번엔 **NodePort**입니다.

Node의 특정 Port를 이용하는 방식입니다.

먼저 NodePort 타입의 서비스를 하나 만들어 보겠습니다.
아래와 같은 파일을 준비해주세요. (~/lab 디렉토리 아래의 파일을 사용하셔도 됩니다.)

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

> 파일명은 nginx-nodeport-service.yaml로 합니다.

---

그리고, 생성하고 조회까지 해볼게요.

```bash
controlplane $ kubectl apply -f nginx-nodeport-service.yaml
service/nginx-nodeport-service created

controlplane $ kubectl get services
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes                ClusterIP   10.96.0.1      <none>        443/TCP        70d
nginx-clusterip-service   ClusterIP   10.106.200.8   <none>        80/TCP         6m14s
nginx-nodeport-service    NodePort    10.97.15.144   <none>        80:30007/TCP   10s
```

> **명령어** : `kubectl apply -f nginx-nodeport-service.yaml`{{exec}} , `kubectl get services`{{exec}}  

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `kubectl apply -f ~/lab/nginx-nodeport-service.yaml`{{exec}} , `kubectl get services`{{exec}}


**ClusterIP**와 **NodePort** 유형의 Service간 차이가 보이시나요? (힌트 : PORT(S))

**NodePort**는 아래와 같은 방법으로 접근 가능합니다.

먼저 Node의 IP를 알아야합니다.

```bash
controlplane $ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   70d   v1.24.0   172.30.1.2    <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic   containerd://1.5.9
node01         Ready    <none>          70d   v1.24.0   172.30.2.2    <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic   containerd://1.5.9
```

> **명령어** : `kubectl get nodes -o wide`{{exec}}

---

이제는 Node의 IP를 통해서 내부의 Pod로 연결이 가능합니다.
Node까지의 경로가 열려있다면 어디서든 이 IP로 접근 가능합니다.

```bash
controlplane $ curl http://172.30.2.2:30007
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

> **명령어** : `curl http://[NODE_IP]:30007`{{copy}}  
> [NODE_IP] 는 Node의 IP

같은 클러스터 내에 있는 다른 Node의 IP,Port로도 해보세요. 될까요?   ＿φ(°-°=)