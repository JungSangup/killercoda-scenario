이제 **Secret**을 작성하고 사용하는 방법을 알아보겠습니다.

**Secret**은 **ConfigMap**과 동일하게 **Key:Value** 형식으로 저장이 됩니다.

차이점은, 저장될 때 base64 encoding이 되어서 저장된다는 점입니다. 사실 암호화되어 저장되는 것도 아니고 단순히 base64 encoding만 되기 때문에 안전하다고 할 수는 없으나 공격자(?)에게는 혼란을 줄 수 있습니다.

### Secret from Yaml

Secret도 ConfigMap과 동일하게 `--from-literal` 이나 `--from-file`, `--from-env-file`을 사용할 수 있습니다.  
이번 실습에는 YAML파일을 사용하는 방법만 사용해보겠습니다.



먼저 Secret을 위한 yaml파일을 하나 작성합니다. (~/lab 디렉토리 아래의 파일을 사용하셔도 됩니다.)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: yaml-secret
data:
  location: SmFtc2ls
  business: SVRTZXJ2aWNl
```

> 파일명은 yaml-secret.yaml 로 합니다.

---

configmap과 달리 secret을 생성할 때는, value를 base64 encoding한 값으로 작성해야만 합니다.  
location의 value는 Jamsil 을 base64 encoding 한 값이며, business의 value는 ITService를 base64 encoding 한 값입니다.

아래를 참고하세요.

```bash
controlplane $ echo -n 'Jamsil' | base64
SmFtc2ls
controlplane $ echo -n 'ITService' | base64
SVRTZXJ2aWNl
```

> **명령어** : `echo -n 'Jamsil' | base64`{{exec}}, `echo -n 'ITService' | base64`{{exec}}

작성된 yaml을 적용하겠습니다.

```bash
controlplane $ kubectl apply -f yaml-secret.yaml
secret/yaml-secret created
```

> **명령어** : `kubectl apply -f yaml-secret.yaml`{{exec}}

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어(Tab 1)** : `kubectl apply -f ~/lab/yaml-secret.yaml`{{exec}}


---

동일하게 describe로 확인해 봅니다.

```bash
controlplane $ kubectl describe secret yaml-secret
Name:         yaml-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
business:  9 bytes
location:  6 bytes
```

> **명령어** : `kubectl describe secret yaml-secret`{{exec}}

---

이제 앞에서 만든 Secret을 사용할 Pod를 준비하겠습니다. (~/lab 디렉토리 아래의 파일을 사용하셔도 됩니다.)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: secret-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "while true; do echo hi; sleep 10; done" ]
      env:
        - name: LOCATION
          valueFrom:
            secretKeyRef:
              name: yaml-secret
              key: location
        - name: BUSINESS
          valueFrom:
            secretKeyRef:
              name: yaml-secret
              key: business
  restartPolicy: Never
```

> 파일명은 secretpod.yaml 로 합니다.
>
> Manifest를 보면, 아주 가벼운 busybox shell 만 포함하고 있는 이미지를 사용합니다.

- yaml-secret에서 location과 business key에 해당하는 value를 LOCATION과 BUSINESS 환경 변수에 담아주도록 하였습니다.

---

이제 Pod을 생성합니다.

```
controlplane $ kubectl apply -f secretpod.yaml
pod/secret-pod created
```

> **명령어** : `kubectl apply -f secretpod.yaml`{{exec}}

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어(Tab 1)** : `kubectl apply -f ~/lab/secretpod.yaml`{{exec}}



해당 pod의 환경변수에 어떤 값들이 들어갔는지 확인해 보겠습니다.

```bash{4,5}
controlplane $ kubectl exec -it secret-pod -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-pod
LOCATION=Jamsil
BUSINESS=ITService
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
TERM=xterm
HOME=/root
```

> **명령어** : `kubectl exec -it secret-pod -- env`{{exec}}

이번 실습은 여기까지 입니다.   ＿〆(。╹‿ ╹ 。)