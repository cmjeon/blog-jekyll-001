---
layout: single
title: "docusaurus Algolia 적용하기"
categories:
  - "javascript"
tags:
  - "functional programming"
---

[https://docsearch.algolia.com/docs/legacy/run-your-own](https://docsearch.algolia.com/docs/legacy/run-your-own) 문서를 보고 Algolia 를 적용했다.

만약 algolia 계정이 없다면 하나 생성하여야 한다.

설정파일인 config.json 파일을 만든다.

```shell
$ ./docsearch bootstrap
```

위 명령어는 작동하지 않아서 도큐사우르스의 config.json 파일을 참고하였다.

[https://github.com/algolia/docsearch-configs/blob/master/configs/docusaurus-2.json](https://github.com/algolia/docsearch-configs/blob/master/configs/docusaurus-2.json)

도큐사우루스의 파일에서 index_name, start_urls, sitemap_urls 는 수정하였다.

```json
{
  "index_name": { 인덱스명 },
  "start_urls": [
    { 도메인주소 }
  ],
  "sitemap_urls": [
    { 도메인주소/sitemap.xml }
  ],
  "sitemap_alternate_links": true,
  "stop_urls": [
    "/tests"
  ],
  "selectors": {
    "lvl0": {
      "selector": "(//ul[contains(@class,'menu__list')]//a[contains(@class, 'menu__link menu__link--sublist menu__link--active')]/text() | //nav[contains(@class, 'navbar')]//a[contains(@class, 'navbar__link--active')]/text())[last()]",
      "type": "xpath",
      "global": true,
      "default_value": "Documentation"
    },
    "lvl1": "header h1",
    "lvl2": "article h2",
    "lvl3": "article h3",
    "lvl4": "article h4",
    "lvl5": "article h5, article td:first-child",
    "lvl6": "article h6",
    "text": "article p, article li, article td:last-child"
  },
  "strip_chars": " .,;:#",
  "custom_settings": {
    "separatorsToIndex": "_",
    "attributesForFaceting": [
      "language",
      "version",
      "type",
      "docusaurus_tag"
    ],
    "attributesToRetrieve": [
      "hierarchy",
      "content",
      "anchor",
      "url",
      "url_without_anchor",
      "type"
    ]
  },
  "conversation_id": [
    "833762294"
  ],
  "nb_hits": 46250
}
```

.env 파일에 APPLICATION_ID, API_KEY 를 설정한다.

```yaml
APPLICATION_ID= 
API_KEY= # admin API KEY 를 등록해야 한다.
```

jq 와 docker 를 설치한다.

> jq is a lightweight and flexible command-line JSON processor

```shell
$ brew install jq
```

docker 를 설치한다.

```shell
$ brew install --cask docker
```

프로젝트가 있는 폴더로 이동하여 아래 명력어를 실행한다.

docker 가 구동되면서 크롤링을 진행한다.

```shell
$ docker run -it --env-file=.env -e "CONFIG=$(cat ./config.json | jq -r tostring)" algolia/docsearch-scraper
```

문서상에서는 1주일에 한번씩 추가된 문서를 다시 크롤링한다고 되어 있는데, 실제로 어떤지 확인해보아야 한다.

> Crawls are handled by the Algolia Crawler and scheduled to run once a week by default, you can then trigger new crawls yourself and monitor them directly from the Crawler interface, which also offers a live editor where you can maintain your config.

---

하지만...

1주일이 한번씩 크롤링되지는 않았다.

대신 위의 명령어로 jq 를 구동해주면 크롤링하면서 문서가 검색되도록 해준다.
