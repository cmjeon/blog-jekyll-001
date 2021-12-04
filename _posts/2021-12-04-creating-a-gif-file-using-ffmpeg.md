---
layout: single
title: ffmpeg를 이용해 gif 파일 만들기 - 001
categories: 
  - about something
tags:
  - ffmpeg
---

## 맥에서 캡쳐한 영상을 gif 로 만드는 방법

업무중에 구동되는 화면을 캡쳐해서 공유해야하는 일이 종종 생김

맥의 캡쳐프로그램으로 캡쳐를 하면 .mov 파일이 생기는데 이를 gif 로 변경하고 싶어서 알아봄

원하는 조건은 용량이 mov 보다 작고 사이즈 조절이 가능했으면 좋겠다는 것이었음

유료툴보다는 가급적 무료로 가능한 방법을 찾고 싶음

그러다 ffmpeg 를 사용한 방법을 찾게 되었고 ffmpeg 는 .mov 뿐 아니라 다른 포맷도 가능함

## 왜 gif 인가

플레이어 없이 바로 볼 수 있는 파일이라서?

## ffmpeg 설치

```bash
$ brew update
$ brew install ffmpeg
```

설치시간은 약 10분정도

## ffmpeg 사용하기

다른 옵션도 엄청나게 많지만 나는 단순하게 사용함

```bash
$ ffmpeg -i { inputFileName } -r 30 -vf scale=1920:-1 -f gif { outputFileName }
```

옵션은 크게 3가지임

`-r 30` : 프레임속도, 눈에 거슬리지 않을정도를 위해 30 프레임을 사용함. 프레임속도값이 커질수록 용량이 증가함

`-vf scale=1920:-1` : 해상도, 보통 모니터의 화면폭이 1920 이기에 폭을 1920 으로 사용함, -1 은 높이를 원본비율로 지켜줌 

`-f gif`  : 출력 파일의 포맷, gif 임

해당 옵션으로 변환한 결과 16MB 짜리 .mov 파일이 3.4MB .gif 파일로 변환되었고, 원본대비 약 20% 정도의 용량으로 작아짐

## 참고 
- [http://www.ffmpeg.org/](http://www.ffmpeg.org/)