---
layout: single
title:  "ProGit 정리 - 대화형 명령 / Stashing & Cleaning"
date:   2019-09-07 17:00:00 +0900
classes: wide
categories: git
tags: git
---

# 대화형 명령

`git add` 명령에 -i나 --interactive 옵션을 주고 실행하면 Git은 아래와 같은 대화형 모드로 들어간다.

![git add -i](/assets/img/git_interactive/add.png)

현재 Staging Area의 상태를 보여주고 어떤 일을 할 수 있는지 보여준다.
위처럼 

staged | unstaged
--- | ---
Staged 상태인 파일들 | Unstaged 상태인 파일들..

이런 식으로 보여준다.

Command 부분에서는 할 수 있는 일들을 보여준다. Stage 하기, Unstage하기, Untracked 상태로 바꾸기, Stage한 파일 Diff, 수정한 파일의 일부만 Staging Area에 추가하기 등..

## Update

Command에서 2나 u를 입력해서 update를 해본다. 그러면 Staging Area에 추가할 수 있는 파일을 보여준다. 그리고 내가 원하는 파일을 선택해본다.

![update](/assets/img/git_interactive/update.png)

그리고 아무것도 입력하지 않고 엔터치면 선택한 파일을 Staging Area로 추가한다.

![after update](/assets/img/git_interactive/after_update.png)


## Revert

이 상태에서 다시 1,2 번 파일을 Unstage 하고 싶으면 3이나 r을 입력한다.

![after update](/assets/img/git_interactive/revert.png)

Staged 파일의 변경 내용을 보려면 6이나 d를 입력하고 파일을 선택한다.

## Diff

![diff](/assets/img/git_interactive/diff.png)

![diff-contents](/assets/img/git_interactive/diff_contents.png)

이 명령의 결과는 `git diff --cached`의 결과와 같다.

## Patch - 파일의 일부분만 Staging Area에 추가

5나 p를 입력하고 파일을 선택한다.

![patch](/assets/img/git_interactive/patch.png)

![patch](/assets/img/git_interactive/patch_man.png)

y나 n으로 각 부분을 stage 할 지 결정할 수 있다. 어쨌든 일부분을 stage 하고 status를 살펴보면 아래와 같다..

![status](/assets/img/git_interactive/status.png)

이 상태에서 대화형 add를 종료하고 commit하면 파일의 일부분만 커밋할 수 있다.

사실 이렇게 일부분만 stage하는 것은 `git add -p`나 `git add --patch`로도 할 수 있다.

`reset --patch`로 파일 일부만 Stage Area에서 내릴 수도 있고, `checkout --patch`로 파일 일부를 다시 Checkout 할 수 있다. `stash save --patch`로 파일 일부만 Stash 할 수도 있다.

# Stashing과 Cleaning

## Stash

Stash는 상당히 자주 사용한다. 

만약에 어떤 프로젝트에서 한 부분을 담당하고 있다고 하자. 그리고 이제 작업하는 일(A, dev-mybranch)이 있었고, 다른 이슈가 터져서 그 이슈를 수정하기 위해 브랜치(issue branch)를 변경해야 한다고 치자. 그런데 아직 작업하던 일(A)는 완료되지 않았기 때문에 커밋하기가 좀 그렇다.. 그렇지만 파일을 날려버리기에는 너무 아깝다!

이런 경우에 `git stash` 명령이 유용할 것이다.

Stash 명령을 사용하면 워킹 디렉토리에서 수정한 파일들만 저장한다.
Stash는 Modified이면서 Tracked 상태인 파일과 Staging Area에 있는 파일들을 보관해두는 장소이다. 마무리 되지 않은 수정사항들(위의 예에서 A)을 스택에 잠깐 저장해두고 나중에 다시 적용할 수 있다.

팀 프로젝트 할 때 사용했던 경험이 있는데, 

나는 dev branch에서 새로운 기능을 개발하기 위해 dev-feature1과 같은 브랜치를 따서 작업 중이었다. 

![stash](/assets/img/git_interactive/ex_stash.png)

그런데 다른 팀원이 다른 기능개발을 마치고 dev 브랜치에 pull request를 날리고 이를 코드 리뷰 후에 merge를 했다. 그러면 아래와 같은 상태가 된다.

![after-merge](/assets/img/git_interactive/ex_stash2.png)

그러면 어떻게 해야 할까? 여러 해결 방법이 있지만, 일단 두 가지 경우로 나누어 본다.

1. 만약 내가 dev-feature1 브랜치에서 한 번도 커밋을 하지 않았다면, `git stash` 명령으로 로컬의 변경사항을 저장해두고, `git pull origin dev` 명령으로 다시 dev branch의 내용을 최신화 하고 `git stash pop`으로 변경 사항 수정을 다시 시작하면 된다.

![after-stash-pull](/assets/img/git_interactive/ex_stash3.png)

2.  dev-feature1 브랜치에서 커밋을 엄청나게 쌓아두었다.. 근데 이 커밋로그들이 필요하진 않다. 그러면 커밋로그들을 모두 날려버리고 stash한 후에 다시 pull 한다.


![reset-stash](/assets/img/git_interactive/stash-commit.png)

위와 같은 상황에서 2번 방법으로 이 문제를 해결하려면 다음과 같은 커맨드로 하면 된다.

```bash
$ git reset --soft A3
$ git stash
$ git pull origin dev
$ git stash pop
```

![after-stash-pull](/assets/img/git_interactive/ex_stash3.png)

그렇게 되면 위의 그림에서 현재 내 로컬에는 위 그림에서 B1, B2의 변경 사항까지 모두 가지고 있으면서 remote branch와 동기화까지 완료된 상태이다. 이 방법의 단점은 지금까지 내가 쌓아둔 커밋로그가 사라진다는 것이다. 하지만 불필요한 머지 커밋을 남기고 싶지 않을 때 사용한 방법이다.


## Stash 옵션들

`git stash apply --index` 명령으로 stash하기 전에 staged 상태였던 파일까지 그대로 staged 상태로 만들어준다. 

`git stash apply` 명령은 stash를 적용하기만 한다. 스택에는 여전히 남아있다. 그렇기 때문에 `git stash drop` 명령으로 해당 stash를 제거 한다.

위 두 명령을 합친 것이 `git stash pop`이다.

`git stash --keep-index`라는 명령도 있다. 이는 이미 Staging Area에 있는 파일은 Stash하지 않는다. 

`git stash --include-untracked` 또는 `git stash -u` 명령으로 추적중이지 않은 파일을 같이 stash 할 수 있다.

`git stash --patch` 명령으로 변경된 데이터 중 골라서 stash 할 수 있게 된다.


**Stash를 적용한 브랜치 만들기**
Stash를 꺼낼 때.. 충돌이 일어날 수 있다. `git stash branch` 명령으로 stash할 당시의 커밋을 checkout 한 후 새로운 브랜치를 만들고 여기에 적용한다. 이 명령은 브랜치를 새로 만들고 stash를 복원해준다.

## Clean

작업하던 내용을 Stash 하지 않고 지워버리고 싶다면, `git clean` 명령을 쓰면 된다.

이 명령을 수행하면 워킹 디렉토리 안의 추적하고 있지 않은 모든 파일이 지워지기 때문에 신중하게 사용해야 한다.

이 명령이 실행되면 어떤 일이 일어날지 알고 싶다면 -n 옵션을 달아준다.

`git clean -d -n` 명령을 실행한다.

기본적으로 `git clean` 명령은 추적 중이지 않은 파일만 지운다. .gitignore에 명시해서 무시되는 파일은 지우지 않는다. 무시된 파일도 지우려면 -x 옵션이 필요하다.

clean 명령도 동일하게 -i 옵션으로 대화형으로 실행 시킬 수 있다.