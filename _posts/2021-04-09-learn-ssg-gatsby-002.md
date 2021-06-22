---
layout: single
title:  정적 사이트 생성기 gatsby 배우기 - 002
categories: 
  - learning ssg gatsby
tags: 
  - static site generator
  - gatsby
---

## 정적 사이트 생성기 gatsby 배우기 - 002

- Gatsby 사이트 S3에 배포하기

### AWS CLI 설치하기

1. https://awscli.amazonaws.com/AWSCLIV2.pkg 다운로드
1. symlink 생성
    ```bash
        $ sudo ln -s /usr/local/aws-cli/aws-cli/aws /usr/local/bin/aws
        $ sudo ln -s /usr/local/aws-cli/aws-cli/aws_completer /usr/local/bin/aws_completer
    ```

### AWS Configure

- `aws configure` 를 통해 빠른 구성
    ```bash
    $ aws configure
    AWS Access Key ID [None]: # AWSAccessKeyId
    AWS Secret Access Key [None]: # AWSSecretKey
    Default region name [None]: # ap-northeast-2
    Default output format [None]: # json
    ```

### S3 만들기

1. S3 서비스 선택
1. `Create Bucket` 선택
1. Bucket name 입력
    1. 3자 ~ 63자, 중복불가
    1. 소문자, 숫자, `.`, `-` 으로만 구성
1. `Region` 선택 > `...Seoul...` 선택
1. Bucket setting for Block Public Access
    1. `Block all public access` 체크 해제
    1. `I acknowledge that ...` 체크
1. `Create Bucket` 선택

### 정적 웹 사이트 호스팅 활성화

1. 생성된 Bucket 선택
1. `Properties` 탭 선택
    1. `Static website hosting` 의 `Edit` 선택
    1. `Use this bucket to host a website` 선택
        1. `Static website hosting` 에 `Enable` 선택
        1. `Index document` 에 `index.html` 입력
        1. `Save changes` 선택
1. `Permissions` 탭 선택
    1. `Bucket Policy` 의 `Edit` 선택
        1. Bucket ARN 의 `arn:aws:s3...` 복사
        1. `Policy generator` 선택
            1. Step 1: Select Policy Type
                1. Select type of policy `S3 Bucket Policy` 선택
            1. Step 2: Add Statement(s)
                1. Effect : Allow
                1. Principal : *
                1. Actions : `All Actions` 체크
                1. Amazon Resource Name : `arn:aws...` 붙여넣기
                1. `Add Statment` 선택
            1. Step 3: Generate Policy
                1. `Generate Policy` 선택
            1. Policy JSON Document 내용 복사
        1. `Bucket Policy` 페이지로 돌아오기
        1. `Policy` 부분에 붙여넣기
        1. `"Resource": "arn:aws:s3..."` 문장 끝에 `/*` 추가
            - "Resource": "arn:aws:s3:::YOUR S3 BUCKETS NAME/*"
        1. `Save changes` 선택

### S3 셋업

1. gatsby site 디렉토리로 이동
1. gatsby-plugin-s3 설치
    ```bash
    $ npm install gatsby-plugin-s3
    ```

### gatsby-config.js 수정

1. gatsby site 디렉토리로 이동
1. gatsby-config.js 수정
    ```javascript
    module.exports = {
        ...
        plugins: [
            ...
            {
                resolve: `gatsby-plugin-s3`,
                options: {
                    bucketName: "YOUR S3 BUCKETS NAME",
                },
            },
            ...
        ],
    };
    ```

### package.json 수정

1. gatsby site 디렉토리로 이동
1. package.json 수정
    ```javascript
    {
        ...
        "scripts": {
            ...
            "deploy": "gatsby-plugin-s3 deploy --yes"
        },
        ...
    }
    ```

### build & deploy

```bash
$ npm run build
$ npm run deploy
```

## 참고
- [https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/deploying-to-s3-cloudfront/](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/deploying-to-s3-cloudfront/)
- [https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-mac.html](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-mac.html)
- [https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-quickstart.html](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-quickstart.html)
- [https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_root-user.html#id_root-user_manage_add-key](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_root-user.html#id_root-user_manage_add-key)