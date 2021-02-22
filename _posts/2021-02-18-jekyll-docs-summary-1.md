---
layout: posts
title: jekyll 문서 요약 1
categories: 
  - jekyll
tags: 
  - jekyll
  - jekyll docs
---
# contents
## page - 페이지
- 페이지는 콘텐츠의 기본요소
- 루트 디렉토리에 만들어도 되고, 하위 폴더로 정리할 수도 있음
- html, md(html로 변환됨) 파일로 생성가능
```
  |- about.md # https://example.com/about.html
  `- docs
   |- doc1.md # https://example.com/docs/doc1.html
   `- doc2.md # https://example.com/docs/doc2.html
```

---

## post - 포스트
### 포스트 폴더
- 포스트는 `_post`에 모여있어야 함

### 포스트 생성하기
- 일반적으로 마크다운으로 작성되며, HTML로도 작성가능
- 포스트의 파일명 규칙
```
  YYYY-MM-DD-title.md
  ex) 2011-12-31-new-years-eve-is-awesome.md
```

- 모든 포스트는 front matter로 시작해야 함
```yml
  ---
  layout: post
  title: "Welcome to Jekyll"
  ---
  # Welcome

  **Hello world**, this is my first Jekyll blog post.

  I hope you like it!
```

### 포스트에 이미지 삽입하기
```
... which is shown in the screenshot below:
![My helpful screenshot](/assets/screenshot.jpg)
```

- pdf 다운로드 링크 삽입하기
```
... you can [get the PDF](/assets/mydoc.pdf) directly.
```

### 포스트 목록 표시하기
```{% raw %}
  <ul>
    {% for post in site.posts %}
      <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
      </li>
    {% endfor %}
  </ul>
{% endraw %}```

### 카테고리와 태그 생성하기
```
---
layout: post
title: A Trip
categories: [blog, travel]
tags: [hot, summer]
---
```

- jekyll에서는 site.categories로 카테고리를 사용할 수 있음
```html{% raw %}
  {% for category in site.categories %}
    <h3>{{ category[0] }}</h3> <!-- category[0]은 카테고리의 이름 -->
    <ul>
      {% for post in category[1] %} <!-- category[1]은 카테고리의 포스트 배열 -->
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
    </ul>
  {% endfor %}
{% endraw %}```
- 태그는 `site.tags`로 사용할 수 있음

### 포스트 발췌
- 발췌를 지정하고, 포스트 목록에서 발췌를 출력할 수 있음
```md
  # 발췌 지정하기
  ---
  excerpt_separator: <!--more-->
  ---

  Excerpt with multiple paragraphs

  Here's another paragraph in the excerpt.
  <!--more-->
  Out-of-excerpt
```
```html{% raw %}
  # 발췌 표시하기
  <ul>
    {% for post in site.posts %}
      <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
        {{ post.excerpt }}
      </li>
    {% endfor %}
  </ul>
{% endraw %}```

### Drafts
- Draft는 파일명에 날짜가 없는 포스트
- `_drafts` 폴더에 생성 및 보관

---

## front matter - 머리말
- front matter는 레이아웃이나 메타 데이터를 설정하는데 사용함
```md
  ---
  layout: post
  title: Blogging Like a Hacker
  ---
```
### 사전-정의 전역 변수
- layout: _layout 디렉토리에 있는 파일을 레이아웃으로 사용하는 것
- permalink: 최종 URL
- published: 발행여부 true, false

### 포스트의 사전-정의 전역 변수
- date: 파일명보다 우선 순위가 높은 발행일자를 입력할 수 있음
  형식: YYYY-MM-DD HH:MM:SS +/-TTTT
- category, categories: 카테고리 목록으로 yaml 목록 또는 공백으로 구분된 문자열로 표현
  형식: [milk, pumpkin pie] or milk pumplkin pie
- tag: 태그 목록으로 yaml 목록 또는 공백으로 구분된 문자열로 표현
  형식: [milk, pumpkin pie] or milk pumplkin pie

### 사용자-정의 변수
- 머릿말에서 food를 정의했다면 본문에서 liquid로 사용할 수 있음
```md{% raw %}
  ---
  food: Pizza
  ---

  <h1>{{ page.food }}</h1>
{% endraw %}```
