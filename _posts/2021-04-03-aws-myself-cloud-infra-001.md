---
layout: single
title:  스스로 구축하는 AWS 클라우드 인프라 - 기본편 001
categories: 
  - learning aws cloud infra
tags: 
  - aws
  - cloud
  - infra
---

## 서버리스 웹 호스팅과 CloudFront로 웹 가속화 구성하기

- 서버없이 구성가능한 정적 웹 호스팅을 만들고, 웹 속도를 높이기 위한 콘텐츠 전송 네트워크(CDN) 서비스를 연동

## S3 정적 웹 호스팅 구성하기

### S3 Bucket 생성

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
  1. `Static website hosting` 선택
  1. `Use this bucket to host a website` 선택
    1. `Index document` 에 `index.html` 입력
    1. `Save` 선택
  1. `Permissions` 탭 선택
1. `Bucket Policy` 선택
  1. `arn:aws...` 복사
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
    1. 내용 복사
  1. `Bucket Policy` 페이지로 돌아오기
  1. 정책 부분에 붙여넣기
  1. `"Resource": "arn:aws..."` 끝에 `/*` 추가
  1. `Save` 선택

### 웹 사이트 엔드포인트 테스트

1. `Overview` 탭 선택
1. `Upload` 선택
1. 파일 추가
1. `Uplaod` 선택
1. 파일 선택

## CloudFront를 이용해 웹 사이트 속도 높이기

### CloudFront 배포 만들기

1. CloudFront 서비스 선택
1. `Create Distribution` 선택
1. Web의 `Get Started` 선택
1. Create Distribution
  1. Origin Settings
    1. Origin Domain Name 선택 > S3 Bucket 선택
  1. Distribution Settings
    1. Price Class 선택 > `Use All Edge...` 선택
  1. `Create Distribution` 선택
  1. 생성까지는 5 ~ 20분 소요 > Status 의 Deployed 확인

### 생성된 CloudFront 도메인 확인

## 참고
- [https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard]