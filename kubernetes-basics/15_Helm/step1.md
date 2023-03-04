자주 사용되는 Helm 명령어들을 실습해 보겠습니다.  
먼저 어떤 명령어들이 있는지 살펴볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ helm --help
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

...생략...

Helm stores cache, configuration, and data based on the following configuration order:

- If a HELM_*_HOME environment variable is set, it will be used
- Otherwise, on systems supporting the XDG base directory specification, the XDG variables will be used
- When no other location is set a default location will be used based on the operating system

By default, the default directories depend on the Operating System. The defaults are listed below:

| Operating System | Cache Path                | Configuration Path             | Data Path               |
|------------------|---------------------------|--------------------------------|-------------------------|
| Linux            | $HOME/.cache/helm         | $HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                 | %APPDATA%\helm          |

Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies

... 생략 ...

Use "helm [command] --help" for more information about a command.
```

> 💻 명령어
>```bash
>helm --help
>```

<br><br><br>

**Common actions for Helm** 의 명령어들을 하나씩 해볼까요?
```bash
Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts
```

첫 번째는 `helm search`인데, 그 전에 **helm repository**를 먼저 추가(**add**)해줘야 합니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

> 💻 명령어
>```bash
>helm repo add bitnami https://charts.bitnami.com/bitnami
>```

<br><br><br>

Repository 목록도 볼 수 있습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm repo list
NAME   	URL
bitnami	https://charts.bitnami.com/bitnami
```

> 💻 명령어
>```bash
>helm repo list
>```

<br><br><br>

이제 검색(`helm search`) 가능합니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm search repo bitnami
NAME                                        	CHART VERSION	APP VERSION  	DESCRIPTION
bitnami/airflow                             	14.0.11      	2.5.1        	Apache Airflow is a tool to express and execute...
bitnami/apache                              	9.2.15       	2.4.55       	Apache HTTP Server is an open-source HTTP serve...
bitnami/appsmith                            	0.1.12       	1.9.7        	Appsmith is an open source platform for buildin...
bitnami/argo-cd                             	4.4.9        	2.6.1        	Argo CD is a continuous delivery tool for Kuber...
bitnami/argo-workflows                      	5.1.6        	3.4.5        	Argo Workflows is meant to orchestrate Kubernet...
bitnami/aspnet-core                         	4.0.5        	7.0.3        	ASP.NET Core is an open-source framework for we...
bitnami/cassandra                           	10.0.2       	4.1.0        	Apache Cassandra is an open source distributed ...
bitnami/cert-manager                        	0.9.0        	1.11.0       	cert-manager is a Kubernetes add-on to automate...
bitnami/clickhouse                          	3.0.1        	23.1.3       	ClickHouse is an open-source column-oriented OL...
bitnami/common                              	2.2.3        	2.2.3        	A Library Helm Chart for grouping common logic ...
bitnami/concourse                           	2.0.3        	7.9.0        	Concourse is an automation system written in Go...
bitnami/consul                              	10.9.11      	1.14.4       	HashiCorp Consul is a tool for discovering and ...
bitnami/contour                             	10.2.2       	1.23.3       	Contour is an open source Kubernetes ingress co...
bitnami/contour-operator                    	3.0.3        	1.23.0       	The Contour Operator extends the Kubernetes API...
...생략...
```

> 💻 명령어
>```bash
>helm search repo bitnami
>```

<br><br><br>

Wordpress를 한 번 찾아볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ helm search repo wordpress
NAME                   	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/wordpress      	15.2.42      	6.1.1      	WordPress is the world's most popular blogging ...
bitnami/wordpress-intel	2.1.31       	6.1.1      	DEPRECATED WordPress for Intel is the most popu...
```

> 💻 명령어
>```bash
>helm search repo wordpress
>```

<br><br><br>

다음은 `helm pull` 명령어 입니다.  
**Helm repository**에 등록되어 있는 Helm chart를 다운로드(pull)하는 명령어 입니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm pull bitnami/wordpress --version 15.0.7
ubuntu@ip-172-31-23-60:~$ ls wordpress*
wordpress-15.0.7.tgz
```

> 💻 명령어
>```bash
>helm pull bitnami/wordpress --version 15.0.7
>```
>```bash
>ls wordpress*
>```

<br><br><br>

tar 파일로 받아지네요.  
압축도 풀어볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ tar -xvf wordpress-15.0.7.tgz
wordpress/Chart.yaml
wordpress/Chart.lock
wordpress/values.yaml
wordpress/values.schema.json
wordpress/templates/NOTES.txt
wordpress/templates/_helpers.tpl
wordpress/templates/config-secret.yaml
wordpress/templates/deployment.yaml
...생략...
```

> 💻 명령어
>```bash
>tar -xvf wordpress-15.0.7.tgz
>```

<br><br><br>

어떤 파일들이 있는지 한 번 살펴보겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ tree ./wordpress
./wordpress
├── Chart.lock
├── Chart.yaml
├── README.md
├── charts
│   ├── common
│   │   ├── Chart.yaml
│   │   ├── README.md
│   │   ├── templates
│   │   │   ├── _affinities.tpl
│   │   │   ├── _capabilities.tpl
│   │   │   ├── _errors.tpl
│   │   │   ├── _images.tpl
│   │   │   ├── _ingress.tpl
│   │   │   ├── _labels.tpl
│   │   │   ├── _names.tpl
│   │   │   ├── _secrets.tpl
│   │   │   ├── _storage.tpl
│   │   │   ├── _tplvalues.tpl
│   │   │   ├── _utils.tpl
│   │   │   ├── _warnings.tpl
│   │   │   └── validations
│   │   │       ├── _cassandra.tpl
│   │   │       ├── _mariadb.tpl
│   │   │       ├── _mongodb.tpl
│   │   │       ├── _mysql.tpl
│   │   │       ├── _postgresql.tpl
│   │   │       ├── _redis.tpl
│   │   │       └── _validations.tpl
│   │   └── values.yaml
│   ├── mariadb
│   │   ├── Chart.lock
│   │   ├── Chart.yaml
│   │   ├── README.md
│   │   ├── charts
│   │   │   └── common
│   │   │       ├── Chart.yaml
│   │   │       ├── README.md
│   │   │       ├── templates
│   │   │       │   ├── _affinities.tpl
│   │   │       │   ├── _capabilities.tpl
│   │   │       │   ├── _errors.tpl
│   │   │       │   ├── _images.tpl
│   │   │       │   ├── _ingress.tpl
│   │   │       │   ├── _labels.tpl
│   │   │       │   ├── _names.tpl
│   │   │       │   ├── _secrets.tpl
│   │   │       │   ├── _storage.tpl
│   │   │       │   ├── _tplvalues.tpl
│   │   │       │   ├── _utils.tpl
│   │   │       │   ├── _warnings.tpl
│   │   │       │   └── validations
│   │   │       │       ├── _cassandra.tpl
│   │   │       │       ├── _mariadb.tpl
│   │   │       │       ├── _mongodb.tpl
│   │   │       │       ├── _mysql.tpl
│   │   │       │       ├── _postgresql.tpl
│   │   │       │       ├── _redis.tpl
│   │   │       │       └── _validations.tpl
│   │   │       └── values.yaml
│   │   ├── templates
│   │   │   ├── NOTES.txt
│   │   │   ├── _helpers.tpl
│   │   │   ├── extra-list.yaml
│   │   │   ├── networkpolicy-egress.yaml
│   │   │   ├── primary
│   │   │   │   ├── configmap.yaml
│   │   │   │   ├── initialization-configmap.yaml
│   │   │   │   ├── networkpolicy-ingress.yaml
│   │   │   │   ├── pdb.yaml
│   │   │   │   ├── statefulset.yaml
│   │   │   │   └── svc.yaml
│   │   │   ├── prometheusrules.yaml
│   │   │   ├── role.yaml
│   │   │   ├── rolebinding.yaml
│   │   │   ├── secondary
│   │   │   │   ├── configmap.yaml
│   │   │   │   ├── networkpolicy-ingress.yaml
│   │   │   │   ├── pdb.yaml
│   │   │   │   ├── statefulset.yaml
│   │   │   │   └── svc.yaml
│   │   │   ├── secrets.yaml
│   │   │   ├── serviceaccount.yaml
│   │   │   └── servicemonitor.yaml
│   │   ├── values.schema.json
│   │   └── values.yaml
│   └── memcached
│       ├── Chart.lock
│       ├── Chart.yaml
│       ├── README.md
│       ├── charts
│       │   └── common
│       │       ├── Chart.yaml
│       │       ├── README.md
│       │       ├── templates
│       │       │   ├── _affinities.tpl
│       │       │   ├── _capabilities.tpl
│       │       │   ├── _errors.tpl
│       │       │   ├── _images.tpl
│       │       │   ├── _ingress.tpl
│       │       │   ├── _labels.tpl
│       │       │   ├── _names.tpl
│       │       │   ├── _secrets.tpl
│       │       │   ├── _storage.tpl
│       │       │   ├── _tplvalues.tpl
│       │       │   ├── _utils.tpl
│       │       │   ├── _warnings.tpl
│       │       │   └── validations
│       │       │       ├── _cassandra.tpl
│       │       │       ├── _mariadb.tpl
│       │       │       ├── _mongodb.tpl
│       │       │       ├── _mysql.tpl
│       │       │       ├── _postgresql.tpl
│       │       │       ├── _redis.tpl
│       │       │       └── _validations.tpl
│       │       └── values.yaml
│       ├── templates
│       │   ├── NOTES.txt
│       │   ├── _helpers.tpl
│       │   ├── deployment.yaml
│       │   ├── extra-list.yaml
│       │   ├── hpa.yaml
│       │   ├── metrics-svc.yaml
│       │   ├── pdb.yaml
│       │   ├── secrets.yaml
│       │   ├── service.yaml
│       │   ├── serviceaccount.yaml
│       │   ├── servicemonitor.yaml
│       │   └── statefulset.yaml
│       └── values.yaml
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── config-secret.yaml
│   ├── deployment.yaml
│   ├── externaldb-secrets.yaml
│   ├── extra-list.yaml
│   ├── hpa.yaml
│   ├── httpd-configmap.yaml
│   ├── ingress.yaml
│   ├── metrics-svc.yaml
│   ├── networkpolicy-backend-ingress.yaml
│   ├── networkpolicy-egress.yaml
│   ├── networkpolicy-ingress.yaml
│   ├── pdb.yaml
│   ├── postinit-configmap.yaml
│   ├── pvc.yaml
│   ├── secrets.yaml
│   ├── serviceaccount.yaml
│   ├── servicemonitor.yaml
│   ├── svc.yaml
│   └── tls-secrets.yaml
├── values.schema.json
└── values.yaml

19 directories, 134 files
```

> 💻 명령어
>```bash
>tree ./wordpress
>```

<br><br><br>

이제 설치(`helm install`)를 진행해 보겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```

> 💻 명령어
>```bash
>helm repo update
>```

<br><br><br>

```bash
ubuntu@ip-172-31-23-60:~$ helm install my-wordpress bitnami/wordpress
NAME: my-wordpress
LAST DEPLOYED: Thu Feb 16 08:08:04 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: wordpress
CHART VERSION: 15.2.42
APP VERSION: 6.1.1

** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    my-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w my-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default my-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default my-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
```

> 💻 명령어
>```bash
>helm install my-wordpress bitnami/wordpress
>```

<br><br><br>

설치된 Helm chart는 **[Release](https://helm.sh/ko/docs/glossary/#release)**라고 합니다.  
**Release**의 목록은 `helm list`명령으로 조회할 수 있구요.
```bash
ubuntu@ip-172-31-23-60:~$ helm list
NAME        	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART            	APP VERSION
my-wordpress	default  	1       	2023-02-16 08:08:04.880473857 +0000 UTC	deployed	wordpress-15.2.42	6.1.1
```

> 💻 명령어
>```bash
>helm list
>```

<br><br><br>

쿠버네티스 명령어로 어떤 리소스들이 생성됐나 볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/my-wordpress-5786598c5-5fqln   1/1     Running   0          2m59s
pod/my-wordpress-mariadb-0         1/1     Running   0          2m59s

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/kubernetes             ClusterIP      10.96.0.1        <none>        443/TCP                      4d17h
service/my-wordpress           LoadBalancer   10.109.136.241   <pending>     80:30606/TCP,443:32687/TCP   2m59s
service/my-wordpress-mariadb   ClusterIP      10.111.195.166   <none>        3306/TCP                     2m59s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-wordpress   1/1     1            1           2m59s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-wordpress-5786598c5   1         1         1       2m59s

NAME                                    READY   AGE
statefulset.apps/my-wordpress-mariadb   1/1     2m59s
```

> 💻 명령어
>```bash
>kubectl get all
>```

<br><br><br>

와우~ 뭔가 Wordpress 소프트웨어에 필요한 모든게 한 번에 설치가 된 것 같네요. 패키지로...

삭제도 한 번에 가능합니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm uninstall my-wordpress
release "my-wordpress" uninstalled
```

> 💻 명령어
>```bash
>helm uninstall my-wordpress
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