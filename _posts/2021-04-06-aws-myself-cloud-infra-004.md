---
layout: single
title:  스스로 구축하는 AWS 클라우드 인프라 - 기본편 004
categories: 
  - learning aws cloud infra
tags: 
  - aws
  - cloud
  - infra
  - MySQL
---

## RDS for MySQL 생성하기

1. RDS 서비스 선택
    1. `Databases` 선택
    1. `Create Database` 선택
        1. `Standard Create` 선택
        1. Engine Info : `MySQL` 선택
        1. Templates : `Dev/Test` 선택
        1. Settings
            1. Auto generate a password : 체크해제
            1. Master password : 입력
            1. Confirm password : 입력
        1. DB instandce size : `Burstable classes (includes t classes)` 선택
        1. Connectivity
            1. Virtual private cloud(VPC) : `lab-vpc` 선택
            1. Publicly accessible (DB 공개접근) : `No` 선택
            1. Availability Zone : `ap-northeast-2a` 선택
        1. Additional Configuration
            1. Initial database name : sample 입력
            1. Backup : Enable automatic backups 체크해제
            1. Monitoring : 체크
        1. `Create database` 선택
    1. `Connectivity & security` 의 Endpoint & port 확인
1. EC2 서비스 선택
    1. `Security group` 선택
    1. `Create security group` 선택
        1. Security group name : lab-web-rds-mysql-sg 입력
        1. VPC : `lab-vpc` 선택
        1. Inbound rules
            1. Type : `MySQL/Aurora` 선택
        1. Outbound rules
            1. Type : `All trafic` 선택
        1. `Create Security group`
1. RDS 서비스 선택
    1. `Databases` 선택
    1. `Modify` 선택
        1. Network & Security - Security group : `lab-web-rds-mysql-sg` 선택

## SSH 또는 Web으로 Database 관리를 위한 RDS for MYSQL 접속

1. RDS 서비스 선택
    1. `Databases` 선택
    1. `RDS Endporint` 복사
1. EC2 서비스 선택
    1. `lab-web-srv-bastion` 선택
    1. `IPv4 Public IP` 복사
1. putty 실행
    1. Session
        1. Host Name : ec2-user@`IPv4 Public IP` 입력
    1. Connection > SSH > Auth
        1. `Browse...` 선택
        1. `seoul-lab-web-bastion.ppk` 열기
    1. `Open` 선택
1. mysql 접속
    ```bash
    $ mysql -u admin -p -h {RDS endpoint}
    Enter password : {mysql 암호입력}
    ```
1. mysql 테스트
    ```bash
    > show databases; # database 보기
    > use mysql; # mysql 데이터베이스 사용하기
    > show tables; # tables 보기
    > select * from user; # user 테이블 보기
    > exit # sql 접속종료
    ```
1. mysql php 설정
    ```bash
    $ cd /var/www/html/phpMyAdmin/
    $ sudo vi comfig.inc.php
    ```
    1. 주소변경
        ```bash
        # 주소입력
        $cfg['Servers'][$i]['host'] = {RDS Endpoint}
        ```
1. 브라우저에서 확인
    1. `lab-web-srv-bastion`의 IPv4 Public IP 로 접속
    1. `lab-web-srv-bastion`의 Security group 확인
        1. 80 port 확인
        1. `Security Groups` 선택
            1. `lab-web-bastion-sg` 선택
            1. `Inbound rules` 선택
                1. `Edit inbound rules` 선택
                1. `Add rule` 선택
                    1. `HTTP` 선택
                    1. `0.0.0.0/0` 입력
    1. `lab-web-srv-bastion`의 IPv4 Public IP 로 재접속
    1. {`lab-web-srv-bastion`의 IPv4 Public IP}/phpMyAdmin 으로 접속

## 참고
- [https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard](https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard)