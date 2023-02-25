이전 Step에서 Java Application을 포함하는 docker 이미지를 생성해 보았습니다.  
Base image로 openjdk:8 (JDK)을 사용했기 때문에 이미지의 크기가 큽니다. (앞의 결과는 **526MB**)

docker는 multi-stage build 기능을 제공하기 때문에 최종 docker 이미지에는 binary만 포함될 수 있도록 할 수가 있습니다.

<br>

![](./img/docker-multi-stage-build.png)


애플리케이션 파일인 HelloDocker.java 파일은 그대로 두고 **Dockerfile**을 하나 더 준비하겠습니다.
```dockerfile
# Build stage
FROM openjdk:8 as build-stage
COPY HelloDocker.java /hello/
WORKDIR /hello
RUN javac HelloDocker.java

# Production stage
FROM openjdk:8-jre as production-stage
COPY --from=build-stage /hello/HelloDocker.class /hello/HelloDocker.class
WORKDIR /hello
CMD ["java","HelloDocker"]
```
> 파일명은 **Dockerfile2**로 합니다.

| Build stage | Production stage |
| :--- | :--- |
| 1. openjdk8을 build-stage로 정하고<br>2. /hello 경로에 HelloDocker.java파일을 복사<br>3. /hello 경로로 이동<br>4. HelloDocker.java를 컴파일 | 1. openjdk8-jre를 production-stage로 정하고<br>2. /hello/HelloDocker.class 파일 복사 (build -> production)<br>3. 작업 경로를 /hello로 변경<br>4. docker container가 구동되면 `java HelloDocker`를 실행 |

<br><br><br>

이제 아래 명령어로 hellodocker 이미지 v2를 생성합니다.
```bash
ubuntu@ip-172-31-23-60:~$ docker build -f Dockerfile2 -t hellodocker:v2 .
Sending build context to Docker daemon  4.096kB
Step 1/8 : FROM openjdk:8 as build-stage
 ---> b273004037cc
Step 2/8 : COPY HelloDocker.java /hello/
 ---> Using cache
 ---> 586ddfe9c462
Step 3/8 : WORKDIR /hello
 ---> Using cache
 ---> 31be5909e298
Step 4/8 : RUN javac HelloDocker.java
 ---> Using cache
 ---> a12ffad6b75b
Step 5/8 : FROM openjdk:8-jre as production-stage
8-jre: Pulling from library/openjdk
001c52e26ad5: Already exists
d9d4b9b6e964: Already exists
2068746827ec: Already exists
8510da692cda: Pull complete
c34215579d03: Pull complete
73d77b4774a9: Pull complete
Digest: sha256:667a15e7bc533a90fb39ddb7e5bed63162ac3c13a97e6c698bf4f139f51b7d33
Status: Downloaded newer image for openjdk:8-jre
 ---> 0c14a0e20aa3
Step 6/8 : COPY --from=build-stage /hello/HelloDocker.class /hello/HelloDocker.class
 ---> e6a283c7bcb4
Step 7/8 : WORKDIR /hello
 ---> Running in 0c29098b40e6
Removing intermediate container 0c29098b40e6
 ---> a7703fe64d5f
Step 8/8 : CMD ["java","HelloDocker"]
 ---> Running in beddc772d5b9
Removing intermediate container beddc772d5b9
 ---> 9435a2a2311d
Successfully built 9435a2a2311d
Successfully tagged hellodocker:v2
```

> 💻 명령어 `docker build -f Dockerfile2 -t hellodocker:v2 .`{{exec}}

<br><br><br>

빌드가 성공하면 `docker images`명령어를 실행해서 결과를 볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ docker images hellodocker
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
hellodocker   v2        9435a2a2311d   About a minute ago   274MB
hellodocker   v1        91d22f496ae4   7 minutes ago        526MB
```

> 💻 명령어 `docker images hellodocker`{{exec}}

v1 과 v2 는 Java Application은 동일하지만, base image의 차이 때문에 이미지 전체의 사이즈가 크게 차이가 납니다.
- **v1** : 526MB -> **v2** : 274MB   （°o°；）

**Cloud native** 환경에서는 가능하면 이미지 사이즈를 작게 가져가는게 좋겠죠?

<br><br><br>

컨테이너 실행결과는 아래처럼 차이가 없습니다.
```bash
ubuntu@ip-172-31-23-60:~/mspt3/hands_on_files/dockerfile$ docker run --rm hellodocker:v2
Hello Docker!!!
```

> 💻 명령어 `docker run --rm hellodocker:v2`{{exec}}
- `--rm` : Automatically remove the container when it exits

<br><br><br>

수고하셨습니다. (〃･ิ‿･ิ)ゞ