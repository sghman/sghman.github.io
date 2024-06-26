---
title: 멱등성을 위해
author: gyuhwan
date: 2024-1-2 8:30:00 +0800
categories: [Project, Reservation]
tags: [Reservation]
image:
  path: /commons/previews/idempotent.png
  alt: idempotent
---
# 멱등성을 위하여

이번 글을 MSA 환경에서 멱등성을 위해 고민했던 것들을 적어보려고 한다. 

# 1. 장애 발생 경우

먼저 그림을 보자.

![idempotent_flow](/commons/reservation/idempotent_flow.png)

간단한 흐름도이다. A 서비스에서 이벤트를 발행하여 메시지 큐에 전달하고 B 서비스는 이를 구독한다. 여기서 발생할 수 있는 장애가 뭐가 있을까?

### A 서비스의 장애(Producer)

- 이벤트 생성시 예외 발생
- 네트워크 장애(Message Queue와 통신 장애)
- 메시지 포맷 예외

### 메시지 큐 장애

- 메시지 큐 서버 다운
- 메시지 큐 용량 초과
- 많은 요청으로 서버 부하

### B 서비스 장애(Consumer)

- 메시지 수신 실패
- 비즈니스 로직상 예외
- 메시지 큐에 응답 실패

이 밖에도 여러 장애가 발생할 수 있을 것이다. 이벤트 생성과 수신시 발생하는 예외에는 다른 글에 적은 것처럼 최소한의 데이터만 전달하도록 처리했다. 

메시지 큐의 장애는 실제로 하나의 메시지 큐로 통신을 했을 때, 서버 부하로 인해 데이터의 유실이 발생했다. 그래서 클러스터 구성으로 변경했다. 

그럼 남은 경우 중에 장애로 인해 **멱등성을 해칠 수 있는 경우**가 뭐가 있을까?

- 메시지 큐 자체에서 오류가 발생했을 때, A 서비스는 재전송을 할 수 있다.
- B 서비스에서 여러 이유로 딜레이가 걸렸을 때, A 서비스에서는 장애가 발생했다고 생각하고 다시 보냈을 수 있다.
- B 서비스에서 브로커에게 응답을 보내지 않았을 때, 카프카는 재전송을 한다.

# 2. 멱등성을 위한 대안들

명칭의 편의를 위해 A 서비스를 **클라이언트**라 하고 B 서비스를 **서버**라고 하겠다. 

## 1) Idempotent Receiver

클라이언트에서 서버에 요청을 보내지만 **응답을 받지 못했을 때**, 클라이언트는 응답이 손실이 됐는지, 메시지가 손실이 됐는지 알 수가 없다. 

요청이 처리됐는지 확인하려면 클라이언트가 다시 요청을 보낼 수밖에 없다.

![idempotent_flow](/commons/reservation/idompotent_receiver1.png)

이를 위한 대안으로 클라이언트는 서버에게 요청 전에 **고유 ID 값을 요청**한다. 서버는 **고유 ID를 저장**하고 요청에 이 **고유 ID 값을 담아서 보낸다.** 

![idempotent_flow](/commons/reservation/idompotent_receiver2.png)

`Client` → `Server`로 고유 ID를 포함하여 요청 시, 서버에서는 다음과 같이 처리한다. 

1. 고유 ID가 등록이 되어있는지 확인한다.
2. 고유 ID로 응답 값이 있는지 확인하고 없으면 요청의 내용을 처리한다.
3. 응답 값을 고유 ID와 함께 저장 후 클라이언트에 응답한다. 

이 과정에서 고유 ID와 응답 값을 확인과 저장을 위해 Session이나 Cache 혹은 DB를 이용하면 된다. 

### Spring Integration

Idempotent Receiver를 직접 구현해도 좋지만 **Spring Integration**에서 이를 제공해준다. 바로 [IdompotentReceiver](https://docs.spring.io/spring-integration/docs/4.1.7.RELEASE/reference/html/messaging-endpoints-chapter.html#idempotent-receiver)를 사용하면 된다. 하지만 여러 인스턴스가 있는 MSA 환경에서는 저장소가 동일해야 멱등성을 보장할 수 있다. 

이를 위해 **Spring Integration**은 공통 저장소를 위한 [Metadata Store](https://docs.spring.io/spring-integration/docs/5.0.5.RELEASE/reference/html/system-management-chapter.html#metadata-store)를 제공해준다. 저장할 수 있는 곳은 mysql, redis, mongodb 등 다양하게 사용할 수 있다. 

## 2) Kafka Exactly Once

카프카를 사용하여 Producer가 Broker에 메시지를 보낼 때, 멱등성을 해칠 경우를 살펴보자. 먼저 다음은 정상 요청이다. 

![idempotent_flow](/commons/reservation/kafka_idompotent.png)

Message를 보내면 Broker 는 커밋 후 응답을 보낸다. 그런데 여기서 다음과 같은 경우에는 Producer는 재요청을 보내게 된다.  

- 응답이 시간 초과 되거나 오류를 보냈을 때
- 브로커 자체에서 실패했을 때
- Consumer 쪽에서 실패했을 때

이러한 장애로 인해 여러 번 요청을 보내게 되고 멱등성을 깨질 위험이 될 수 있다. 이를 위해 카프카 스트림즈에서는 다음과 같은 설정을 제공한다. 

`processing.guarantee=exactly_once`

기본 값은 `At-least-once semantics` ****로 설정되어 있다. `exactly_once` 뜻 그대로 정확히 한 번만 보낸다는 설정인데. 흐름도는 다음과 같다. 

1. Producer는 `transactional.id` 를 사용하여 Broker에게 트랜잭션을 시작하라 요청한다. 
2. Broker는 시퀀스 번호를 부여하고 트랜잭션 상태를 시작으로 변경한다.
3. Producer가 메시지를 전달하면 Broker는 파티션에 임시로 저장하고 Producer에 응답을 보낸다.(이때 컨슈머는 아직 읽을 수 없다.)
4. 모든 메시지를 전송하면 브로커는 트랜잭션을 커밋한다.
5. 컨슈머는 커밋된 메시지만 읽을 수 있다.

이 설정을 통해 정확히 한 번만 메시지를 보낼 수 있게 된다. 다만 파티션에 모두 저장하거나 중간 과정을 통해 약간의 성능 하락이 있을 수 있다

## 3) Outbox Pattern

**Outbox Pattern**은 동일한 트랜잭션 안에서 `INSERT`, `UPDATE`, `DELETE`를 하는 동시에 `Outbox table`에 메시지를 저장하는 것이다. 

그리고 **Message Relay**를 통해 주기적으로 `Outbox table`에 있는 메시지를 가져와서 메시지를 발행한다. 

**Outbox table**에 발행된 메시지는 삭제한다.

![outbox-pattern](https://microservices.io/i/patterns/data/ReliablePublication.png)
https://microservices.io/i/patterns/data/ReliablePublication.png

단, 이 방법은 **Consumer에도 멱등성을 보장**해야 한다. 이유는 Outbox table에 중복된 메시지가 저장될 수 있기 때문이다. 또 분산 환경에서는 Outbox table을 공통적으로 사용할 DB를 적용하는게 나을 수 있다. 

## 3. 내가 적용한 방법

멱등성에 대해 고민하다 이 세 가지 방법을 적절히 섞어보기로 결정했다. 각 서비스에는 모두 Outbox table과 같은 테이블을 추가했다. 

이 테이블에는 메시지 ID, 상태, 응답 값을 저장한다. 

![idempotent_flow](/commons/reservation/me_idompotent.png)

각 서비스는 모두 이벤트 ID와 상태를 저장한다. 처음 이벤트는 Pending 혹은 Retry 상태로 저장하고 응답 값이 정상으로 오면 Success로 바뀌게 된다. 

![idempotent_flow](/commons/reservation/me_idompotent2.png)

만약 응답 과정에서 장애가 발생하면 Client는 Retry 인 이벤트를 재전송한다.

![idempotent_flow](/commons/reservation/me_idompotent3.png)

이 방법을 통해 멱등성을 어느 정도 해결했다. 이벤트를 각 서비스에 저장하기 때문에 이벤트의 장애 지점을 확인할 수 있었다. 모니터링 시스템은 구축했지만 로깅 시스템을 구축하지 못했기에 했던 대안으로 한 처리였다. 만약 추후에 학습을 통해 로깅 시스템까지 구축이되면 조금 다르게 바꿔볼까 한다.

또 각 이벤트를 저장하는 과정에서 상태 값을 관리해야 하며 상태 값에 따라 재전송 유무를 처리와 같은 추가적인 작업이 생겼다. 

## 느낀점

개인적으로 Outbox pattern으로 모두 설계해보면 어땠을까 싶었다. 만약 다음 기회가 된다면 이 방식을 사용해 볼 것 같다. 처음에 Spring Integration으로 구현하지 않았기 때문에 갑자기 바꾸기엔 무리가 있었고 카프카는 아직 미숙한 실력이기 때문에 좀 더 공부를 해보고 제대로 적용해 보고 싶다. 분명 저 설정으로도 추가적인 문제가 발생할 수 있을거라 생각이 든다.

그래도 좋은 경험이었다.

### 참고

https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/

https://www.confluent.io/blog/enabling-exactly-once-kafka-streams/

https://microservices.io/patterns/data/transactional-outbox.html

https://blog.naver.com/fbfbf1/223101741560

https://docs.spring.io/spring-integration/docs/5.0.5.RELEASE/reference/html/system-management-chapter.html#metadata-store

https://docs.spring.io/spring-integration/docs/4.1.7.RELEASE/reference/html/messaging-endpoints-chapter.html#idempotent-receiver

https://martinfowler.com/articles/patterns-of-distributed-systems/idempotent-receiver.html