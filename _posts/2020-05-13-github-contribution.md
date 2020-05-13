---
layout: single
title:  "Git 커밋 조작과 Github의 Contribution"
date:   2020-05-13 16:30:59 +0900
classes: wide
categories: git
tags: git
toc: true
toc_sticky: true
---

1일 1커밋을 하다보면 Github의 my profile의 잔디에 집착하게 됩니다. 잔디를 막 심다보니.. 이전에 실수로, 또는 실제로 쉬는 기간에 비어있는 contribution이 아쉬워집니다.

실제 제 프로필을 보면 아래와 같이 비어있는 부분이 있습니다.

![contribution hole](/assets/img/github_contribution/contribution_hole.png)

이렇게 비어있는 잔디를 심을 방법은 없을까? 라는 생각으로 시작했습니다. 생각으로는 커밋 날짜만 바꿔서 커밋하고, 원격 레포에 커밋을 해야하니 해당 날짜(예를 들어 위 사진의 4/3) 이후의 커밋이 존재하면 안될 것 같기도 합니다.

그러면 3가지 조건에 대해 실험을 해봅시다. *2020년 4월 3일은 임의의 날짜입니다.* ~~(사실 잔디가 비어있다)~~

1. 원격 레포(`https://github.com/oeeen/git-practice`) 커밋 기록은 2020년 4월 3일 이전의 커밋만 있음 -> 현재 날짜(2020년 5월 13일)에 4월 3일 날짜로 커밋을 생성 -> 해당 원격레포에 푸시
2. 원격 레포 커밋 기록은 2020년 4월 3일 이후에 커밋 기록이 있음 -> 현재 날짜(2020년 5월 13일)에 4월 3일 날짜로 커밋을 생성 -> 원격 레포에 푸시하면 푸시가 될까?(된다면 커밋 사이에 끼워져서 들어갈까?)
3. 원격 레포를 2020년 4월 3일 이후에 생성 -> 현재 날짜(2020년 5월 13일)에 4월 3일 날짜로 커밋을 생성 -> 원격 레포에 푸시하면 푸시가 될까?

일단 해보기 전에 예상을 해보면 1번 가능 2, 3번은 불가능할 것 같습니다.

본격적으로 실험을 해봅시다.

## 1번 조건

1번 조건은 다음과 같습니다.

1. 원격 레포(`https://github.com/oeeen/git-practice`) 커밋 기록은 2020년 4월 3일 이전의 커밋만 있음
2. 현재 날짜(2020년 5월 13일)에 4월 3일 날짜로 커밋을 생성
3. 해당 원격레포에 푸시

그림으로 보면 다음과 같습니다.

![condition - 1](/assets/img/github_contribution/condition1.png)

그리고 제가 원하는 것은 다음과 같은 그림입니다.

![condition - 1.2](/assets/img/github_contribution/condition1-2.png)

```sh
echo "date change" > change.txt
git add change.txt
git commit -m "chore: change date test" --date="Fri 3 Apr 2020 22:21:20 KST"
```

위와 같이 수행하면 다음과 같은 결과가 나옵니다.

![change date result](/assets/img/github_contribution/change_date_result.png)

실제로 날짜가 변경된 상태로 커밋이 들어가는 것을 알 수 있습니다.

그럼 이걸 원격 레포에 푸시하면 어떻게 될까요? (`git push origin master`)

아래와 같이 커밋이 들어가는 것을 확인할 수 있습니다. 물론 profile에 잔디도 찍히네요?!

![repository](/assets/img/github_contribution/change_date_result_repository.png)

![repository](/assets/img/github_contribution/change_date_result_contribution.png)

예상은 했지만 가능했습니다.

## 2번 조건

2번 조건은 다음과 같습니다.

1. 원격 레포 커밋 기록은 2020년 3월 15일 이후(2020년 4월 3일)에 커밋 기록이 있음
2. 현재 날짜(2020년 5월 13일)에 3월 15일 날짜로 커밋을 생성
3. 원격 레포에 푸시하면 푸시가 될까?(된다면 커밋 사이에 끼워져서 들어갈까?)

실험을 위해서 1번 조건에서 사용한 원격 레포를 사용하기 위해 날짜는 변경할 예정입니다. 확실한 차이를 위해 2020년 3월 15일 커밋을 심을 예정입니다.

그림으로 보면 다음과 같습니다.

![interpolation](/assets/img/github_contribution/condition2.png)

실험을 위해 다음과 같은 커맨드로 조작합니다.

```sh
echo "Sun 15 Mar 2020 commit" > march_15.txt
git add march_15.txt
git commit -m "chore: 끼워넣기 커밋" --date="Sun 15 Mar 2020 15:16:17 KST"
git log
```

위 커맨드를 수행한 후 git log를 살펴보면 이상한 로그를 볼 수 있습니다.

![abnormal git log](/assets/img/github_contribution/abnormal_git_log.png)

위 그림처럼 4월 3일 커밋은 2번째에 있는 가장 최근 커밋은 3월 15일 커밋이 돼버립니다. 시간 순이 아니다..

일단 알겠고 원격 레포에 푸시를 하면 어떻게 될까요? 푸시 전의 제 프로필의 3월 15일 contribution은 아래와 같습니다.(1 contribution)

![before commit profile](/assets/img/github_contribution/condition2_before_profile.png)

원격 레포에 푸시 후 레포의 커밋 내역과 제 프로필은 다음과 같습니다.

![after commit repository](/assets/img/github_contribution/condition2_result_repository.png)

![after commit profile](/assets/img/github_contribution/condition2_after_profile.png)

잔디도 심기고(contribution 상승) 커밋 내역에도 잘 찍히는 것을 알 수 있습니다.

안될 것이라고 생각했는데, 이상한 로그를 만들어버리면서 날짜 변경 커밋이 들어갔습니다. 3번 조건은 2번 조건과 비슷하기 때문에 해볼 필요도 없을 것 같지만, 그래도 해보겠습니다.

## 3번 조건

3번 조건은 다음과 같습니다.

1. 원격 레포를 2020년 5월 13일에 생성
2. 현재 날짜(2020년 5월 13일)에 지금보다 과거 날짜(2020년 4월 2일)로 커밋을 생성
3. 원격 레포에 푸시하면 푸시가 될까?

처음 생각은 당연히 원격 레포 생성 시점보다 이전 기록의 커밋이니 문제가 되지 않을까 생각했지만.. 앞의 실험을 해보고 나니, 가능할 것 같습니다.

일단 github에 원격 레포를 하나 만듭니다. (어차피 다시 지울 것이니 test-repository라고 하나 만들었습니다.)

날짜 변경 커밋을 푸시하기전 4월 2일의 contribution은 아래와 같습니다.(No contribution)

![April 4 contribution - Before](/assets/img/github_contribution/april4_contribution_before.png)

```sh
git clone https://github.com/oeeen/test-repository.git
cd test-repository
echo "date change test" > date_change.txt
git add date_change.txt
git commit -m "chore: 날짜 변경 커밋" --date="Thu 2 Apr 2020 12:13:14 KST"
git log
```

![Condition3 git log](/assets/img/github_contribution/condition3_result_log.png)

2번 조건에서의 이상한 로그와 비슷하게 나타납니다. 이를 원격 레포에 푸시해봅니다.(`git push origin master`)

![Condition3 repository result](/assets/img/github_contribution/condition3_result_repository.png)

![Condition3 profile result](/assets/img/github_contribution/condition3_result_profile.png)

contribution이 1로 늘어난 것을 볼 수 있고, repository의 커밋 로그도 2번 조건과 비슷하게 이상하게 나타남을 볼 수 있었습니다.

실제로 잔디 심는 것에 이를 사용할 생각은 없지만, 가능하다는 것을 알게 되었습니다!
