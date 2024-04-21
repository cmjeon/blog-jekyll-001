---
layout: single
title:  My JSON Server 로 백엔드 API 서버 배포하기
categories: 
  - backend
tags: 
  - backend
---

## My JSON Server 란

My JSON Server 는 db.json 이라는 하나의 파일로 API 서버를 구축할 수 있게 해주는 편리한 서비스임

db.json 을 github 에 업로드 함으로써 손쉽게 API 서버를 만들 수 있음

[My JSON Server](https://my-json-server.typicode.com/)

1. db.json 파일을 생성한다.

    

    ```json
    // db.json 파일 예시
    {
      "products": [
        {
          "id": 0,
          "name": "Refined Fresh Chicken",
          "price": "209.00",
          "imageUrl": "http://placeimg.com/640/480/fashion"
        },
        {
          "id": 1,
          "name": "Intelligent Metal Mouse",
          "price": "84.00",
          "imageUrl": "http://placeimg.com/640/480/fashion"
        },
        {
          "id": 2,
          "name": "Handcrafted Frozen Pizza",
          "price": "315.00",
          "imageUrl": "http://placeimg.com/640/480/fashion"
        },
        {
          "id": 3,
          "name": "Practical Metal Gloves",
          "price": "702.00",
          "imageUrl": "http://placeimg.com/640/480/fashion"
        },
      ],
      "carts": [
        {
          "id": 1,
          "name": "Intelligent Metal Mouse",
          "price": "84.00",
          "imageUrl": "http://placeimg.com/640/480/fashion"
        },
        {
          "id": 7,
          "name": "Awesome Concrete Shirt",
          "price": "165.00",
          "imageUrl": "http://placeimg.com/640/480/fashion"
        },
        {
          "id": 12,
          "name": "Awesome Metal Hat",
          "price": "376.00",
          "imageUrl": "http://placeimg.com/640/480/fashion"
        },
      ]
    }
    ```

    db.json 은 resources 단위로 구분되며 사용됨

1. db.json 파일을 github 의 리포지토리에 업로드함

    cmjeon 계정의 inflearn_nuxtjs001_api 리포지토리에 업로드 하였음

    [https://github.com/cmjeon/inflearn_nuxtjs001_api](https://github.com/cmjeon/inflearn_nuxtjs001_api)

1. `my-json-server.typicode.com/{user}/{repo}` 로 접근하면 db.json 의 내용을 확인가능함

    특정 resources 에도 접근가능함

    [https://my-json-server.typicode.com/cmjeon/inflearn_nuxtjs001_api/products](https://my-json-server.typicode.com/cmjeon/inflearn_nuxtjs001_api/products)

1. POSTMAN 등을 통해 GET, POST, PUT, PATCH, DELETE 사용

    201 created 등 정상적인 response 을 받을 수 있지만 db.json 이 바뀌지 않으므로 변경된 데이터를 다시 확인할 수는 없음

## 참고
- [https://my-json-server.typicode.com/]
