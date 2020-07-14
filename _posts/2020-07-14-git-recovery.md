---
layout: single
title:  "ProGit 정리 - 데이터 복구"
date:   2020-07-14 22:00:00 +0900
classes: wide
categories: git
tags: git
toc: true
toc_sticky: true
---

바쁘신 분은 맨 밑의 요약 해둔 과정만 읽고 따라하시면 됩니다!

실제로 Git을 사용하다보면 커밋을 잃어버리는 일이 종종 발생한다. 작업 중이던 브랜치를 잠깐 develop branch에 갔다가 지워버린다거나, Hard Reset을 해버렸다거나 하면 종종 발생한다. 이럴 경우 어떻게 커밋을 다시 찾을 수 있을까?

## 잃어버린 커밋 복구하기(reflog)

master 브랜치에서 hard reset 한 경우 잃어버린 커밋을 복구해보자.

![git log before reset](/assets/img/git_recovery/git-log-before-reset.png)

일단 위와 같은 커밋로그를 만들어두었다. 이제 여기서 세번째 커밋으로 hard reset 해보자.(`git reset --hard fa0e799ce14bfb4d3719af3749fd665057cc35eb`)

![git log after reset](/assets/img/git_recovery/git-log-after-reset.png)

이제 위에 두 개의 커밋은 해시값을 기억하고 있지 않는 이상 잃어버렸다고 할 수 있다. 보통은 `git reflog` 명령을 사용하는게 좋다. HEAD가 가리키는 커밋이 바뀔 때마다 Git은 그 커밋이 무엇인지 기록한다. 일단 그럼 `git reflog`를 실행해보자.

![git reflog](/assets/img/git_recovery/git-reflog.png)

이걸 좀 더 자세히 보려면 `git log -g` 명령을 사용하면 된다.

![git log -g](/assets/img/git_recovery/git-log-g.png)

여기서 보면 두 번째 커밋으로 복구했으면 좋겠다는 것을 알 수 있으므로, 그 커밋을 가리키는 브랜치를 만들어서 복구하면 된다.(`git branch recover f1bb5bee31cfba88638358572ce24acb2ad2cd9d`)

## 좀 더 심각한 상황

이제 recover 브랜치에 가보면 총 5개의 커밋을 모두 확인할 수 있게 되었다. 하지만 잃어버린 커밋을 reflog에서도 찾을 수 없는 경우가 있을 수 있다. 이 상황도 재연해보자.

일단 recover 브랜치를 지우고, reflog에서 지우기 위해서 .git/logs/ 디렉토리를 지운다.(reflog는 .git/logs/ 에 존재한다.)

```sh
git branch -D recover # master 브랜치나, 다른 브랜치에서
rm -rf .git/logs/
```

이 명령 이후 `git reflog` 명령을 수행해보면 reflog가 텅 비어있음을 볼 수 있다.

이제 어떻게 복구할 수 있을까? `git fsck` 명령으로 데이터베이스의 Integrity를 검사할 수 있다. 일단 해보자(`git fsck --full`)

![git fsck](/assets/img/git_recovery/git-fsck.png)

여기서 보이는 dangling commit이 바로 잃어버린 커밋이므로 아까와 동일하게 이 커밋을 가리키는 브랜치를 만들어서 복구하면 된다.

## 네 줄 요약

1. 일단 `git reflog`를 확인하고 여기서 내가 복구하고 싶은 커밋을 찾아본다.
2. reflog에 없을 경우, `git fsck --full`을 통해 dangling commit 을 찾아본다.
3. 찾아낸 commit의 hash값을 이용하여 `git branch 브랜치명 해시값` 명령을 통해 잃어버린 커밋으로 복구한다.
4. `git checkout 브랜치명`으로 해당 브랜치로 작업을 계속한다.

## 참고자료

- ProGit - 10. Git 개체
