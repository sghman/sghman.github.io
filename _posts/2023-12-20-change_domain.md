---
title: 도메인 변경에 따른 영향
author: gyuhwan
date: 2023-12-20 18:14:00 +0800
categories: [Project, Reservation]
tags: [Reservation]
---
# 도메인의 수정이 타 서비스에 영향이?

## 문제 상황

![이벤트클래스1.png](/commons/reservation/이벤트클래스1.png)

기존 설계로 인해 도메인 필드의 수정이 테스트 코드뿐 아니라 이벤트 클래스, 타 서비스에까지 영향을 줬다. 테스트 코드는 도메인 변경시 당연히 영향이 가겠지만, 타 서비스에 영향은 생각지 못했다.

## 이벤트 클래스 설계

먼저 이벤트 클래스를 설계는 다음과 같이 했었다.

공통 Event를 설계하면서 중요하게 생각했던 것이 두 가지였는데. **필수** **필드**와 **설계의 목적 전달**이었다. 필수 필드는 **이벤트 생성 시간**, **이벤트 ID**, **이벤트 상태**, **소스(출처)**, **페이로드**이고 설계의 목적 전달로는 **Event 내부에 Payload가 포함된다는 구조**를 전달하고 싶었다

| 필드             | 필드명        |
| ---------------- | ------------- |
| 이벤트 ID        | eventId       |
| 이벤트 생성 시간 | eventDateTime |
| 이벤트 상태      | status        |
| 소스             | source        |
| 페이로드         | payload       |

### **1) 추상 클래스**

```
// Event 추상 클래스
public abstract class Event<T extend Payload> {
  private final String id;
  private final LocalDateTime eventDateTime;
  private final String source;
  private final String status;
  private final T payload;
 ...
}

// Payload 인터페이스
public interface Payload{}
```

처음 생각했던 방식은 Event 클래스를 추상클래스로 만들고 Payload를 인터페이스로 만드는 방식이었다. 이로써 목적했던 모든 것들이 다 담겼다고 생각했다.

**원했던 필드들이 포함될 뿐더러 Payload가 Event 클래스 안에 있다는 것을 확인할 수 있었다** 그런데 실제 사용해보니 **예상치 못한** **불편함**을 느낄 수 있었다.

막상 상속 받아서 각 서비스에서 사용할 Event를 생성하려고 하니 ‘**추상 클래스에 있는 필드 값이 뭐였더라?’** 결국 추상 클래스를 들어가서 확인해야 했다. 그리고 추상 클래스 내부 필드를 임의대로 수정하고자 하니 추상 클래스에 메서드를 추가했어야 했다. 각 서비스의 요청와 필요에 따라 **추상 클래스의 수정은 불가피해 보였다**.

결국 다른 방법을 찾아보는데..

### **2) 인터페이스 상속**

```java
// Payload 인터페이스
public interface Payload{}

// Payload가 부모 클래스?
public inferface Event<T extend Payload> extend Payload {}

// 기본 구현 클래스
public DefaultEvent implements Event<Payload>{
  private ..eventDateTime;   // 구현 클래스에 직접 필드 지정
  ...
  private T payload;
}
```

다음 방법은 **Event, Payload**를 **인터페이스**로 만들고 **Event**가 **Payload**를 **상속**하게 하는 방법이었다. 이제 Event를 구현할 때, 내부 필드로 payload 타입으로 가지고 있기만 하면 됐다. 다만 필수 필드는 구현 클래스에 따로 정의해야 했다.

된건가 싶었지만 처음에 목표로 했던 **두 가지는 단 하나도 담을 수 없었다**. 필드는 유연하게 원하는 값을 넣을 수 있지만 필수 필드를 강제할 순 없었다. 또 Event가 Payload 를 상속하면서 직관적으로 내가 원하는 구조가 아닌 그 반대의 구조가 떠올렸기 때문에 이번 역시 실패였다.

### **3) 메시지 전달**

```java
// Event 인터페이스
public interface Event {
  String getId();

  Payload getPayload();

  SourceType getSource();

  EventStatusType getStatus();

  LocalDateTime getEventDateTime();
}

// Payload 인터페이스
public interface Payload {}

// 기본 클래스
public class DefaultEvent<T extends Payload> implements Event{
  private String id;

  @JsonSerialize(using = LocalDateTimeSerializer.class)
  @JsonDeserialize(using = LocalDateTimeDeserializer.class)
  private LocalDateTime eventDateTime;
  private EventStatusType status;
  private T payload;
  private String eventType;
  private SourceType source;
}

// 사용 예시
// 받고 싶은 Payload 타입으로 지정
Function<DefaultEvent<PerformanceCreatedPayload>, DefaultEvent<PerformanceCreatedPayload>> performanceCreatedEventConsumer() {
  return event -> {
    // 비즈니스 로직
    return event;
  };
}
```

**Spring Security**에서 아이디어를 얻었다. Spring Security의 **AuthenticationManager**나 **UserDetailService** 등을 보면 이 **인터페이스의 역할과 책임을 이름과 메서드(메시지)로 표현**했다. 이 아이디어를 착안해서 Event 인터페이스가 해야할 책임과 메시지를 메서드로 표현했다. Event를 구현한 클래스는 필수 값들을 뱉어내야 한다. 라는 메시지를 전달했다.

원했던 구조도 getPayload를 통해 나름 전달했다고 생각한다. 사실 더 좋은 방법이 안떠올랐다.. 최종적으로는 **DefaultEvent**를 만들어서 각 서비스에서는 Payload 인터페이스를 원하는 구조로 구현해서 교체하면서 사용할 수 있도록 만들었다.

## 문제 원인

![이벤트클래스2.png](/commons/reservation/이벤트클래스2.png)

문제의 원인은 동일한 Payload 값으로 인해 서비스 1, 2 연결이었다. 처음 서비스 1에서 이벤트를 발행할 때 **변경된 값을 포함하여 모든 값을 전달한다**. 서비스 2는 이 값을 받기 위해 **동일한 구조**로 클래스를 선언하고 받게 된다.  바로 이 부분이 문제였다.

## 해결 방법

해결 하기 위해 [우아한 형제들 Zero-Payload](https://techblog.woowahan.com/7835/)를 참고했다.  Zero-Payload는 이벤트에 꼭 필요한 값만 포함시키는 것이었다.

즉, 서비스 1에서는 이벤트 클래스의 Payload에 식별 값만 포함해서 전달한다. 그리고 전달받은 서비스 2는 식별 값을 통해 서비스 1에 API 요청을 하여 필요한 값만 가져온다.

![이벤트클래스3.png](/commons/reservation/이벤트클래스3.png)

이렇게 함으로써 각 서비스에 API 요청을 추가해야 했지만 변경의 영향은 최소화 할 수 있었다. 단, 관리 포인트가 늘어난다는 단점이 있다.

## Zero-Payload 방식의 문제?

![zero-payload](/commons/reservation/zero-payload.png)

만약 변경이 대량으로 온다면 어떻게 될까? 서비스에 부하가 올 수 있어 응답 시간이 늘어나거나 서버가 다운될 수 있을 것이다. 

이를 위한 해결 방법은 어떤게 있을까? 생각해 본 대안은 두 가지였다. 

**1) 쿠버네티스의 autoscaling**

성능의 부하가 됨에 따라 새로운 서비스를 확장하고 로드 밸런싱을 사용하여 부하를 분산한다. 

**2) Outbox pattern([link](https://microservices.io/patterns/data/transactional-outbox.html))**

별도의 테이블을 두고 Message Relay를 통하여 변경된 내용을 메시지 브로커에게 발행한다. 

개인적으로 힘들게 수정한 코드를 다시 바꿀 엄두가 나지 않아 쿠버네티스의 학습(?)을 생각했었다. 시간이 된다면 두 가지 방법을 모두 적용해 보고자 한다.