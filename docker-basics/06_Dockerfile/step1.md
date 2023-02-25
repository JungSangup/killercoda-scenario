이번 실습은 Dockerfile을 이용하여 이미지를 생성하는 방법을 알아보겠습니다.  
어떻게 하면 좀 더 효율적인 이미지를 만들 수 있는지도 알아볼게요.

<br>

#### Java Application

먼저 간단한 Java 파일(`HelloDocker.java`)을 준비합니다.

```java
public class HelloDocker {
    public static void main(String[] args) {
        System.out.println("Hello Docker!!!");
    }
}
```

- VI Editor를 사용하거나, Editor탭을 눌러 Web-IDE를 사용하여 작성합니다.  
- 작성이 어려운 경우 lab 디렉토리 아래에 있는 파일(`HelloDocker.java`)을 사용하셔도 됩니다.

<br>

#### Dockerfile

이제 Dockerfile 하나를 준비합니다.
이미지를 만드는 여러가지 방법 중 하나인 Dockerfile을 이용한 빌드를 해볼게요.

```dockerfile
FROM openjdk:8
COPY HelloDocker.java /hello/
WORKDIR /hello
RUN javac HelloDocker.java
CMD ["java","HelloDocker"]
```

- VI Editor를 사용하거나, Editor탭을 눌러 Web-IDE를 사용하여 작성합니다.  
- 작성이 어려운 경우 lab 디렉토리 아래에 있는 파일(`Dockerfile1`)을 사용하셔도 됩니다.

---

위에서 만든 Dockerfile을 간단히 설명하자면 다음과 같습니다.

> 1. openjdk8을 Base image로 사용하고
> 2. /hello 경로에 HelloDocker.java 파일을 복사하고
> 3. /hello 경로로 이동한 뒤
> 4. HelloDocker.java 를 컴파일하고
> 5. docker container가 구동되면 `java HelloDocker`를 실행

준비를 마친 상태는 아래와 같습니다.

```bash
$ tree
.
├── Dockerfile
└── HelloDocker.java

0 directories, 2 files
```

> `tree` 명령이 실행되지 않는경우, 아래 명령어로 설치 후 해보세요.  
> **명령어** : `sudo apt-get update && sudo apt-get install tree`{{exec}}

이제 우리 애플리케이션 소스파일인 `HelloDocker.java` 와 이미지를 만들 때 사용할 `Dockerfile`이 준비됐습니다.