이제 **Container probe**를 알아보겠습니다.  
여러 종류가 있지만, 그 중 한 가지만 실습을 통해서 알아보겠습니다.

먼저 아래와 같은 파일을 작성합니다. (httpGet을 이용하는 livenessProbe 입니다.)
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
> 파일명은 livenessProbe-httpGet.yaml로 합니다.  
> `livenessProbe` 부분이 설정 부분입니다.

<br><br><br>

그리고, 아래 명령어를 이용해서 Pod를 생성합니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl apply -f livenessProbe-httpGet.yaml
pod/liveness-http created
```

> 💻 명령어 `kubectl apply -f livenessProbe-httpGet.yaml`{{exec}}

<br><br><br>

어느정도(10초이상) 시간이 지난 후 조회를 해보면 아래와 같이 보일거예요.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods -o wide
NAME            READY   STATUS    RESTARTS     AGE     IP           NODE              NOMINATED NODE   READINESS GATES
liveness-http   1/1     Running   1 (6s ago)   24s     172.17.0.9   ip-172-31-23-60   <none>           <none>
my-nginx1       1/1     Running   0            19m     172.17.0.2   ip-172-31-23-60   <none>           <none>
my-nginx2       1/1     Running   0            10m     172.17.0.3   ip-172-31-23-60   <none>           <none>
my-nginx3       1/1     Running   0            6m25s   172.17.0.8   ip-172-31-23-60   <none>           <none>
```

> 💻 명령어 `kubectl get pods -o wide`{{exec}}

<br><br><br>

테스트에 사용된 컨테이너는 libenessProbe 테스트를 위해서 생성 후 10초가 지난 뒤부터는 httpGet 요청에 대해서 500 Error를 발생하도록 되어있습니다.  
([소스코드](https://github.com/kubernetes/kubernetes/blob/master/test/images/agnhost/liveness/server.go) 참고)

livenessProbe는 계속해서 Pod의 상태를 살피고, 문제가 발생하면(500 error) 컨테이너를 재시작(RESTARTS) 합니다.  
`kubectl get pods` 명령어의 결과에서 **RESTARTS** 의 숫자가 바로 이 재시작 횟수입니다.

<br><br><br>

이번 실습은 여기까지 입니다.