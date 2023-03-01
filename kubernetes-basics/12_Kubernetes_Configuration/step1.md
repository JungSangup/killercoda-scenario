**ToDo App**을 또 다른 방법으로 실행해보려고 합니다.  
**Docker Network** 실습에서 했던 방법과 동일하게

- MySQL
- Todo App

을 각각 실행해서 연동하려고 합니다.  
이렇게요.

![](./img/multi-app-architecture.png)

기본적인 구조와 방법은 **Docker Network** 실습과 같고, Kubernetes환경에 맞게 리소스들을 생성해서 해볼게요.

<br>

먼저 MySQL을 실행합니다.
Docker에서는 아래와 같이 했습니다.
```bash
ubuntu $ docker run -d \
    --network todo-app --network-alias mysql \
    --volume todo-mysql-data:/var/lib/mysql \
    --env MYSQL_ROOT_PASSWORD=secret \
    --env MYSQL_DATABASE=todos \
    --env LANG=C.UTF-8 \
    --name my-mysql \
    mysql:5.7 \
    --character-set-server=utf8mb4 \
    --collation-server=utf8mb4_unicode_ci
c9d83cbd2ac8941da32d8d64103223fe1c6937c9c28507c6e19ed91fca740c98
```
> 환경변수를 이용해서 MySQL 설정을 하고, user-defined bridge 네트워크를 이용했습니다.

쿠버네티스에서는 설정에 필요한 환경변수를 **ConfigMap**과 **Secret**으로 만들어서 사용해볼게요.

- **ConfigMap**으로 만들 환경변수
  - MYSQL_DATABASE
  - LANG
- **Secret**으로 만들 환경변수
  - MYSQL_ROOT_PASSWORD

<br><br><br>

ConfigMap과 Secret은 각각 아래와 같이 준비합니다.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  database: "todos"
  lang: "C.UTF-8"
```
> 파일명은 **mysql-configmap.yaml**로 합니다.

<br>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  password: c2VjcmV0
```
> 파일명은 **mysql-secret.yaml**로 합니다.

<br><br><br>

password("c2VjcmV0")는 "secret"을 base64 encoding한 것입니다.  
base64 encoding 은 아래처럼 하면 됩니다.
```bash
controlplane $ echo -n 'secret' | base64
c2VjcmV0
```

<br><br><br>

ConfigMap과 Secret을 위한 파일들이 준비됐으면, 다음은 pv와 pvc를 위한 파일도 하나 준비해주세요.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mysql"
```
> 파일명은 **mysql-pv.yaml**로 합니다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
> 파일명은 **mysql-pvc.yaml**로 합니다.

<br><br><br>

그리고, 이제 다들 아시겠지만 Service도 만들어야하니 준비해 주시구요.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: todo-mysql-svc
spec:
  type: ClusterIP
  selector:
    app: todo-mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```
> 파일명은 **mysql-clusterip-service.yaml**로 합니다.

<br><br><br>

마지막으로 Deployment를 준비합니다.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: todo-mysql
  template:
    metadata:
      labels:
        app: todo-mysql
    spec:
      containers:
      - name: todo-mysql-pod
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: database
        - name: LANG
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: lang
        ports:
        - containerPort: 3306
        args: ["--character-set-server=utf8mb4", "--collation-server=utf8mb4_unicode_ci"]
        volumeMounts:
          - mountPath: "/var/lib/mysql"
            name: mysql-storage
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```
> 파일명은 **mysql-deployment.yaml**로 합니다.

<br><br><br>

이제 하나씩 생성해줍니다.  
모두 여섯 개의 리소스를 생성합니다.
```bash
controlplane $ kubectl apply -f mysql-configmap.yaml
configmap/mysql-config created
controlplane $ kubectl apply -f mysql-secret.yaml
secret/mysql-secret created
controlplane $ kubectl apply -f mysql-pv.yaml
persistentvolume/mysql-pv created
controlplane $ kubectl apply -f mysql-pvc.yaml
persistentvolumeclaim/mysql-pvc created
controlplane $ kubectl apply -f mysql-clusterip-service.yaml
service/todo-mysql-svc created
controlplane $ kubectl apply -f mysql-deployment.yaml
deployment.apps/todo-mysql-deployment created
```

> 💻 명령어
>```bash
>kubectl apply -f mysql-configmap.yaml
>kubectl apply -f mysql-secret.yaml
>kubectl apply -f mysql-pv.yaml
>kubectl apply -f mysql-pvc.yaml
>kubectl apply -f mysql-clusterip-service.yaml
>kubectl apply -f mysql-deployment.yaml
>```{{exec}}

<br><br><br>

ConfigMap이 잘 생성됐나 볼까요?
```bash
controlplane $ kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      5d17h
mysql-config       2      55s
controlplane $ kubectl describe configmaps mysql-config
Name:         mysql-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
lang:
----
C.UTF-8
database:
----
todos

BinaryData
====

Events:  <none>
```

> 💻 명령어 `kubectl get configmaps`{{exec}}  
> 💻 명령어 `kubectl describe configmaps mysql-config`{{exec}}  

lang(C.UTF-8)과 database(todos)두 개의 데이터가 보입니다.

<br><br><br>

Secret도 볼게요.
```bash
controlplane $ kubectl get secrets
NAME           TYPE     DATA   AGE
mysql-secret   Opaque   1      75s
controlplane $ kubectl describe secrets mysql-secret
Name:         mysql-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
```

> 💻 명령어 `kubectl get secrets`{{exec}}  
> 💻 명령어 `kubectl describe secrets mysql-secret`{{exec}}

Secret의 Data는 값이 보이지는 않네요.
Secret이니까요. -_-

<br><br><br>

Pod도 확인해보세요. (Environment, Mounts, Volumes 부분을 잘 보세요.)
```bash
controlplane $ kubectl get pod
NAME                                     READY   STATUS    RESTARTS   AGE
todo-mysql-deployment-7cf4865b86-gx4lc   1/1     Running   0          108s
controlplane $ kubectl describe pod todo-mysql-deployment-7cf4865b86-gx4lc
Name:             todo-mysql-deployment-7cf4865b86-gx4lc
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Wed, 01 Mar 2023 06:18:42 +0000
Labels:           app=todo-mysql
                  pod-template-hash=7cf4865b86
Annotations:      cni.projectcalico.org/containerID: 16b783b053e91d46942cc6ac73989ea73086013aa2f6ab8489192a87628f14da
                  cni.projectcalico.org/podIP: 192.168.1.3/32
                  cni.projectcalico.org/podIPs: 192.168.1.3/32
Status:           Running
IP:               192.168.1.3
IPs:
  IP:           192.168.1.3
Controlled By:  ReplicaSet/todo-mysql-deployment-7cf4865b86
Containers:
  todo-mysql-pod:
    Container ID:  containerd://862829d266a54371c4d19d971e23955772b1bc580d90124e73210466407a3919
    Image:         mysql:5.7
    Image ID:      docker.io/library/mysql@sha256:8cf035b14977b26f4a47d98e85949a7dd35e641f88fc24aa4b466b36beecf9d6
    Port:          3306/TCP
    Host Port:     0/TCP
    Args:
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
    State:          Running
      Started:      Wed, 01 Mar 2023 06:18:56 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-secret'>      Optional: false
      MYSQL_DATABASE:       <set to the key 'database' of config map 'mysql-config'>  Optional: false
      LANG:                 <set to the key 'lang' of config map 'mysql-config'>      Optional: false
    Mounts:
      /var/lib/mysql from mysql-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hdlqf (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  mysql-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mysql-pvc
    ReadOnly:   false
  kube-api-access-hdlqf:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  119s  default-scheduler  Successfully assigned default/todo-mysql-deployment-7cf4865b86-gx4lc to node01
  Normal  Pulling    119s  kubelet            Pulling image "mysql:5.7"
  Normal  Pulled     105s  kubelet            Successfully pulled image "mysql:5.7" in 13.447227849s (13.447233165s including waiting)
  Normal  Created    105s  kubelet            Created container todo-mysql-pod
  Normal  Started    105s  kubelet            Started container todo-mysql-pod
```

> 💻 명령어 `kubectl get pod`{{exec}}  
> 💻 명령어 `kubectl describe pod [POD-NAME]`{{copy}}
> - [POD-NAME] 에는 MySQL POD의 이름을 넣어주세요.

<br><br><br>

이제 두 번재 워크로드인 **ToDo App**을 실행해볼까요?  
MySQL과 마찬가지로 ConfigMap과 Secret을 사용하고, 외부에서 접속을 해야하니 Ingress까지 만들어볼게요.

이번엔 몽땅 하나의 yaml파일로 만들어 보겠습니다.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: todo-secret
type: Opaque
data:
  mysql-user: cm9vdA==
  mysql-password: c2VjcmV0
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: todo-config
data:
  mysql-host: "todo-mysql-svc.default.svc.cluster.local"
  mysql-db: "todos"
---
apiVersion: v1
kind: Service
metadata:
  name: todo-app-svc
spec:
  type: NodePort
  selector:
    app: todo-app
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30007
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-app-deployment
  labels:
    app: todo-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: todo-app
  template:
    metadata:
      labels:
        app: todo-app
    spec:
      containers:
      - name: todo-app
        image: [USER-NAME]/todo-app:1.0.0
        env:
        - name: MYSQL_HOST
          valueFrom:
            configMapKeyRef:
              name: todo-config
              key: mysql-host
        - name: MYSQL_DB
          valueFrom:
            configMapKeyRef:
              name: todo-config
              key: mysql-db
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: todo-secret
              key: mysql-user
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: todo-secret
              key: mysql-password
        ports:
        - containerPort: 3000
      imagePullSecrets:
      - name: regcred
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todo-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: todo-app.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: todo-app-svc
                port:
                  number: 3000
```
> 파일명은 **todo-all.yaml**로 합니다.  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요. (52번째 라인 -> Deployment의 .spec.containers[0].image 입니다.)  
> - 만약 [USER-NAME]/todo-app:1.0.0가 준비가 안되어있다면, **rogallo/101-todo-app:1.0.0**을 사용해주세요.

<br>

> 하나의 yaml파일 안에 여러개의 K8s [Manifest](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-manifest)를 정의할때는, `---`를 구분자로 해서 여러개를 담으면 됩니다.

<br><br><br>

생성하기 전에 한 가지 더 해줘야할 일이 있습니다.
이 과정은 Public repository의 이미지인 **rogallo/101-todo-app:1.0.0**를 사용하는 경우는 생략해도 됩니다.

컨테이너 이미지([USER-NAME]/todo-app:1.0.0)를 여러분의 private repository에서 pull해야하기 때문에, **자격증명**을 해야합니다.  
여러가지 방법이 있지만, 간단하게 [커맨드 라인에서 자격 증명을 통하여 시크릿 생성하기](https://kubernetes.io/ko/docs/tasks/configure-pod-container/pull-image-private-registry/#%EC%BB%A4%EB%A7%A8%EB%93%9C-%EB%9D%BC%EC%9D%B8%EC%97%90%EC%84%9C-%EC%9E%90%EA%B2%A9-%EC%A6%9D%EB%AA%85%EC%9D%84-%ED%86%B5%ED%95%98%EC%97%AC-%EC%8B%9C%ED%81%AC%EB%A6%BF-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0)방법으로 해볼게요.

```bash
controlplane $ kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=rogallo --docker-password=XXXXXX    
secret/regcred created
```

> 💻 명령어 `kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=[USER-NAME] --docker-password=[PASSWORD]`{{copy}}  
> [USER-NAME]과 [PASSWORD]는 여러분의 정보로 채워넣어 주세요.

이것도 많이 쓰이는 Secret의 용도 중 하나입니다.  
조회도 한 번 해보세요.
```bash
controlplane $ kubectl describe secrets regcred
Name:         regcred
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  114 bytes
```

> 💻 명령어 `kubectl describe secrets regcred`{{exec}}

<br><br><br>

이제 ToDo App을 생성해볼게요.  
생성은 아래처럼 한 번에 됩니다.
```bash
controlplane $ kubectl apply -f todo-all.yaml
secret/todo-secret created
configmap/todo-config created
service/todo-app-svc created
deployment.apps/todo-app-deployment created
ingress.networking.k8s.io/todo-app-ingress created
```

> 💻 명령어 `kubectl apply -f todo-all.yaml`{{exec}}

<br><br><br>

앞서 MySQL에서 한 것과 비슷하게 ConfigMap, Secret, Pod도 확인해보세요.  
명령어만 알려드릴게요.
> 💻 명령어 `kubectl describe configmaps todo-config`{{exec}}  
> 💻 명령어 `kubectl describe secrets todo-secret`{{exec}}  
> 💻 명령어 `kubectl get pods`{{exec}}  
> 💻 명령어 `kubectl describe pod [POD-NAME]`{{copy}}  
> [POD-NAME] 에는 ToDo App POD중 하나의 이름을 넣어주세요.

<br><br><br>

POD의 환경변수를 확인하려면 아래처럼도 가능합니다.
```bash
controlplane $ kubectl exec -it todo-app-deployment-69b59d644-4t6j8 -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=todo-app-deployment-69b59d644-4t6j8
NODE_VERSION=10.24.1
YARN_VERSION=1.22.5
MYSQL_HOST=todo-mysql-svc.default.svc.cluster.local
MYSQL_DB=todos
MYSQL_USER=root
MYSQL_PASSWORD=secret
TODO_APP_SVC_SERVICE_PORT=3000
TODO_APP_SVC_PORT_3000_TCP=tcp://10.96.150.70:3000
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT_443_TCP_PORT=443
TODO_MYSQL_SVC_PORT_3306_TCP=tcp://10.105.231.101:3306
TODO_MYSQL_SVC_PORT_3306_TCP_PROTO=tcp
TODO_APP_SVC_SERVICE_HOST=10.96.150.70
KUBERNETES_PORT_443_TCP_PROTO=tcp
TODO_MYSQL_SVC_SERVICE_PORT=3306
TODO_MYSQL_SVC_PORT_3306_TCP_PORT=3306
TODO_MYSQL_SVC_PORT_3306_TCP_ADDR=10.105.231.101
TODO_APP_SVC_PORT_3000_TCP_PORT=3000
TODO_APP_SVC_PORT_3000_TCP_ADDR=10.96.150.70
TODO_MYSQL_SVC_SERVICE_HOST=10.105.231.101
TODO_MYSQL_SVC_PORT=tcp://10.105.231.101:3306
TODO_APP_SVC_PORT=tcp://10.96.150.70:3000
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
TODO_APP_SVC_PORT_3000_TCP_PROTO=tcp
TERM=xterm
HOME=/root
```

> 💻 명령어 `kubectl exec -it [POD-NAME] -- env`{{copy}}  
> [POD-NAME] 에는 ToDo App POD중 하나의 이름을 넣어주세요.

- [kubectl exec](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec) 명령어의 사용방법은 [동작중인 컨테이너의 셸에 접근하기](https://kubernetes.io/ko/docs/tasks/debug/debug-application/get-shell-running-container/)를 참고하세요.

<br><br><br>

이제 브라우저에서 어떻게 나오나 볼까요?

🔗 [ToDo List Manager]({{TRAFFIC_HOST1_30007}})

<br><br><br>

MySQL DB의 테이블에 잘 저장됐는지도 확인해보세요.
```bash
controlplane $ kubectl exec -it todo-mysql-deployment-7cf4865b86-gx4lc -- mysql -p todos
Enter password: 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.7.41 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select * from todo_items;
+--------------------------------------+-----------------------+-----------+
| id                                   | name                  | completed |
+--------------------------------------+-----------------------+-----------+
| 4f7c3980-640d-4bec-a04a-467a55fa1de5 | 엠에스피티쓰리            |         0 |
| 2f6da7dd-859d-4a47-8991-908e239cc1b8 | mspt3                 |         0 |
+--------------------------------------+-----------------------+-----------+
2 rows in set (0.01 sec)

mysql> exit;
Bye
```

> 💻 명령어 `kubectl exec -it [POD-NAME] -- mysql -p todos`{{copy}}  
> [POD-NAME] 에는 MySQL POD의 이름을 넣어주세요.

<br><br><br>

이것저것 확인해보시고, 마지막은 생성된 리소스들을 정리해주세요.
```bash
controlplane $ kubectl delete -f todo-all.yaml
secret "todo-secret" deleted
configmap "todo-config" deleted
service "todo-app-svc" deleted
deployment.apps "todo-app-deployment" deleted
ingress.networking.k8s.io "todo-app-ingress" deleted
controlplane $ kubectl delete secret regcred
secret "regcred" deleted
controlplane $ kubectl delete -f mysql-deployment.yaml
deployment.apps "todo-mysql-deployment" deleted
controlplane $ kubectl delete -f mysql-clusterip-service.yaml
service "todo-mysql-svc" deleted
controlplane $ kubectl delete -f mysql-pvc.yaml
persistentvolumeclaim "mysql-pvc" deleted
controlplane $ kubectl delete -f mysql-pv.yaml
persistentvolume "mysql-pv" deleted
controlplane $ kubectl delete -f mysql-secret.yaml
secret "mysql-secret" deleted
controlplane $ kubectl delete -f mysql-configmap.yaml
configmap "mysql-config" deleted
```
> 💻 명령어
>```bash
>kubectl delete -f todo-all.yaml
>kubectl delete secret regcred
>kubectl delete -f mysql-deployment.yaml
>kubectl delete -f mysql-clusterip-service.yaml
>kubectl delete -f mysql-pvc.yaml
>kubectl delete -f mysql-pv.yaml
>kubectl delete -f mysql-secret.yaml
>kubectl delete -f mysql-configmap.yaml
>```{{exec}}

<br>

이번 실습은 여기까지 입니다.   ＿〆(。╹‿ ╹ 。)