---
layout: posts
title: mysql 'access denied for user ...' 오류 해결
categories: 
  - mysql
tags: 
  - mysql
  - error
---
# Mysql “Access denied for user ‘root’@’localhost'” 오류 해결하기
- root의 권한을 수정하여 해결
```
$ sudo mysql -u root
$ use mysql;
$ grant all on *.* to ‘root’@‘localhost’ identified by ‘root’ with grant option;
$ flush privileges;
$ exit;
```