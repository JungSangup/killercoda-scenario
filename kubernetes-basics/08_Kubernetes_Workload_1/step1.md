이번에는 Pod를 관리하는 명령어들을 알아보겠습니다.

먼저 Pod를 생성하는 세 가지 방법을 알아보겠습니다.

첫 번째는 [명령형 커맨드(Imperative commands)](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/object-management/#%EB%AA%85%EB%A0%B9%ED%98%95-%EC%BB%A4%EB%A7%A8%EB%93%9C) 입니다.
Pod를 직접 동작시키는 방법입니다.

```bash
ubuntu@ip-172-31-23-60:~$ kubectl run my-nginx1 --image=nginx:1.19.3
pod/my-nginx1 created
```

> 💻 명령어 `kubectl run my-nginx1 --image=nginx:1.19.3`{{exec}}

<br><br><br>

생성된 Pod를 볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
my-nginx1   1/1     Running   0          53s   172.17.0.2   ip-172-31-23-60   <none>           <none>
```

> 💻 명령어 `kubectl get pods -o wide`{{exec}}


nginx:1.19.3 이미지를 이용해서 my-nginx1 pod를 생성했습니다.

<br><br><br>

두 번째는, [명령형 오브젝트 구성 (Imperative object configuration)](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/object-management/#%EB%AA%85%EB%A0%B9%ED%98%95-%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8-%EA%B5%AC%EC%84%B1) 입니다.
미리 정의된 yaml파일을 이용해서 생성(create) 합니다.

먼저 아래와 같은 파일을 작성합니다.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    tier: frontend
  name: my-nginx2
spec:
  containers:
  - image: nginx:1.19.3
    name: my-nginx2
    ports:
    - containerPort: 80
```
> 파일명은 **nginx2-pod.yaml**로 합니다.

<br>

그리고, 아래와 같이 Pod를 생성합니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl create -f nginx2-pod.yaml
pod/my-nginx2 created
```

> 💻 명령어 `kubectl create -f nginx2-pod.yaml`{{exec}}

<br><br><br>

생성된 Pod를 볼까요?

```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
my-nginx1   1/1     Running   0          11m     172.17.0.2   ip-172-31-23-60   <none>           <none>
my-nginx2   1/1     Running   0          2m14s   172.17.0.3   ip-172-31-23-60   <none>           <none>
```

> 💻 명령어 `kubectl get pods -o wide`{{exec}}

두 번째 Nginx Pod가 생성된 걸 볼 수 있습니다.
첫 번째와 다른 명령어를 사용하였지만, 결과는 동일한 걸 알 수 있습니다.

<br><br><br>

세 번째는, [선언형 오브젝트 구성 (Declarative object configuration)](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/object-management/#%EB%AA%85%EB%A0%B9%ED%98%95-%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8-%EA%B5%AC%EC%84%B1) 입니다.
어떤 작업을 할지(**create**, **update**) 명시하지 않고 단순히 `apply`라는 키워드를 씁니다.
무엇을 할지는 쿠버네티스가 알아서 해줍니다. (ง˙∇˙)ว

먼저 아래와 같은 파일을 작성합니다.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    tier: frontend
  name: my-nginx3
spec:
  containers:
  - image: nginx:1.19.3
    name: my-nginx3
    ports:
    - containerPort: 80
```
> 파일명은 nginx3-pod.yaml로 합니다.

nginx2-pod.yaml과 동일하고 name만 다르게 작성했습니다.

<br>

그리고, 아래 명령어로 Pod를 생성해보겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ kubectl apply -f nginx3-pod.yaml
pod/my-nginx3 created
```

> 💻 명령어 `kubectl apply -f nginx3-pod.yaml`{{exec}}

<br><br><br>

생성된 Pod를 볼까요?

```bash
ubuntu@ip-172-31-23-60:~$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
my-nginx1   1/1     Running   0          14m     172.17.0.2   ip-172-31-23-60   <none>           <none>
my-nginx2   1/1     Running   0          5m32s   172.17.0.3   ip-172-31-23-60   <none>           <none>
my-nginx3   1/1     Running   0          117s    172.17.0.8   ip-172-31-23-60   <none>           <none>
```

> 💻 명령어 `kubectl get pods -o wide`{{exec}}

역시 동일한 결과를 얻을 수 있습니다.

**선언형(Declarative)** 이라는 말을 잘 기억해두세요.
그리고 `kubectl apply ~` 명령어도 많이 쓰이니, 잘 기억해 두시구요.

