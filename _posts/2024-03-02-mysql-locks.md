---
title: MySQL 락 실습해보기
author: gyuhwan
date: 2024-03-03 13:40:00 +0800
categories: [DB, MySQL]
tags: [Lock]
---
# 1. 레코드 락(Record Lock)

레코드 락(Record Lock)이란, InnoDB 스토리지 엔진 영역에서 적용되는 락이며 인덱스 레코드를 락을 거는 것을 의미한다. 인덱스 레코드를 락을 건다는 의미는 ‘**인덱스를 통해서 접근하는 레코드’**에 락을 건다는 뜻이다. 만약 **인덱스가 없다면** InnoDB에서는 자동으로 6 byte 크기의 시스템 컬럼인 ‘hidden crusted index’를 생성해서 관리하게 된다.

그러면 이 레코드 락에 대해 실습을 통해 확인해보자. 

## Schema

```sql
mysql> CREATE TABLE member(
  id INT AUTO_INCREMENT PRIMARY KEY,
  last_name VARCHAR(50),
  first_name VARCHAR(50),
  register_date DATE
);

mysql> CREATE INDEX idx_last_name ON member(last_name);

INSERT INTO member (last_name, first_name, register_date) VALUES 
                  ('ZUser1', 'Name_1', CURDATE()),
                  ('ZUser2', 'Name_2', CURDATE()),
                  ('ZUser3', 'Name_3', CURDATE()),
                  ('ZUser4', 'Name_4', CURDATE()),
                  ('ZUser5', 'Name_5', CURDATE()),
                  ('ZUser6', 'Name_6', CURDATE()),
                  ('ZUser7', 'Name_7', CURDATE()),
                  ('ZUser8', 'Name_8', CURDATE()),
                  ('ZUser9', 'Name_9', CURDATE()),
                  ('ZUser10', 'Name_10', CURDATE()),
                  ('ZUser11', 'Name_11', CURDATE()),
                  ('ZUser12', 'Name_12', CURDATE()),
                  ('ZUser299', 'Name_13', CURDATE()),
                  ('ZUser300', 'ONLY_ONE', CURDATE());
```

생성된 Member 테이블과 삽입된 데이터를 확인해보면 `last_name`으로 인덱스를 생성했다. 그리고 `last_name` 이 모두 **Z**로 시작된다는 것을 확인할 수 있다. 이제 두 개의 **cmd** 창을 열고 각각 다른 쿼리를 사용해보자.

`

## 1) 인덱스 없는 컬럼으로 조건 검색

```sql
mysql> START TRANSACTION;
mysql> UPDATE member SET register_date = NOW() WHERE first_name = 'ONLY_ONE'
```

실행되는 쿼리는 `UPDATE member SET register_date = NOW() WHERE first_name = 'ONLY_ONE'` 이다. 검색 조건으로 인덱스가 없는 `first_name`을 사용한 것을 확인할 수 있는데. 과연 실제로는 어떤 레코드가 락이 걸릴까?

### 락 확인

![lock1](/commons/mysql_lock/lock1.png)

`SELECT * FROM performance_schema.data_locks\g` 명령어로 현재 락 테이블을 확인할 수 있는데. 테이블의 자세한 정보는 [lock_data_table](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-locks-table.html)에 자세히 나와있다. 

* `SELECT * FROM performance_schema.data_locks WHERE OBJECT_SCHEMA = 'your_database_name' AND OBJECT_NAME = 'member';` 쿼리를 통해 특정 데이터베이스와 테이블을 명시할 수 있다.

몇 가지만 살펴보자면 **LOCK_TYPE**을 통해 어떤 락을 걸었는지 확인이 가능하고 **LOCK_MODE**를 통해서 어떤 잠금 모드를 사용했는지 알 수 있다. **X(exclusive lock)**, **S(shared lock)** 을 의미한다. 보통 UPDATE, DELETE와 같은 작업은 X 락이 사용된다. 테이블을 살펴보면 전체 테이블에 X 락을 건 것을 확인할 수 있다. 

실제로 다른 **cmd** 창에서 데이터를 조회하거나 업데이트 하려고 하면 대기상태로 빠지고 결과적으로 **타임아웃 초과 에러**가 발생하는 것을 확인할 수 있다. 

![lock2](/commons/mysql_lock/lock2.png)

- 단순 `SELECT * FROM member` 는 잘 조회된다.

## 2) 인덱스 있는 컬럼으로 조건 검색

```sql
mysql> start trasaction;
mysql> UPDATE member SET register_date = NOW() WHERE last_name = 'ZUser1'
```

이번에 실행되는 쿼리는 `UPDATE member SET register_date = NOW() WHERE last_name = 'ZUser1'`이다. 인덱스가 있는 컬럼을 검색 조건으로 사용했다. 

### 락 확인

![lock3](/commons/mysql_lock/lock3_real.png)

락을 확인해보면 해당 레코드에 락이 걸린 것을 확인할 수 있다. 실제로 다른 칼럼들을 조회해보면 다 조회가 가능한 것을 확인할 수 있다. 

![lock4](/commons/mysql_lock/lock4.png)

## 3) 인덱스있는 컬럼과 없는 컬럼 혼합하여 조건 검색

```sql
mysql> UPDATE member SET register_date=NOW() WHERE last_name like 'Z%' AND first_name='ONLY_ONE';
```

이번엔 인덱스가 있는 컬럼과 없는 컬럼을 검색 조건으로 하여 검색해보자. 실제 검색 조건에 해당하는 데이터는 1건이다. 그런데 **인덱스를 통해 Z로 시작되는 모든 데이터를 조회**하게 된다. 

### 락 확인

![lock6](/commons/mysql_lock/lock6.png)

확인해보면 모든 컬럼에 X 락이 걸린 것을 확인하게 된다. 여기서 레코드 락은 인덱스를 통해 검색되는 레코드를 모두 락을 건다는 것을 확인할 수 있다.

## 4) 결론

레코드 락(Record Lock)이란, 특정 데이터를 다른 트랜잭션의 접근으로부터 보호하기 위해 사용되는 기능이다. 이를 통해 동시성 제어와 데이터 무결성을 유지할 수 있다. 레코드 락은 특정 레코드를 베타적으로 잠그는 것을 의미하며, 다른 트랜잭션은 해당 레코드를 수정할 수 없다. 

**인덱스 레코드를 락을 거는 이유**

- 인덱스를 통해 검색되는 레코드를 락을 걸면 다른 트랜잭션에서 다른 레코드에 대한 접근에 영향을 주지 않을 수 있습니다.
- 만약 인덱스를 통하지 않고 전체 테이블을 락으로 걸게 되면 다른 레코드도 접근할 수 없게 됩니다.

잘못된 인덱스 설정은 오히려 성능을 악화시킬 수 있다. 그러니 중요한 것은 인덱스를 잘 생성하는 것이다. 적절한 필드를 인덱스로 설정하고 인덱스의 크기와 유형을 고려하여 데이터베이스 성능 최적화하는 것이 중요하다.

# 2. 갭 락(Gap Lock) & 넥스트 키 락(**Next Key Lock)**

**Gap Lock**은 인덱스 레코드 사이의 갭을 잠금하는 것을 의미하는데. 예를 들어, 1 ~ 10까지 인덱스 범위를 검색한다고 했을 때, 인덱스에서 이 범위 첫 번째 또는 마지막 인덱스 레코드 후의 갭에 대한 잠금이다. MySQL에서 Gap Lock은 **팬텀 리드와 같은 현상을 방지**하는 데 중요한 역할을 한다.

팬텀 리드는 **REPEATABLE READ**와 같은 격리 수준에서 발생하며 이를 방지하기 위해 Gap Lock이 사용된다. 만약 격리 수준을 **READ COMMITED** 로 설정하면 Gap Lock은 비활성화된다.

## Schema

```sql
mysql> create table human(
		id INT AUTO_INCREMENT PRIMARY KEY,
		name VARCHAR(20),
	  age INT
);

mysql> CREATE INDEX idx_age ON human(age);

mysql> INSERT INTO human(name, age) VALUES
	 ('name1', 10),('name2', 12), ('name3', 14), ('name4', 20), ('name5', 25);
```

`SELECT age FROM human WHERE age BETWEEN 10 and 20 FOR UPDATE` 스키마를 설정하고 이 쿼리를 실행하고 잠금 상태를 확인해보자. 

![lock7](/commons/mysql_lock/lock7.png)

실제로 10과 20 사이에 INSERT를 하려고하면 X락이 걸려있기 때문에 대기 상태에 빠지게 된다. 예를 들어 15와 같은 값을 삽입하면 **타임아웃 예외**가 발생한다.

![lock8](/commons/mysql_lock/lock8.png)

**넥스트 키 락(Next Key Lock)** 은 **갭 락(Gap Lock)** 과 **레코드 락(Record Lock)** 을 합친 것으로 특정 레코드와 그 레코드 앞의 갭에 대한 락을 동시에 설정한다. 이게 무슨 말인가 하면

- 갭 락: 10과 20 사이에 새로운 레코드를 삽입하는 것을 막는다.
- 넥스트 키 락: 10과 그 앞의 갭과 20과 뒤의 갭에 새로운 레코드를 삽입하는 것을 막는다.

실제로 age 값이 9와 21을 넣으려고 하면 **타임아웃 예외**가 발생한다.

참고>   
[MySQL docs](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-record-locks)

[MySQL data_locks](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-locks-table.html)