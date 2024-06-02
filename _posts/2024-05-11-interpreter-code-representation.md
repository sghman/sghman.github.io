---
title: 3. 인터프리터 구현 - 코드 표현
author: gyuhwan
date: 2024-05-27 11:40:00 +0800
categories: [Interpreter, Java]
tags: [Interpreter]
---

스캐너로 원시 코드를 토큰으로 만들었으면 이 토큰을 표현(representation)으로 만들 차례다. 코틀린은 `var time = 10,` 자바는 `int num = 10;` 처럼 말이다. 

자바에서 `num int = 10;` 는 허용하지 않는 문법이다. 먼저 **타입**이 나오고 **식별자**, **‘=’**, **값**, **‘;’** 이 나와야 한다. 이처럼 각 언어는 자신만의 표현식이 있다.  이번 장의 핵심 목표는 표현식의 정의다.

## 컨텍스트 자유 문법

표현식을 정의하기 앞서, 스캐너는 문자들을 토큰으로 묶는 데. 이러한 규칙을 정규 언어(regular language)라고 한다. 그런데 이러한 정규 언어는 중첩한 표현식을 처리하기 쉽지 않다. ex) `4 * (2 + 1 / (3 - 2)))`

중첩되어 있으면 가장 안쪽의 괄호부터 밖의 괄호 범위로 확장하며 처리해야 하는데. 이러한 표현식을 처리하기 위한 방법으로 **컨텍스트 자유 문법(Context-Free Grammar)**가 있다. 컨텍스트 자유 문법은 복잡한 언어 구조를 분석하고 이해하는 데 필수적인 도구다.  컨텍스트 자유 문법에 대해 다음의 예시와 함께 살펴보자. 

```
[Header] → [Body]

breakfast → protein “with” breakfast “on the side”;
breakfast → protein;
breakfast → bread;

protein → crispiness “crispy” “bacon”;
protein → “sausage”;
protein → cooked “eggs”;

crispiness → “really”;
crispiness → “really” crispiness;

cooked → scrambled”;
cooked → poached”;
cooked → fried”;

bread → “toast”;
bread → “biscuits”;
bread → “English muffin”;
```

처음에는 이게 뭔가 싶을 수 있는데. 의외로 간단하다. 모든 문장은 **Header** → **Body** 형식으로 이루어진다. 간단하게 용어를 살펴보자.

**넌터미널(nonterminal)** : ‘”’ 에 포함되지 않는 부분이며, 각각을 문법으로 표현하며 다른 문법 규칙을 가리키는 기명 참조(named reference)이다. 예를 들어, 처음의 세 번째 ‘**breakfast** → **bread**;’ 에 ‘bread’ 부분에는 제일 아래에 있는 “toast”, “biscuits”, “English muffin”을 삽입할 수 있다. 

**터미널(terminal)** :  리터럴 값으로 보면 되고 스캐너가 내보내는 개별 렉심, 즉 **토큰**을 의미한다.

자유 문법을 통해 하나의 예시를 만들어보자. 

‘**breakfast** → **protein** “with” **breakfast** “on the side” →  

‘”sausage” “with” **breakfast** “on the side”’→ 

‘“sausage” “with” “toast” “on the side”’ 최종적으로 이러한 식이 만들어지는데. 이런 식으로 유한한 문법으로 정말 다양한 표현식을 만들 수 있게 된다. 

### 표기법 개선

조금 장황했던 표기법을 몇 가지 규칙을 추가해서 깔끔하게 표현할 수 있다. 헤더를 중복해서 나열하는 것보다. ‘|
’를 사용해서 여러 옵션 중에 한 가지를 선택할 수 있게 한다. 정규표현식을 생각하면 이해하기 쉽다.

**bread** → “toast” | “biscuits” | “English muffin”;

반복을 뜻하는 ‘*’ 를 사용하여 최소 0회 이상 반복을 표현한다.

**crispiness** → “really” “really”*;

접미어 ‘+’는 적어도 한 번 이상 나와야 한다.

**crispiness** → “really”+ ;

접미어 ‘?’ 는 선택적인 프로덕션을 의미한다. 한 번도 나오지 않거나 한 번만 나올 수 있다. 

**breakfast** → **protein** (”with” **breakfast** “on th side” )? ;

아래는 최종 정리 형태이다. 

```
breakfast -> protein ("with" breakfast "on the side" )? 
							| bread ;
protein -> "really" + "crispy" "bacon" | "sausage"
						| ( "scrambled" | "poached" | "fried" ) "eggs" ;
bread  -> "toast" | "biscuits" | "English muffin" ;
```

### 록스 표현식 문법

이제 록스 표현식 문법을 컨텍스트 자유 문법으로 표현해보자.

```
expression -> literal | unary | binary | grouping;
literal -> NUMBER | String | "true" | "false" | "nil";
grouping -> "(" expression ")";
unary -> ("-"|"!") expression;
binary -> expression operator expresision;
operator -> "==" | "!=" | ">" | ">=" | "<" | "<=" | "+" | "-"| "*" | "/";
```

이제 각 부분을 살펴보자.

- **`expression`**: 표현식은 리터럴, 단항 연산자, 이항 연산자, 그룹화로 구성된다.
- **`literal`**: 리터럴은 숫자, 문자열, 불리언 값, 또는 **`nil`**을 나타내는데. nil은 null 과 같다.
- **`grouping`**: 괄호로 둘러싸인 표현식이다.
- **`unary`**: 단항 연산자로 `-****`또는 **`!`**를 사용할 수 있다.
- **`binary`**: 이항 연산자로 **`==`**, **`!=`**, **`>`**, **`>=`**, **`<`**, **`<=`**, **`+`**, `-****`, `*****`, **`/`**를 사용할 수 있다.

## 표현식 클래스

록스 표현식은 다음과 같은 데이터만 가지고 있는 트리 클래스로 만들어진다. 

```java
abstract class Expr {

 static class Binary extends Expr {
  Binary(Expr left, Token operator, Expr right) {
   this.left = left;
   this.operator = operator;
   this.right = right;
   }
  ...
  
  static class Grouping extends Expr {
    Grouping(Expr expression) {
     this.expression = expression;
    }
  }
  ...
}
```

### 표현식 문제

지금은 단순히 4개의 표현식만 정의했을 뿐이다. 여기서 표현식이 늘어나면 어떻게하면 될까? 단순히 Expr을 확장하여 클래스를 생성하면 된다. 그러면 각각 공통된 메서드가 있다고 해보자. `interpret()`, `resolve()`, `analyze()` 와 같이 공통적으로 필요한 메서드를 만들었다. 

그런데 여기서 메서드를 추가하고 싶으면 어떡해야할까? 먼저 Expr에 추상 메서드를 추가할 것이고 각 클래스는 컴파일 에러가 발생한다. 그러면 하나 하나 메서드를 구현하면 될 것이다. 간단한 방법이지만 클래스가 많다면 상당히 손이 많이간다는 것은 부정할 수 없다.(확장성)

새로운 클래스와 메서드 확장을 해결하기 위한 패턴이 있는데. 바로 **비지터 패턴(visitor pattern)**이다.

### 비지터 패턴(visitor pattern)

이해를 돕기위해 네트워크 장비에 대한 모델명 설정을 예로 들겠다. 장비는 처음에는 라우터, 스위치가 있다. 부모 인터페이스로 NetworkDevice가 있고  공통적으로 accept 메서드를 가지고 있다.

```java
public interface NetworkDevice {

  void setModel(String name);

  void accept(Visitor visitor);
}

class Router implements NetworkDevice {

  @Override
  public void setModel(String name) {
    System.out.println("this model : " + name);
  }

  @Override
  public void accept(Visitor visitor) {
    visitor.visit(this);
  }
}

class Switch implements NetworkDevice {

  @Override
  public void setModel(String name) {
    System.out.println("this model : " + name);
  }

  @Override
  public void accept(Visitor visitor) {
    visitor.visit(this);
  }
}
```

accept 메서드는 Visitor 인터페이스를 인수로 가지는데. 새로운 처리가 필요할 때마다 이 Visitor 메서드를 새롭게 만들면 된다. 코드는 아래와 같다.

```java
interface Visitor {

  void visit(Router router);

  void visit(Switch switch);
}

// 이름을 설정하는 비지터
class SetNameVisitor implements Visitor {

  @Override
  public void visit(Router router) {
    router.setModel("router v1");
  }

  @Override
  public void visit(Switch swit) {
    swit.setModel("switch v1");
  }
}
```

그런데 방화벽을 추가해야 할 일이 생겼다. 그러면 어떡해야 할까? 간단하다. 방화벽 클래스를 만들고 비지터 인터페이스에 추가하면 된다.

```java
// 1) 방화벽 클래스 생성
class FireWall implements NetworkDevice{

  @Override
  public void setModel(String name) {
    System.out.println("this model : " + name);
  }

  @Override
  public void accept(Visitor visitor) {
   visitor.visit(this);
  }
}

// 2) 비지터 인터페이스에 메서드 추가
interface Visitor {

  void visit(Router router);

  void visit(Switch swit);

  void visit(FireWall fireWall);
}

// 3) 비지터 구현체에 방화벽 전용 메서드 구현
class SetNameVisitor implements Visitor {
	....

	// 추가
  @Override
  public void visit(FireWall fireWall) {
    fireWall.setModel("firewall v1");
  }
}
```