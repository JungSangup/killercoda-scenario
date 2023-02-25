이제 샘플 애플리케이션을 컨테이너 이미지로 만들어 보겠습니다.  
아래 명령어는 도커 이미지를 만드는(build) 명령어 입니다.

```bash
$ docker build --tag docker-101 .
Sending build context to Docker daemon  6.489MB
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
 ---> Running in bf92de6fa1c4
Removing intermediate container bf92de6fa1c4
 ---> ff7e4526409e
Step 3/5 : COPY . .
 ---> 027fa7ba112b
Step 4/5 : RUN yarn install --production
 ---> Running in 1e6469e6dd8d
yarn install v1.22.5
[1/4] Resolving packages...
[2/4] Fetching packages...
info fsevents@1.2.9: The platform "linux" is incompatible with this module.
info "fsevents@1.2.9" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 9.08s.
Removing intermediate container 1e6469e6dd8d
 ---> d86f278413c5
Step 5/5 : CMD ["node", "/app/src/index.js"]
 ---> Running in 6aa4603e6f88
Removing intermediate container 6aa4603e6f88
 ---> 25d534982391
Successfully built 25d534982391
Successfully tagged docker-101:latest
```

> 💻 명령어 `docker build --tag docker-101 .`{{exec}}

뭔가 열심히 만든 것 같네요.  
Download도 하고, COPY도 하고, Install도 하고...  
​
이제 잘 만들어졌는지 볼까요?  
현재 Host에 있는 이미지를 조회하는 명령어입니다.  

```bash
$ docker images
REPOSITORY   TAG         IMAGE ID       CREATED         SIZE
docker-101   latest      25d534982391   2 minutes ago   172MB
```

> 💻 명령어 `docker images`{{exec}}

위 처럼 docker-101 이 보이면 성공입니다. ٩(ˊᗜˋ*)و