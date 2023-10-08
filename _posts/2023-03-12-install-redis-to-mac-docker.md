---
layout: single
title: "Mac 에서 Docker 로 redis 설치하기"
categories:
  - "docker"
tags:
  - "redis"
---

## Mac 에서 Docker 로 redis 설치하기

### Mac 에 Docker 설치하기

brew 를 이용해서 Docker 를 설치합니다.

```shell
$ brew install docker

# 설치확인
$ docker --version
$ docker-compose --version
```

이미지는 Docker 의 컨테이너를 띄우기 위한 실행 패키지 (executable package) 입니다.

> An image in Docker is a lightweight, stand-alone, executable package that includes everything needed to run an application: code, libraries, dependencies, and configuration files.

### 이미지 추가하기

redis latest 이미지를 받습니다.

```shell
$ docker pull redis:latest
```

> 이미지는 Dockerfile 로 만들어집니다.


redis:latest 의 Dockerfile 위치 : [https://github.com/docker-library/redis/blob/d77143afb3dc8d0b05225ab23b001cf6c41e1b62/7.0/Dockerfile](https://github.com/docker-library/redis/blob/d77143afb3dc8d0b05225ab23b001cf6c41e1b62/7.0/Dockerfile)

### 이미지 삭제하기

```shell
$ docker rmi { IMAGE_NAME }

# 다른 방법
$ docker image rm { IMAGE_NAME }
```

### 이미지를 이용해서 컨테이너를 생성하고 구동하기

```shell
$ docker run --name myRedis -d -p 6379:6379 redis

# 다른 방법
$ docker container run ...
```

컨테이너는 이미지로 의해 생성된 환경 (environments) 입니다.

> Containers are lightweight, portable, and self-contained environments that can run applications and their dependencies consistently across different environments.

명령어 설명
- --name myRedis : 컨테이너명은 'myRedis'
- -d : detached mode 로 구동, 실행 후 터미널이 프롬프트로 빠져나옴
- -p 6379:6379 : 호스트 포트 6379, 컨테이너 포트 6379
- redis : redis 이미지를 사용하여 생성 

### 구동하고 있는 컨테이너 목록 확인하기

```shell
$ docker ps --all

# 다른 방법
$ docker container ls --all
```

- --all : 모든 상태의 컨테이너를 확인

### 컨테이너 중지하기

```shell
$ docker stop { CONTAINER_ID }

# 다른 방법
$ docker container rm { CONTAINER_ID }
```

> CONTAINER_ID 대신 NAME 도 가능합니다.

### 중지된 컨테이너 재기동하기

```shell
$ docker start { CONTAINER_ID }
```

`$ docker run` 과 헷갈리지 않게 주의합니다.

### 컨테이너 삭제하기

```shell
$ docker rm { CONTAINER_ID }

# 다른 방법
$ docker container rm { CONTAINER_ID }
```


## 컨테이너에 접속하기

컨테이너에 접속하는 방법은 크게 3가지입니다.

### myRedis 컨테이너에 접속해서 redis 에 접속하기

> Mac -> myRedis Container -> redis (in myRedis Container)

myRedis 컨테이너에 접속하는 명령어입니다.

```shell
$ docker exec -t -i { CONTAINER_ID } /bin/bash
```

myRedis 컨테이너에서 redis 에 접속할 수 있습니다.

```shell
redis 접속하기
$ redis-cli -p { REDIS_PORT } -n { DATABASE_NUM }
```

한번에 접속하는 방법입니다.

```shell
$ docker exec -it { CONTAINER_ID } redis-cli -p { REDIS_PORT } -n { DATABASE_NUM }
```

### Mac 에서 바로 myRedis 컨테이너의 redis 에 접속하기

> Mac -> redis (in myRedis Container)

myRedis 컨테이너에 있는 redis 에 접속하기 위해서는 redis-cli 가 필요합니다.

redis 를 설치해서 redis-cli 를 사용할 수도 있습니다.

```shell
# redis 설치하기
$ brew install redis

# redis 접속하기
$ redis-cli -p { REDIS_PORT } -n { DATABASE_NUM }

# redis 원격 접속하기
$ redis-cli -h { REDIS_HOST } -p { REDIS_PORT } -a { MY_REDISPASSWORD } -n { DATABASE_NUM }
```

### redis-cli 용 임시 컨테이너를 통해 redis 에 접속하기

> Mac -> Temp redis Container -> redis (in myRedis Container)

redis-cli 용 임시 컨테이너를 생성하고, 임시 컨테이너를 통해 redis 에 접속할 수도 있습니다.

```shell
$ docker run -it --link myredis:redis --rm redis redis-cli -h redis -p 6379
```

명령어 설명

- --it : 도커를 인터랙티브 모드로 실행
- --link myredis:redis : 현재 컨테이너와 myredis 라는 컨테이너를 연결
- --rm : 현재 컨테이너는 연결이 끊어질 때 자동으로 삭제됨
- redis : 컨테이너를 만들 때 사용할 이미지
- redis-cli -h redis -p 6379 : 현재 컨테이너 내부에서 사용하는 명령어


## 참고

- [https://docs.docker.com/engine/reference/run/](https://docs.docker.com/engine/reference/run/)
- [http://redisgate.kr/redis/education/docker_intro.php](http://redisgate.kr/redis/education/docker_intro.php)
- [https://hub.docker.com/_/redis](https://hub.docker.com/_/redis)
- [https://wooono.tistory.com/123](https://wooono.tistory.com/123)
