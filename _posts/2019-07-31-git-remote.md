---
layout: single
title:  "ProGit 정리 - Git Remote"
date:   2019-07-31 23:00:00 +0900
classes: wide
categories: git
tags: git
---
# 리모트 저장소

## 리모트 저장소 추가하기

`git remote add [내가 해당 원격 저장소를 부르고 싶은 이름(alias)] [원격 저장소 주소]`

이렇게 하면 원격 저장소를 이제 alias로 정한 이름으로 부를 수 있다.

`git fetch [remote alias name]`

이 명령을 실행하면 remote 저장소에 있는 데이터를 모두 가져온다.

리모트 저장소의 모든 브랜치를 로컬에서 접근할 수 있게 되어서 merge 하거나 내용을 볼 수 있게 된다.

하지만 `git fetch` 명령은 remote의 모든 데이터를 가져오긴 하지만 merge하지는 않는다. 

그래서 로컬에서 하던 작업을 모두 정리한 후 수동으로 merge를 진행 해야한다.

`git clone` 명령은 자동으로 local의 master 브랜치가 remote 저장소의 master 브랜치를 추적하도록 한다.

그리고 `git pull` 명령은 Clone한 서버에서 데이터를 가져오고 그 데이터를 자동으로 현재 작업 하는 코드와 Merge 시킨다.


## 원격 저장소에 Push하기

`git push origin master` 하면 origin 서버의 master 브랜치로 push 된다.

`git remote show origin` 같은 작업을 하면 

```
remote origin
Fetch URL: ~~
Push URL: ~~~
HEAD branch: master
Remote branch:
    oeeen tracked
Local branch configured for 'git pull':
    oeeen merges with remote oeeen
Local ref configured for 'git push':
    oeeen pushes to oeeen (up to date)
```

이런 식으로 나온다.

살펴보면 `git pull`와 `git push` 명령을 수행 했을 때 일어나는 일이 무엇인지 알 수 있다.

여기서는 pull 하면 원격 저장소의 oeeen에서 로컬의 oeeen 브랜치로 머지 된다.

push를 하면 로컬의 oeeen 브랜치에서 원격의 oeeen으로 푸시된다.

## 기타 명령어
```bash
git remote rm [alias]
git remote rename [기존 alias] [new alias]
git remote set-url [지우길 원하는 alias] —delete [지우길 원하는 주소]
git remote set-url [추가하길 원하는 alias] —add [추가하길 원하는 주소]
```