지금까지는 만들어진 차트를 사용하는 방법을 알아보았습니다.  
이제 내 차트를 직접 만들어보도록 해요. ( from scratch ＿〆(。╹‿ ╹ 。)  )

<br><br><br>

새로운 차트를 생성하는 명령어는 [helm create](https://helm.sh/ko/docs/helm/helm_create/)입니다.  
**my-chart**라는 차트를 하나 만들어볼까요?
```bash
controlplane $ helm create my-chart
Creating my-chart
controlplane $ ls ./my-chart/
Chart.yaml  charts  templates  values.yaml
```

> 💻 명령어 `helm create my-chart`{{exec}}  
> 💻 명령어 `ls ./my-chart/`{{exec}}

<br><br><br>

앞에서 배운것 처럼, chart.yaml파일과 templates디렉토리가 만들어져 있습니다.  
templates디렉토리에는 deployment, service, ingress등의 manifest 파일이 있구요.

<br><br><br>

이제 몇 가지만 수정해볼게요.  
먼저 **values.yaml** 파일에서 **service**부분을 수정해볼게요.  

아래와 같이 수정해주세요.
```yaml

...생략...

service:
  type: NodePort
  port: 80
  nodePort: 30007

...생략...

```

<br>

> 💻 명령어 `vi ./my-chart/values.yaml`{{exec}}  
> vi 에디터나 Editor탭의 웹 에디터를 사용해서 변경해주세요.

<br>

바뀐 부분은 다음과 같습니다.
- service.type : ClusterIP -> NodePort
- service.nodePort 추가

<br><br><br>

service 생성을 위한 yaml파일도 수정해주세요.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-chart.fullname" . }}
  labels:
    {{- include "my-chart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      {{- if eq .Values.service.type "NodePort" }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "my-chart.selectorLabels" . | nindent 4 }}
```

<br>

> 💻 명령어 `vi ./my-chart/templates/service.yaml`{{exec}}  
> vi 에디터나 Editor탭의 웹 에디터를 사용해서 변경해주세요.

바뀐 부분은 다음과 같습니다.
- service.ports.nodePort 부분 추가 (line 11 ~ 13)  

들여쓰기 주의해서 작성해주세요.

> service.type만 NodePort로 해도 생성은 됩니다.  
> 이 경우 nodePort는 **30000~32767**사이의 값이 자동으로 할당됩니다.  
> 우리 예제는 30007로 지정하기 위해서 차트를 수정했습니다.

<br><br><br>

그리고, templates 디렉토리에서 필요없는 파일들은 삭제해주세요.  
templates 디렉토리에서 삭제된 파일들은 후에 생성되지 않습니다.
```bash
controlplane $ rm -r ./my-chart/templates/tests
```

> 💻 명령어 `rm -r ./my-chart/templates/tests`{{exec}}

<br><br><br>

이제 우리가 수정한 부분에 문제가 없는지 볼까요?

```bash
controlplane $ helm lint ./my-chart
==> Linting ./my-chart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

> 💻 명령어 `helm lint ./my-chart`{{exec}}

<br>

만약 잘 못된 부분이 있으면 고쳐주세요.

<br><br><br>

자, 이제 문법적으로 문제는 없는 나만의 차트가 준비됐습니다.

내 차트로 어떤 K8s 리소스들이 만들어지는지 확인을 해보려면 [Helm Template](https://helm.sh/ko/docs/helm/helm_template/)명령어를 사용하면 됩니다.

```bash
controlplane $ helm template my-nginx ./my-chart
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
  type: NodePort
  ports:
    - port: 80
      nodePort: 30007
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
```

> 💻 명령어 `helm template my-nginx ./my-chart`{{exec}}

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
controlplane $ helm install my-nginx --dry-run ./my-chart
NAME: my-nginx
LAST DEPLOYED: Sun Mar 12 03:38:16 2023
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
  type: NodePort
  ports:
    - port: 80
      nodePort: 30007
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

NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services my-nginx-my-chart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

> 💻 명령어 `helm install my-nginx --dry-run ./my-chart`{{exec}}

<br><br><br>

지금까지 문제가 없다면, 이제 설치를 진행하면 됩니다.  
```
controlplane $ helm install my-nginx ./my-chart
NAME: my-nginx
LAST DEPLOYED: Sun Mar 12 03:38:35 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services my-nginx-my-chart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

> 💻 명령어 `helm install my-nginx ./my-chart`{{exec}}

접속해서 테스트도 해보세요. (NodePort로 접속)

🔗 [My-chart - Nginx]({{TRAFFIC_HOST1_30007}})

<br><br><br>

잘 되죠?  
그렇다면, 이제 남은것은 내 차트를 잘 패키징하면 됩니다.
```
controlplane $ helm package ./my-chart
Successfully packaged chart and saved it to: /root/lab/my-chart-0.1.0.tgz
```

> 💻 명령어 `helm package ./my-chart`{{exec}}

**my-chart-0.1.0.tgz**라는 파일이 만들어집니다.

helm chart repository에 업로드하거나, github같은곳에 업로드 해서 사용하시면 됩니다.

<br><br><br>

여기까지 Helm 에 대해 알아보았습니다.

정리하고 마치겠습니다.
```bash
controlplane $ helm uninstall my-nginx
release "my-nginx" uninstalled
```

> 💻 명령어 `helm uninstall my-nginx`{{exec}}

<br>

정리 후 상태는 아래와 같습니다.
```bash
controlplane $ helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
ubuntu@ip-172-31-28-216:~$
```

> 💻 명령어 `helm list`{{exec}}

수고하셨습니다. (〃･ิ‿･ิ)ゞ