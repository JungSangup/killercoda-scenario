이제 도커 볼륨(Volume)을 이용해서 데이터를 유지하는 방법을 알아보겠습니다.

우리 샘플 애플리케이션(Todo List Manager)는 SQLite database를 사용하고 있습니다.  
데이터는 `/etc/todos/todo.db` 에 파일로 저장이 되고 있구요.

이제 도커 볼륨을 이용해서 데이터가 저장되는 위치를 host 머신의 경로로 바꿔보겠습니다.

먼저 도커 볼륨을 하나 생성합니다.

```bash
$ docker volume create todo-db
todo-db
```

> **명령어** : `docker volume create todo-db`{{exec}}

생성된 볼륨을 확인하려면 아래 명령어를 사용하면 됩니다.

```bash
$ docker volume list
DRIVER    VOLUME NAME
local     todo-db
```

> **명령어** : `docker volume list`{{exec}} 또는 `docker volume ls`{{exec}}

---

그리고, 볼륨의 더 자세한 정보를 알아보려면 아래 명령어를 사용하면 됩니다.

```bash
$ docker volume inspect todo-db
[
    {
        "CreatedAt": "2022-06-26T08:49:50Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

> **명령어** : `docker volume inspect todo-db`{{exec}}

Mountpoint가 바로 실제 데이터가 저장되는 Host 머신의 위치입니다.

---

이제 방금 생성한 볼륨을 우리 애플리케이션의 데이터 저장경로로 마운트해서 실행해 보겠습니다.

```bash
$ docker run --detach --publish 3000:3000 --volume todo-db:/etc/todos --name my-todo-manager rogallo/101-todo-app:1.0.0
Unable to find image 'rogallo/101-todo-app:1.0.0' locally
1.0.0: Pulling from rogallo/101-todo-app
ddad3d7c1e96: Pull complete
de915e575d22: Pull complete
7150aa69525b: Pull complete
d7aa47be044e: Pull complete
ac899a26a529: Pull complete
6aa912a6e5d1: Pull complete
1b3f4279bcb1: Pull complete
Digest: sha256:18e19953a27c5575840214c7a8d0a3acbcd78bf695d7c8884f4c401939de8913
Status: Downloaded newer image for rogallo/101-todo-app:1.0.0
a2c03d99dc1808287614353a48cf0b95b21bc648bb757454a25fbff7e88058e3
```

> **명령어** : `docker run --detach --publish 3000:3000 --volume todo-db:/etc/todos --name my-todo-manager [USER-NAME]/101-todo-app:1.0.0`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

여러분은 여러분의 이미지를 도커허브에서 받아와서 실행해보세요.

`--volume todo-db:/etc/todos`에서 콜론(:)을 구분자로 사용해서 첫 번째로는 volume의 이름을,
두 번째로는 마운트할 컨테이너의 경로를 적어줍니다.

---

이제 실행된 애플리케이션에 접속하고 오늘 할 일을 몇 개 적어볼까요? (아래 링크 클릭!)  

![ ](./img/hyperlink.png) [ToDo List Manager]({{TRAFFIC_HOST1_3000}})

![](./img/todo-list-sample3.png)

그리고, 컨테이너를 멈추고 삭제합니다.

```bash
$ docker stop my-todo-manager
my-todo-manager
$ docker rm my-todo-manager
my-todo-manager
```

> **명령어** : `docker stop my-todo-manager`{{exec}} , `docker rm my-todo-manager`{{exec}}  
> 컨테이너는 생성할때 --name 옵션으로 이름을 정하면, 이후에 이 이름을 이용할 수 있습니다.

---

이전 같으면(Volume을 사용하지 않았을 때는) 방금 저장한 할 일이 모두 사라지고 없겠죠?

이제 다시한번 같은 명령어로 우리 애플리케이션을 실행해 볼까요?

```bash
$ docker run --detach --publish 3000:3000 --volume todo-db:/etc/todos --name my-todo-manager rogallo/101-todo-app:1.0.0
bc7ab606fe61d12ec50ec8580963f0c169c4b6da428a3e67ecc384653cd1d161
```

> **명령어** : `docker run --detach --publish 3000:3000 --volume todo-db:/etc/todos --name my-todo-manager [USER-NAME]/101-todo-app:1.0.0`{{copy}}  
> [USER-NAME] 에는 여러분의 정보로 채워넣어 주세요.

그리고 다시 우리 애플리케이션으로 접속해보세요. (아래 링크 클릭!)

![ ](./img/hyperlink.png) [ToDo List Manager]({{TRAFFIC_HOST1_3000}})

어떤가요? 오늘 할 일 목록이 그대로 남아있나요? **정국이와 저녁식사**도 **장보기**도 잊지않고 할 수 있게 되었습니다.

![](./img/todo-list-sample3.png)
이제, 좀 제대로 된 애플리케이션이 된 것 같네요.... (ง˙∇˙)ว