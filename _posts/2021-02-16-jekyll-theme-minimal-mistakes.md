---
layout: posts
title: jekyll theme, minimal mistakes
categories:
  - jekyll
tags:
  - jekyll
  - theme
---
# jekyll 테마
- jekyll의 사용을 돕거나 더 이쁘게 웹사이트를 꾸밀 수 있게 여러가지 테마가 제공되고 있음
- 미리 제공되는 테마를 채택하면 내용 자체에만 집중할 수 있음. 추후에 수정도 가능
- [jekyll 홈페이지](https://jekyllrb-ko.github.io/docs/themes/){:target="_blank"}에서 테마를 찾을 수 있도록 제공하고 있음
- [깃헙 jekyll theme 토픽](https://github.com/topics/jekyll-theme){:target="_blank"}에서 가장 많은 star를 받은 minimal mistakes를 채택하기로 함

# minimal mistakes 설치하기
1. [minimal mistakes 홈페이지](https://mmistakes.github.io/minimal-mistakes/){:target="_blank"}에서 최신버전 다운로드
2. 압축해제한 파일들을 현재 블로그에 덮어쓰기
3. 로컬환경에서 실행
```bash
$ bundle exec jekyll serve
```
4. mininal mistakes 테마 적용 확인

# 테마 주요 설정(Configuration) 살펴보기
- _config.yml 파일이 테마의 기본 설정파일
- _config.yml 파일은 수정 시 서버를 재실행해야 함

## Skin
- 스킨은 텍스트 수정으로 10가지 스킨으로 변경가능
```yml
minimal_mistakes_skin: "default" # "air", "aqua", "contrast", "dark", "dirt", "neon", "mint", "plum" "sunrise"
```

## 발행 시간 표시
- show_date: true 설정으로 포스트의 발행 일자 표시가 가능

```yml
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      show_date: true # 발행 일자 표시
```

## locale
- 사이트 메인언어 설정 가능
```yml
# 한국어-한국으로 설정한 경우
locale : "ko-KR"
```

## title
- `<title>` 태그와 동일한 사이트 전체의 이름

## search
- search를 활성화하면 검색기능 활성화 가능
```yml
search : true
```

# 리눅스 history 명령어
- 과거 실행한 명령어 확인
```
$ history
```
- history 중 특정 명령어 검색
```bash
# sudo가 포함된 명령어
$ history | grep sudo
```
- history애 있는 명령어 실행
```bash
# 10번 명령어 실행
$ !10
```
