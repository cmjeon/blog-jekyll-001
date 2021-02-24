---
layout: posts
title: jekyll 문서 요약 2
categories: 
  - jekyll
tags: 
  - jekyll
  - jekyll docs
---
# contents
## collections - 컬렉션
- 컬렉션은 서로 관련된 정보를 그룹화하는데 용이함
### 셋업
- 컬렉션을 사용하려면 먼저 _config.yml에 컬렉션을 정의해야 함

```
# staff_members를 컬렉션으로 정의한 예시
collections:
  - staff_members
```
### 콘텐츠 추가하기
- 컬렉션에 설정한 이름 `staff_members` 에 맞는 폴더인 `<source>/_staff_members` 를 추가함
- 머릿말이 있다면 먼저 처리되고 이후의 모든 내용은 문서의 content 속성에 들어감
- 머릿말이 없다면 jekyll은 내용 처리를 수행하지 않음
- 만약 컬렉션의 메타데이터에 output: true 가 설정되어 있는 경우, _site에 결과물을 생성
- Jane이라는 스테프를 추가하는 예시
```
# _staff_members/jane.md
---
name: Jane Doe
position: Developer
---
Jane has worked on Jekyll for the past *five years*.
```

### 출력
- 페이지에서 `site.staff_members`를 사용하여 각 스태프의 내용을 출력할 수 있음

```
# _staff_members 컬렉션 출력 예시
{% for staff_member in site.staff_members %}
  <h2>{{ staff_member.name }} - {{ staff_member.position }}</h2>
  <p>{{ staff_member.content | markdownify }}</p>
{% endfor %}
```
- posts랑 비슷하게 머릿말 하단은 `content` 변수로 출력할 수 있음
- 머릿말의 존재여부에 상관없이 컬렉션의 모든 문서를 렌더링하고 싶다면 `_config.yml`의 컬렉션 정의를 수정해야 함

```
# _config.yml의 collection 부분 수정
collections:
  staff_members:
    output: true
```
- 생성된 페이지에 대한 링크는 `url`속성으로 얻을 수 있음

```{% raw %}
{% for staff_member in site.staff_members %}
  <h2>
    <a href="{{ staff_member.url }}">
      {{ staff_member.name }} - {{ staff_member.position }}
    </a>
  </h2>
  <p>{{ staff_member.content | markdownify }}</p>
{% endfor %}
{% endraw %}```

### 고유주소
- 고유주소 변수로 컬렉션의 url 결과물을 제어할 수 있음

### 문서 순서 조정
- 컬렉션 안의 모든 문서들이 머릿말에 date를 가지고 있다면 date를 기준으로 정렬
- 아니면 문서명을 기준으로 정렬
- 컬렉션의 메타데이터를 기준으로 정렬방법을 조회할 수 있음

#### 머리말 키에 따른 정렬
- 메타데이터에 `sort_by`를 설정하면 머리말 내용을 기반하여 문서 정렬이 가능
```
# lesson을 기준으로 정렬한 예시
collections:
  tutorials:
    sort_by: lesson
```
- 문서는 키 값에 따라 오름차순으로 정렬됨
- 머릿말에 키가 정의되지 않은 문서는 정렬된 문서 뒤에 옴 > 날짜나 경로로 정렬

#### 문서 수동 정렬
- 메타데이터에서 order 에 원하는 순서로 파일명을 나열하여 수동으로 정렬 가능
```
# 수동 정렬 예시
collections:
  tutorials:
    order:
      - hello-world.md
      - introduction.md
```
- order에 존재 하지 않는 문서는 정렬된 문서 뒤에 옴 > 날짜나 경로로 정렬

### Liquid 속성
#### 컬렉션

- 컬렉션은 site.collections 로도 사용가능
- _config.yml에 정의한 메타데이터와 아래 정보를 사용할 수 있음
```
label : 컬렉션의 이름. 예시, my_collection.
docs : 문서들의 배열.
files : 컬렉션 내의 정적 파일 배열.
relative_directory : 컬렉션 소스 디렉토리 (Site Source 디렉토리를 기준으로 한 상대 경로)
directory : 컬렉션 소스 디렉토리 전체 경로
output : 컬렉션의 문서들이 독립적인 파일로 출력될 것인지에 대한 여부.
```

#### 문서
- 모든 문서는 머리말에서 지정한 속성과 아래의 기본 속성을 가지고 있음

```
content : 문서의 (렌더링되지 않은) 컨텐츠. 머리말이 없다면, Jekyll 은 컬렉션에 어떠한 파일도 생성하지 않는다. 머리말이 사용되었다면, 이 변수는 머리말의 종료표시인 `---` 이후의 모든 내용이다.
output : content 에 기반하여 렌더링 된, 문서의 출력
path : 문서 소스 파일의 전체 경로
relative_path : 사이트 소스 경로를 기준으로 한, 문서 소스 파일의 상대 경로
url : 렌더링 된 컬렉션의 URL. 사이트 환경설정에 output: true 를 가지고 있는 컬렉션의 파일들만 Site Destination 에 생성된다.
collection : 해당 문서가 포함된 컬렉션의 이름
date : 문서가 속한 컬렉션의 날짜
```
