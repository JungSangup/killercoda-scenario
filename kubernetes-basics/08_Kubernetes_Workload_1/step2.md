이제 **Container probe**를 알아보겠습니다.
여러 종류가 있지만, 그 중 한 가지만 실습을 통해서 알아보겠습니다.

먼저 아래와 같은 파일을 작성합니다. (~/lab 디렉토리 아래의 파일을 사용하셔도 됩니다.)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

> 파일명은 livenessProbe_httpGet.yaml로 합니다.  
> `livenessProbe` 부분이 설정 부분입니다.

---

그리고, 아래 명령어를 이용해서 Pod를 생성합니다.

```bash
controlplane $ kubectl apply -f livenessProbe_httpGet.yaml
pod/liveness-http created
```

> **명령어** : `kubectl apply -f livenessProbe_httpGet.yaml`{{exec}}  

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `kubectl apply -f ~/lab/livenessProbe_httpGet.yaml`{{exec}}

어느정도(10초이상) 시간이 지난 후 조회를 해보면 아래와 같이 보일거예요.

```bash
controlplane $ kubectl get pods -o wide
NAME            READY   STATUS    RESTARTS      AGE     IP            NODE     NOMINATED NODE   READINESS GATES
liveness-http   1/1     Running   1 (16s ago)   34s     192.168.1.7   node01   <none>           <none>
my-nginx1       1/1     Running   0             7m53s   192.168.1.4   node01   <none>           <none>
my-nginx2       1/1     Running   0             6m40s   192.168.1.5   node01   <none>           <none>
my-nginx3       1/1     Running   0             119s    192.168.1.6   node01   <none>           <none>
```

> **명령어** : `kubectl get pods -o wide`{{exec}}  

테스트에 사용된 컨테이너는 libenessProbe 테스트를 위해서 생성 후 10초가 지난 뒤부터는 httpGet 요청에 대해서 500 Error를 발생하도록 되어있습니다.  
([소스코드](https://github.com/kubernetes/kubernetes/blob/master/test/images/agnhost/liveness/server.go) 참고)

livenessProbe는 계속해서 Pod의 상태를 살피고, 문제가 발생하면(500 error) 컨테이너를 재시작(RESTARTS) 합니다.  
`kubectl get pods` 명령어의 결과에서 **RESTARTS** 의 숫자가 바로 이 재시작 횟수입니다.

---

이번 실습은 여기까지 입니다.

다음 실습을 위해서 생성한 모든 Pod를 삭제할게요.

```bash
controlplane $ kubectl delete pod --all
pod "liveness-http" deleted
pod "my-nginx1" deleted
pod "my-nginx2" deleted
pod "my-nginx3" deleted
```

> **명령어** : `kubectl delete pod --all`{{exec}}  

