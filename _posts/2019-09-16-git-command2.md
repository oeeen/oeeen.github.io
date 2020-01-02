---
layout: single
title:  "ProGit 정리 - 검색, 히스토리"
date:   2019-09-16 23:00:00 +0900
classes: wide
categories: git
tags: git
toc: true
toc_sticky: true
---

## 검색

함수의 정의나 함수가 호출되는 곳을 검색해야 하는 경우가 있다.

`git grep` 명령을 이용해서 커밋 트리의 내용이나 워킹 디렉토리의 내용을 쉽게 찾을 수 있다.

명령을 실행할 때 -n 옵션을 추가해서 찾을 문자열이 위치한 라인 번호도 같이 출력한다.

`git grep -n signin` 이라는 명령으로 signin이 들어간 곳을 찾아보았다.

![git grep](/assets/img/git_command2/grep.png)

`--count` 옵션으로 단순히 몇 개를 찾았는지 나타낼 수 있다.

![git grep count](/assets/img/git_command2/grep_count.png)

`-p` 옵션으로 매칭되는 라인이 있는 함수나 메서드를 찾을 수 있다. 찾을 문자열이 호출된 메서드들을 같이 볼 수 있게 된다.

![git grep p](/assets/img/git_command2/grep_p.png)

`--and` 옵션으로 여러 단어가 한 라인에 동시에 나타나는 줄을 찾을 수 있다.

`--break`, `--heading` 옵션으로 더 읽기 쉽게 출력할 수 있다.

## 로그 검색

찾으려는 단어가 어떤 파일에 있는 게 아니라 언제 추가 되었고 깃 로그 상에 어느 시점에 나타나는 지를 찾을 수 있다.

예시로 `PASSWORD_EXCEPTION_MESSAGE`라는 상수가 어떤 커밋에 추가되었는지 알고 싶으면 -S 옵션을 추가해서 `git log -SPASSWORD_EXCEPTION_MESSAGE`라고 명령하면 된다.

### 라인로그 검색

라인 히스토리를 검색할 수 있다. `git log`에 -L 옵션을 붙여서 어떤 함수나 한 라인의 히스토리를 볼 수 있다.

`git log -L :git_deflate_bound:zlib.c` 라는 명령을 실행하면 zlib.c 파일 내의 git_deflate_bound 함수의 모든 변경 사항을 볼 수 있다.

![git log -L](/assets/img/git_command2/log_L.png)

나는 `git log -L '/public List<ArticleResponseDto> findAll/',/^}/:./src/main/java/com/woowacourse/sunbook/application/service/ArticleService.java` 이런 명령으로 라인 조회를 한 번 해봤다.

그 결과 해당 메서드가 수정된 커밋의 내역이 쭈욱 나온다. 내역은 너무 자세한 내용이라 생략한다.

## 히스토리

Git을 쓰다보면, 커밋 히스토리를 수정할 일이 많다.

### 마지막 커밋 수정하기

마지막 커밋을 수정할 일은 보통 두 가지가 있는데, 커밋 메시지 수정과 커밋할 파일 목록 수정이다.

커밋 메시지 수정은 `git commit --amend` 명령으로 할 수 있다.

커밋할 파일 목록 수정은, `git add`로 파일을 추가하거나, `git rm`으로 파일 추적을 제거한 후 `git commit --amend` 명령을 수행하면 된다.

단, 당연하지만 **이미 푸시한 커밋에 대해서는 수정하면 안된다!**

### 커밋 메시지 여러개 수정하기

가장 마지막 커밋이 아닌 예전 커밋을 수정하려면 다른 도구를 써야 한다.

rebase를 이용해서 수정할 수 있는데, 어느 시점부터 HEAD까지의 커밋을 한꺼번에 Rebase할 수 있다.

`git rebase -i HEAD~3` 명령을 실행 시켜보자. 여기서 `HEAD~3`은 편집하려는 마지막 커밋의 `부모 커밋`이다.

이 명령은 Rebase이기 때문에 메시지를 수정하든 안하든 HEAD~3..HEAD 범위의 모든 커밋을 수정한다.

![git log](/assets/img/git_command2/log.png)

`git log` 명령에는 위 그림처럼 나오지만 rebase 명령에는 이와 반대로 아래 그림처럼 나오게 된다.

![git rebase](/assets/img/git_command2/rebase.png)

수정을 하려면 이제 pick이라는 단어를 edit으로 바꾼다.

![git edit](/assets/img/git_command2/edit.png)

그러면 다음에 어떻게 해야할 지 나온다. 시키는 대로 진행하면 `git commit --amend`로 커밋 메시지를 수정하고, `git rebase --continue`로 계속해서 rebase를 진행하면 된다.

![git edit after](/assets/img/git_command2/edit_after.png)

### 커밋 순서 바꾸기, 지우기

대화형 Rebase 명령으로 커밋을 지울 수도 있고, 순서를 바꿀 수도 있다.

Squash 명령으로 이전 커밋과 합쳐서 하나의 커밋으로 만들 수도 있다. 다양한 대화형 rebase 명령이 있기 때문에 설명을 읽어보면서 진행하면 많은 일들을 할 수 있다.

![git rebase command](/assets/img/git_command2/rebase_command.png)

### 커밋 분리하기

기존의 하나의 커밋을 두개, 세개로 쪼개서 커밋을 하고 싶다. 그러면 아래처럼 하면 된다.

아래 상태에서 시작해보자

![git rebase - 1](/assets/img/git_command2/rebase_1.png)

이 상황에서 `git reset HEAD^`를 하면 현재 edit으로 골랐던 [테스트]테스트 보완(#56) 커밋이 풀린다.

그 이후 `git add` 명령으로 원하는 파일을 추가하고 `git commit` 명령으로 커밋한다. 이런 식으로 진행해서 커밋을 쪼갤 수 있다.

## filter-branch

수정해야 하는 커밋이 너무 많아서 Rebase로는 수정하기 어려울 것 같으면 filter-branch라는 명령을 써볼 수도 있다.

ProGit에서 비유하기를 Rebase가 삽이라면 filter-branch는 포크레인이라고 할 수 있단다. 어떤 상황에서 쓸 수 있을 지를 살펴보자.

### 모든 커밋에서 파일을 제거하기

누군가 생각 없이 `git add .` 명령으로 왠 이상한 파일을 커밋했다. 아니면 누군가가 암호나 secret key가 담겨있는 파일을 커밋해서, 이런 파일을 다시 삭제해야 하는 상황이 있을 수 있다.

filter-branch는 히스토리 전체에서 필요한 것만 골라내는 데 사용하는 도구다. filter-branch의 --tree-filter라는 옵션을 사용하면 히스토리에서 특정 파일만 제거할 수 있다.

예시를 위해서 deploy.sh파일을 삭제 해보았다.

`git filter-branch --tree-filter 'rm -f deploy.sh' HEAD` 명령을 수행시켰다.

![git filter-branch](/assets/img/git_command2/filter-branch.png)

filter-branch 명령에 --all 옵션으로 모든 브랜치에 적용할 수 있다.

### 모든 커밋의 이메일 주소를 수정하기

모든 커밋 중에 자신의 이메일 주소만 변경하고 싶을 수 있다. 그 경우에 filter-branch 명령의 --commit-filter 옵션으로 해당 커밋만 골라서 이메일 주소를 수정할 수 있다.

이 예시는 ProGit에 있는 예시를 그대로 써놓는다.

```bash
#!/bin/bash
git filter-branch --commit-filter '
    if [ "$GIT_AUTHOR_EMAIL" = "oeeen3@gmail.com" ];
    then
        GIT_AUTHOR_NAME="Martin";
        GIT_AUTHOR_EMAIL="martin@gmail.com";
        git commit-tree "$@";
    else
        git commit-tree "$@";
    fi' HEAD
```

위 명령을 수행하면 이메일이 oeeen3@gmail.com인 커밋의 author name은 Martin으로, author email은 martin@gmail.com 으로 바뀌게 된다.

이 명령을 수행하면 이메일 주소가 바뀌는 커밋의 해시값만 바뀌는 것이 아니라, 부모가 바뀐 커밋의 해시값들도 모두 바뀐다.

마지막으로 ProGit에서도 계속해서 강조를 하기 때문에 나도 강조를 해보면, 당연하지만 **이미 푸시한 커밋에 대해서는 수정하면 안된다!**
