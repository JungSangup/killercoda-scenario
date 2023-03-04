우리가 익숙한 **ToDo App**을 이용해서 좀 더 자세히 볼게요.  
차트는 아래와 같은 구조를 가지고 있습니다. 우리가 배운 여러가지가 다 들어있네요.

![h:500](./img/helm_todo_app.png)

<br><br><br>

설치는 간단합니다. 명령어 하나면 끝. （°o°；）
```bash
ubuntu@ip-172-31-23-60:~$ helm install my-todo-app https://github.com/JungSangup/mspt3/raw/main/hands_on_files/todo-app-1.0.0.tgz
NAME: my-todo-app
LAST DEPLOYED: Thu Feb 16 08:19:57 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  http://todo-app.info/
```

> 💻 명령어
>```bash
>helm install my-todo-app https://github.com/JungSangup/mspt3/raw/main/hands_on_files/todo-app-1.0.0.tgz
>```

<br>

위의 방법은 Helm chart 패키지 파일의 URL(깃헙에 올려놓은 파일)을 직접 지정해서 설치한 것입니다.  
위의 방법 외에도 아래와 같은 다양한 방법으로 설치 가능합니다. 
> - `helm install my-todo-app ./todo-app-1.0.0.tgz` -> 로컬 경로의 tgz파일(패키징 된 Helm chart)
> - `helm install my-todo-app ./todo-app` -> 로컬 경로의 차트 디렉토리  
- **hands_on_files** 아래에 위의 두 가지 방법을 위한 파일/디렉토리도 준비해 놓았습니다.

<br><br><br>

우선 이 Helm release는 Uninstall을 할게요. 뒤에 다른 방법으로 다시 설치하겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm uninstall my-todo-app
release "my-todo-app" uninstalled
```

> 💻 명령어
>```bash
>helm uninstall my-todo-app
>```

<br>

정리 후 상태는 아래와 같습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
ubuntu@ip-172-31-28-216:~$
```

> 💻 명령어
>```bash
>helm list
>```

<br><br><br>

이번에는 구성을 조금 달리해서 설치하겠습니다.  
여러분의 **Docker private repository**에 올려놓은 이미지를 사용하도록 하고, 이미지 pull을 위해서 **자격증명**을 사용하도록 할게요.

역시 아래와 같이 간단하게 실행할 수 있습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm install my-todo-app \
>     --set image.repository=rogallo/todo-app \
>     --set imageCredentials.create=true \
>     --set imageCredentials.username=rogallo \
>     --set imageCredentials.password=XXXXXX \
>     https://github.com/JungSangup/mspt3/raw/main/hands_on_files/todo-app-1.0.0.tgz
NAME: my-todo-app
LAST DEPLOYED: Thu Feb 16 08:31:07 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  http://todo-app.info/
```

> 💻 명령어
>```bash
>helm install my-todo-app \
>     --set image.repository=[USER-NAME]/todo-app \
>     --set imageCredentials.create=true \
>     --set imageCredentials.username=[USER-NAME] \
>     --set imageCredentials.password=[PASSWORD] \
>     https://github.com/JungSangup/mspt3/raw/main/hands_on_files/todo-app-1.0.0.tgz
>```
> [USER-NAME]과 [PASSWORD]는 여러분의 정보로 채워넣어 주세요.

설치 시점에 아래 키-값 들을 변경해서 적용한 것입니다.  
image.repository는 여러분의 Private repository에서 pull해서 사용하도록 하고, imageCredentials 값들을 이용해서 도커허브 자격증명을 위햔 Secret을 생성합니다.
- image.repository=[USER-NAME]/todo-app
- imageCredentials.create=true
- imageCredentials.username=[USER-NAME]
- imageCredentials.password=[PASSWORD]

<br><br><br>

브라우저에서 http://todo-app.info/ 로 접속해서 테스트도 해보세요.

![h:400](./img/k8s_todo_ingress.png)

잘 되나요?

<br><br><br>

생성된 K8s 리소스들도 확인해보세요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/my-todo-app-6b8b4887d5-l5kwf        1/1     Running   0          3m41s
pod/my-todo-app-mysql-7d8c985b5-5kwk2   1/1     Running   0          3m41s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP    4d17h
service/my-todo-app         ClusterIP   10.99.25.35      <none>        3000/TCP   3m41s
service/my-todo-app-mysql   ClusterIP   10.101.118.220   <none>        3306/TCP   3m41s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-todo-app         1/1     1            1           3m41s
deployment.apps/my-todo-app-mysql   1/1     1            1           3m41s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/my-todo-app-6b8b4887d5        1         1         1       3m41s
replicaset.apps/my-todo-app-mysql-7d8c985b5   1         1         1       3m41s

NAME                                              REFERENCE                TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/my-todo-app   Deployment/my-todo-app   <unknown>/80%   1         10        1          3m41s
```

> 💻 명령어
>```bash
>kubectl get all
>```

<br><br><br>

private repository의 이미지를 pull 하기 위해서 자격증명도 secret으로 생성했습니다.
```base
ubuntu@ip-172-31-23-60:~$ kubectl describe secrets regcred
Name:         regcred
Namespace:    default
Labels:       app.kubernetes.io/managed-by=Helm
Annotations:  meta.helm.sh/release-name: my-todo-app
              meta.helm.sh/release-namespace: default

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  135 bytes
```

> 💻 명령어
>```bash
>kubectl describe secrets regcred
>```

<br><br><br>

다른 리소스들 (ConfitMap, Secret, PVC, PV, Ingress) 도 한 번 확인해보세요.
```base
ubuntu@ip-172-31-23-60:~$ kubectl get configmaps
NAME                 DATA   AGE
kube-root-ca.crt     1      4d18h
my-todo-app-config   2      5m5s
mysql-config         2      5m5s
ubuntu@ip-172-31-23-60:~$ kubectl get secrets
NAME                                TYPE                                  DATA   AGE
default-token-dskrr                 kubernetes.io/service-account-token   3      4d18h
my-todo-app-mysql-token-d25jp       kubernetes.io/service-account-token   3      5m10s
my-todo-app-secret                  Opaque                                2      5m10s
my-todo-app-token-5n24p             kubernetes.io/service-account-token   3      5m10s
mysql-secret                        Opaque                                1      5m10s
regcred                             kubernetes.io/dockerconfigjson        1      5m10s
sh.helm.release.v1.my-todo-app.v1   helm.sh/release.v1                    1      5m10s
ubuntu@ip-172-31-23-60:~$ kubectl get pvc
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-my-wordpress-mariadb-0   Bound    pvc-94d614ca-8cb3-499c-a76f-35e73eca936e   8Gi        RWO            standard       28m
mysql-pvc                     Bound    pvc-775be31c-d9b9-4e21-b55c-8280af4e342f   3Gi        RWO            standard       5m14s
ubuntu@ip-172-31-23-60:~$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS   REASON   AGE
pvc-775be31c-d9b9-4e21-b55c-8280af4e342f   3Gi        RWO            Delete           Bound    default/mysql-pvc                     standard                5m18s
pvc-94d614ca-8cb3-499c-a76f-35e73eca936e   8Gi        RWO            Delete           Bound    default/data-my-wordpress-mariadb-0   standard                28m
ubuntu@ip-172-31-23-60:~$ kubectl get ingress
NAME          CLASS   HOSTS           ADDRESS        PORTS   AGE
my-todo-app   nginx   todo-app.info   172.31.23.60   80      5m23s
```

> 💻 명령어
>```bash
>kubectl get configmaps
>```
>```bash
>kubectl get secrets
>```
>```bash
>kubectl get pvc
>```
>```bash
>kubectl get pv
>```
>```bash
>kubectl get ingress
>```

<br><br><br>

이제 Helm 에서 **업그레이드**를 해볼게요.  
여러가지 업그레이드가 있겠지만, 간단히 이미지의 Tag를 변경하는 경우만 해보겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm upgrade --set image.tag=2.0.0 my-todo-app https://github.com/JungSangup/mspt3/raw/main/hands_on_files/todo-app-1.0.0.tgz
Release "my-todo-app" has been upgraded. Happy Helming!
NAME: my-todo-app
LAST DEPLOYED: Thu Feb 16 10:41:50 2023
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  http://todo-app.info/
```

> 💻 명령어
>```bash
>helm upgrade --set image.tag=2.0.0 my-todo-app https://github.com/JungSangup/mspt3/raw/main/hands_on_files/todo-app-1.0.0.tgz
>```
> **image.tag**만 변경해서 새로운 버젼으로 업그레이드 합니다.

<br><br><br>

바뀐 Deployment도 확인해 보시구요.
이렇게요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl describe deployments my-todo-app | grep Image
    Image:      rogallo/101-todo-app:2.0.0
```

> 💻 명령어
>```bash
>kubectl describe deployments my-todo-app | grep Image
>```

<br><br><br>

브라우저에서 http://todo-app.info/로 접속해서 업그레이드 결과도 보세요.

![h:400](./img/k8s_todo_ingress2.png)

> 구분하기 위해서 하단에 버젼을 표시하도록 해 놓았습니다.

<br><br><br>

롤백도 해볼까요?  
간단히 History를 조회하고, 원하는 **Revision**으로 돌아가면 됩니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm history my-todo-app
REVISION	UPDATED                 	STATUS    	CHART         	APP VERSION	DESCRIPTION
1       	Thu Feb 16 08:31:07 2023	superseded	todo-app-1.0.0	1.0.0      	Install complete
2       	Thu Feb 16 10:41:50 2023	deployed  	todo-app-1.0.0	1.0.0      	Upgrade complete
ubuntu@ip-172-31-23-60:~$ helm rollback my-todo-app 1
Rollback was a success! Happy Helming!
ubuntu@ip-172-31-23-60:~$ kubectl describe deployments my-todo-app | grep Image
    Image:      rogallo/todo-app:1.0.0
```

> 💻 명령어
>```bash
>helm history my-todo-app
>```
>```bash
>helm rollback my-todo-app 1
>```
>```bash
>kubectl describe deployments my-todo-app | grep Image
>```

http://todo-app.info/ 화면도 확인 해 보시구요.

<br><br><br>

역시 마지막은 정리.   
아래 명령어로 삭제(Uninstall) 해 주세요.

```bash
ubuntu@ip-172-31-23-60:~$ helm uninstall my-todo-app
release "my-todo-app" uninstalled
```

> 💻 명령어
>```bash
>helm uninstall my-todo-app
>```

<br>

정리 후 상태는 아래와 같습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
ubuntu@ip-172-31-28-216:~$
```

> 💻 명령어
>```bash
>helm list
>```

한꺼번에 설치(install), 업데이트(update), 롤백(rollback), 삭제(uninstall)되니 편하네요.

여기까지 Helm 에 대해 알아보았습니다.

수고하셨습니다. (〃･ิ‿･ิ)ゞ