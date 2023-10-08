---
layout: single
title: "Synology 에서 Docker 로 Ghost 5 설치하기"
categories:
  - "synology"
tags:
  - "docker"
  - "ghost5"
---

Synology 에서 Docker 로 Ghost 5 를 설치해보겠습니다.

Ghost 5 란 워드프레스와 유사한 CMS 입니다.

## Docker 레지스트리에서 이미지 받기

Ghost 5 설치를 위해서는 2개 도커 이미지를 받아야 합니다.

Docker 를 실행하고 레지스트리 탭으로 이동합니다.

2개의 이미지를 검색해서 받습니다.

- mysql
- ghost

이미지를 받을 때 태그를 선택하는데, latest 로 받으면 됩니다.

이미지를 받으면 이미지 탭에서 2개의 이미지가 보이게 됩니다.

## mysql 용 docker 디렉토리 만들기

```text
/docker
`- ghost-db
    |- conf.d
    `- mysql
```

컨테이너 생성 시에 연결할 디렉토리를 만들어둡니다.

## mysql 이미지로 컨테이너 생성하기

mysql:latest 를 생성합니다.

### 네트워크

브릿지

### 일반설정

이름을 입력합니다.

### 고급설정

일반설정에서 '고급 설정'을 선택합니다.

환경 탭에서 추가를 선택해서 아래의 변수를 입력해줍니다.

값은 임의로 넣어줍니다.

- MYSQL_ROOT_PASSWORD: {my_rootpassword}
- MYSQL_DATABASE: {my_database}
- MYSQL_USER: {my_user}
- MYSQL_PASSWORD: {my_password}

### 포트 설정

로컬 포트:컨테이너 포트를 설정합니다.

3306:3306

### 볼륨 설정

디렉터리를 선택하고 아래와 같이 마운트 경로를 잡아줍니다.

- /docker/ghost-db/mysql : /var/lib/mysql
- /docker/ghost-db/conf.d : /etc/mysql/conf.d

이제 mysql 이 구동됩니다.

## mysql 용 docker 디렉토리 만들기

```text
/docker
`- ghost-app
    `- content
```

컨테이너 생성 시에 연결할 디렉토리를 만들어둡니다.

## ghost 이미지로 Docker 생성하기

ghost:latest 를 생성합니다.

### 네트워크

브릿지

### 일반설정

이름을 입력합니다.

### 고급설정

일반설정에서 '고급 설정'을 선택합니다.

환경 탭에서 추가를 선택해서 아래의 변수를 입력해줍니다.

값은 mysql 생성 시에 등록한 값을 넣거나 임의로 넣어줍니다.

- database__client: mysql
- database__connection__host: {databasehost}
- database__connection__port: {포트 번호}
- database__connection__user: {MYSQL_USER 의 값}
- database__connection__password: {MYSQL_PASSWORD 의 값}
- database__connection__database: {MYSQL_DATABASE 의 값}
- url: {Ghost5 의 주소}

### 포트 설정

로컬 포트:컨테이너 포트를 설정합니다.

2368:2368

## 볼륨 설정

디렉터리를 선택하고 아래와 같이 마운트 경로를 잡아줍니다.

- /docker/ghost-app/content : /var/lib/ghost/content

이제 ghost 5 가 구동됩니다.

## 접속하기

{NAS_주소}:2368 로 접속 시 ghost 5 블로그에 접속이 가능합니다.

## 참고

- [https://studio.cinntiq.synology.me/install-ghost-5-on-the-synology-nas/](https://studio.cinntiq.synology.me/install-ghost-5-on-the-synology-nas/)
- [https://svrforum.com/svr/130908](https://svrforum.com/svr/130908)
- [https://ten20four.co.uk/ghost-on-docker-on-synology/](https://ten20four.co.uk/ghost-on-docker-on-synology/)
- [https://www.lainyzine.com/ko/article/how-to-install-and-use-docker-on-your-synology-nas/](https://www.lainyzine.com/ko/article/how-to-install-and-use-docker-on-your-synology-nas/)
- [https://dev.to/lrth06/deploy-a-ghost-blog-with-docker-j5m](https://dev.to/lrth06/deploy-a-ghost-blog-with-docker-j5m)
