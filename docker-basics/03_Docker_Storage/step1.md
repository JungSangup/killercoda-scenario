앞서 우리는 도커가 사용하는 레이어 파일시스템에 대해 알아보았습니다.  
컨테이너가 실행될 때 마다 새로운 레이어(Container layer, R/W layer)가 생성되고, 컨테이너가 삭제될 경우 그 레이어의 내용은 사라지게 됩니다.

직접 한번 컨테이너를 생성해서 볼까요?  
먼저 ubuntu 컨테이너를 하나 실행합니다.
```bash
ubuntu@ip-172-31-23-60:~$ docker run --name my-ubuntu1 --detach ubuntu bash -c "echo 'Hello Docker...' > /test.txt && tail -f /dev/null"
69a984336a12e4d186399730f1ff3f51300caf95ea9d70a59a235c334da26ea2
```

> 💻 명령어 `docker run --name my-ubuntu1 --detach ubuntu bash -c "echo 'Hello Docker...' > /test.txt && tail -f /dev/null"`{{exec}}

"Hello Docker..." 라는 문자열을 담은 txt파일(**/test.txt**)을 하나 만들고, 컨테이너를 **running**상태로 두기 위해서 `tail`명령을 실행했습니다.

<br><br><br>

이제 우리가 만든 파일을 한 번 볼까요?
```bash
ubuntu@ip-172-31-23-60:~$ docker exec my-ubuntu1 cat /test.txt
Hello Docker...
```

> 💻 명령어 `docker exec my-ubuntu1 cat /test.txt`{{exec}}

<br>

우리가 적어놓은 테스트문구("Hello Docker...")가 보일거예요.  
컨테이너가 실행되는 동안 처리된 내용이기 때문에 Container layer에 이 내용이 기록되게 됩니다.

<br><br><br>

이제 같은 ubuntu이미지를 이용해서 새로운 컨테이너를 실행하고, test.txt파일이 있나 살펴봅시다.
```bash
ubuntu@ip-172-31-23-60:~$ docker run -it --name my-ubuntu2 ubuntu ls /
bin   dev  home  lib32	libx32	mnt  proc  run	 srv  tmp  var
boot  etc  lib	 lib64	media	opt  root  sbin  sys  usr
```

> 💻 명령어 `docker run -it --name my-ubuntu2 ubuntu ls /`{{exec}}

당연히 없겠죠... 왜 그럴까요? (◔,◔o)

우리가 앞에서 알아본 **레이어**의 개념을 잘 떠올려 보세요.

앞에서 테스트한 컨테이너는 삭제하고 다음으로 넘어갈게요.  
아래 명령어를 실행해주세요.

```bash
ubuntu@ip-172-31-23-60:~$ docker rm -f $(docker ps -a -f "name=my-ubuntu1" -f "name=my-ubuntu2" -q)
06388b0ee44f
69a984336a12
```

> 💻 명령어 `docker rm -f $(docker ps -a -f "name=my-ubuntu1" -f "name=my-ubuntu2" -q)`{{exec}}