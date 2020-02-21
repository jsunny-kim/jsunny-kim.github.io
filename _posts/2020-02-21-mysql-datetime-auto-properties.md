---
title:  "MySQL TIMESTAMP, DATETIME 타입 DEFAULT, ON UPDATE 속성 설정"
category: posts
tags:
  - MySQL
comments: true
last_modified_at: 2020-02-21T16:50:00
---

> [MySQL 8.0 Reference Manual - Automatic Initialization and Updating for TIMESTAMP and DATETIME](https://dev.mysql.com/doc/refman/8.0/en/timestamp-initialization.html) 참고

데이터 타입이 날짜( `TIMESTAMP`, `DATETIME`) 인 경우, 자동으로 초기화 및 업데이트 속성을 지정할 수 있다.

해당 속성은 `DEFAULT CURRENT_TIMESTAMP`, `ON UPDATE CURRENT_TIMESTAMP` 구문을 통해서 지정된다. 여기서 **DEFAULT** 구문은 데이터 초기 값(**INSERT**) 속성을 의미하고, **ON UPDATE** 구문은 자동 업데이트 값(**UPDATE**)에 대한 속성을 의미한다. 각 구문의 순서는 중요하지 않다. 지정할 수 있는 자동 업데이트 속성으로는 `CURRENT_TIMESTAMP`, `NOW`, `LOCALTIME`, `LOCALTIMESTAMP` 가 있다.

```sql
CREATE TABLE t1 (
  ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  dt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

이때 데이터 타입이 [밀리초 속성이 부여된 날짜 타입](https://dev.mysql.com/doc/refman/8.0/en/fractional-seconds.html)(`type_name(fsp)`)인 경우에는 해당 속성이 데이터 타입과 일치해야 한다. 가령 데이터 타입이 **TIMESTAMP(6)** 인 경우, 자동 속성도 **CURRENT_TIMESTAMP(6)** 이어야 한다. 

```sql
CREATE TABLE t1 (
  ts TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6)
);
```

```sql
// 잘못된 쿼리
CREATE TABLE t1 (
  ts TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP(3)
);
```

SQL 쿼리는 학교에서 분명 열심히 배운 것 같지만, 정작 필요할 때는 정확히 기억나지 않아 검색해보는 경우가 많은 것 같다. 그렇게 다시 찾아온 반성의 시간🤔
