
### Multi-stage builds

이전 Step에서 Java Application을 포함하는 docker 이미지를 생성해 보았습니다.  
Base image로 openjdk:8 (JDK)을 사용했기 때문에 이미지의 크기가 큽니다. (앞의 결과는 **526MB**)

docker는 multi-stage build 기능을 제공하기 때문에 최종 docker 이미지에는 binary만 포함될 수 있도록 할 수가 있습니다.

<br>

![](./img/docker_multi_stage_build.PNG)

---

#### Dockerfile 수정

애플리케이션 파일인 HelloDocker.java 파일은 그대로 두고 Dockerfile만 아래와 같이 수정합니다.

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

- VI Editor를 사용하거나, Editor탭을 눌러 Web-IDE를 사용하여 작성합니다.  
- 작성이 어려운 경우 lab 디렉토리 아래에 있는 파일(`Dockerfile2`)을 사용하셔도 됩니다.


| Build stage                                                  | Production stage                                             |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1. openjdk8을 build-stage로 정하고<br>2. /hello 경로에 HelloDocker.java파일을 복사<br>3. /hello 경로로 이동<br>4. HelloDocker.java를 컴파일 | 1. openjdk8-jre를 production-stage로 정하고<br>2. /hello/HelloDocker.class 파일 복사 (build -> production)<br>3. 작업 경로를 /hello로 변경<br>4. docker container가 구동되면 `java HelloDocker`를 실행 |

---

#### Docker image 생성

이제 아래 명령어로 hellodocker 이미지 v2를 생성합니다.

```bash
$ docker build -t hellodocker:v2 .
Sending build context to Docker daemon  3.072kB
Step 1/8 : FROM openjdk:8 as build-stage
 ---> 2a8331246713
Step 2/8 : COPY HelloDocker.java /hello/
 ---> Using cache
 ---> 898708a1fb93
Step 3/8 : WORKDIR /hello
 ---> Using cache
 ---> f6d8741cd695
Step 4/8 : RUN javac HelloDocker.java
 ---> Using cache
 ---> 24ec44b764c2
Step 5/8 : FROM openjdk:8-jre as production-stage
 ---> d991802804b7
Step 6/8 : COPY --from=build-stage /hello/HelloDocker.class /hello/HelloDocker.class
 ---> d1ee4db7b623
Step 7/8 : WORKDIR /hello
 ---> Running in 98a5b3f359c0
Removing intermediate container 98a5b3f359c0
 ---> e5eef1dd32d5
Step 8/8 : CMD ["java","HelloDocker"]
 ---> Running in 0c3c772a09dd
Removing intermediate container 0c3c772a09dd
 ---> 61de5a0b96a9
Successfully built 61de5a0b96a9
Successfully tagged hellodocker:v2
```

> **명령어** : `docker build -t hellodocker:v2 .`{{exec}}  

> 또는 lab 디렉토리의 파일을 그대로 사용하려면 아래 명령어를 실행하세요.  
> **명령어** : `docker build -t hellodocker:v2 -f ~/lab/Dockerfile2 ~/lab`{{exec}}

---

빌드가 성공하면 `docker images`명령어를 실행해서 결과를 볼까요?

```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
hellodocker   v2        61de5a0b96a9   About a minute ago   274MB
hellodocker   v1        d187b50492c2   29 minutes ago       526MB
openjdk       8-jre     d991802804b7   6 days ago           274MB
openjdk       8         2a8331246713   6 days ago           526MB
```

> **명령어** : `docker images`{{exec}}

v1 과 v2 는 Java Application은 동일하지만, base image의 차이 때문에 이미지 전체의 사이즈가 크게 차이가 납니다.

- **v1** : 526MB -> **v2** : 274MB   （°o°；）

Cloud native 환경에서는 가능하면 이미지 사이즈를 작게 가져가는게 좋겠죠?

컨테이너 실행결과는 아래처럼 차이가 없습니다.

```bash
$ docker run hellodocker:v2
Hello Docker!!!
```

> **명령어** : `docker run hellodocker:v2`{{exec}}