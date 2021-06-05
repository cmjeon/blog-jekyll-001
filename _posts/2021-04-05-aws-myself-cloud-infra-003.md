---
layout: single
title:  스스로 구축하는 AWS 클라우드 인프라 - 기본편 003
categories: 
  - learning aws cloud infra
tags: 
  - aws
  - cloud
  - infra
---

## VPC와 중계 서버(Bastion) 구성하기

### Custom VPC-Subnet 생성하기

#### VPC 생성하기

1. VPC 서비스 선택
1. `Your VPCs` 선택
    1. `Create VPC` 선택
        1. Name tag : lab-vpc 선택
        1. IPv4 CIDR 블록 : 10.0.0.0/16 입력
        1. `Create` 선택


#### Subnet 생성하기

- 2개의 public, 4개의 private 생성

1. `Subnets` 선택
    1. `Create subnet` 선택
        1. VPC : lab-vpc 선택
        1. Name tag : lab-web-pub1-2a 입력
        1. Availability Zone : ap-northeast-2a 선택
        1. IPv4 CIDR block : 10.0.1.0/24 입력
        1. `Create` 선택
    1. `Create subnet` 선택
        1. VPC : lab-vpc 선택
        1. Name tag : lab-web-pub2-2c 입력
        1. Availability Zone : ap-northeast-2c 선택
        1. IPv4 CIDR block : 10.0.2.0/24 입력
        1. `Create` 선택
    1. `Create subnet` 선택
        1. VPC : lab-vpc 선택
        1. Name tag : lab-web-pri1-2a 입력
        1. Availability Zone : ap-northeast-2a 선택
        1. IPv4 CIDR block : 10.0.3.0/24 입력
        1. `Create` 선택
    1. `Create subnet` 선택
        1. VPC : lab-vpc 선택
        1. Name tag : lab-web-pri2-2c 입력
        1. Availability Zone : ap-northeast-2c 선택
        1. IPv4 CIDR block : 10.0.4.0/24 입력
        1. `Create` 선택
    1. `Create subnet` 선택
        1. VPC : lab-vpc 선택
        1. Name tag : lab-web-pri3-2a 입력
        1. Availability Zone : ap-northeast-2a 선택
        1. IPv4 CIDR block : 10.0.5.0/24 입력
        1. `Create` 선택
    1. `Create subnet` 선택
        1. VPC : lab-vpc 선택
        1. Name tag : lab-web-pri4-2c 입력
        1. Availability Zone : ap-northeast-2c 선택
        1. IPv4 CIDR block : 10.0.6.0/24 입력
        1. `Create` 선택

### Internet Gateaway-Routing Table 설정하기

#### Internet Gateway 만들기

1. `Internet Gateway` 선택
    1. `Create Internet gateway` 선택
        1. Name tag : lab-web-igw
        1 `Create internet gateway` 선택
    1. Internet gateway 선택
    1. `Actions` 선택 > `Attach to VPC` 선택
        1. `Available VPCs` 선택
        1. `Attach internet gateway` 선택

#### Routing table-Public 만들기

1. `Route Tables` 선택
    1. VPC 가 연결된 Route Table 선택
    1. Name 편집 : lab-web-rt-pub
    1. 하단 `Routes` 선택
        1. `Edit routes` 선택
            1. `Add route` 선택
                1. Destination : 0.0.0.0/0
                1. Target : Internet gateway
                1. 생성한 Internet gateway 선택
            1. `Save routes` 선택
    1. 하단 `Subnet Associations` 선택
        1. `Edit subnet associations` 선택
            1. pub 서브넷 2개 선택
            1. `Save associations` 선택

#### Routing table-Private 만들기

1. `Route Tables` 선택
    1. `Create route table` 선택
        1. Name tag : lab-web-rt-pri
        1. VPC : lab-vpc 선택
        1. `Create route table` 선택
    1. 하단 `Subnet Associations` 선택
        1. `Edit subnet associations` 선택
            1. pri 서브넷 4개 선택
            1. `Save associations` 선택


### NAT Gateway 구성하기

1. `NAT Gateway` 선택
    1. `Create NAT gateway` 선택
        1. Name : lab-web-nat-2a 입력
        1. Subnet : lab-web-pub1-2a 선택
        1. Elastic IP allocation ID :  `Allocate Elastic IP` 선택
    1. `Create NAT gateway` 선택
        1. Name : lab-web-nat-2c 입력
        1. Subnet : lab-web-pub2-2c 선택
        1. Elastic IP allocation ID :  `Allocate Elastic IP` 선택
1. `Route Tables` 선택
    1. lab-web-rt-pri 선택
    1. Name 편집 : lab-web-rt-pri-2a
    1. 하단 `Subnet Associations` 선택
        1. `Edit subnet associations` 선택
            1. pri-2a 서브넷 2개 선택
            1. `Save associations` 선택
    1. `Create route table` 선택
        1. Name tag : lab-web-rt-pri-2c
        1. VPC : lab-vpc 선택
        1. `Create route table` 선택
    1. 하단 `Subnet Associations` 선택
        1. `Edit subnet associations` 선택
            1. pri-2c 서브넷 2개 선택
            1. `Save associations` 선택

### Bastion Host 생성하기

1. EC2 서비스 선택
1. `AMI` 선택
    1. lab-web-... AMI 선택
    1. `Launch` 선택
        1. `Next: Configure Instance Details` 선택
            1. Network : lab-vpc 선택
            1. Subnet : lab-web-pub1-2a 선택
            1. Auto-assign Public IP : Enable 선택
        1. `Next: Add Storage` 선택
        1. `Next: Add Tags` 선택
            1. Add tag
                1. Key: Name 입력
                1. Value: lab-web-srv-bastion 입력
        1. `Next: Configure Security Group` 선택
            1. Assign a security group : Create a new security group
            1. Security group name : lab-web-bastion-sg
        1. `Review and Launch` 선택
        1. `Launch` 선택
        1. key pair 생성
            1. `Create a new key pair` 선택
            1. key pair name : seoul-lab-web-bastion
            1. `Download Key Pair` 선택
        1. `Launch Instances` 선택
    1. lab-web-... AMI 선택
    1. `Launch` 선택
        1. `Next: Configure Instance Details` 선택
            1. Network : lab-vpc 선택
            1. Subnet : lab-web-pri1-2a 선택
            1. Auto-assign Public IP : Enable 선택
        1. `Next: Add Storage` 선택
        1. `Next: Add Tags` 선택
            1. Add tag
                1. Key: Name 입력
                1. Value: lab-web-srv-pri-2c 입력
        1. `Next: Configure Security Group` 선택
            1. Assign a security group : Create a new security group
            1. Security group name : lab-web-srv-sg
        1. `Review and Launch` 선택
        1. `Launch` 선택
        1. key pair 생성
            1. `Create a new key pair` 선택
            1. key pair name : seoul-lab-web-srv
            1. `Download Key Pair` 선택
        1. `Launch Instances` 선택
1. pem > ppk 변환
    1. `PUTTYGEN.EXE` 실행
    1. `Load` 선택
        1. `seoul-lab-web-bastion.pem` 열기
        1. `Save private key` 선택 > `예` 선택
        1. `seoul-lab-web-bastion.ppk`로 저장
    1. `Load` 선택
        1. `seoul-lab-web-srv.pem` 열기
        1. `Save private key` 선택 > `예` 선택
        1. `seoul-lab-web-srv.ppk`로 저장
1. Bastion 접속
    1. `PUTTY.EXE` 실행
        1. Host Name : Bastion EC2의 IPv4 Public IP 입력
        1. `SSH` 선택 > `Auth` 선택 > `Browse...` 선택
        1. `seoul-lab-web.ppk` 열기
        1. `Open` 선택 > `Accept` 선택
    1. login as: 접속
        1. ec2-user 입력 > 접속 확인
1. Private subnet pem 키 복사하기
    1. seoul-lab-web-srv.pem 내용 복사
    1. putty에 입력
        ```bash
        $ sudo vi seoul-lab-web-srv.pem
        # vi Insert 모드
        `i` 엔터
        # 내용 붙여 넣기
        SHIFT + Insert 입력
        # vi Insert 모드 나가기
        ESE 입력
        # 저장 + 나가기
        `:` 엔터 > `wq` 엔터
        ```
1. Private subnet ec2에 연결하기
    1. putty에 입력
        ```bash
        $ ssh -i seoul-lab-web-srv.pem ec2-user-@{내부 EC2의 Private IPs}
        # Are you sure you want to continue connecting?
        yes 입력
        # Private subnet ec2 에 접속, 외부로 통신불가 확인
        $ ping google.co.kr
        ```
1. VPC 서비스 선택
1. `Route Tables` 선택
    1. lab-web-rt-pri-2a 선택
    1. 하단 `Routes` 선택
        1. `Edit routes` 선택
            1. `Add route` 선택
                1. Destination : 0.0.0.0/0
                1. Target : NAT gateway
                1. 생성한 2a 영역의 NAT gateway 선택
            1. `Save routes` 선택
    1. putty에 입력
        ```bash
        # Private subnet ec2 에 접속, 외부로 통신가능 확인
        $ ping google.co.kr
        ```
    1. lab-web-rt-pri-2c 선택
    1. 하단 `Routes` 선택
        1. `Edit routes` 선택
            1. `Add route` 선택
                1. Destination : 0.0.0.0/0
                1. Target : NAT gateway
                1. 생성한 2c 영역의 NAT gateway 선택
            1. `Save routes` 선택

## 참고
- [https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard](https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard)