
이제 **도커 레지스트리**에 대해 알아보고, 우리가 만든 애플리케이션을 등록해 보겠습니다.

먼저 [https://hub.docker.com/](https://hub.docker.com/) 에 가입(**Register**)을 합니다.
![h:350](./img/docker_create_repository1.png)
`Register`를 클릭하고, Docker account를 하나 만듭니다. (이미 있으면 있는 Account를 사용해도 됩니다.)

<br><br><br>

이제 실습을 위해서 **Repository**를 하나 생성합니다.  
로그인 후 `Create Repository` 버튼을 클릭해서 시작하면 됩니다.  
이름은 **todo-app** 으로 하고, Visibility는 **Private**으로 합니다.

![h:350](./img/docker_create_repository2.png)
> 무료 계정인 경우 Private repository는 하나만 만들 수 있습니다.
> 이미 사용중인 Private repository가 있으면, Public으로 만들어도 됩니다.

<br><br><br>

이제 여러분의 Docker repository가 하나 생겼습니다.
앞으로 이 곳에 여러분의 컨테이너 이미지를 저장하고 사용하면 됩니다.

![h:450](./img/docker_create_repository3.png)
> `[USER-NAME]/[REPOSITORY-NAME]` 이 여러분의 Repository 입니다. (e.g. `rogallo/todo-app`)

<br><br><br>

샘플 애플리케이션 이미지를 만들어 볼까요?  
먼저 소스코드를 Github에서 clone 합니다.
```bash
ubuntu@ip-172-31-23-60:~$ git clone https://github.com/JungSangup/todo_list_manager.git app
Cloning into 'app'...
remote: Enumerating objects: 131, done.
remote: Counting objects: 100% (131/131), done.
remote: Compressing objects: 100% (123/123), done.
remote: Total 131 (delta 51), reused 53 (delta 7), pack-reused 0
Receiving objects: 100% (131/131), 1.68 MiB | 24.63 MiB/s, done.
Resolving deltas: 100% (51/51), done.
```

> 💻 명령어 `git clone https://github.com/JungSangup/todo_list_manager.git app`{{exec}}

<br><br><br>

그리고, 소스코드가 있는 경로로 이동합니다.
```bash
ubuntu@ip-172-31-23-60:~$ cd app
ubuntu@ip-172-31-23-60:~/app$
```

> 💻 명령어 `cd app`{{exec}}

<br><br><br>

샘플 소스코드에는 두 개의 Tag가 있습니다.
```bash
ubuntu@ip-172-31-23-60:~/app$ git tag
v1.0.0
v2.0.0
```

> 💻 명령어 `git tag`{{exec}}

<br><br><br>

먼저 v1.0.0 이미지를 만듭니다. (`docker build` 명령어을 이용합니다.)  
v1.0.0 tag로 checkout을 하구요,
```bash
ubuntu@ip-172-31-23-60:~/app$ git checkout v1.0.0
Note: switching to 'v1.0.0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at c7a54f7 .
```

> 💻 명령어 `git checkout v1.0.0`{{exec}}

<br><br><br>

이제 Dockerfile을 이용해서 빌드를 합니다.
```bash
ubuntu@ip-172-31-23-60:~/app$ docker build -t rogallo/todo-app:1.0.0 .
Sending build context to Docker daemon  6.483MB
Step 1/5 : FROM node:10-alpine
10-alpine: Pulling from library/node
ddad3d7c1e96: Pull complete
de915e575d22: Pull complete
7150aa69525b: Pull complete
d7aa47be044e: Pull complete
Digest: sha256:dc98dac24efd4254f75976c40bce46944697a110d06ce7fa47e7268470cf2e28
Status: Downloaded newer image for node:10-alpine
 ---> aa67ba258e18
Step 2/5 : WORKDIR /app
 ---> Running in 8acf6accd870
Removing intermediate container 8acf6accd870
 ---> d63db0834be0
Step 3/5 : COPY . .
 ---> 58c34aafdf8f
Step 4/5 : RUN yarn install --production
 ---> Running in 86d59beb231e
yarn install v1.22.5
[1/4] Resolving packages...
[2/4] Fetching packages...
info fsevents@1.2.9: The platform "linux" is incompatible with this module.
info "fsevents@1.2.9" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 7.37s.
Removing intermediate container 86d59beb231e
 ---> f83bb9189656
Step 5/5 : CMD ["node", "/app/src/index.js"]
 ---> Running in 7e4c6cce23f8
Removing intermediate container 7e4c6cce23f8
 ---> df16f7f47728
Successfully built df16f7f47728
Successfully tagged rogallo/todo-app:1.0.0
```

> 💻 명령어 `docker build -t [USER-NAME]/todo-app:1.0.0 .`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

<br><br><br>

이제 v2.0.0 이미지를 만듭니다.  
v2.0.0 tag로 checkout을 하구요,
```bash
ubuntu@ip-172-31-23-60:~/app$ git checkout v2.0.0
Previous HEAD position was c7a54f7 .
HEAD is now at d1c1aaf Update index.html
```

> 💻 명령어 `git checkout v2.0.0`{{exec}}

<br><br><br>

Dockerfile을 이용해서 빌드를 합니다.
```bash
ubuntu@ip-172-31-23-60:~/app$ docker build -t rogallo/todo-app:2.0.0 .
Sending build context to Docker daemon  6.483MB
Step 1/5 : FROM node:10-alpine
 ---> aa67ba258e18
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> d63db0834be0
Step 3/5 : COPY . .
 ---> df9345ba8d5f
Step 4/5 : RUN yarn install --production
 ---> Running in 039e7f732624
yarn install v1.22.5
[1/4] Resolving packages...
[2/4] Fetching packages...
info fsevents@1.2.9: The platform "linux" is incompatible with this module.
info "fsevents@1.2.9" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 7.40s.
Removing intermediate container 039e7f732624
 ---> 69b3437c84a4
Step 5/5 : CMD ["node", "/app/src/index.js"]
 ---> Running in a3fa94d42b81
Removing intermediate container a3fa94d42b81
 ---> 7a79571ef432
Successfully built 7a79571ef432
Successfully tagged rogallo/todo-app:2.0.0
```

> 💻 명령어 `docker build -t [USER-NAME]/todo-app:2.0.0 .`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

<br><br><br>

그리고, 만들어진 이미지를 확인합니다.
```bash
ubuntu@ip-172-31-23-60:~/app$ docker images rogallo/todo-app
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
rogallo/todo-app   2.0.0     7a79571ef432   56 seconds ago   172MB
rogallo/todo-app   1.0.0     df16f7f47728   2 minutes ago    172MB
```

> 💻 명령어 `docker images [USER-NAME]/todo-app`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

<br><br><br>

이제 우리가 만든 이미지를 우리의 Docker hub repository에 업로드(push)해 보겠습니다.
먼저 로그인을 하구요,
```bash
ubuntu@ip-172-31-23-60:~/app$ docker login -u rogallo
Password:
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

> 💻 명령어 `docker login -u [USER-NAME]`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

<br><br><br>

아래 명령어로 docker hub의 우리 repository에 업로드(push) 해볼까요?
```bash
ubuntu@ip-172-31-23-60:~/app$ docker push rogallo/todo-app:1.0.0
The push refers to repository [docker.io/rogallo/todo-app]
a07f0156c43d: Pushed
8c40db749504: Pushed
4efca0eb4778: Pushed
edff9ff691d5: Layer already exists
cbe4b9146f86: Layer already exists
a6524c5b12a6: Layer already exists
9a5d14f9f550: Layer already exists
1.0.0: digest: sha256:5cee6f196aa06a6ba00a1b7c40a0b674510cf9f931785d9491daaa31af0d9de1 size: 1787
```

> 💻 명령어 `docker push [USER-NAME]/todo-app:1.0.0`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

<br><br><br>

똑 같은 방법으로 두 번재 이미지도 push합니다.
```bash
ubuntu@ip-172-31-23-60:~/app$ docker push rogallo/todo-app:2.0.0
The push refers to repository [docker.io/rogallo/todo-app]
ca2b6e73233e: Pushed
96fd2c585529: Pushed
4efca0eb4778: Layer already exists
edff9ff691d5: Layer already exists
cbe4b9146f86: Layer already exists
a6524c5b12a6: Layer already exists
9a5d14f9f550: Layer already exists
2.0.0: digest: sha256:13ba0a28f43b581d260611cd0ecb446eb5b1105334fb5bf9c86e08bbd66bfadb size: 1787
```

> 💻 명령어 `docker push [USER-NAME]/todo-app:2.0.0`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

<br><br><br>

[https://hub.docker.com/](https://hub.docker.com/) 에 방금 push한 이미지가 잘 올라가 있나요?

![h:300](./img/docker_create_repository4.png)
> 위의 두 개 이미지는 뒤의 과정에서 계속 필요하니, 잘 준비해두세요.

<br><br><br>

축하합니다.  (๑˃̵ᴗ˂̵)و

이제 여러분들의 저장공간도 생겼고, 언제 어디서든 방금 올려둔 이미지를 이용해서 여러분의 샘플 애플리케이션을 실행할 수 있게 됐습니다.  

<br>

이번 실습은 여기까지 입니다. 