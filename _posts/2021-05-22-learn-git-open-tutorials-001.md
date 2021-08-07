---
layout: single
title:  생활코딩 git 강좌 정리 - 001
categories: 
  - learning git by open tutorials
tags: 
  - git
---

## git 이란

리눅스를 개발하기 위해 만들어진 분산버전관리시스템

문서의 version(commit)관리를 지원하는 시스템. 버전관리 & 백업 & 협업이 가능

git은 CLI(Git Bash), TortoiseGit, Sourcetree 로 학습&사용 가능

CLI 학습 > TortoiseGit, Sourcetree 사용은 쉬움

TortoiseGit, Sourcetree 학습 > CLI 사용은 어려움

따라서 CLI를 기반으로 학습진행

## git 설치

https://git-scm.com > 다운로드

Next... Next.. 계속 Next

시작프로그램에서 git bash 검색 및 실행

```bash
$ git
usage: … 설명나오면 성공
```

## git 으로 버전관리 시작

git bash 실행

```bash
$ mkdir hello-git-cli-a
$ cd hello-git-cli-a
$ git init # 버전관리 시작
$ ls -al # .git이라는 폴더 생성 확인
```

## 버전 만들기

working tree : 버전을 만들기 전 작업공간

staging area : 버전으로 올리고 싶은 파일을 준비시키는 공간

repository : 저장소, 생성된 버전이 저장되는 곳

```bash
$ vi hello1.txt
# 1 입력
$ cat hello1.txt # hello1.txt 생성 및 내용 확인

$ git status
No commits yet... Untracked files...

$ git add hello1.txt  # hello1.txt는 git의 추적대상이 됨

$ git status
Changes to be commitedted

$ git commit -m 'v1' # 현재 staging area에 있는 파일의 변경사항을 v1 이라는 메시지를 넣은 버전으로 생성
```

---

**NOTE**

초기 commit 시에 email과 name을 입력해야 함

---

```bash
$ git status
Nothing to commit

$ git log # git 의 변경사항 확인. v1 있음

$ vi hello1.txt
# 2 입력
$ cat hell1.txt # hello1.txt 생성 및 내용 확인

$ git status
Changes not staged for commit

$ git add hello1.txt # hello1.txt 의 변화를 staging area에 올림

$ git status
Changes to be committed

$ git commit -m 'v2' # 버전을 생성

$ git status
Nothing to commit

$ git log # git 의 변경사항 확인. v1, v2 있음
```




## 

## 참고
- [https://www.youtube.com/playlist?list=PLuHgQVnccGMCNJESahrVV-uYGMNYK_vMf](https://www.youtube.com/playlist?list=PLuHgQVnccGMCNJESahrVV-uYGMNYK_vMf)