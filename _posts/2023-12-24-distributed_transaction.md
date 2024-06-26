---
title: 분산 트랜잭션 해결 방법1
author: gyuhwan
date: 2023-12-24 20:14:00 +0800
categories: [Project, Reservation]
tags: [Reservation]
image:
  path: /commons/previews/distributed_transaction1.png
  alt: distributed transaction
---
# **1. 사가 패턴의 Choreography**

분산 트랜잭션을 관리하기 위한 사가 패턴의 Choreography를 적용한 내용을 적어보고자 한다. **공연 등록**과 **공연 수정**에 패턴을 적용했는데. 

사실 공연 등록과 공연 수정의 흐름에는 API 통신으로 바로 처리하는게 더 낫겠다는 생각이 들었지만 학습의 목적이 있기 때문에 억지로 적용했다. 

## 1) ACID

분산 트랜젝션에서는 **격리성, 일관성을 보장하기 어렵다.** 

먼저 격리성이란 **하나의 트랜잭션**이 **다른 트랜잭션에 영향을 미치지 않는 것**을 의미한다. 각각의 트랜잭션이 서로 격리되어 있기 때문에 영향을 주지 않으므로 트랜잭션의 독립성을 보장해준다. 그런데 분산된 트랜잭션을 관리하는 환경에서는 쉽지 않다. 

예를 들어보자. 상품의 상태를 변경하는 요청이 왔을 때, 재고 서비스에 이 변경된 요청을 전달한다고 가정해보자.

![Isolation1.png](/commons//reservation/Isolation1.png)

1. 상품 서비스에서는 ‘상품 상태 변경 요청’ 을 받게된다. 그리고 상품의 상태가 변경되고 DB에 변경된 데이터를 저장한다.
2. ‘재고 상태 변경 요청’ or ‘상품 상태 변경’ 이벤트를 메시지 큐에 발행한다.
3. **재고 서비스**에서는 **‘재고 상태 변경’** 이벤트를 전달 받는다. 이벤트를 전달받은 재고 서비스는 요청에 대한 프로세스를 진행한다. 재고 서비스에서는 재고에 대한 개수의 변경이나 이름의 변경이 일어날 수 있다.

![isolation_rollback.png](/commons/reservation/isolation_rollback.png)

여기서 재고 서비스에서 작업도중 **장애**가 발생하여 *보상 트랜잭션*을 진행한다고 가정해보자.

재고 서비스에서는 ‘**롤백 요청 이벤트**’를 발행한다.
하지만 이때 ***롤백 이전에 사용자가 커밋된 상품 상태를 조회하면 어떻게 될까?*** 그럼 사용자에게는 **롤백 전의 상태를 조회하게 된다.** 이미 DB에 저장된 상태는 잘못된 상태이기 때문에 사용자에게 적절하지 못한 정보를 제공해주는 것과 같다.

![isolation_new.png](/commons/reservation/isolation_new.png)



이와 같이 분산 트랜잭션에서는 **격리성의 부족으로 인해 일관성을 보장하지 못하게 될 수 있다.**

구현하는 과정에서 이러한 일관성의 부족은 받아드리기로 했고 **최종 일관성**을 보장하는 방식으로 처리했다. 최종 일관성이란, 데이터의 일관성이 즉시 보장되지 않는 것을 의미한다. 어느 정도의 지연 시간이나 오차가 발생할 수 있지만 결국에는 일관된 상태로 수렴하는 것을 의미한다.

## 2) Choreography

**choreography** 방식이란, ***중앙에서 제어하는 시스템*** 없이 서비스들이 직접 서로 이벤트를 구독하고 발행하여 트랜잭션을 관리하는 방식이다. 예를 들어, 아래와 같은 흐름이 될 수 있다.

**주문 요청 → 주문 서비스 → 재고 서비스 → 결제 서비스**

# **2. 이벤트 설계**

먼저 Choreography 방식으로 설계하기 전에 이벤트를 설계하기로 했다. 

## 1) **이벤트 모듈화**

공연 서비스에 이벤트를 적용하기 앞서 이벤트 처리의 구분을 위해 이벤트 모듈을 구성했다.

```
├─out
│  └─production
│      ├─classes
│      │  └─com
│      │      └─sim
│      │          └─performance
│      │              └─event
│      │                  ├─core
│      │                  │  ├─payload
│      │                  │  └─type
│      │                  ├─external
│      │                  └─internal
```

기본 구조는 **core**에는 핵심이 되는 클래스를 모아놨다. 그리고 외부 이벤트와 내부 이벤트를 분리했다. 모듈을 분리하고 도메인에서의 처리가 필요한 handler는 domain 서비스에 구현했다.

### **내부 이벤트 발행**

```java
public class InternalEventPublisher {
    private final ApplicationEventPublisher eventPublisher;
     
    //공연 생성 내부 이벤트 발행
    public void publishPerformanceCreatedEvent(InternalEventDto internalEventDto) {
      log.info("Creation of performances Publication of internal events. performanceId : {}", internalEventDto.getPerformanceId());
      eventPublisher.publishEvent(PerformanceCreatedPayload.from(internalEventDto));
    }
      
    // 공연 생성 결과, 내부 이벤트 발행
    public void publishPerformanceCreatedEventResult(EventResult eventResult) {
      log.info("Creation of performances Publication of internal events result. eventId : {}", eventResult.getId());
      eventPublisher.publishEvent(CreatedEventResultDto.from(eventResult));
    }
}
```

내부 이벤트 발행에 대한 처리를 진행하는 클래스이다. 이벤트에 대해 각각 메서드명으로만 분리했다. 

공연 생성 내부 이벤트가 발행되면 두 가지 작업이 진행된다.

![internal_event1.png](/commons/reservation/internal_event1.png)

- **이벤트 저장**
- **외부 이벤트 발행**

이벤트의 발행과 결과를 조회하기 위해서 이벤트를 저장하고 다른 서비스에게 이벤트를 전달하기 위한 외부 이벤트 발행이 이루어진다.

### **이벤트 저장**

```java
public class InternalEventHandler {
	private final PerformanceEventService performanceEventService;

  /**
  * 공연 생성 이벤트 저장
  */
  @EventListener
  public void saveEvent(PerformanceCreatedPayload performanceCreatedPayload**) {
    performanceEventService.saveEvent(performanceCreatedPayload, EventType.PERFORMANCE_CREATED);
  }
}
```

도메인 모듈에 구성된 내부 이벤트 핸들러에서는 이벤트를 받아서 DB에 저장한다.

### **외부 이벤트 발행**

```java
public interface ExternalEventPublisher {
    /**
    * 공연 생성 이벤트 발행
    */
    void publishPerformanceCreatedEvent(PerformanceEvent performanceEvent);

    /**
    * 공연 수정 이벤트 발행
    */
    void publishPerformanceUpdatedEvent(PerformanceEvent performanceEvent);
}
```

외부 이벤트 발행은 메시지 큐를 사용하는 **RabbitMQ**나 **kafka**를 사용하는 다른 구현체를 사용할 수 있도록 인터페이스로 구현했다. 실제 외부 이벤트 발행은 **kafka**로 진행했다.

```java
public class KafkaExternalEventPublisher implements ExternalEventPublisher {
	private final StreamBridge streamBridge;
	
    /**
     * 외부 서비스에게 공연 생성 이벤트 발행
     *
     * @param performanceEvent 공연 이벤트
    */
    @Override
    public void publishPerformanceCreatedEvent(PerformanceEvent performanceEvent) {
      log.info("Publish events to external services : {}", performanceEvent);
      streamBridge.send("performance-service.performance.created", performanceEvent);
    }
}
```

# 3. **공연 등록**

공연 등록의 이벤트의 흐름은 다음과 같다. 

### Event Flow

![performance_flow.png](/commons/reservation/performance_flow.png)

공연 등록을 **Choreography 방식**으로 설계하면서 고민했던 내용은 다음과 같다.

- **이벤트로 전달할 값**
- **보상 트랜잭션**
- **멱등성**

### **이벤트로 전달할 값**

공연 서비스에서 공연 등록 신청이 완료되면 예약 서비스로 이벤트가 발행한다. 예약 서비스에서는 공연 정보를 가지고 공연 가능한 예약 정보와 좌석 정보를 저장한다. 이때 **예약 서비스로 전달할 이벤트 내부 값**이 문제였다.

처음에는 단순히 이벤트에 *공연 정보를 모두 담아서* 전달했다. 필요한 데이터가 모두 있었기 때문에 처리 시 문제는 없었다. 하지만 다른 작업의 수정을 통해 데이터의 변경이 잦았고 그때마다 관여된 모든 필드를 수정하는 작업이 이루어졌었다. 이 이벤트 내부 값도 그 중에 하나였는데. 하나하나 다 수정하는 작업은 너무 힘들었다.

이를 해결하기 위해 우아한 형제들의 Zero-Payload 방식을 적용했고 [블로그 글](https://simgyuhwan.github.io/%EC%98%88%EC%95%BD%EC%8B%9C%EC%8A%A4%ED%85%9C_%ED%83%80%EC%84%9C%EB%B9%84%EC%8A%A4%EB%A1%9C%EC%98%81%ED%96%A5/)에 게시해놨다.

**Payload**

```java
public class PerformanceCreatedPayload implements Payload {
	private String id;
	private Long performanceId;
	private Long memberId;
}
```

공연 서비스에서 예약 서비스에는 이벤트 처리에 필요한 **식별값**을 포함해서 전달해준다. 

**API 추가**

추가된 API를 사용하여 예약 서비스는 필요한 정보를 조회하여 이벤트를 처리한다

```java
@GetMapping("/{performanceId}/status/pending")
public PerformanceInfoResponse unregisteredPerformanceInfo(@PathVariable Long performanceId) {
	PerformanceDto performanceDto = performanceQueryService.selectPendingPerformanceById(performanceId);
	return PerformanceInfoResponse.from(**performanceDto);
}
```

### **보상 트랜잭션**

이벤트의 시작점인 **공연 서비스에서 실패**한다고 가정해보자. 이곳은 간단히 어노테이션을 추가함으로써 로컬 트랜잭션 관리가 되기 때문에 **장애가 발생 시 간단하게 롤백할 수 있다**.

하지만 커밋이 완료된 후, 공연 서비스에서 이벤트를 발행하고 예약 서비스에서 장애가 발생한다고 해보자. 

![compensation.png](/commons/reservation/compensation.png)

이때는 이미 공연 서비스에서 커밋 된 내용을 롤백 처리해야 한다. 이러한 보상 트랜잭션 처리에 관해서도 고민을 많이 했다.

1. 사용자에게 이미 등록됐다고 응답했는데. 실패했다고 다시 알려야 하나?
    
    → 등록 처리 완료가 아니라 등록 대기 상태를 추가할까?
    
    → 알림 서비스를 통해 사용자에게 메시지를 전달할까?
    
2. 공연 서비스에서 등록된 내용을 어떻게 롤백을 하지?
    
    → 등록 실패 상태로 바꿀까?
    
    → 등록할 때, 등록 대기 상태로 저장하고 롤백시 삭제 처리할까?
    

결과적으로 등록 대기 상태를 추가하기로 했다. 공연 Entity에 상태 값을 추가했다. 공연 등록 시 공연 정보는 Pending 상태로 저장한다.

```java
public enum RegisterStatusType {
	PENDING("등록 요청 대기중"),
	COMPLETED("등록 완료"),
	FAILED("등록 실패");
    ...
}
```

사용자는 따로 추가한 API를 통해서 공연 등록 상태를 조회할 수 있게 했다. 아래의 API를 통해서 현재 등록 신청 대기, 완료, 실패 여부를 조회할 수 있다.

```java
@GetMapping("/{performanceId}/status")
public PerformanceStatusDto performanceStatus(@PathVariable Long performanceId) {
	return performanceQueryService.getPerformanceStatusByPerformanceId(performanceId);
}
```

예약 서비스에서 장애가 발생시 롤백 이벤트를 발행하도록 했다. 롤백 이벤트를 전달 받은 공연 서비스는 해당 도메인의 상태를 실패로 바꾼다. 

보상 트랜잭션을 적용하면서 사용자 친화적으로 만들 수 있도록 노력했다. 일반적인 공연 조회에서는 상태가 **COMPLETE** 인 공연만 조회하도록 변경했으며 예약 서비스에서 실패시, 롤백 진행 결과로 **FAILED** 상태로 변경한다. 사용자는 **FAILED** 된 공연을 조회하면 재등록 요청 메시지를 받게 된다. 이 밖에도 이벤트 발행과 처리는 모두 저장되도록 구현했다.

### **멱등성**

멱등성에 대해서 따로 글을 작성하려고 한다. 여기서는 가볍게 적겠다. 만약 동일한 이벤트가 여러 개가 전달 된다면 전달된 수 만큼 이벤트가 처리될 것이다. 이러한 처리를 방지하기 위해 **이벤트 응답에 대해서 처리 결과 저장과 조회시 캐시**를 이용하기로 했다. 만약 동일한 요청이 여러 번 온다면 *캐시 값을 조회하여 반환하게 되고 캐시에 없다면 DB를 조회해서 반환해준다*.

```java
public EventResult savePerformanceInfo(DefaultEvent<PerformanceEventPayload> event) {
    String eventId = event.getId();
    Payload payload = event.getPayload();

    // 처리된 이벤트라면 이전 처리 결과 반환
    if(isProcessedEvent(eventId)) {
      return getProcessedEventResult(eventId);
    }
    // 이벤트 처리
    return processCreatedEvent(eventId, payload);
}
```

## 3) **공연 수정**

공연 등록의 경우는 단순히 **공연 등록 대기 상태**로 바꾸는 것으로 손쉽게 해결이 가능했다. 하지만 공연 수정은 쉽지 않았다. 그 이유는 공연 정보가 수정이 되고 커밋이 됐다고 해보자. 예약 서비스에서 실패한다면 보상 트랜잭션을 진행할 것이다. 이때 공연 서비스에서는 수정되기 전 데이터로 롤백해야 한다. 

그런데 ***이 수정 전의 데이터는 어디있는가***?

커밋되기 전의 데이터가 존재해야 롤백이 가능한데. 이미 변경된 데이터밖에는 조회할 수 없었다. 그러면 이 변경 전의 데이터를 저장할 수 밖에 없는데. 어디에 어떻게 저장해야할까??

한참을 고민했고 결과적으로는 세 가지 방안을 떠올렸다.

### **임시 저장소**

동일한 DB의 다른 테이블이나 NoSQL에 이러한 정보를 저장하는 방법이었다. 캐시를 사용하자니 유실됐을 때의 손실이 크기 때문에 제외했다.

동일한 DB의 다른 테이블에 저장하는 방법은 테이블의 스키마를 어떻게 구성해야 할지였다. 모든 테이블이 복합적으로 사용할 수 있게 만들어야 하는건가? 아니면 각각 하나씩 만들어야 하는건가? 아무리 생각해도 비효율이었다.

NoSQL로 구성하는 방법은 모든 서비스에서 사용하게 해야 하나? 아니면 각 서비스에 하나씩 구성해야할까? 그러면 각 서비스 당 두 개의 DB를 사용하게 될텐데. 너무 과하지 않을까 싶었다. 실 사용에 대한 경험이 없었기 때문에 섣불리 해볼 수 없었다.

### **버전**

임시 저장소를 떠나보내고 다른 방안을 떠올렸는데. **공연 Entity의 식별 값과 버전을 복합키**로 사용하는 방법이다. 공연에 대한 수정 작업이 있을 때 버전을 올려서 저장한다. 그리고 조회 시 최신 버전만 조회한다.

만약 **롤백해야 한다면 이전 버전으로 돌려주기만 하면 된다**. 가장 괜찮은 방법으로 생각되었다. 하지만 이전 작업으로 인한 피로로 인해 이 방법은 다음으로 미뤘다.

### **최종 일관성**

**스케줄러를 통한 이벤트 재발행**을 통해서 **최종 일관성을 보장**한다. 공연 서비스에서 공연 수정이 완료되었다면 연관된 서비스에서 장애가 나더라도 재전송을 통해 최종적으로 일관성을 보장하게 했다.

예약 서비스에서 이벤트 실패 시, 공연 서비스에선 해당 이벤트를 **RETRY 상태**로 변경한다. 그리고 스케줄러를 통해 이벤트 타입과 상태를 통해 재발행해야 할 이벤트를 조회하여 다시 발행한다.

그 잠깐 사이의 정보의 오류는 일단 감수하기로 했다. 스케줄러로 이벤트를 재발행하기 전에 모니터링을 통해 장애를 확인하고 처리해야 하기 때문에 장애에 대한 저장 처리를 추가했다. 이 부분은 간단하게만 작성했다.

```java
/**
 * 공연 수정 Retry 이벤트 재발행
 */
@Scheduled(cron = "0 0/10 * * * *")
public void rePublishUpdateEvent(){
	performanceEventService.rePublishPerformanceUpdateEvent();
}
```

이 부분의 아쉬웠던 점은 스케줄러를 모듈로 분리하고 처리하고 싶었으나 도메인에서 이벤트를 관리하도록 해버려서 어려웠다. 조금 더 고민해야할 부분이다.

# **느낀점**

**Choreography** 방식으로 설계하면서 느낀 점은 만약 처리할 내용이 많아진다면 점점 더 복잡해지고 추적이 힘들어질 것 같다는 것이다.

지금은 단순한 흐름에 적용했지만 보상 트랜잭션을 구현하면서도 놓치고 있는 부분이 분명히 있을 것 같다는 생각이든다. 경험이 적어 모든 것을 파악하기 힘들었지만 프로젝트가 어느정도 완성이 되고 성능 테스트와 각종 테스트를 통해 파악해 보려고 한다.

다시금 설계의 중요성을 깨닫게 되는 시간이었다.

### **참고**

https://learn.microsoft.com/ko-kr/azure/architecture/reference-architectures/saga/saga

https://azderica.github.io/01-architecture-msa/