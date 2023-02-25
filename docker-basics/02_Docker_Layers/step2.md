이제 우리만의 새로운 이미지를 만들어 보겠습니다.  
여러가지 방법이 있지만 이번 실습은 실행중인 컨테이너의 내용을 반영한 새로운 이미지를 만드는 것입니다.  

앞에서 실행한 nginx를 활용하도록 하겠습니다.  

먼저 잘 실행되고 있나 보구요.  

```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
237e34821eea   nginx     "/docker-entrypoint.…"   20 minutes ago   Up 20 minutes   80/tcp    ecstatic_visvesvaraya
```

> **명령어** : `docker ps`{{exec}}

잘 살아있으면, 다음 명령으로 새로운 이미지를 만들어 보겠습니다.  

```bash
$ docker commit $(docker ps --filter "label=color=red" --quiet) nginx:my-tag
sha256:58a92488d2a1246d6caa14d2b5b46e9999f2a5abffac86356188f128985288ef
```

> **명령어** : `docker commit $(docker ps --filter "label=color=red" --quiet) nginx:my-tag`{{exec}}

뭔가 복잡해 보이지만 별거 아닙니다.

`$(...)` 이 부분은 우리가 Label(`color=red`)을 붙인 컨테이너의 ID를 조회하는 명령입니다.
git의 `commit` 명령어 처럼, docker도 `commit` 명령을 이용해서 새로운 이미지를 만들 수 있습니다.

---

이제 어떤 이미지가 있나 조회해볼까요?

```bash
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
nginx        my-tag    58a92488d2a1   About a minute ago   142MB
nginx        latest    0e901e68141f   2 weeks ago          142MB
```

> **명령어** : `docker images`{{exec}}

방금 우리가 `docker commit` 명령으로 만든 `nginx:my-tag` 이미지가 보이네요.

`overlay2` 디렉토리는요?

```bash
$ ls -alt /var/lib/docker/overlay2
total 56
drwx--x--- 12 root root 12288 Jun 17 06:26 .
drwx--x---  4 root root  4096 Jun 17 06:26 a17bce650b53c0a640af9ca29de030366978a37f2874e68ecc2fe5bd349db935
drwx------  2 root root  4096 Jun 17 06:26 l
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

---

네, 여기도 하나가 더 생겼네요.
​     
이제 실행중인 컨테이너까지 멈추고 삭제까지 해볼게요.

```bash
$ docker rm -f $(docker ps --filter "label=color=red" -q)
237e34821eea
```

> **명령어** : `docker rm -f $(docker ps --filter "label=color=red" -q)`{{exec}}

​     
이제 `overlay2` 디렉토리는 어떻게 되었을까요?

```bash
$ ls -alt /var/lib/docker/overlay2
total 48
drwx--x--- 10 root root 12288 Jun 17 06:30 .
drwx------  2 root root  4096 Jun 17 06:30 l
drwx--x---  4 root root  4096 Jun 17 06:26 a17bce650b53c0a640af9ca29de030366978a37f2874e68ecc2fe5bd349db935
drwx--x---  4 root root  4096 Jun 17 06:04 d060bd3b029045629a36a42774c68afc43716d9bfdbbb2bcca810e530262e604
drwx--x---  4 root root  4096 Jun 17 06:01 c77d0c3677b1d3d949038beef16bcf0dfe6b9da76c79efcf558463c937779022
drwx--x---  4 root root  4096 Jun 17 06:01 d0cd4377c363119f63c59a2d32db03e0deba873c11cc9f0885a9b343b57387a1
drwx--x---  4 root root  4096 Jun 17 06:01 5dc378c99eedd46fc3aca29e771c7a044b91ac71f4ebff2da0be8260da057350
drwx--x---  4 root root  4096 Jun 17 06:01 73b619a2efde2fe0bac0940fa6892ad5c7edc9fef14ba2b0a5605ce6ae4963c0
drwx--x---  3 root root  4096 Jun 17 06:01 fccb8e1376f1c76242fc79698262f7498b78c3c8090fc9f8b79bd29d6b8e460d
drwx--x--- 13 root root  4096 Jun 17 02:51 ..
```

> **명령어** : `ls -alt /var/lib/docker/overlay2`{{exec}}

---

​     
처음 `docker pull`해서 생성된 layer에, `docker commit`해서 생성된 레이어만 하나 추가되어 있네요. ( +1 )

아래 그림을 다시한 번 떠올려 보면서 마무리 하겠습니다.

![](./img/container-layers.jpeg) ![](./img/sharing-layers.jpeg)


이번 실습은 여기까지 입니다.

＿〆(。╹‿ ╹ 。)