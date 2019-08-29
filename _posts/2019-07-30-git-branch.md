---
layout: single
title:  "ProGit 정리 - Git Branch"
date:   2019-07-30 23:00:00 +0900
classes: wide
categories: git
---
## Branch란?
Git은 데이터를 일련의 스냅샷으로 기록한다.

### 예시
파일이 3개 있는 디렉토리가 하나 있고, 이 파일을 Staging Area에 저장하고 커밋 하는 과정.

`git commit`으로 커밋하면 루트 디렉토리와 각 하위 디렉토리의 트리 개체를 체크섬(파일 이름인듯)과 함께 저장소에 저장한다. 그다음에 커밋 개체를 만들고 메타 데이터(author, commit message, snapshot pointer)와 루트 디렉토리 트리 개체를 가리키는 포인터 정보를 커밋 개체에 넣어서 저장한다.

끝나면 각 파일 별로 Blob 하나 씩(총 3개), 트리 개체 하나, 커밋 개체 하나 - 총 5개의 파일이 생긴다.

Git의 브랜치는 커밋 사이를 이동할 수 있는 포인터 라고 생각하면 된다.

### 연습
`git branch test` 로 새로운 브랜치를 만들면 현재 작업하고 있던 마지막 커밋을 가리키는 새로운 브랜치가 생성된다. (현재 작업 중인 브랜치는 HEAD 기준)

`git checkout [브랜치명]`으로 해당 브랜치로 이동할 수 있다.

그 브랜치로 checkout 하고 commit을 하면 HEAD가 가리키고 있는 브랜치에만 commit이 쌓인다. 

**예시**
```bash
git branch test
git checkout test
echo "test branch" > testFile.txt
git add testFile.txt
git commit -m "test branch commit"
git checkout master
echo "master branch" > masterFile.txt
git add masterFile.txt
git commit -m "master branch commit"
```

위와 같은 과정을 진행한다고 해보자.

**초기 모습**

master branch에 Commit 3개가 쌓여있다.

![Initial Condition](/assets/img/gitbranch/0.png)

**Test branch로 Checkout**

![Test Checkout](/assets/img/gitbranch/1.png)

**Test branch에서 Commit**

test branch에서 testFile.txt 파일을 추가하고 commit을 진행 한다.

아래 그림처럼 A4라는 Commit이 생긴다.

![Test Commit](/assets/img/gitbranch/2.png)

**Master branch로 Checkout**

HEAD만 master branch쪽으로 이동했다.

![Master Checkout](/assets/img/gitbranch/3.png)

**Master branch에서 Commit**

test branch와 상관 없이 새로운 Commit A5가 생성된다.

![Master Commit](/assets/img/gitbranch/4.png)


이렇게 브랜치를 나누어서 커밋을 쌓을 수 있기 때문에..

## Merge
위와 같은 상황에서 master branch에 test branch를 merge 해보자.

```bash
git checkout master
git merge test
```

그러면 현재 브랜치가 가리키는 커밋이 merge할 브랜치(master)의 조상이 아니기 때문에 Fast-forward로 merge 하지 않는다.

대신에 3-way merge를 한다.

이 경우에는 A4와 A5의 공통 조상인 A3를 찾는다. 그리고 그 공통 조상에서부터 Merge를 진행한다.

3-way merge의 결과를 별도의 커밋(merge commit)으로 만들고 그 브랜치(master)가 그 커밋을 가리키도록 이동 시킨다.

![Merge](/assets/img/gitbranch/5.png)

merge가 끝난 이후에는 test branch가 더이상 필요 없으니 `git branch -d test`와 같은 명령으로 지우면 된다.

## Conflict
위와 같은 merge 작업을 진행하다가 실패 할 때가 있다. 두 브랜치(test, master)가 서로 같은 파일의 같은 부분을 수정했다면 merge가 실패한다.

Conflict가 발생하면 이 Conflict를 해결 해주면 된다.

이 conflict 발생 후 `git status` 명령으로 어떤 파일이 문제였는지 파악하고, 그 파일을 vim이나 각종 diff툴을 이용하여 해결하면 된다.

## Remote Branch
리모트 Refs는 리모트 저장소에 있는 포인터인 레퍼런스다. 리모트 저장소에 있는 브랜치, 태그 등..

`git ls-remote (remote)`로 모든 리모트 Refs를 조회할 수 있다.

리모트 Refs가 있지만 보통은 리모트 트래킹 브랜치를 사용한다.

리모트 트래킹 브랜치는 리모트 브랜치를 추적하는 브랜치다.

이 브랜치는 로컬에 있지만 움직일 수는 없다.

리모트 트래킹 브랜치는 북마크라고 생각하면 된다. 마지막으로 서버에 연결한 순간에 브랜치가 어떤 커밋을 가리키고 있었는지 나타낸다.

리모트 브랜치의 이름은 (remote)/(branch) 의 형식으로 되어있다.

예시로 origin/master 이런 브랜치가 있을 것이다. 이걸 예시로 다른 팀원과의 작업을 살펴본다.

초기 상태는 아래와 같다.

![remote-initial](/assets/img/gitbranch/7.jpg)

![local-initial](/assets/img/gitbranch/8.jpg)

이 상황에서 각 local과 remote의 commit이 추가 된다고 생각해보자.

그러면 아래와 같이 상황이 바뀐다.

![remote-committed](/assets/img/gitbranch/9.jpg)


![local-committed](/assets/img/gitbranch/10.jpg)

이 상황에서 `git fetch origin` 명령어로 리모트 서버의 상황을 로컬에 동기화 하면 아래 상황처럼 바뀐다. 



![remote-committed](/assets/img/gitbranch/9.jpg)

![local-committed](/assets/img/gitbranch/11.jpg)

여기에 다른 원격 서버를 연결해서 fetch 해온다고 생각해보자. 아래와 같은 상황이 될 것이다.

![another-remote](/assets/img/gitbranch/12.jpg)

![local-another-remote](/assets/img/gitbranch/13.jpg)

이렇게 fetch로 리모트 트래킹 브랜치를 내려받아도 로컬 저장소에 수정할 수 있는 브랜치가 새로 생기는 것은 아니고. 그냥 참조할 수 있는 브랜치 포인터가 생기는 것이다.

새로 받은 브랜치의 내용을 merge하려면 merge하려는 브랜치에서(예를 들어 master 브랜치에서) git merge remote명/branch명 으로 머지할 수있다.

Merge를 하지 않고 fetch 해온 리모트 트래킹 브랜치에서 시작하는 새로운 브랜치를 만들려면

```bash
git checkout -b newbranch명 remote명/branch명
```

으로 시작할 수 있다.

우아한테크코스에서 pair의 코드를 가져와서 이 코드를 새로운 나의 로컬 브랜치에서 수정을 하고 싶다면..?

일단 페어의 remote를 alias로 등록하고, 해당 remote에서 pair의 branch를 가져오면 된다.

```bash
git remote add pair https://github.com/페어의아이디/페어의레포주소.git
git fetch pair 페어의브랜치명

git checkout -b 원하는브랜치명 pair/페어의원격브랜치명
```

이런 식으로 진행하면 된다.

## Branch Tracking
리모트 트래킹 브랜치를 로컬 브랜치로 checkout 하면 자동으로 트래킹 브랜치가 만들어진다. 

트래킹 브랜치는 리모트 브랜치와 직접 연결고리가 있는 로컬 브랜치이다.

위에서 했던 `git checkout -b 원하는브랜치명 pair/페어의원격브랜치명` 이 명령으로 간단히 트래킹 브랜치를 만들 수 있고, --track 옵션을 사용해서 로컬 브랜치명을 자동으로 생성할 수도 있다. (가져오는 페어의 원격 브랜치명과 같은 로컬 브랜치 명)

`git checkout --track pair/페어의원격브랜치명`

이렇게 트래킹 브랜치를 만들면, 이 브랜치에서 push나 pull 하면 연결되어 있는 pair/페어의원격브랜치명 에서 자동으로 push, pull을 하게 된다.

이미 로컬에 있는 브랜치를 리모트의 특정 브랜치를 추적하게 하려면

`git branch -u pair/페어의원격브랜치명` 또는 `git branch --set-upstream-to pair/페어의원격브랜치명` 으로 설정하면 된다.

트래킹 브랜치가 어떻게 설정되어 있는지 보려면
`git branch -vv` 명령으로 확인 할 수 있다.

**리모트 브랜치 삭제**

리모트 브랜치를 만들었다가 작업을 끝내고 master브랜치로 merge 했다. 그러면 이제 그 리모트 브랜치는 필요가 없기 때문에 삭제해도 된다.

그러면 `git push origin --delete 리모트브랜치명` 를 실행하면 된다.
그러면 그 remote branch를 가리키는 포인터가 사라진다! (가비지 컬렉터가 동작하기 전에는 다시 살릴수도 있다.)


## Rebase
위에서 merge 했던 상황으로 돌아가본다.

![Master Commit](/assets/img/gitbranch/4.png)

![Merge](/assets/img/gitbranch/5.png)

rebase는 이 merge와 비슷한 방식이다.
```bash
git checkout test
git rebase master
```

이 명령을 수행하면 아래와 같이 된다.

![rebase](/assets/img/gitbranch/14.jpg)

그 이후에 master 브랜치에서 merge 명령을 실행해서 Fast-forward 한다.

```bash
git checkout master
git merge test
```

![fast-forward](/assets/img/gitbranch/15.jpg)

## Rebase의 위험성
**공개 저장소(리모트 저장소)에 Push한 커밋을 Rebase하지 마라!**

rebase는 기존의 커밋을 그대로 사용하는 것이 아니라 내용은 같은데 다른 커밋을 새로 만든다.

새 커밋을 서버에 push하고 그 커밋(C0)을 동료가 pull 해서 작업을 한다고 하자.

그런데 그 이후에 그 커밋(C0)을 rebase로 바꿔서 push 해버리면 동료가 다시 push 했을 때는 동료는 다시 merge를 해야 한다. 그리고 그 merge commit을 내가 다시 pull 하면 내 코드는 이상해진다..

아무튼 꼬인다.. 하지말자..

자세한 내용은 ProGit 111 page 부터 나와있는 **Rebase의 위험성** 을 참조하자.
