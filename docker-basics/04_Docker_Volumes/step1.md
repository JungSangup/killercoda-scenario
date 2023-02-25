앞서 우리는 도커가 사용하는 레이어 파일시스템에 대해 알아보았습니다.

컨테이너가 실행될 때 마다 새로운 레이어(Container layer, R/W layer)가 생성되고, 컨테이너가 삭제될 경우 그 레이어의 내용은 사라지게 됩니다.

직접 한번 컨테이너를 생성해서 볼까요?

먼저 ubuntu를 하나 실행합니다.

```bash
$ docker run --name my-ubuntu --detach ubuntu bash -c "echo 'Hello Docker...' > /test.txt && tail -f /dev/null"
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
405f018f9d1d: Pull complete
Digest: sha256:b6b83d3c331794420340093eb706a6f152d9c1fa51b262d9bf34594887c2c7ac
Status: Downloaded newer image for ubuntu:latest
6f4a471389915ed1d2b47d7814899968a3f05aab34de2b62721a0ee694a38f70
```

> **명령어** : `docker run --name my-ubuntu --detach ubuntu bash -c "echo 'Hello Docker...' > /test.txt && tail -f /dev/null"`{{exec}}

"Hello Docker..." 라는 문자열을 담은 txt파일(`/test.txt`)을 하나 만들고,
컨테이너를 running상태로 두기 위해서 `tail`명령을 실행했습니다.

---

이제 우리가 만든 파일을 한 번 볼까요?

```bash
$ docker exec my-ubuntu cat /test.txt
Hello Docker...
```

> **명령어** : `docker exec my-ubuntu cat /test.txt`{{exec}}

우리가 적어놓은 테스트문구("Hello Docker...")가 보일거예요.
컨테이너가 실행되는 동안 처리된 내용이기 때문에 Container layer에 이 내용이 기록되게 됩니다.

이제 같은 ubuntu이미지를 이용해서 새로운 컨테이너를 실행하고, test.txt파일이 있나 살펴봅시다.

```bash
$ docker run -it ubuntu ls /
bin   dev  home  lib32	libx32	mnt  proc  run	 srv  tmp  var
boot  etc  lib	 lib64	media	opt  root  sbin  sys  usr
```

> **명령어** : `docker run -it ubuntu ls /`{{exec}}

당연히 없겠죠... 왜 그럴까요? (◔,◔o)

우리가 앞에서 알아본 레이어의 개념을 잘 떠올려 보세요.