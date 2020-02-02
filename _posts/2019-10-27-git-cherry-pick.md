---
layout: single
title:  "Git cherry-pick"
date:   2019-10-27 15:00:00 +0900
classes: wide
categories: git
tags: git
toc: true
toc_sticky: true
---

누군가의 요청을 받아 cherry-pick에 대해 정리한 문서입니다. 사용법을 이해하고 싶으시면 아래 **예시 부분**을 먼저 보세요.

## cherry-pick

체리픽이 뭘까? 말만 들고 생각해보면 뭔가 체리 따듯 커밋을 하나 따와서 뭔가 작업을 할 수 있는 건가? 이렇게 생각할 수 있다.

그래서 Git Manual을 살펴봤다. `git cherry-pick --help`를 쳐보면 설명이 자세하게 나와있다.

일단 사용은 아래와 같이 한다고 한다.

```bash
#!/bin/bash
$git cherry-pick [--edit] [-n] [-m parent-number] [-s] [-x] [--ff] [-S[<keyid>]] <commit>...
$git cherry-pick --continue
$git cherry-pick --quit
$git cherry-pick --abort
```

이 중 제일 간단한 사용만! 알아보기로 하자

설명은 `Given one or more existing commits, apply the change each one introduces, recording a new commit for each. This requires your working tree to be clean (no modifications from the HEAD commit)` 이라고 한다.

그러니까 뒤에 나오는 커밋들(하나 or 그 이상)에 변화를 만들고 새로운 커밋으로 기록한다. (working tree는 clean 해야 한다.(HEAD commit과 차이가 없어야 한다.))

## Option

- -e, --edit
  - commit 하기 전에 커밋 메시지를 수정할 수 있다.
- -x
  - 커밋을 기록할 때, 원래 커밋의 커밋 메세지를 기록한다. ("cherry picked from commit XXX" 같은..)
  - conflict 없는 커밋에만 적용된다.
- -r
  - 위의 -x가 default라면, -r은 -x를 disable 하는 것이다.
- -m parent-number, --mainline parent-number
  - merge commit에는 종종 체리픽을 못하는데, 왜냐하면 merge 커밋의 메인라인의 부모가 무엇인지 모르기 때문이다.
  - 이 옵션은 merge commit의 부모를 특정 지어 준다.(1번부터 시작)
- -n, --no-commit
  - 이 옵션은 체리픽으로 변경된 내용을 커밋을 하지 않는다.
- -s, --signoff
  - 커밋 메세지 끝에 signed-off-by line을 추가한다.
- -S, --gpg-sign
  - 인증된 커밋을 날리는 것 같다. (인증 부분은 아직 보지 않아서 잘 모른다.)
- --ff
  - 현재 HEAD가 cherry-pick으로 골라진 커밋의 부모와 동일하다면 fast-forward 한다.
- --allow-empty
- --allow-empty-message
- --keep-redundant-commits
- --strategy=\<strategy\>
- -X\<option\>, --strategy-option=\<option\>
- --continue
  - .git/sequencer에 있는 정보를 써서 과정을 계속 진행한다. 보통 conflict 이후에 conflict 해결 후 continue한다.
- --quit
  - Forget about the current operation in progress. Can be used to clear the sequencer state after a failed cherry-pick or revert.
- --abort
  - cherry-pick을 취소한다.

## 예시

![before cherry pick](/assets/img/git_cherrypick/before_cherrypick.png)

위와 같은 커밋 내역이 있다고 해보자. 이 상태에서 master 브랜치에 checkout 된 상태에서 `git cherry-pick oeeen` 명령을 수행하면, 아래와 같이 변하게 된다.

![after cherry pick](/assets/img/git_cherrypick/after_cherrypick.png)

보면 oeeen branch의 cherry pick test 라는 커밋이 master 브랜치의 master cherry pick 커밋 위로 옮겨진 것을 알 수 있다. (하지만 기존 oeeen 브랜치의 커밋의 해시값과는 다르다. 애초에 다른 커밋)

![before cherry pick](/assets/img/git_cherrypick/cherrypick_total.png)

조금 더 어려운 예시를 위해 아래와 같은 상태를 만들었다.

![before cherry pick](/assets/img/git_cherrypick/before_cherrypick2.png)

![before cherry pick](/assets/img/git_cherrypick/before_cherrypick2_terminal.png)

이 상태에서 topic master에는 없지만 topic 브랜치에는 있는 E, F, G 커밋을 master 브랜치로 이동 시키고 싶다면 `git cherry-pick ..topic`을 해보면 된다. 저 명령어와 동일한 명령어는 `git cherry-pick master..topic` 또는 `git cherry-pick ^master topic`이다. 이 부분에 대한 자세한 설명은 [progit 정리 - 조회하기](https://smjeon.dev/git/git-refs/) 편에서 double dot 부분을 살펴보자.

![after cherry pick](/assets/img/git_cherrypick/after_cherrypick2.png)

![after cherry pick](/assets/img/git_cherrypick/after_cherrypick2_terminal.png)

커밋 로그는 체리픽 이후에 위와 같이 변화하게 된다.

정리하자면 체리픽 커맨드 이후에 원하는 커밋의 리스트를 둘 수 있는데 double dot, triple dot 등의 참조 방식으로 여러개의 커밋을 지정할 수도 있고 여러 커밋을 그냥 순차적으로 나열 하는 방법으로도 체리픽 할 수 있다.

위 명령을 수행시키기 전에 이전 cherry-pick의 quit을 사용하지 않아서 아래와 같은 문제가 발생 했었다.

![after cherry pick](/assets/img/git_cherrypick/cherrypick_fail.png)

cherry-pick을 실행하면 진행했던 오퍼레이션이 .git/sequencer에 저장되는 것 같은데, 이 sequencer에 있는 operation에 대한 것을 초기화 시켜주지 않아서 다음 새로운 규칙의 cherry-pick이 먹지 않았다. 그래서 `git cherry-pick --quit` 명령을 실행 후 다음 cherry-pick을 진행 했다. (정확하지 않은 추측성 내용입니다.)

- 참고자료: `git cherry-pick --help` 또는 `man git-cherry-pick`
