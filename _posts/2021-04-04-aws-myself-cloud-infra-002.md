---
layout: single
title:  스스로 구축하는 AWS 클라우드 인프라 - 기본편 002
categories: 
  - learning aws cloud infra
tags: 
  - aws
  - cloud
  - infra
---

## EC2-LAMP-ELB 구성하기

### Amazon Linux 2에 LAMP 웹 서버 설치하기

---

**NOTE**

#### LAMP 웹 서버 설치방법

1. EC2에 접속해서 명령어로 개별 프로그램 설치
1. EC2 생성단계에서 User Data 스크립트 추가하여 자동 설치
1. LAMP 웹 서버가 설치된 EC2를 AMI로 저장한 후 필요할 때마다 생성

---

1. LAMP 서버 설치 및 테스트   
    1. EC2 생성 시 User Data 스크립트 추가하여 자동으로 설치   
        1. EC2 서비스 선택   
        1. `Launch Instance` 선택   
        1. `Amazon Linux 2 AMI (HVM), SSD Volume Type` 선택
        1. `t2.micro` 선택
        1. `Next: Configure instance Details` 선택
            1. Subnet : `ap-northeast-2a` 선택   
            1. Advanced Details
                1. User data 입력
                    ```bash
                    #!/bin/bash
                    yum update -y
                    amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
                    yum install -y httpd mariadb-server
                    systemctl start httpd
                    systemctl enable httpd
                    usermod -a -G apache ec2-user
                    chown -R ec2-user:apache /var/www
                    chmod 2775 /var/www
                    find /var/www -type d -exec chmod 2775 {} \;
                    find /var/www -type f -exec chmod 0664 {} \;
                    echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
                    if [ ! -f /var/www/html/bootcamp-app.tar.gz ]; then
                    cd /var/www/html
                    wget https://s3.amazonaws.com/immersionday-labs/bootcamp-app.tar
                    tar xvf bootcamp-app.tar
                    chown apache:root /var/www/html/rds.conf.php
                    wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
                    mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
                    cd /var/www/html/phpMyAdmin/
                    cp config.sample.inc.php config.inc.php
                    fi
                    ```
        1. `Next: Add Storage` 선택
        1. `Next: Add Tags` 선택
            1. `Add tag` 선택
                1. Key : Name 입력
                1. Value : lab-web-pub-2a 입력
        1. `Next: Configure Security Group` 선택
            1. `Create a new security group` 선택
            1. security group name : lab-web-sg 입력
            1. `Add Rull` 선택
                1. HTTP 선택
                1. Source : Custom 0.0.0.0/0
        1. `Review And Launch` 선택
        1. `Launch` 선택
        1. key pair 생성
            1. `Create a new key pair` 선택
            1. key pair name : seoul-lab-web
            1. `Download Key Pair` 선택
        1. `Launch Instances` 선택
    1. LAMP 서버 테스트
        1. Putty 다운로드 : https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html 에서 `putty.zip` 다운로드
        1. 압축 해제
        1. pem > ppk 변환
            1. `PUTTYGEN.EXE` 실행
            1. `Load` 선택
            1. `seoul-lab-web.pem` 열기
            1. `Save private key` 선택 > `예` 선택
            1. `seoul-lab-web.ppk`로 저장
    1. `PUTTY.EXE` 실행
        1. Host Name : AWS Instance의 IPv4 Public IP 입력
        1. `SSH` 선택 > `Auth` 선택 > `Browse...` 선택
        1. `seoul-lab-web.ppk` 열기
        1. `Open` 선택 > `Accept` 선택
    1. login as: 접속
        1. ec2-user 입력 > 접속 확인
    1. Browser 실행
    1. 주소창에 AWS Instance의 IPv4 Public IP 입력 > 접속 확인
1. Custom AMI 생성
    1. 인스턴스 선택
    1. `Actions` 선택
    1. `Image & Template` > `Create Image` 선택
        1. Image name : lab-web-20210605 입력
        1. Image description : lab-web-20210605
        1. No reboot 체크
        1. `Create Image` 선택
    1. Images > `AMI` 선택 : AMI 생성 확인
1. Custom AMI로 두번째 LAMP 서버 생성
    1. Images > `AMI` 선택
    1. AMI 선택
    1. `Launch` 선택
    1. `t2.micro` 선택
    1. `Next: Configure instance Details` 선택
        1. Network : `Default` 선택
        1. Subnet : `Default in ap-northeast-2c` 선택
    1. `Next: Add Storage` 선택
    1. `Next: Add Tags` 선택
        1. Key : Name 입력
        1. Value : lab-web-pub-2c 입력
    1. `Next: Configure Security Group` 선택
        1. `Select existing security group` 선택
        1. `lab-web-sg` 선택
    1. `Review And Launch` 선택
    1. `Launch` 선택
    1. key pair 선택
        1. `Choose an exsiting key pair` 선택
        1. `seoul-lab-web` 선택
    1. `Launch Instances` 선택
1. PuTTY로 SSH 접속하여 데이터베이스 보안 설정
    1. ?

### Application Load Balancer 시작하기

1. Load Balancer 유형 선택
    1. Load Balancing > `Load Balancers` 선택
    1. `Create Load Balancer` 선택
    1. Application Load Balancer 의 `Create` 선택
        1. Name : lab-web-alb
        1. scheme : internet-facing 선택
        1. Availability Zones
            1. `ap-northeast-2a`, `ap-northeast-2c` 체크
        1. Tags
            1. Key : Name
            1. Value : lab-web-alb
    1. `Next: Configure Security Settings` 선택
    1. `Next: Configure Security Groups` 선택
        1. `Create a new security group` 선택
        1. security group name : lab-web-alb-sg 입력
            1. Source : Custom 0.0.0.0/0 입력
    1. `Next: Configure Routing` 선택
        1. Target group : New target group 선택
        1. Name : lab-web-alb-tg 입력
    1. `Next: Register Targets` 선택
        1. Instances에서 EC2 선택
        1. `Add to registered` 선택
    1. `Next: Review` 선택
    1. `Create` 선택

## 참고
- [https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard]