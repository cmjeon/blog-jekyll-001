---
layout: posts
title: m1 macOS에서 어플리케이션 설치하는 방법
categories: 
  - spring
tags: 
  - spring
  - m1 macOS
---
# .dmg 다운로드
- 가장 평범한 방법으로 m1 macOS용 .dmg 파일을 다운받아서 실행

# homebrew 설치
- homebrew는 linux의 apt와 같은 패키지 관리 프로그램
- homebrew는 현재 안정성때문에 인텔용프로그램을 설치하는 것이 좋음
- 로제타 설치 > homebrew 설치 > 세팅으로 진행
## 설치방법
- 로제타 설치
```
$ /usr/sbin/softwareupdate --install-rosetta --agree-to-license
```

- homebrew 설치
```
# 로제타를 이용해서 brew를 설치
$ arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- 세팅
```
# brew를 실행할 때마다 arch -x86_64로 실행하도록 약어를 등록
$ echo "alias brew='arch -x86_64 /usr/local/bin/brew'" >> .zshrc
```

  5. brew 설치
    1. https://brew.sh/index_ko
    2. https://velog.io/@mordred/Apple-M1-Mac%EC%97%90%EC%84%9C-HomeBrew-%EC%84%A4%EC%B9%98
    3. .zshrc 에 alias brew='arch -x86_64 /usr/local/bin/brew' 추가
    4. https://www.44bits.io/ko/post/setup-apple-silicon-m1-for-developers 참고

# 참고
- https://brew.sh/index_ko
- https://velog.io/@mordred/Apple-M1-Mac%EC%97%90%EC%84%9C-HomeBrew-%EC%84%A4%EC%B9%98
- https://www.44bits.io/ko/post/setup-apple-silicon-m1-for-developers