---
title: (JPA)영속성 컨택스트 실제 쿼리 확인하기
author: gyuhwan
date: 2024-05-21 14:43:00 +0800
categories: [JPA]
tags: [JPA]
---

이번은 트랜잭션별로 영속성 컨택스트로 관리중인 엔티티들의 실제 쿼리가 어떻게 나가는지 확인해보는 시간을 가졌다. 

## 1. 단순 Repository Save 쿼리 후 findById 사용시 쿼리

```java
  @DisplayName("Basic Insert Test")
  @Test
  void basicInsertTest() {
    Temp temp = Temp.of("first");
    Temp saveTemp = repository.save(temp);
    Temp temp1 = repository.findById(saveTemp.getId()).get();
  }
```

### **실제 쿼리**

- insert 쿼리 하나만 나가게 된다. 처음에 save를 하면서 영속성 컨텍스트에 저장하기 떄문인 것같다.

```java
Hibernate: 
    insert 
    into
        temp
        (name,id) 
    values
        (?,default)

```

## 2. 부모 트랜잭션(O) + 자식 트랜잭션(X) with findByName

```java
  @Transactional
  public boolean question1() {
    String name = "hong";
    Temp temp = Temp.of(name);
    repository.save(temp); // (1)
    question1method(name); 
    return true;
  }

  @Transactional
  public boolean question1method(String name) {
    Temp temp = repository.findByName(name) // (2)
												    .orElseThrow(EntityNotFoundException::new);
    temp.changeName("pot"); // (3)
    return true;
  }
```

두 메서드는 트랜잭션 어노테이션이 달려있다. 각각 save, findByName, changeName이 진행되는데. 어느 순서에 어떤 쿼리가 나가게 될까? 예상가는 범위는 1,2,3으로 표시 해놨다. 과연 맞을까?

### 실제 쿼리

```java
Hibernate: insert into temp (name) values (?)
Hibernate: select t1_0.id,t1_0.name from temp t1_0 where t1_0.name=?
Hibernate: update temp set name=? where id=?
```

실제 사용되는 쿼리는 insert, select, update 그대로 진행됐다. save를 하는 순간 영속성 컨택스트에 저장되기 때문에 select 이 안되지 않을까 했는데. 아니었다. 모두 진행됐다. 그리고 changeName을 할 때 update 쿼리가 나갈거라 생각했는데 아니었다. 정리하자면

- save로 저장했을 때, insert query가 나간다.
- **save를 통해 영속성 컨텍스트에 저장해도 findByName을 사용하면 select 쿼리가 나가게 된다.**
- **트랜잭션이 최종적으로 commit 될 때, update 쿼리가 나가게 된다. (3) 에는 나가지 않는다.**

## 3. 부모 트랜잭션(O) + 자식 트랜잭션(X) with findById

```java
  @Transactional
  public boolean question2() {
    String name = "hong";
    Temp temp = Temp.of(name);
    Temp save = repository.save(temp);
    question2method(save.getId());
    return true;
  }

  @Transactional
  public boolean question2method(Long id) {
    Temp temp = repository.findById(id).orElseThrow(EntityNotFoundException::new);
    temp.changeName("pot");
    return true;
  }
```

### 실제 쿼리

```java
Hibernate: insert into temp (name) values (?)
Hibernate: update temp set name=? where id=?
```

findById를 사용하여 조회할 경우, 영속성 컨텍스트에서 가져오게 된다.

## 4. 부모 트랜잭션(O) + 자식 트랜잭션(X) with findByName

```java
@Transactional
public boolean question3() {
String name = "hong";
Temp temp = Temp.of(name);
repository.save(temp); 
question3method(name);
return true;
}

public boolean question3method(String name) {
Temp temp = repository.findByName(name).orElseThrow(EntityNotFoundException::new);
temp.changeName("pot");
return true;
}
```

이번에는 트랜잭션이 상위에 하나만 존재할 경우이다. 공부하기로는 하위 트랜잭션까지 전파되지 않는다고 알고 있다. 실제 쿼리는 어떤 변화가 있을까?

### 실제 쿼리

```java
Hibernate: insert into temp (name) values (?)
Hibernate: select t1_0.id,t1_0.name from temp t1_0 where t1_0.name=?
Hibernate: update temp set name=? where id=?
```

쿼리는 [트랜잭션 + 트랜잭션](https://www.notion.so/1881f1951a554341bf561d080f0ff3f7?pvs=21)과 동일하게 나가게 된다. 동일한 위치에 나간다.

## 5. 부모 트랜잭션(O) + 자식 트랜잭션(O) with Requires_New

```java
@Transactional
public boolean question4() {
  String name = "hong";
  Temp temp = Temp.of(name);
  repository.save(temp);
  question4method(name);
  return true;
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public boolean question4method(String name) {
  Temp temp = repository.findByName(name).orElseThrow(EntityNotFoundException::new);
  temp.changeName("pot");
  return true;
}
```

REQUIRES_NEW 는 항상 새로운 트랜잭션을 생성한다. 

### 실제 쿼리

```java
Hibernate: insert into temp (name) values (?)
Hibernate: select t1_0.id,t1_0.name from temp t1_0 where t1_0.name=?
Hibernate: update temp set name=? where id=?
```

이번에도 이전과 동일한 위치에 쿼리가 발생한다. 

## 6. 부모 트랜잭션(O) + 자식 트랜잭션(O) with Requires_new + findById

```java
@Transactional
public boolean question5() {
  String name = "hong";
  Temp temp = Temp.of(name);
  Temp save = repository.save(temp);
  question5method(save.getId());
  return true;
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public boolean question5method(Long id) {
  Temp temp = repository.findById(id).orElseThrow(EntityNotFoundException::new);
  temp.changeName("pot");
  return true;
}
```

### 실제 쿼리

```java
Hibernate: insert into temp (name) values (?)
Hibernate: update temp set name=? where id=?
```

아무래도 새로운 트랜잭션으로 분리한다고 해도 기존 트랜잭션을 이어가서 하는것과 동일하게 작동한다.

## 8.  부모 트랜잭션(X) + 자식 트랜잭션(X)

```java
public boolean question6() {
  String name = "hong";
  Temp temp = Temp.of(name);
  repository.save(temp);
  question6method(name);
  return true;
}

public boolean question6method(String name) {
  Temp temp = repository.findByName(name).orElseThrow(EntityNotFoundException::new);
  temp.changeName("pot");
  return true;
}
```

이번에는 트랜잭션이 모두 없는 상황이다. 

### 실제 쿼리

```java
Hibernate: insert into temp (name) values (?)
Hibernate: select t1_0.id,t1_0.name from temp t1_0 where t1_0.name=?
```

엔티티가 영속성 컨택스트 안에 존재하지 않기 때문에 더티 체킹이 불가능하다. 그렇기 때문에 엔티티를 수정해도 변경 사항이 반영되지 않는다.

## 8.  부모 트랜잭션(X) + 자식 트랜잭션(O)

```java
  public boolean question7() {
    String name = "hong";
    Temp temp = Temp.of(name);
    repository.save(temp);
    question7method(name);
    return true;
  }

  @Transactional
  public boolean question7method(String name) {
    Temp temp = repository.findByName(name).orElseThrow(EntityNotFoundException::new);
    temp.changeName("pot");
    return true;
  }
```

상위 메서드에는 트랜잭션이 없고 하위 메서드에는 트랜잭션이 있따. 이럴 경우, 더티 체킹이 이루어질까?

### 실제 쿼리

```java
Hibernate: insert into temp (name) values (?)
Hibernate: select t1_0.id,t1_0.name from temp t1_0 where t1_0.name=?
```

트랜잭션 어노테이션은 AOP를 사용해서 프록시 객체를 만들어 메서드를 사용하는데. 내부 메서드를 사용할 경우, 자기 참조이기 때문에 트랜잭션이 적용되지 않는다. 그렇기 때문에 변경 감지가 되지 않아 변경 사항은 적용되지 않는다.