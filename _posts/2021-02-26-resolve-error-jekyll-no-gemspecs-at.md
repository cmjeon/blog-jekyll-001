---
layout: single
title: 지킬 'There are no gemspecs' 오류 해결
categories: 
  - jekyll
tags: 
  - jekyll
  - minimal-mistakes
  - error
---

- 열심히 글을 쓰고 올리던 올리다가 테마색상을 바꾸고 싶어서 _config.yml를 변경하고 로컬에서 확인해보고 싶었음
- 로컬에서 구동을 하였더니 오류가 발생함

# 오류 내용
```
❯ bundle exec jekyll serve

[!] There was an error parsing `Gemfile`: There are no gemspecs at /{ repo }. Bundler cannot continue.

 #  from /{ repo }/Gemfile:2
 #  -------------------------------------------
 #  source "https://rubygems.org"
 >  gemspec 
 #  -------------------------------------------
```

# 원인
- minimal-mistakes의 Unnecessary 파일 목록 삭제 시 발생하는 것으로 생각됨
- [minimal-mistakes의 install guide](https://mmistakes.github.io/minimal-mistakes/docs/installation/#install-dependencies){:target="_blank"} 에 따라 Gemfile을 변경해야함

# 해결방법
- Gemfile 변경

```
source "https://rubygems.org"

gem "jekyll"
gem "minimal-mistakes-jekyll"

group :jekyll_plugins do
end
```

- _config.yml 변경

```
# 하단 내용 추가 or remote_theme setting 주석제거
remote_theme: minimal-mistakes-jekyll
```

- bundle update & install

```
$ bundle update
$ bundle install
```

- bundle 실행

```
$ bundle exec jekyll serve
```

# 참고
- https://github.com/mmistakes/minimal-mistakes/issues/1407
- https://mmistakes.github.io/minimal-mistakes/docs/installation/#install-dependencies
