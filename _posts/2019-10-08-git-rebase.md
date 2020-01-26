---
layout: post
title:  "Git Rebase"
date:   2019-10-08 23:30:00 +0900
classes: wide
categories: git
tags: git
toc: true
toc_sticky: true
---

우아한테크코스의 woowacourse 브랜치에 Pull Request 날린 이후에 Step2 미션을 진행하기 전 커밋 로그를 한 줄로 예쁘게 rebase를 하고 싶을 때 방법을 정리해둔 글입니다.

***바쁘신 분은 맨 밑에 세 줄 정리만 보시면 됩니다**

## Rebase

일단 git rebase 명령의 기본을 알아봅시다. `git rebase --help` 명령으로 쉽게 그림으로 알아볼 수 있습니다. 아래 내용은 help 명령에 나와있는 그대로를 그림으로 옮긴 내용입니다.

이중에 --onto 옵션을 살펴보면 `git rebase [--onto <newbase>] [<upstream> [<branch>]]` 라고 쓸 수 있습니다.

이 명령에서 branch가 특정 지어진다면, rebase 명령은 다른 명령을 실행하기 전에 `git checkout <branch>` 명령을 먼저 수행합니다.

그리고 upstream에 없지만 branch에는 있는 커밋들(변경사항들)은 임시 공간에 저장됩니다.

--onto 옵션이 있으면, 현재 체크아웃 되어있는 브랜치는 `git reset --hard <newbase>`와 동일한 효과를 냅니다. 그 다음으로 임시공간에 저장했던 커밋들을 다시 적용합니다.(하나씩, 순서대로)

*Note that any commits in HEAD which introduce the same textual changes as a commit in `HEAD..<upstream>` are omitted (i.e., a patch already accepted upstream with a different commit message or timestamp will be skipped).

![before-rebase-master](/assets/img/rebase/rebase-master.png)

위와 같은 상황에서 `git rebase master topic` 이나 `git rebase master` 명령을 수행하면 아래 그림처럼 바뀌게 됩니다.

![after-rebase-master](/assets/img/rebase/after-rebase-master.png)

또다른 예시로 master branch에 topic branch에서 커밋했던 내용이 들어있다면, 그 커밋은 rebase시에 생략 됩니다.

![before-rebase-master2](/assets/img/rebase/rebase-master-ex2.png)

위와 같은 상황에서 `git rebase master` 또는 `git rebase master topic` 명령을 수행하면 아래 그림처럼 바뀌게 됩니다.

![after-rebase-master2](/assets/img/rebase/after-rebase-master-ex2.png)

다음으로 `git rebase --onto` 옵션에 대해서는 실제 사례에서 적용해보면서 알아보면 좋을 것 같습니다.

## 실제 사례에 적용

일단 현재 상황을 가정해봅시다.

`git remote -v` 명령을 쳐보면 아래와 같은 상태입니다.

![remote -v](/assets/img/rebase/remote.png)

 현재 woowacourse의 jwp-mvc의 master 브랜치 및 내 브랜치의 상태는 아래와 같습니다.

![remote-initial](/assets/img/rebase/remote-initial.png)

현재 상황은 woowacourse의 jwp-mvc의 1단계(step1 branch)를 구현하고, Pull Request를 날리고 코드리뷰를 기다리고 있는 상태입니다. (아래 그림)

![remote-step1](/assets/img/rebase/remote-my-step1.png)

근데 마냥 기다리고 있을 수는 없어서 jwp-mvc의 2단계를 새로운 브랜치(step2 branch)에서 차근차근 구현하며 커밋을 쌓고 있는 상태입니다.(로컬에서, 아래 그림처럼)

![remote-step2](/assets/img/rebase/local-step2.png)

이 상황에서 1단계 Pull Request 날렸던 것이 머지가 되었습니다.(보통 리뷰어님께서 스쿼시 머지 해주십니다.) 그러면 아래와 같은 상황이 됩니다.

![remote-step2](/assets/img/rebase/after-merge.png)

그러면 이제 upstream(woowacourse)의 oeeen브랜치와 local의 step2 브랜치를 비교해보면 아래와 같습니다.

![remote-step2](/assets/img/rebase/after-merge-step2.png)

위와 같은 상황을 실제로 로컬에서 보면 아래와 같습니다.

![local-log](/assets/img/rebase/local-log.png)

그리고 remote repository의 상태를 보면 다음과 같습니다.

![local-log](/assets/img/rebase/merge-remote.png)

이 상황에서 여기서 이제 저희가 원하는 것은 B1(PR Test#3 commit) 커밋 다음부터 Step2 구현한 커밋들을 이어 붙이고 싶습니다.

커밋 순서대로 보면 A1 - A2 - A3 - A4 - B1 - A8 - A9 - A10 상태를 만들려고 합니다.

일단 로컬의 step2 브랜치로 checkout 합니다. 그리고 upstream(woowacourse의 repo)에서 자신의 브랜치(oeeen)을 fetch 해옵니다. fetch_head의 log는 다음과 같이 나옵니다.

![fetch_head-log](/assets/img/rebase/fetch_head-log.png)

다음으로 fetch_head 위에 A8, A9, A10 커밋을 이동 시키고 싶기 때문에, rebase를 합니다. 아래와 같은 커맨드를 순서대로 실행시켜보면 원하는대로 실행이 됩니다.

```bash
#!/bin/bash
$ git checkout step2
$ git fetch upstream oeeen
$ git rebase --onto FETCH_HEAD step1
```

`git rebase --onto FETCH_HEAD step1` 명령을 하나하나 뜯어보면, 현재 브랜치는 step2 브랜치이고, FETCH_HEAD를 new base로 하고 step1 브랜치를 upstream으로 rebase 하는 것입니다.

풀어서 설명하면 step1 branch(A1부터 A7까지)에는 없지만, step2 branch(A1부터 A10까지)에만 포함되어 있는 커밋들(A8, A9, A10)을 FETCH_HEAD 위로 rebase 하는 것입니다. 그래서 이 결과로 아래와 같은 결과가 나타나게 됩니다.

![after-rebase](/assets/img/rebase/after-rebase.png)

## 세 줄 정리

### upstream은 woowacourse의 repository

1. `git checkout [PR이후 커밋을 쌓은 브랜치]`
2. `git fetch upstream [본인의 브랜치명]`
3. `git rebase --onto FETCH_HEAD [옮기기를 원하는 커밋들 중 시작의 부모] [옮기기를 원하는 마지막 커밋]`
