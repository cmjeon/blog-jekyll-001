---
layout: single
title: "redis 기본 명령어 모음"
categories:
  - "redis"
---

## Database

### database 선택하기

database 번호는 0 ~ 16 이 기본입니다.

기본값은 0 이고, 다른 번호를 선택하면 명령어 창에  해당 데이터베이스 번호가 표시됩니다.

```
> select { DATABASE_NUMBER }

# 다른 DATABASE_NUMBER 를 선택한 경우
172.0.0.1:6379> select 12
OK
172.0.0.1:6379[12]>
```

### database 에 저장된 키 개수 확인하기

```
> DBSIZE
```

## key 저장

### key 저장하기

```
> set { KEY } { VALUE }
```

### key 에 만료시간(초)을 정하여 저장하기

```
> setex { KEY } { SECOND } { VALUE }
```

## key 조회

### keys 확인하기

적재되어있는 모든 키를 확인하는 방법입니다.

```
> keys *
```

### key 의 값을 확인하기

```
> get { KEY }
```

### 여러 키의 값을 확인하기

```
> mget { KEY1 } { KEY2 } ...
```

### key 를 패턴으로 조회하기

```
> get *{ KEY_PATTERN }*
```

### key 의 유효시간 확인하기

```
> ttl { KEY }
```

## key 변경

### key 이름 바꾸기

```
> rename { KEY } { RENAMED_KEY }
```

### key 의 값 바꾸기

```
> set { KEY } { ANOTHER_VALUE }
```

## key 삭제

### key 삭제하기

```
> del { KEY }
```

### 모든 데이터 삭제

```
> flushall
```

## 참고

- [https://redis.io/commands/?group=string](https://redis.io/commands/?group=string)
- [https://tgyun615.com/192](https://tgyun615.com/192)
