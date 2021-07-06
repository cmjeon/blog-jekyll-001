---
layout: single
title:  만들면서 학습하는 리액트(react):소개편 - Hello world - 017
categories: 
  - learning react by jeonghwan-kim
tags: 
  - react
---

## 소개편 

### Hello world

브랜치 이동

    ```bash
    $ git checkout -f intro/hello-world
    ```

index.html 확인

    ```html
    <!DOCTYPE html>
    <html lang="ko">
      <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Lecture React</title>
      </head>
      <body>
        <div id="app"></div>

        <script
          crossorigin
          src="https://unpkg.com/react@17/umd/react.development.js"
        ></script>

        <script
          crossorigin
          src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"
        ></script>

        <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

        <script type="text/babel">
          const element = React.createElement("h1", null, "Hello world");
          ReactDOM.render(element, document.querySelector("#app"));
        </script>
      </body>
    </html>
    ```

동작확인

    ```bash
    $ npx lite-server --baseDir 2-react/
    ```

"Hello world"가 출력됨 어떻게?

## 참고
- [https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard](https://www.inflearn.com/course/%EB%A7%8C%EB%93%A4%EB%A9%B4%EC%84%9C-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard)
- [https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html](https://jeonghwan-kim.github.io/series/2021/04/08/lecture-react-intro.html)