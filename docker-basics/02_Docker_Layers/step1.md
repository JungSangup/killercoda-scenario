이번 실습은 컨테이너와 이미지, 그리고 Layer에 대한 실습입니다.

먼저 현재 Host에 있는 컨테이너 이미지를 모두 삭제하겠습니다.

```bash
$ docker rmi --force $(docker images -aq)
Untagged: docker-101:latest
Deleted: sha256:25d5349823915cda37e5cb03dc0e3520ca689fa9c9c67dae71e2168d9cb1f00e
Deleted: sha256:d86f278413c56f277591d2ea3de6739ffc8bf5b88c2cd8ba5ebaee22a28db4e2
Deleted: sha256:36e8ca2bb99a00e2b026794c5f20cd5d586e82dbf21319039d30dc166c65afa4
Deleted: sha256:027fa7ba112b507788a369d6a59ccd78a3bd17c0f4bcd883e9ee1be57ea5bfc5
Deleted: sha256:40d929a5abbd730be7f99fe0a7f8c98d9eba45ff79a20ab0a46958e7b911e5c8
Deleted: sha256:ff7e4526409ea0d633ef76c6e9d4df0ac7bb67fa649643b03eadef1ea00f1e0a
Deleted: sha256:6df32015dd8fca2d2723961255c5131350122e035b0e57b9ba52ad47a50a9231
```

> **명령어** : `docker rmi --force $(docker images -aq)`{{exec}}

모두 삭제됐는지 볼까요?

```bash
$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
$
```

> **명령어** : `docker images`{{exec}}

위와 같이 조회되는 image가 하나도 없는 상태여야 합니다.

---

이제 우리 도커가 어떤 스토리지 드라이버를 사용하는지 알아보겠습니다.

```bash
$ docker info | grep -i storage
 Storage Driver: overlay2
```

> **명령어** : `docker info | grep -i storage`{{exec}}

[스토리지 드라이버](https://docs.docker.com/storage/storagedriver/select-storage-driver/)에 따라서 이미지가 저장되는 장소가 달라집니다.
우리 시스템은 `overlay2` 드라이버이기 때문에 아래 경로가 사용됩니다.

```bash
$ ls -al /var/lib/docker/overlay2
total 20
drwx--x---  3 root root 12288 Jun 17 05:55 .
drwx--x--- 13 root root  4096 Jun 17 02:51 ..
drwx------  2 root root  4096 Jun 17 05:55 l
```

> **명령어** : `ls -al /var/lib/docker/overlay2`{{exec}}

위 그림처럼 깨끗하게 정리된 상태에서 시작해볼게요.

---

nginx 이미지를 하나 pull 하구요.

```bash
$ docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
42c077c10790: Pull complete
62c70f376f6a: Pull complete
915cc9bd79c2: Pull complete
75a963e94de0: Pull complete
7b1fab684d70: Pull complete
db24d06d5af4: Pull complete
Digest: sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981bafbb884165e514
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

> **명령어** : `docker pull nginx`{{exec}}

이미지 목록을 확인을 합니다.

```bash
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    0e901e68141f   2 weeks ago   142MB
```

> **명령어** : `docker images`{{exec}}

---

이제 다시 저장된 위치(`/var/lib/docker/overlay2`)가 어떻게 바뀌었나 확인해볼까요?

```bash
$ ls -al /var/lib/docker/overlay2
total 44
drwx--x---  9 root root 12288 Jun 17 06:01 .
drwx--x--- 13 root root  4096 Jun 17 02:51 ..
drwx--x---  4 root root  4096 Jun 17 06:01 5dc378c99eedd46fc3aca29e771c7a044b91ac71f4ebff2da0be8260da057350
drwx--x---  4 root root  4096 Jun 17 06:01 73b619a2efde2fe0bac0940fa6892ad5c7edc9fef14ba2b0a5605ce6ae4963c0
drwx--x---  4 root root  4096 Jun 17 06:01 c77d0c3677b1d3d949038beef16bcf0dfe6b9da76c79efcf558463c937779022
drwx--x---  4 root root  4096 Jun 17 06:01 d060bd3b029045629a36a42774c68afc43716d9bfdbbb2bcca810e530262e604
drwx--x---  4 root root  4096 Jun 17 06:01 d0cd4377c363119f63c59a2d32db03e0deba873c11cc9f0885a9b343b57387a1
drwx--x---  3 root root  4096 Jun 17 06:01 fccb8e1376f1c76242fc79698262f7498b78c3c8090fc9f8b79bd29d6b8e460d
drwx------  2 root root  4096 Jun 17 06:01 l
```

> **명령어** : `ls -al /var/lib/docker/overlay2`{{exec}}

뭔가 많이 생겼네요.
혹시 눈치 채셨나요?
`docker pull` 할때 표시된 layer만큼 overlay 아래 디렉토리가 생성된걸...

이번엔 컨테이너를 실행해 보겠습니다.

```bash
$ docker run --detach --label "color=red" nginx
237e34821eea34627bc4de44741318a45a386d3d76c249ff958cad89b0ffb176
```

> **명령어** : `docker run --detach --label "color=red" nginx`{{exec}}

다음 실습을 위해서 label(`color=red`)을 달아뒀습니다.

---

​     
잘 실행되고 있나 살펴볼게요.

```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS     NAMES
237e34821eea   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    ecstatic_visvesvaraya
```

> **명령어** : `docker ps`{{exec}}

이제 `overlay2` 디렉토리는 어떻게 바뀌어 있을까요?

```bash
$ ls -alt /var/lib/docker/overlay2
total 52
drwx--x---  5 root root  4096 Jun 17 06:04 85790c7c79f6a9ed42c56001ecbcca73a247e3dae52e70642cfa1deb8717b345
drwx--x--- 11 root root 12288 Jun 17 06:04 .
drwx--x---  4 root root  4096 Jun 17 06:04 85790c7c79f6a9ed42c56001ecbcca73a247e3dae52e70642cfa1deb8717b345-init
drwx------  2 root root  4096 Jun 17 06:04 l
drwx--x---  4 root root  4096 Jun 17 06:04 d060bd3b029045629a36a42774c68afc43716d9bfdbbb2bcca810e530262e604
drwx--x---  4 root root  4096 Jun 17 06:01 c77d0c3677b1d3d949038beef16bcf0dfe6b9da76c79efcf558463c937779022
drwx--x---  4 root root  4096 Jun 17 06:01 d0cd4377c363119f63c59a2d32db03e0deba873c11cc9f0885a9b343b57387a1
drwx--x---  4 root root  4096 Jun 17 06:01 5dc378c99eedd46fc3aca29e771c7a044b91ac71f4ebff2da0be8260da057350
drwx--x---  4 root root  4096 Jun 17 06:01 73b619a2efde2fe0bac0940fa6892ad5c7edc9fef14ba2b0a5605ce6ae4963c0
drwx--x---  3 root root  4096 Jun 17 06:01 fccb8e1376f1c76242fc79698262f7498b78c3c8090fc9f8b79bd29d6b8e460d
drwx--x--- 13 root root  4096 Jun 17 02:51 ..
```

> **명령어** : `ls -alt /var/lib/docker/overlay2`{{exec}}

두 개의 디렉토리가 더 생긴걸 볼 수 있습니다. ( `-t` 옵션을 사용하여 최근 디렉토리를 상위에 표시함.)

---

​      
이게 우리가 배운 R/W Layer인 Container layer입니다.
실행되는 컨테이너에서 발생하는 모든 변경사항은 바로 여기 기록되게 됩니다.

그럼, 하나를 더 실행하면 어떻게 될까요?

```bash
$ docker run --detach --label "color=blue" nginx
670302b2287fe6bc297ba9e0499c285febd4752a3faffb4eda981ee55805190c
```

> **명령어** : `docker run --detach --label "color=blue" nginx`{{exec}}

이번엔 `color=blue` label을 달아뒀습니다.

컨테이너를 확인해볼까요?

```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS     NAMES
670302b2287f   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    friendly_leakey
237e34821eea   nginx     "/docker-entrypoint.…"   12 minutes ago       Up 12 minutes       80/tcp    ecstatic_visvesvaraya
```

> **명령어** : `docker ps`{{exec}}

---

​     
그리고, `overlay2`디렉토리에는 R/W Layer만 추가된 걸 확인할 수 있습니다.

```bash
$ ls -alt /var/lib/docker/overlay2
total 60
drwx--x---  5 root root  4096 Jun 17 06:15 e00fa3a0cd51eb127520ba0e2fc6cd507b0b8c6a374a26c42813dcac5190afad
drwx--x--- 13 root root 12288 Jun 17 06:15 .
drwx--x---  4 root root  4096 Jun 17 06:15 e00fa3a0cd51eb127520ba0e2fc6cd507b0b8c6a374a26c42813dcac5190afad-init
drwx------  2 root root  4096 Jun 17 06:15 l
drwx--x---  5 root root  4096 Jun 17 06:04 85790c7c79f6a9ed42c56001ecbcca73a247e3dae52e70642cfa1deb8717b345
drwx--x---  4 root root  4096 Jun 17 06:04 85790c7c79f6a9ed42c56001ecbcca73a247e3dae52e70642cfa1deb8717b345-init
drwx--x---  4 root root  4096 Jun 17 06:04 d060bd3b029045629a36a42774c68afc43716d9bfdbbb2bcca810e530262e604
drwx--x---  4 root root  4096 Jun 17 06:01 c77d0c3677b1d3d949038beef16bcf0dfe6b9da76c79efcf558463c937779022
drwx--x---  4 root root  4096 Jun 17 06:01 d0cd4377c363119f63c59a2d32db03e0deba873c11cc9f0885a9b343b57387a1
drwx--x---  4 root root  4096 Jun 17 06:01 5dc378c99eedd46fc3aca29e771c7a044b91ac71f4ebff2da0be8260da057350
drwx--x---  4 root root  4096 Jun 17 06:01 73b619a2efde2fe0bac0940fa6892ad5c7edc9fef14ba2b0a5605ce6ae4963c0
drwx--x---  3 root root  4096 Jun 17 06:01 fccb8e1376f1c76242fc79698262f7498b78c3c8090fc9f8b79bd29d6b8e460d
drwx--x--- 13 root root  4096 Jun 17 02:51 ..
```

> **명령어** : `ls -alt /var/lib/docker/overlay2`{{exec}}

같은 이미지로 여러개의 컨테이너를 실행해도, R/O Layer는 공유하고 R/W Layer만 추가해서 만들어지네요.

다음 실습을 위해서 blue는 삭제할게요.

```bash
$ docker rm --force $(docker ps --filter "label=color=blue" --quiet)
670302b2287f
```

> **명령어** : `docker rm --force $(docker ps --filter "label=color=blue" --quiet)`{{exec}}