---
layout: single
title:  "Github CLI 사용기"
date:   2020-09-19 18:00:59 +0900
classes: wide
categories: etc
tags: github
toc: true
toc_sticky: true
---

9월 17일 날짜로 Github CLI 1.0 버전이 릴리즈 되었습니다. Github에 들어가보니 옆에 뭔가 거슬리는 것이 있길래 확인해보고 사용해본 후기를 정리하여 포스팅합니다.

## 무엇을 할 수 있나

릴리즈 페이지에 가보면, Github CLI 1.0 으로 다음과 같은 것들을 할 수 있다고 나와있다.

1. 전체 GitHub 워크플로우를 터미널에서 할 수 있다.
2. GitHub API를 호출해서 거의 모든 작업을 스크립트화 할 수 있고, 어떤 커맨드에도 커스텀 alias를 설정할 수 있다.
3. Github enterprise도 연결할 수 있다.

## 실제 사용법

소개 페이지에 나와있는 예시를 기반으로 실제로 사용해보자. 다음과 같은 순서로 해볼 생각이다.

1. Github에 public repository 생성
2. master 브랜치에 커밋 푸시
3. issue 생성, label: enhancement
4. develop 브랜치에 커밋 푸시
5. develop -> master로 머지하는 Pull Request 생성
6. PR 내용 확인 후 머지

MacOS 기준으로 설치하려면 `brew install gh` 명령을 통해 간단하게 설치할 수 있다. GitHub CLI를 설치 후 Github 계정에 인증을 받는 과정을 기본으로 거쳐야 다음 작업을 할 수 있다.

### Repository 생성

Github CLI로 repository를 생성해본다. 명령어를 어떻게 쳐야하는지 모르기 때문에 `gh repo --help`명령으로 가능한 조합을 확인해봤다.

![gh repo help](/assets/img/gh_cli/gh-repo-help.png)

`gh repo create [repo명]` 커맨드로 레포를 만들 수 있는 것을 알 수 있다.

![gh repo create](/assets/img/gh_cli/gh-repo-create.png)

이후에 master 브랜치에 커밋을 푸시하기 위해 README.md 파일을 생성하여 마스터 브랜치에 커밋을 하나 만들고 원격 레포에 푸시를 했다.

![gh initial commit](/assets/img/gh_cli/github-initial-commit.png)

### Issue 생성

이제 이슈를 생성해보자. `gh issue create --label [원하는라벨]` 명령으로 생성한다. 명령어를 치면 대화형으로 선택지가 나와서 간단하게 이슈를 생성할 수 있다.

![gh issue create](/assets/img/gh_cli/gh-issue-create.png)

![gh issue create web](/assets/img/gh_cli/gh-issue-create-web.png)

다음으로 PR을 생성하기 전에 develop branch에 새로운 커밋을 쌓는다. 아래 사진과 같은 커밋로그를 만들었다.

![commit log](/assets/img/gh_cli/commit-log.png)

### PR 생성

이제 develop branch에서 master branch로 merge 요청하는 PR을 생성한다. PR 내용에 issue close하는 커맨드까지 작성하여 PR merge시 위에서 생성한 Issue를 close 하도록 작성해보자

터미널에서 develop branch에 checkout 한 상태로 `gh pr create` 명령으로 develop에서 master로 PR을 생성한다. PR의 Body는 resolve: #1으로 작성했다. Reviewer, Assignee, Metadata들을 설정할 수도 있다.

Github CLI로 커맨드로 PR 관련하여 file diff(`gh pr diff [pr#]`)를 확인한다거나 pr 코드로 checkout(`gh pr checkout [pr#]`) 하여 코드를 확인한다거나 할 수 있다. 그러나 터미널로 파일 변경사항 비교하는 것은 개인적으로 어려워서 이 방법으로는 못할 것 같다. 그 대신 IntelliJ등을 사용하면서 PR로 checkout 후 추가된 테스트코드를 확인한다거나, 직접 코드를 보면서 리뷰를 할 수 있을 것 같다.

### PR 확인 후 Merge

`gh pr diff 2` 명령으로 파일 변경사항을 확인 할 수 있다.(아래 사진처럼 나와서 확인하기 어렵다)

![pr diff](/assets/img/gh_cli/gh-pr-diff.png)

변경사항을 모두 확인했고, PR을 머지해도 될 상태라면 `gh pr merge [pr#]` 명령을 통해 merge를 한다. Github에서 PR Merge 했을 때의 옵션들이 나온다.(Merge commit 생성, Rebase merge, Squash merge)

![pr merge](/assets/img/gh_cli/gh-pr-merge.png)

여기서 squash merge를 선택하여 merge 한 결과는 아래와 같다.

![pr merge web](/assets/img/gh_cli/gh-pr-merge-web.png)

PR도 머지가 됬고 해당 PR이 머지 될 때 자동으로 연결된 Issue가 close 되도록 작성해두었기 때문에 자동으로 Issue까지 Close 되었다.(PR 머지 후 branch 삭제까지도 선택할 수 있다.)

## 사용후기

Github CLI가 정식으로 릴리즈 되었다길래 일단 한번 사용해봤다. 익숙해지면 Web에서 굳이 확인하지 않고도 Github의 모든 기능을 사용할 수 있게 되었기 때문에 사용해봐도 괜찮을 것 같다. 이 포스팅에서 사용한 것들 외에도 굉장히 많은 기능들이 있지만, 내가 실제로도 Github에서 사용하는 기능들이 위에 나와있는 내용 정도라서 이정도면 충분한 것 같다.

## 참고자료

- [GitHub CLI 1.0 is now available](https://github.blog/2020-09-17-github-cli-1-0-is-now-available/)
