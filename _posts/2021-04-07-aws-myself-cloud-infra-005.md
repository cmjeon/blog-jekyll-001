---
layout: single
title:  스스로 구축하는 AWS 클라우드 인프라 - 기본편 005
categories: 
  - learning aws cloud infra
tags: 
  - aws
  - cloud
  - infra
  - MySQL
---

## Auto Scaling으로 확장성 및 탄력성 구현하기

### Application Load Balancer 생성

1. EC2 서비스 선택
    1. Load Balancing > `Load Balancers` 선택
    1. `Create Load Balancer` 선택
    1. Application Load Balancer 의 `Create` 선택
        1. Name : lab-web-alb-asg 입력
        1. scheme : internet-facing 선택
        1. Availability Zones
            1. lab-vpc
            1. `ap-northeast-2a` 체크
            1. lab-web-put1-2a 선택
            1. `ap-northeast-2c` 체크
            1. lab-web-pub2-2c 선택
        1. Tags
            1. Key : Name
            1. Value : lab-web-alb-asg
    1. `Next: Configure Security Settings` 선택
    1. `Next: Configure Security Groups` 선택
        1. `Create a new security group` 선택
        1. security group name : lab-web-alb-asg 입력
            1. Source : Custom 0.0.0.0/0 입력
    1. `Next: Configure Routing` 선택
        1. Target group : New target group 선택
        1. Name : lab-web-alb-asg-tg 입력
    1. `Next: Register Targets` 선택
        1. Instances에서 EC2 선택
        1. `Add to registered` 선택
    1. `Next: Review` 선택
    1. `Create` 선택

### Launch Configurations 설정

1. EC2 서비스 선택
    1. Auto Scaling > `Launch Configuration` 선택
    1. `Create launch configuration` 선택
        1. Name : lab-web-lc
        1. AMI : `lweb-web-0000` 설정
        1. Instance type : t2.micro 선택
        1. Advanced details
            1. User data
                1. `As text` 선택
        1. Security groups
            1. `Select an existing security group` 선택
            1. `lab-web-srv-sg` 선택
        1. Key pair
            1. `seoul-lab-web-srv` 선택
        1. `I acknowledge that... ` 체크
        1. `Create launch Configuration` 선택

### Auto Scaling Groups 설정

1. EC2 서비스 선택
    1. Auto Scaling > `Auto Scaling Groups` 선택
    1. `Create Auto Scaling group` 선택
        1. Name : lab-web-asg
        1. Switch to launch template 선택
        1. `lab-web-lc` 선택
        1. `Next` 선택
        1. VPC
            1. `lab-vpc` 선택
        1. Subnets
            1. `lab-web-pri1-2a` 선택
            1. `lab-web-pri2-2c` 선택
        1. `Next` 선택
        1. `Enable load balancing` 체크
        1. `Application Load Balancer or NEtwork Load Balancer` 선택
        1. Target group : `lab-web-alb-asg-tg` 선택
        1. `Next` 선택
        1. Group size - optional
            1. Desired capacity : 2
            1. Minimum capacity : 2
            1. Maximum capacity : 4
        1. `Target tracking scaling policy` 선택
            1. Metric type : Average CPU utilization
            1. Target value : 50
        1. `Next` 선택
        1. `Next` 선택
        1. `Add tag` 선택
            1. Key : Name
            1. Value : asg
        1. `Next` 선택
        1. `Create Auto Scaling Group` 선택

## 참고
- [https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard](https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard)
