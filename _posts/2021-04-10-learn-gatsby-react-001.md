---
layout: single
title:  React 기반 Gatsby로 기술 블로그 개발하기 001
categories: 
  - learning gatsby(react)
tags: 
  - static site generator
  - gatsby
  - react
---

## Gatsby 알아보기 및 개발 환경 구성하기

### Gatsby란 무엇일까?

#### JAM Stack 
- javascript, API, MarkUp Stack
- 빠르다 : JAM Stack은 렌더링할 화면들을 모두 Pre-Render하여 제공되기 때문
- 안전하다 : API를 통해 정적 사이트를 생성하므로, 공격 노출 범위가 감소
- 스케일링하기 쉽다 : 미리 빌드된 파일을 CDN을 통해 가능

#### 왜 Gatsby일까?

- 20년도 기준으로 next.js가 가장 많이 쓰이지만, next.js는 서버사이드렌더링이다.
- gatsby.js는 두번째로 많이 쓰이지만 정적 사이트 생성에 알맞다. > 기술블로그에 적합함

### Gatsby를 위해 필수로 알아야 할 기술

#### react

- Gatsby 는 react를 사용하는 JAM Stack 기반 프레임워크

#### GraphQL

- 페이스북에서 개발한 쿼리 언어
- 단일 엔드포인트에서 우너하는 데이터만을 받을 수 있음

### Gatsby 프로젝트 생성하기

#### 기본 개발 환경 구성

```bash
# node.js
$ brew install node
# yarn
$ brew install yarn
# git/github
$ brew install git
```

#### Gatsby 프로젝트 생성하기

- 프로젝트 생성하기

```bash
$ npx gatsby-cli new "[프로젝트 명]"
$ cd "[프로젝트 명]"
$ gatsby develop
```
- http://localhost:8000 에 접속하여 확인

#### 디렉토리 구조

```
- Root Directory
|- contents : 블로그 포스트 관련 파일을 저장
`- src
  |- components : react component를 저장
  |- hooks : Custom Hooks을 저장
  |- pages : 페이지 역할을 하는 컴포넌트를 저장
  `- templates : 같은 형식의 콘텐츠들을 보여주는 컴포넌트를 저장
```

- PWA(Progresssive Web Applicaton) 라이브러리, Gatsby Cloud 라이브러리 삭제

```bash
$ yarn remove gatsby-plugin-manifest gatsby-plugin-gatsby-cloud
```

### TypeScript 개발 환경 구성하기

#### TypeScript, gatsby 플러그인 설치

- TypeScript, gatsby 플러그인 설치

```bash
# 타입스크립트
$ yarn add typescript --dev
# 타입스크립트 플러그인
$ yarn add gatsby-plugin-typescript
```

- gatsby-config.js 파일 수정

```
module.exports = {
  siteMetadata: {
    title: `Gatsby Default Starter`,
    description: `Kick off your next, great Gatsby project with this default starter. This barebones starter ships with the main Gatsby configuration files you might need.`,
    author: `@gatsbyjs`,
  },
  plugins: [
    {
      resolve: 'gatsby-plugin-typescript',
      options: {
        isTSX: true,
        allExtensions: true,
      },
    },
    `gatsby-plugin-react-helmet`,
    // {
    //   resolve: `gatsby-source-filesystem`,
    //   options: {
    //     name: `contents`,
    //     path: `${__dirname}/contents`,
    //   },
    // },
    `gatsby-transformer-sharp`,
    `gatsby-plugin-sharp`,
    // this (optional) plugin enables Progressive Web App + Offline functionality
    // To learn more, visit: <https://gatsby.dev/offline>
    // `gatsby-plugin-offline`,
  ],
};
```

#### TypeScript 설정

- tsconfig.json 생성

```bash
$ yarn tsc --init
```

- tsconfig.json 수정

```
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "allowJs": true,
    "jsx": "preserve",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "baseUrl": "./src",
    "paths": {
      "components/*": ["./components/*"],
      "utils/*": ["./utils/*"],
      "hooks/*": ["./hooks/*"]
    },
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*.tsx"],
  "exclude": ["node_modules"]
}
```

- paths 옵션
  - paths 옵션은 절대경로를 사용하기 위해 경로를 매핑해주는 옵션임
  - baseUrl을 선언해줘야 함

- gastby-node.js 수정

```
/**
 * Implement Gatsby's Node APIs in this file.
 *
 * See: <https://www.gatsbyjs.com/docs/node-apis/>
 */

// You can delete this file if you're not using it

const path = require('path');

// Setup Import Alias
exports.onCreateWebpackConfig = ({ getConfig, actions }) => {
  const output = getConfig().output || {};

  actions.setWebpackConfig({
    output,
    resolve: {
      alias: {
        components: path.resolve(__dirname, 'src/components'),
        utils: path.resolve(__dirname, 'src/utils'),
        hooks: path.resolve(__dirname, 'src/hooks'),
      },
    },
  });
};
```

- alias 옵션
  - components로 시작하는 경로는 src/compoents 폴더로 매핑

#### ESLint와 Prettier 설정

- `ESLint`는 코드 분석을 통해 문법 오류 또는 코드 규칙에 어긋나는 부분을 찾아주는 `정적 코드 분석 도구`
- `Prettier`는 개발자가 작성한 코드를 정해진 규칙에 따르도록 변환해주는 `Code Formmater`

```bash
$ yarn add eslint prettier eslint-config-prettier eslint-plugin-prettier @typescript-eslint/eslint-plugin@latest @typescript-eslint/parser@latest --dev
```

- .eslintrc.json 생성

```bash
$ touch .eslintrc.json
```

- .eslintrc.json 설정

```
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "plugin:prettier/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": 12,
    "sourceType": "module",
    "project": "./tsconfig.json"
  },
  "plugins": ["react", "@typescript-eslint"],
  "ignorePatterns": ["dist/", "node_modules/"],
  "rules": {}
}
```

- .prettierrc 수정

```
{
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "quoteProps": "as-needed",
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

-VSCode 에서 저장 시, 자동으로 포매팅 formatting 되도록 설정

- Preferences > Settings
- 'Format On Save' 설정에 체크
- `Open Settings(JSON)` 버튼 선택하여 settings.json 열기
- 설정 등록

```
{
  ...,
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  ...
}
```

- components 폴더에 Text.tsx 생성

```
import React, { FunctionComponent } from 'react';

const Text: FunctionComponent = function ({ text }) {
  return <div>{text}</div>;
};

export default Text;
```

- pages 폴더에 index.tsx 생성

```
import React, { FunctionComponent } from 'react';
import Text from 'components/Text';

const IndexPage: FunctionComponent = function () {
  return <Text text="Home" />;
};

export default IndexPage;
```

- 프로젝트 실행

```
yarn develop
```

## 참고
- [https://www.inflearn.com/course/gatsby-기술블로그/dashboard](https://www.inflearn.com/course/gatsby-기술블로그/dashboard)