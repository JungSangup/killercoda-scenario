첫 번째 실습입니다.  

그냥 일단 무작정 따라해보세요.  
자세한건 천천히 알아볼게요.  

![](./img/todo-list-sample1.png)

이번 과정에서 사용할 샘플 애플리케이션인 **ToDo List Manager** 입니다.  
할 일 목록(Item)을 등록하고 관리할 수 있는 간단한 애플리케이션입니다.

---

먼저 필요한 소스코드를 Github에서 다운로드 합니다.  

```bash
$ git clone https://github.com/JungSangup/todo_list_manager.git app
Cloning into 'app'...
remote: Enumerating objects: 54, done.
remote: Counting objects: 100% (54/54), done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 54 (delta 4), reused 54 (delta 4), pack-reused 0
Receiving objects: 100% (54/54), 1.67 MiB | 4.76 MiB/s, done.
Resolving deltas: 100% (4/4), done.
```

> **명령어** : `git clone https://github.com/JungSangup/todo_list_manager.git app`{{exec}}

소스코드 준비가 됐으면 `app` 디렉토리로 이동해서 어떤 파일들이 있는지 살펴볼까요?

```bash
$ cd app
$ ls -al
total 204
drwxrwxr-x 5 ubuntu ubuntu   4096 Jun 17 02:54 .
drwxr-x--- 8 ubuntu ubuntu   4096 Jun 17 02:54 ..
drwxrwxr-x 8 ubuntu ubuntu   4096 Jun 17 02:54 .git
-rw-rw-r-- 1 ubuntu ubuntu    105 Jun 17 02:54 Dockerfile
-rw-rw-r-- 1 ubuntu ubuntu    626 Jun 17 02:54 package.json
drwxrwxr-x 4 ubuntu ubuntu   4096 Jun 17 02:54 spec
drwxrwxr-x 5 ubuntu ubuntu   4096 Jun 17 02:54 src
-rw-rw-r-- 1 ubuntu ubuntu 179361 Jun 17 02:54 yarn.lock
```

> **명령어** : `cd app`{{exec}} , `ls -al`{{exec}}