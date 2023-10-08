---
layout: single
title: "Synology 에서 Docker 로 redis 설치하기"
categories:
  - "synology"
tags:
  - "docker"
  - "redis"
---

## Docker 레지스트리에서 redis 이미지 받기

redis 를 검색해서 나오는 이미지를 다운받습니다.

## redis 용 docker 디렉토리 만들기

```text
/docker
`- redis
```

/docker/redis 에 redis.conf 파일을 추가합니다.

내용은 링크의 소스와 동일합니다.

[https://github.com/redis/redis/blob/unstable/redis.conf](https://github.com/redis/redis/blob/unstable/redis.conf)

원격 접속을 위해 87 line 의 bind 값을 수정합니다.

```text
# 기존값 
bind 127.0.0.1 -::1

# 변경값
bind * -::*
```

## redis 이미지로 컨테이너 생성하기

mysql:latest 를 생성합니다.

### 네트워크

브릿지

### 일반설정

이름을 입력합니다.

### 고급설정

일반설정에서 '고급 설정'을 선택합니다.

환경 탭에서 추가를 선택해서 아래의 변수를 입력해줍니다.

값은 임의로 넣어줍니다.

- REDIS_PASSWORD: {my_redispassword}

### 포트 설정

로컬 포트:컨테이너 포트를 설정합니다.

6379:6379

### 볼륨 설정

디렉터리를 선택하고 아래와 같이 마운트 경로를 잡아줍니다.

- /docker/redis : /etc/redis

- 이제 redis 가 구동이 됩니다.

## redis-CLI

redis-cli 를 이용해서 원격접속이 가능합니다.

redis-cli 를 이용하기 위해서는 redis 를 설치해야 합니다.

```shell
$ brew install redis
```

## 접속하기

```shell
$ redis-cli -h { REDIS_HOST } -p { REDIS_PORT } -a { MY_REDISPASSWORD } -n { DATABASE_NUM }
```

이제 외부에서 원격접속이 가능한 것을 확인할 수 있습니다.

## 참고

- [https://seokhyun2.tistory.com/63](https://seokhyun2.tistory.com/63)
- [https://www.movingjin.com/30](https://www.movingjin.com/30)
- [https://hub.docker.com/_/redis](https://hub.docker.com/_/redis)
- [https://soyoung-new-challenge.tistory.com/124](https://soyoung-new-challenge.tistory.com/124)
- [https://www.docker.com/blog/how-to-use-the-redis-docker-official-image/](https://www.docker.com/blog/how-to-use-the-redis-docker-official-image/)
