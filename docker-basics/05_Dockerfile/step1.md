이번 실습은 Dockerfile을 이용하여 이미지를 생성하는 방법을 알아보겠습니다.  
어떻게 하면 좀 더 효율적인 이미지를 만들 수 있는지도 알아볼게요.

먼저 간단한 Java 파일을 준비합니다.
```java
public class HelloDocker {
    public static void main(String[] args) {
        System.out.println("Hello Docker!!!");
    }
}
```
> 파일명은 **HelloDocker.java**로 합니다.

<br><br><br>

이제 Dockerfile 하나를 준비합니다.
이미지를 만드는 여러가지 방법 중 하나인 Dockerfile을 이용한 빌드를 해볼게요.
```dockerfile
FROM openjdk:8
COPY HelloDocker.java /hello/
WORKDIR /hello
RUN javac HelloDocker.java
CMD ["java","HelloDocker"]
```
> 파일명은 **Dockerfile1**로 합니다.

<br><br><br>

위에서 만든 Dockerfile을 간단히 설명하자면 다음과 같습니다.
> 1. openjdk8을 Base image로 사용하고
> 2. /hello 경로에 HelloDocker.java 파일을 복사하고
> 3. /hello 경로로 이동한 뒤
> 4. HelloDocker.java 를 컴파일하고
> 5. docker container가 구동되면 `java HelloDocker`를 실행

준비를 마친 상태는 아래와 같습니다.
```bash
ubuntu $ tree
.
├── Dockerfile1
└── HelloDocker.java

0 directories, 2 files
```
> `tree`명령어를 실행하려면 `apt install tree`를 실행해서 설치 후 사용하세요.

<br><br><br>

이제 아래 명령어로 hellodocker 이미지를 생성합니다.
```bash
ubuntu $ docker build -f Dockerfile1 -t hellodocker:v1 .
Sending build context to Docker daemon  4.096kB
Step 1/5 : FROM openjdk:8
8: Pulling from library/openjdk
001c52e26ad5: Pull complete 
d9d4b9b6e964: Pull complete 
2068746827ec: Pull complete 
9daef329d350: Pull complete 
d85151f15b66: Pull complete 
52a8c426d30b: Pull complete 
8754a66e0050: Pull complete 
Digest: sha256:86e863cc57215cfb181bd319736d0baf625fe8f150577f9eb58bd937f5452cb8
Status: Downloaded newer image for openjdk:8
 ---> b273004037cc
Step 2/5 : COPY HelloDocker.java /hello/
 ---> 2d7c9424a882
Step 3/5 : WORKDIR /hello
 ---> Running in 8cd8a2cb2577
Removing intermediate container 8cd8a2cb2577
 ---> 87d65c7cc26e
Step 4/5 : RUN javac HelloDocker.java
 ---> Running in 08ad8bfa412d
Removing intermediate container 08ad8bfa412d
 ---> c0487125c1f3
Step 5/5 : CMD ["java","HelloDocker"]
 ---> Running in c2a8191d98a1
Removing intermediate container c2a8191d98a1
 ---> 296a43ae317f
Successfully built 296a43ae317f
Successfully tagged hellodocker:v1
```

> 💻 명령어 `docker build -f Dockerfile1 -t hellodocker:v1 .`{{exec}}

<br><br><br>

빌드가 성공하면 `docker images`명령어로 조회도 해보세요.
```bash
ubuntu $ docker images hellodocker
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
hellodocker   v1        296a43ae317f   20 seconds ago   526MB
```

> 💻 명령어 `docker images hellodocker`{{exec}}

이미지가 준비됐으니 이제 실행을 해볼게요.
```bash
ubuntu $ docker run --rm hellodocker:v1
Hello Docker!!!
```

> 💻 명령어 `docker run --rm hellodocker:v1`{{exec}}
- `--rm` : Automatically remove the container when it exits

결과가 예상한 것과 같은가요?

일단 첫 번째 단계는 성공입니다. (ง˙∇˙)ว