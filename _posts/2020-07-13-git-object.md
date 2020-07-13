---
layout: single
title:  "ProGit 정리 - Git의 내부"
date:   2020-07-13 22:00:00 +0900
classes: wide
categories: git
tags: git
toc: true
toc_sticky: true
---

Git은 Content-addressable 파일 시스템이다. Git의 핵심은 Key-Value 데이터 저장소라는 것이다.

일단 `git init` 명령을 해보자. 디렉토리에 .git 디렉토리가 생긴다. 그 내부는 다음과 같다.

![git init](/assets/img/git_object/git-init.png)

## Blob 개체

다음 명령으로 Git에 텍스트 파일을 저장해본다. `echo 'test' | git hash-object -w --stdin`

![hash object](/assets/img/git_object/hash-object.png)

이제 `.git/objects` 디렉토리 아래에 어떤 파일이 생겼는지 살펴보면 위에서 나왔던 체크섬의 값을 가진 파일이 생긴 것을 볼 수 있다.

![find](/assets/img/git_object/find.png)

`git cat-file` 명령으로 위에서 나온 해시값으로 파일의 내용을 읽을 수 있다. 위에서 나온 값으로 예시를 들면 `git cat-file -p 9daeafb9864cf43055ae93beb0afd6c7d144bfa4` 라고 명령어를 치면 test 라는 내용이 나올 것이다.(hash값을 몇 자만 쳐도 된다.)

Git이 파일 버전을 관리하는 방식을 이해하기 위해 다음과 같은 상황을 만든다.

1. 새로운 파일(test.txt)을 하나 만든다.
2. Git repository에 저장
3. 해당 파일을 수정 후 git repository에 저장
4. 파일 내용을 첫 번째 버전으로 되돌린다.
5. 두 번째 버전을 다시 적용한다.

위와 같은 과정을 순서대로 진행해보면 아래와 같다.

![cat file](/assets/img/git_object/cat-file.png)

이런 종류의 개체를 Blob 개체라고 부른다. `cat-file -t` 명령으로 무슨 객체인지 확인할 수 있다.

## Tree 개체

Tree 개체에는 파일 이름을 저장한다. 파일 여러 개를 한꺼번에 저장할 수도 있다. 다른 프로젝트에서 `git cat-file -p master^{tree}` 명령을 쳐보자.

![cat file](/assets/img/git_object/cat-file-tree.png)

commerce project에서 해봤다. 여기서 gradle, src는 blob이 아니고 또 다른 tree 개체이다.

그러면 이제 직접 Tree 개체를 만들어보자. Git은 Staging Area의 상태대로 Tree 개체를 만들고 기록하기 때문에, Tree 개체를 만들기 위해서는 먼저 Staging Area에 파일을 추가하여 Index를 만들어야 한다.

`git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt`의 명령을 쳐본다. 여기서 100644는 보통의 파일을 나타낸다.(실행파일은 100755, 심볼릭 링크는 120000이다.)

Staging Area를 Tree 개체로 저장할 때는 write-tree 명령을 사용한다.

![write tree](/assets/img/git_object/write-tree.png)

여기서 새로운 파일도 추가하고 새 버전의 test.txt 파일도 Staging Area에 추가하고 새로운 Tree 개체를 만든다.

![new tree](/assets/img/git_object/new-tree.png)

이걸 보면 test.txt는 2번째 파일의 해시값인 1f7a7a1인 것을 알 수 있다. 그럼 이제 처음에 만든 Tree 개체를 하위 디렉토리로 만들어보자.

```sh
git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
git write-tree
# hash 값이 나옴
git cat-file -p 해시값
```

![read tree](/assets/img/git_object/read-tree.png)

--prefix 옵션을 주면 Tree 개체를 하위 디렉토리로 추가할 수 있다. 이 구조를 보면 다음과 같은 구조로 되어 있을 것이다.

![tree structure](/assets/img/git_object/tree-structure.png)

## Commit 개체

위에서 Tree 개체와 Blob 개체들을 만들어봤다. 근데 해시값으로 불러와야하고, 누가 언제 이 스냅샷을 만들었는지에 대한 정보가 전혀 없다. 이런 정보는 커밋 개체에 기록된다. 이런 커밋 개체는 commit-tree 로 만들 수 있다. 우리가 만들었던 첫 번째 Tree개체로 커밋 개체를 만들어보자.

```sh
echo 'first commit' | git commit-tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
```

![commit tree](/assets/img/git_object/commit-tree.png)

커밋 개체는 단순하다. 해당 스냅샷에서 최상단 Tree를 하나 가리킨다. 그리고 user.name 과 user.email 설정에서 가져온 Author/Committer 정보, 시간, 그리고 커밋 메시지가 들어간다. 이제 커밋 개체를 두개만 더 만들어보자. 각 커밋 개체는 이전 개체를 가리키도록 한다.

```sh
echo 'second commit' | git commit-tree 4d74ff21dadab0bc77516b42884fd9fdfd25a2cc -p b70785c0a00e9b70d102f787cb2ab57fbb8182de
echo 'third commit' | git commit-tree 8357c9d0649c103bc8490e183fa37055127cf867 -p 01f6cfeaf512131559d2b59d4fa40d17d1964a82
```

![commit tree 2](/assets/img/git_object/commit-tree-2.png)

이렇게 만들고 나면 이제 Git history를 만든 것이다. 마지막 커밋 개체의 해시값으로 git log를 실행하면 다음처럼 출력된다.

![git log](/assets/img/git_object/git-log.png)

우리가 단순히 지금까지 git add, git commit으로 해왔던 내용들을 저수준 명령어로 해낸 것이다. Git 개체를 조금 만들어봤으니 이제 .git/objects 디렉토리를 살펴보면 다음처럼 되어있다.

![git object](/assets/img/git_object/git-object.png)

## Git Refs

위에서처럼 `git log --stat fa0e799ce14bfb4d3719af3749fd665057cc35eb` 라고 실행하면 전체 히스토리를 볼 수 있지만, 결국에는 해시값을 계속 기억하고 있어야한다는 단점이 있다. 그래서 우리는 보통 HEAD라거나 master라거나 하는 외우기 쉬운 이름으로 된 포인터를 사용한다.

Git에서는 이런 것을 Refs라고 부른다. 이는 .git/refs 디렉토리에 있다.

지금은 refs에 아무것도 없고 기본 디렉토리만 있을 것이다. 이것을 이용해서 이제 쉬운 이름으로 커밋을 조회 해보자.

아래 사진처럼 refs를 생성하고 이를 이용해보자.

![update refs](/assets/img/git_object/update-refs-master.png)

![git log](/assets/img/git_object/git-log-master.png)

이렇게 직접 refs 파일을 고칠 수도 있고, update-ref 명령을 이용할수도 있다. (`git update-ref refs/heads/master fa0e799ce14bfb4d3719af3749fd665057cc35eb`)

Git 브랜치의 역할이 바로 이것이다. 이를 이용해서 두 번째 커밋을 가리키는 브랜치를 만들어보면 다음처럼 할 수 있다.

```sh
git update-ref refs/heads/test 01f6cfeaf512131559d2b59d4fa40d17d1964a82
git log --pretty=oneline test
```

![git log test](/assets/img/git_object/git-log-test.png)

`git branch 브랜치이름` 명령을 실행하면 내부적으로 Git은 update-ref 명령을 실행한다. 입력받은 브랜치 이름과 현 브랜치의 마지막 커밋의 해시값을 이용해 update-ref 명령을 실행한다.

### HEAD

`git branch 브랜치이름` 명령에서 Git은 어떻게 현 브랜치의 마지막 커밋의 해시값을 알아낼까? HEAD 파일은 현 브랜치를 가리키는 Symbolic Refs다. 실제로 `cat .git/HEAD`를 실행해보면 어떤 ref를 가리키고 있는지 알 수 있다.

`git checkout test`명령으로 현 브랜치를 바꾸고 다시 실행해보면 HEAD가 바뀌어있는 것을 확인할 수 있다.

![head](/assets/img/git_object/after-checkout.png)

그리고 git commit을 실행하면 커밋 개체가 만들어지고, 현재 HEAD가 가리키는 커밋의 해시값이 새로운 커밋 개체의 부모로 사용된다.

이 .git/HEAD 파일도 직접 수정할 수 있지만, symbolic-ref 명령으로 더 안전하게 사용할 수 있다.

`git symbolic-ref HEAD refs/heads/test` 명령으로 위에서 checkout 한 것처럼 할 수 있다.

![symbolic ref](/assets/img/git_object/symbolic-ref.png)

### 리모트

리모트 Refs도 있다. 리모트를 추가하고 push 하면 각 브랜치마다 push 한 마지막 커밋이 무엇인지 refs/remotes 디렉토리에 저장한다. 예를 들어 origin 리모트를 추가하고 master 브랜치를 push 해보자.

![symbolic ref](/assets/img/git_object/remote-ref.png)

리모트 refs는 checkout 할 수 없고, 읽기 용도로만 쓸 수 있는 브랜치다. 리모트 refs는 서버의 브랜치가 가리키는 커밋이 무엇인지 적어둔 북마크라고 보면 된다.

## 태그

태그 개체는 커밋 개체와 비슷하다. 커밋 개체처럼 누가, 언제 태그를 달았고 태그 메세지는 무엇이고 어떤 커밋을 가리키는지에 대한 정보가 들어있다. 태그 개체는 Tree개체가 아니라 Commit 개체를 가리키는 것이 차이점이다.

## 참고자료

- ProGit - 10. Git 개체
