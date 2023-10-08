---
layout: single
title: "Synology 에서 ssh 로 Docker 에 접속하기"
categories:
  - "synology"
tags:
  - "docker"
---

## ssh 접속 설정

Synology 제어판에서 '터미널 및 SNMP' 으로 접속합니다.

'SSH 서비스 활성화' 를 체크합니다.

이 때 포트번호는 임의로 변경합니다.

이제 ssh 나 putty 등으로 접속합니다.

```shell
$ ssh -p {SSH_PORT} {NAS_ID}@{NAS_IP}
```

## docker 컨테이너 접속

ssh 로 접근한 후에 docker 컨테이너 목록을 확인할 수 있습니다.

docker 컨테이너 목록을 확인합니다.

```shell
$ docker ps -a
```

원하는 컨테이너 접속합니다.

```shell
$ docker exec -t -i {CONTAINER_ID} /bin/bash
```

참고
- [https://www.lainyzine.com/ko/article/how-to-enable-and-access-an-ssh-server-to-your-synology-nas/](https://www.lainyzine.com/ko/article/how-to-enable-and-access-an-ssh-server-to-your-synology-nas/)
- [https://deepmal.tistory.com/23](https://deepmal.tistory.com/23)
- [https://dukdo.com/nas/6683](https://dukdo.com/nas/6683)
