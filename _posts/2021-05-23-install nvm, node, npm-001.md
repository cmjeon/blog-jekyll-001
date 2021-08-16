---
layout: single
title:  nvm, nodejs, npm 설치하기 - 001
categories: 
  - env/config
tags: 
  - nvm
  - nodejs
  - npm
---

## nvm

nvm 은 node version manager 임

각 프로젝트별로 다른 버전의 node를 사용할 수 있으니 여러 버전의 node 버전을 설치할 수 있음

### mac, ubuntu 에서 설치

[https://github.com/nvm-sh/](https://github.com/nvm-sh/) 에 접속하여 최신 버전 및 install url 확인

nvm 설치

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

profile file (~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc) 에 아래설정 추가

```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

### mac homebrew 사용자

만약 mac 에서 homebrew 를 사용중이면 아래 명령어를 통해 설치가능

```bash
$ brew install nvm
```

profile file 에 아래설정 추가

```bash
export NVM_DIR="$HOME/.nvm" [ -s "/usr/local/opt/nvm/nvm.sh" ] && . "/usr/local/opt/nvm/nvm.sh" [ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && . "/usr/local/opt/nvm/etc/bash_completion.d/nvm"
```

profile file 갱신

```bash
$ source ~/{prifile file}
```

### nvm 버전 확인

```bash
# nvm 버전 확인
$ nvm --version
```

### windows 에서 설치

[https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases) 에 접속하여 nvm-setup.zip 다운로드 후 설치

### windows chocolately 사용자

만약 chocolately 를 사용중이면 아래 명령어를 통해 설치가능

```bash
> choco install -y nvm
```

### nvm 버전 확인

```bash
> nvm --version
```

## node

### node 설치

nvm 이 설치되었다면 node 설치는 nvm 명령을 통해 진행 가능

```bash
# 설치가능한 node 버전 확인
$ nvm ls-remote

# 최신버전의 node 설치
$ nvm install 

# 특정버전의 node 설치
$ nvm install x.xx.x

# 설치된 node 확인
$ nvm ls

# 특정버전의 node 사용
$ nvm use x.xx.x
```


### node 버전 확인

```bash
# 사용중인 node 버전 확인
$ node -v
```

## npm

npm 은 node 가 설치되면 자동으로 설치됨

### npm 버전 확인

```bash
# 사용중인 npm 버전 확인
$ npm -v
```

## 참고

- [https://github.com/nvm-sh/](https://github.com/nvm-sh/)
- [https://github.com/coreybutler/nvm-windows]