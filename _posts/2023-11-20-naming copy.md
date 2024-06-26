---
title: 실제로 적용하는 네이밍 방법
author: gyuhwan
date: 2023-11-11 15:50:00 +0800
categories: [Backend, Tips]
tags: [Naming]
---

# 메서드명 Guide

# 1. 좋은 네이밍이란?

좋은 네이밍이란 무엇일까? 대다수의 사람들이 어떠한 코드를 보고 이름을 잘 지었구나 라고 느낄 수 있으면 그것이 좋은 네이밍이 아닐까? 그럼 어떻게 작성해야 좋은 네이밍이 될까?

여기서 중점을 둬야하는 것은 **대다수의 사람들이 판단했을 때**이다. 대다수의 사람들이 일반적으로 **자주(일반성)** 사용하며 **가독성이 좋으며** 그 **의미가 명확했을 때**, 그리고 **의미대로 메서드가 이루어질 때(정확성)** 좋은 코드가 만들어지는 것이라고 생각한다.

# 2. 객체 생성

객체를 생성하는 일은 자주 이루어진다. 자바를 처음 시작하다 보면 자주 보는 것이 **new** 생성자를 통해서 객체를 생성하는 것이다

```java
Product product = new Product("신라면", 1000);
```

이팩티브 자바나 다른 블로그들의 경우, **new** 를 통한 객체 생성의 단점을 들면서 정적 팩토리 패턴을 이용하는 게 좋다고 한다. 그러면 new 를 통한 객체 생성의 단점을 무엇일까?

## 1) new의 단점

클래스의 필드가 얼마 안된다면 문제가 안될 수 있다. 하지만 필드가 여러 개라면 어떨까?

```java
public class Product{
	private String name;
	private Integer price;
	private String origin;
	private String owner;
	private LocalDateTime insertDt;
	.....

	public Product(String nanme, Integer price, String origin, String ..........){
}
```

필드가 아주 많은 상품 코드이다. 한 번 생성자를 통해 생성하려고 하면 그 안에 들어갈 값들을 다 넣어줘야 한다.

물론 매개변수 몇 개만 받는 생성자를 따로 만들어도 된다. 다만 그렇게 만들어지는 생성자와 다른 생성자의 차이를 알 수 없다.

매개변수 위치마다 타입도 다르다. 결국 클래스를 왔다 갔다 하면서 타입을 확인해야 하고 많은 필드의 유효성 처리까지 한다면 코드가 지저분해 질 수 있다.

또 단순히 new 라는 단어만 사용하기에는 객체를 생성하는 모든 경우를 표현하기가 어렵다.

**정적 팩토리 메서드**를 사용하면 이러한 단점을 모두 해결할 수 있다.

## 2. 정적 팩토리 메서드 패턴

정적 팩터리 메서드란 static 을 사용해서 객체를 생성할 메서드를 만드는 패턴이다. 아래의 한 예를 살펴보자.

```java
public class Product {
	private String name;
	private Integer price;
	private String origin;
	private String owner;
	private LocalDateTime insertDt;
	....
	
	// 기본 생성자 생성 불가
	**private** Product() {}

	**private** Product(String name, Integer price) {
		this.name = name;
		this.price = price;
	}

	public static Product **create**(String name, Integer price) {
		return new Product(name, price);
	}

}
```

기본적으로 꼭 필요한 필드 값을 **name**과 **price** 라고 가정하겠다. 생성자 부분을 **private**로 처리하여 다른 사람들이 오직 **정적 펙토리 메서드만 사용하도록 강제**한다.

그리고 **create** 라는 이름으로 하나의 정적 팩토리 메서드를 만든다.

여기서 정적 팩토리 메서드의 장점이 보여진다. **메서드 명을 통해 의미를 전달 할 수 있다**는 것이다. create 이라는 명을 통해 생성한다는 의미를 알 수 있다. 물론 new로도 그 의미가 전달된다.

하지만 정적 팩터리 패턴을 사용하면 그 의미를 조금 더 다양하게 전달할 수 있다.

또 생성자를 private로 막아두면서 객체를 생성하는 관리 포인트를 정적 팩토리 메서드로 모을 수 있다.

## 3) 종류

시작할 때 말한 것처럼. 좋은 네이밍이란 일반적(범용)으로 사용해야 한다. 가장 좋은 것은 영어 문법에 맞게 사용하는 것이다. 이 밑은 객체를 생성하는 데에 자주 사용하는 네이밍들을 모아 놨다.

### ▷ of

of는 여러 개의 매개변수를 받아서 생성할 때 사용한다. 전치사 **of**의 일반적인 뜻은 ‘**~의**’ 이다. 아래의 예시는 ‘이름은 사용자이고 13000원 짜리의 Item’이다.

```java
Item newItem = Item.of("사용자", 13000); 
```

### ▷ create, newInstance

create, newInstance 는 이름 그대로 새로운 인스턴스를 생성한다는 뜻으로 쓰인다. 쓰임새는 of와 같다.

```java
Item.create("사용자", 13000);
Item.newInstance("사용자", 14000);
```

### ▷ getInstance

getInstance는 이미 만들어진 인스턴스를 가져온다는 것을 의미한다. 굳이 새로운 인스턴스를 생성할 필요가 없을 때 사용하자. 예시로는 싱글톤 패턴에 사용할 수 있다.

```java
Calendar instance = Calendar.getInstance(Local.KOREA);
```

### ▷from

여러 개의 매개변수를 받아서 생성하는 create, of 등과 다르게 from은 하나의 매개변수를 받아서 인스턴스를 반환해준다.

```java
Product product = Product.from("짬뽕");
```

# 3. 검증

무언가를 검증하거나 확인하는 로직은 반드시 필요하다. 비밀키를 검증하거나 회원 정보를 수정할 때, 자신의 비밀번호를 입력해서 검증받거나 하는 상황을 들 수 있다.

이렇게 확인을 하는 다양한 상황에는 어떤 표현이 적합할까? 흔히 많이 쓰는 단어인 **check, confirm, verify, validate(validated)** 를 보려고 한다.

모두 느낌과 뉘앙스는 다르지만 ‘확인하다’라는 의미로 쓰인다. 어떤 차이일까?

### Check

다른 단어에 비해 아주 가벼운 느낌이다. 체크 리스트를 생각하면 된다. 가볍게 확인하다로 쓰일 수 있다.

### Confirm

Confirm은 **생각하는 결론이 이미 정해진 상태**이고 **그것을 확인**한다는 느낌이 더 강하다. 예를 들어 ‘오늘 7시에 보기로 했는데. **맞지**?’ 와 같다.

또 다른 예시로 이메일이 기억이 안나서 찾는다고 해보자. 이때 이메일 찾기를 위해 기존 비밀번호를 입력 받는 상황에서 비밀번호를 입력하고 버튼을 누를 때, 버튼은 ‘**Confirm’**으로 명하는게 좀 더 어울리는 표현이다.

### Verify

verify 는 confirm과 비슷하다. confirm 과의 차이는 어떤 것이 사실이라는 것을 증명하거나 진짜라는 것을 보여주는 의미이다. 예를 들어 ‘은행 계좌를 새로 만들려면 당신의 신원을 **확인**해야 합니다. ‘ 할 때의 확인에 적합하다.

### Validate(validated)

네 가지의 확인하다 중에 Validate는 가장 무거운 느낌이다. 공식적인 인증을 받은 것으로 표현할 수 있다. validated는 공식적으로 인증 받았다는 뜻으로 볼 수 있다.

토큰을 검증하고 인증 받았을 때, Validated 한 상태라고 볼 수 있다.

# 4. Bool

bool의 반환 값은 true, false 이다. 무엇을 가지고 있는지? 사용할 수 있는지? 완료가 되었는지? 에 대해서 확인하고자 할 때. 메서드명을 잘 작성하면 그 의미를 명확히 파악할 수 있다.

### Is

is는 가장 많이 사용되는 용법이다. ‘~이다’ 라는 뜻으로 쓰이는데. 그 뒤에 있는 문법에 따라 의미는 다르다.

- is + 명사

- is + 현재진행형(~ing)

- is + 형용사

  ### ▷is + 명사

  무엇인지 확인할 때 사용한다. 사과인가? 수박인가?

  ```java
  bool isApple(Fruit fruit);
  ```

  ### ▷is + 현재진행형(~ing)

  현재 하고 있는 중인가? 라는 뜻일 때 사용한다. 활성화중인가?

  ```java
  bool isActivating();
  bool isPending();
  ```

  ### ▷is + 형용사

  형용사는 단어 자체가 형용사이거나 과거분사일 때가 있다.

  ```java
  bool isSelected() // 선택 되었나?
  bool isHidden() // 숨겨졌나?
  ```

  ### ▷is + 동사 원형(X)

  is 에 동사원형을 그대로 넣는 것은 조금 명확하지 않다. 편집이 가능한가? 를 동사 원형을 사용하여

  `isEdit()`

  을 사용할 수 있을 것이다. 다만 이것보다는

  `isEditable()` or `isEditing`

  이 좀 더 해석하기 쉽고 빠르다.

## 조동사

조동사의 종류는 **can, should, will** 등이 있다. 할 수 있는가? 해야 하는가? 할 것인가? 로 표현할 수 있다.

```java
bool canDelete()
bool shouldRefreshServer()
```

## has

has는 가지고 있는가? 라는 뜻으로 쓰인다.

```java
bool hasApple()
bool hasBaby()
```

## 동사원형 용법

3인칭 단수로 사용한 동사원형 방법이 있는데. 보통 뒤에 -s 나 -es를 붙인다.

- supports: ~을 지원하는가?
- includes: ~을 포함하는가?
- shows: ~을 보여줄 것인가?
- allows: ~을 허용할 것인가?
- accepts: ~을 받아 주는가?
- contains: ~을 포함하고 있는가?

```java
bool supportsVideo() 비디오를 지원하는가?
bool includesImages() 이미지를 포함하는가?
```