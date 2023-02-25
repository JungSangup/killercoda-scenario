
이제 **ubuntu** 이미지를 다운로드(**pull**) 해 보겠습니다.
```bash
ubuntu@ip-172-31-23-60:~$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
677076032cca: Pull complete
Digest: sha256:9a0bdde4188b896a372804be2384015e90e3f84906b750c1a53539b585fbbe7f
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

> 💻 명령어 `docker pull ubuntu`{{exec}}

<br><br><br>

**tag**를 특정해서(**18.04**) 다운로드도 해보구요.
```bash
ubuntu@ip-172-31-23-60:~$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
72d9f18d70f3: Pull complete
Digest: sha256:a3765b4d74747b5e9bdd03205b3fbc4fa19a02781c185f97f24c8f4f84ed7bbf
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```

> 💻 명령어 `docker pull ubuntu:18.04`{{exec}}

<br><br><br>

받아온 이미지를 확인해볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ docker images ubuntu
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       18.04     5d2df19066ac   2 weeks ago   63.1MB
ubuntu       latest    58db3edaf2be   2 weeks ago   77.8MB
```

> 💻 명령어 `docker images ubuntu`{{exec}}

**tag**를 명시하지 않은 경우는 default tag인 `latest`를 받아오네요.  