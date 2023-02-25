
#### Docker image 생성

이제 아래 명령어로 hellodocker 이미지를 생성합니다.

```bash
$ docker build -t hellodocker:v1 .
Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM openjdk:8
 ---> 2a8331246713
Step 2/5 : COPY HelloDocker.java /hello/
 ---> 898708a1fb93
Step 3/5 : WORKDIR /hello
 ---> Running in 711b58d64ddb
Removing intermediate container 711b58d64ddb
 ---> f6d8741cd695
Step 4/5 : RUN javac HelloDocker.java
 ---> Running in 6f510b4106f9
Removing intermediate container 6f510b4106f9
 ---> 24ec44b764c2
Step 5/5 : CMD ["java","HelloDocker"]
 ---> Running in 6fdc1dad43f4
Removing intermediate container 6fdc1dad43f4
 ---> d187b50492c2
Successfully built d187b50492c2
Successfully tagged hellodocker:v1
```

> **명령어** : `docker build -t hellodocker:v1 .`{{exec}}  
 
> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `docker build -t hellodocker:v1 -f ~/lab/Dockerfile1 ~/lab`{{exec}}

---

빌드가 성공하면 `docker images`명령어로 조회도 해보세요.

```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hellodocker   v1        d187b50492c2   6 minutes ago   526MB
openjdk       8         2a8331246713   6 days ago      526MB
```

> **명령어** : `docker images`{{exec}}

이미지가 준비됐으니 이제 실행을 해볼게요.

```bash
$ docker run hellodocker:v1
Hello Docker!!!
```

> **명령어** : `docker run hellodocker:v1`{{exec}}

결과가 예상한 것과 같은가요?

일단 첫 번째 단계는 성공입니다. (ง˙∇˙)ว