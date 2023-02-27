첫 번째 실습입니다.  

그냥 일단 무작정 따라해보세요.  
자세한건 천천히 알아볼게요.  

![](./img/todo-list-sample1.png)

이번 과정에서 사용할 샘플 애플리케이션인 **ToDo List Manager** 입니다.  
할 일 목록(Item)을 등록하고 관리할 수 있는 간단한 애플리케이션입니다.

---

먼저 필요한 소스코드를 Github에서 다운로드 합니다.  

```bash
ubuntu $ git clone https://github.com/JungSangup/todo_list_manager.git app
Cloning into 'app'...
remote: Enumerating objects: 131, done.
remote: Counting objects: 100% (131/131), done.
remote: Compressing objects: 100% (123/123), done.
remote: Total 131 (delta 51), reused 53 (delta 7), pack-reused 0
Receiving objects: 100% (131/131), 1.68 MiB | 1.22 MiB/s, done.
Resolving deltas: 100% (51/51), done.
```

> 💻 명령어 `git clone https://github.com/JungSangup/todo_list_manager.git app`{{exec}}

소스코드 준비가 됐으면 `app` 디렉토리로 이동해서 어떤 파일들이 있는지 살펴볼까요?

```bash
ubuntu $ cd app
ubuntu $ ls -al
total 204
drwxr-xr-x 5 root root   4096 Feb 27 03:04 .
drwx------ 5 root root   4096 Feb 27 03:04 ..
drwxr-xr-x 8 root root   4096 Feb 27 03:04 .git
-rw-r--r-- 1 root root    109 Feb 27 03:04 Dockerfile
-rw-r--r-- 1 root root    626 Feb 27 03:04 package.json
drwxr-xr-x 4 root root   4096 Feb 27 03:04 spec
drwxr-xr-x 5 root root   4096 Feb 27 03:04 src
-rw-r--r-- 1 root root 179361 Feb 27 03:04 yarn.lock
```

> 💻 명령어 `cd app`{{exec}}  
> 💻 명령어 `ls -al`{{exec}}