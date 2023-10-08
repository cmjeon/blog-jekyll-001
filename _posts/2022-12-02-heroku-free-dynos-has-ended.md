---
layout: single
title: "heroku free dynos 종료"
categories:
  - "회고"
---

## Heroku free dynos 가 종료되었다.

사실 몇번의 메일이 왔었지만 대충 읽어서 Postgres 와 Redis 만 유료대상이 되는줄 알았다.

[https://blog.heroku.com/next-chapter#focus-on-mission-critical](https://blog.heroku.com/next-chapter#focus-on-mission-critical)

> We will be phasing out our free plan for Heroku Dynos, free plan for Heroku Postgres, and free plan for Heroku Data for Redis®, as well as deleting inactive accounts.

> Dynos 는 Heroku 에서 사용하는 컨테이너 이름이다.
> AWS 의 EC2 같은 느낌이라 생각하면 된다.

Heroku Dynos 가 nodejs 서비스가 돌아가고 있는 서비스이다.

Heroku 요금이 어떻게 바뀌었나 확인해보았다.

[https://www.heroku.com/pricing](https://www.heroku.com/pricing)

가장 저렴한 요금제는 1달에 5달러인 Eco 요금제이다.

단독 dynos 서비스에는 충분한 요금제이다.

이왕 유료요금제로 하는김에 DB 도 heroku postgres 에 올려둘지 고민해본다.

## AWS

nodejs 를 올릴 다른 서비스를 찾아본다.

역시 가장 대표적인 서비스는 AWS 이다.

가격이 heroku 에 비하면 약간 저렴한 편이다.

AWS 의 각종 서비스에 연결되기 쉬우니 확장하기도 편리하다.

하지만 AWS 는 아직은 복잡하다.

배포설정 등을 heroku 같이 github 에 commit & push 만으로 배포하려고 하기에는 배워야하는 부분이 있는 듯 하다.

AWS Elastic Beanstalk 가 Heroku 와 유사하다던데, 확인해봐야겠다.

## 그래서?

일단 Heroku 에서 서비스를 계속 하기로 하였다.

기능이 계속 개발되고 있으니 일단 원래 서비스하는 곳에서 계속 해보려고 한다.

참고
- [https://www.heroku.com/dynos](https://www.heroku.com/dynos)
- [https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html)
