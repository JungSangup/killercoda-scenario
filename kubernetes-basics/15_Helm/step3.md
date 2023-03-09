지금까지는 만들어진 차트를 사용하는 방법을 알아보았습니다.  
이제 내 차트를 직접 만들어보도록 해요. ( from scratch ＿〆(。╹‿ ╹ 。)  )

<br><br><br>

새로운 차트를 생성하는 명령어는 [helm create](https://helm.sh/ko/docs/helm/helm_create/)입니다.  
**my-chart**라는 차트를 하나 만들어볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ helm create my-chart
Creating my-chart
ubuntu@ip-172-31-23-60:~$ tree my-chart
my-chart
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files
```

> 💻 명령어
>```bash
>helm create my-chart
>```{{exec}}
>```bash
>tree my-chart
>```{{exec}}

<br><br><br>

앞에서 배운것 처럼, chart.yaml파일과 templates디렉토리가 만들어져 있습니다.  
templates디렉토리에는 deployment, service, ingress등의 manifest 파일이 있구요.

<br><br><br>

이제 몇 가지만 수정해볼게요.  
먼저 **values.yaml** 파일에서 **ingress**부분을 수정해볼게요.  

아래와 같이 수정해주세요.
```yaml

...생략...

ingress:
  enabled: true
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: my-nginx.io
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

...생략...

```

> 💻 명령어
>```bash
>vi ./my-chart/values.yaml
>```{{exec}}  
> vi 에디터나 익숙한 편집기를 사용해서 변경해주세요.

<br>

바뀐 부분은 다음과 같습니다.
- ingress.enabled : false -> true
- ingress.hosts[0].host : chart-example.local -> my-ninx.io 로 변경

<br><br><br>

그리고, templates 디렉토리에서 필요없는 파일들은 삭제해주세요.  
templates 디렉토리에서 삭제된 파일들은 후에 생성되지 않습니다.
```bash
ubuntu@ip-172-31-23-60:~$ rm -r ./my-chart/templates/tests
```

> 💻 명령어
>```bash
>rm -r ./my-chart/templates/tests
>```{{exec}}

<br><br><br>

이제 우리가 수정한 부분에 문제가 없는지 볼까요?

```bash
ubuntu@ip-172-31-23-60:~$ helm lint ./my-chart
==> Linting ./my-chart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

> 💻 명령어
>```bash
>helm lint ./my-chart
>```{{exec}}

<br>

만약 잘 못된 부분이 있으면 고쳐주세요.

<br><br><br>

자, 이제 문법적으로 문제는 없는 나만의 차트가 준비됐습니다.

내 차트로 어떤 K8s 리소스들이 만들어지는지 확인을 해보려면 [Helm Template](https://helm.sh/ko/docs/helm/helm_template/)명령어를 사용하면 됩니다.

```bash
ubuntu@ip-172-31-23-60:~$ helm template my-nginx ./my-chart
---
# Source: my-chart/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
---
# Source: my-chart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
---
# Source: my-chart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: my-chart
      app.kubernetes.io/instance: my-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: my-chart
        app.kubernetes.io/instance: my-nginx
    spec:
      serviceAccountName: my-nginx-my-chart
      securityContext:
        {}
      containers:
        - name: my-chart
          securityContext:
            {}
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}
---
# Source: my-chart/templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  rules:
    - host: "my-nginx.io"
      http:
        paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
              service:
                name: my-nginx-my-chart
                port:
                  number: 80
```

> 💻 명령어
>```bash
>helm template my-nginx ./my-chart
>```{{exec}}

<br>

템플릿 엔진에 의해서 생성될 K8s 리소스들을 미리 알아볼 수 있습니다.  

생각한 대로인가요?  
아니라면 다시 앞으로 돌아가서 chart를 수정하면 됩니다.

이런 과정이 차트를 개발하는 과정입니다. (수정하고 확인하고... 반복)

<br><br><br>

이제 마지막으로 나의 차트를 이용해서 설치([helm install](https://helm.sh/ko/docs/helm/helm_install/))를 해볼게요.

바로 설치를 할 수도 있지만, 이런것도 가능합니다.  
`--dry-run` 옵션을 사용해서, 실제 설치를 진행하기 전에 미리 확인을 해볼 수 있습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm install my-nginx --dry-run ./my-chart
NAME: my-nginx
LAST DEPLOYED: Tue Mar  7 10:01:38 2023
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: my-chart/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
---
# Source: my-chart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
---
# Source: my-chart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: my-chart
      app.kubernetes.io/instance: my-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: my-chart
        app.kubernetes.io/instance: my-nginx
    spec:
      serviceAccountName: my-nginx-my-chart
      securityContext:
        {}
      containers:
        - name: my-chart
          securityContext:
            {}
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}
---
# Source: my-chart/templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-nginx-my-chart
  labels:
    helm.sh/chart: my-chart-0.1.0
    app.kubernetes.io/name: my-chart
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  rules:
    - host: "my-nginx.io"
      http:
        paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
              service:
                name: my-nginx-my-chart
                port:
                  number: 80

NOTES:
1. Get the application URL by running these commands:
  http://my-nginx.io/
```

> 💻 명령어
>```bash
>helm install my-nginx --dry-run ./my-chart
>```{{exec}}

<br><br><br>

지금까지 문제가 없다면, 이제 설치를 진행하면 됩니다.  
```
ubuntu@ip-172-31-23-60:~$ helm install my-nginx ./my-chart
NAME: my-nginx
LAST DEPLOYED: Tue Mar  7 09:53:06 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  http://my-nginx.io/
```

> 💻 명령어
>```bash
>helm install my-nginx ./my-chart
>```{{exec}}

웹브라우저에서 접속도 해보시구요. (앞에서 사용한 my-nginx.io를 그대로 사용했습니다. 혹시 hosts파일 처리가 필요하다면 추가해주세요.)  

http://my-nginx.io

![h:300](./img/k8s_nginx_ingress.png)

<br><br><br>

잘 되죠?  
그렇다면, 이제 남은것은 내 차트를 잘 패키징하면 됩니다.
```
ubuntu@ip-172-31-23-60:~$ helm package ./my-chart
Successfully packaged chart and saved it to: /home/ubuntu/mspt3/hands_on_files/ch15/my-chart-0.1.0.tgz
```

> 💻 명령어
>```bash
>helm package ./my-chart
>```{{exec}}

**my-chart-0.1.0.tgz**라는 파일이 만들어집니다.

helm chart repository에 업로드하거나, github같은곳에 업로드 해서 사용하시면 됩니다.

<br><br><br>

여기까지 Helm 에 대해 알아보았습니다.

정리하고 마치겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ helm uninstall my-nginx
release "my-nginx" uninstalled
```

> 💻 명령어
>```bash
>helm uninstall my-nginx
>```{{exec}}

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
>```{{exec}}

수고하셨습니다. (〃･ิ‿･ิ)ゞ