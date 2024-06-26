---
title: 대량의 예약 신청으로 인한 동시성 문제 발생
author: gyuhwan
date: 2023-12-10 17:30:00 +0800
categories: [Project, Reservation]
tags: [Reservation]
---

# 대량의 예약 신청으로 인한 동시성 문제 해결 방법

## 문제 상황

중간 계층은 모두 생략했다. 문제 상황은 이렇다. 대량의 예약 신청이 동시에 요청이 왔을 때, 각각의 요청은 gateway server에서 제공하는 Load Balancing 기능을 통해 분산됐지만 예약 신청 과정 중에 동시성 문제가 발생했다.

![동시성문제1](/commons/reservation/동시성문제1.png)

**기존 코드**

```java
public ReservationInfoDto createReservation(Long performanceId, Long scheduleId, ReservationDto reservationDto) {
    PerformanceInfo performanceInfo = findPerformanceById(performanceId);
    PerformanceSchedule schedule = findPerformanceSchedule(performanceInfo, scheduleId);

    validationReservation(performanceInfo); // 예약 가능 검증
    schedule.reserveSeat(); // 예약

    Reservation reservation = reservationRepository.save(Reservation.of(reservationDto, schedule));
    return ReservationInfoDto.of(reservation, schedule, performanceInfo.getName());
}
```

## 해결 방법

결론부터 말하자면 동시성 문제의 해결을 위해 **Redis의 분산락**을 사용했다. 여기서 사용한 Redis Client는 **Redisson** 이다. 먼저 Redis의 분산락을 사용한 이유는 세 가지이다.

1. **공통된 캐시 사용(성능 향상)**
2. **단일 스레드로 인한 원자성 보장**
3. **동시성 문제 해결(분산락)**

먼저 예약 신청을 할 때, 기존 코드의 경우 예약 신청이 불가능해도 DB를 조회하고 예약 가능 여부를 확인하게 된다. 그런데 공통된 캐시인 Redis를 통해 예약 가능 여부를 먼저 확인하면 어떨까? 만약 예약 신청이 불가능하면 DB를 조회하는 비용이 없어지게 될 것이다.

또 Redisson의 경우 단일 스레드를 지원하기 때문에 데이터들의 원자성을 보장해 준다. 그리고 Redis 클라이언트 중에 Lettuce, Jedis, Redisson 중에 **Redisson을** **선택한 이유**는 **성능을 위해서**다. Redisson은 Lock을 가져오기 위해 pub/sub 기능을 사용하기 때문이다. Lettuce는 락을 얻기 위해 반복문을 사용하여 계속 요청할 수밖에 없다. 단순 하나의 서비스면 모를까 여러 서비스에서 요청하면 Redis에 많은 부하를 가하게 될 것이다. 반면 Redisson은 non-blocking 방식으로 락을 얻기 위한 제한 시간 동안 단순히 대기하다가 락을 얻을 수 있으면 락을 가져가라고 알려준다.

## 흐름도

![동시성문제2](/commons/reservation/동시성문제2.png)

1. 예약 서비스에서 예약 신청
2. Redis에서 예약 신청 가능 여부를 확인
   - 좌석이 예약 가능한 경우:
     1. Redis에 락 획득
     2. 데이터베이스에 좌석 예약 정보를 업데이트
     3. Kafka에 예약 정보 업데이트
     4. Redis의 락을 해제
   - 좌석이 예약 불가능한 경우:
     1. 예약 불가능한 상태를 호출한 서비스에 예외로 반환

### 예약 신청 가능 여부

처음에 Redis에 저장할 값에 대해서 고민을 많이 했다. 객체 전체를 캐시로 저장해서 변경이 될 때, 편하게 객체를 그대로 저장하는 방식으로 유지보수성을 위하는게 좋을지? 아니면 필요한 값만 저장하는게 좋을지.

고민해봤지만 다른 곳에서 사용되지 않을 것이라고 판단되어 필요한 부분만 캐시로 저장했다.

```java
HashOperations<String, Object, Object> hashOperations = redisTemplate.opsForHash();
Boolean isAvailable = (Boolean)hashOperations.get(PERFORMANCE_SCHEDULE + scheduleId, "isAvailable");
```

자료 구조는 Hashs를 사용했다. `PERFORMANCE_SCHEDULE + scheduleId` 와 `isAvailable` 의 필드명을 키로 설정했다.

만약 `isAvailable` 이 `null`이라면 캐시에 등록된 것이 없기 때문에 DB를 조회한다.

```java
if (isAvailable == null) {
  PerformanceSchedule schedule = performanceScheduleRepository.findById(scheduleId)
    .orElseThrow(RuntimeException::new);
  hashOperations.put(PERFORMANCE_SCHEDULE + scheduleId, "isAvailable", schedule.isAvailable());
  isAvailable = schedule.isAvailable();
}
```

그러나 `isAvailable`이 `false`라면 예약 신청이 불가능하기 때문에 매진 예외를 던진다.

```java
if (!isAvailable) {
 throw new SoldOutException(ErrorMessage.SOLD_OUT_PERFORMANCE, scheduleId);
}
```

### 분산락 적용

이제 예약 신청이 가능하다면 **분산락**을 적용할 차례다.

```java
RLock lock = redissonClient.getLock(SEAT_LOCK);

try {
  if (!(lock.tryLock(WAIT_TIME, LEASE_TIME, TimeUnit.SECONDS))) {
    throw new RuntimeException("Seat Lock 획득 실패");
  }

  .. 비즈니스 로직

  hashOperations.put(PERFORMANCE_SCHEDULE + scheduleId, "isAvailable", schedule.isAvailable());

  ...
  return ReservationInfoDto.of(reservation, schedule, performanceInfo.getName());
} catch (InterruptedException e) {
  throw new RuntimeException(e);
} finally {
  lock.unlock();
}
```

WAIT_TIME 동안 lock을 기다리기 위해 대기한다. 그리고 LEASE_TIME간 lock을 점유한다.

### 전체 코드

```java
public ReservationInfoDto createReservation(Long performanceId, Long scheduleId, ReservationDto reservationDto) {
  HashOperations<String, Object, Object> hashOperations = redisTemplate.opsForHash();
  Boolean isAvailable = (Boolean)hashOperations.get(PERFORMANCE_SCHEDULE + scheduleId, "isAvailable");

  if (isAvailable == null) {
    PerformanceSchedule schedule = performanceScheduleRepository.findById(scheduleId)
      .orElseThrow(RuntimeException::new);
    hashOperations.put(PERFORMANCE_SCHEDULE + scheduleId, "isAvailable", schedule.isAvailable());
    isAvailable = schedule.isAvailable();
  }

  if (!isAvailable) {
    throw new SoldOutException(ErrorMessage.SOLD_OUT_PERFORMANCE, scheduleId);
  }

  RLock lock = redissonClient.getLock(SEAT_LOCK);

  try {
    if (!(lock.tryLock(WAIT_TIME, LEASE_TIME, TimeUnit.SECONDS))) {
      throw new RuntimeException("Seat Lock 획득 실패");
    }

    PerformanceInfo performanceInfo = findPerformanceById(performanceId);
    PerformanceSchedule schedule = findPerformanceSchedule(performanceInfo, scheduleId);

    validationReservation(performanceInfo);
    schedule.reserveSeat();

    hashOperations.put(PERFORMANCE_SCHEDULE + scheduleId, "isAvailable", schedule.isAvailable());

    Reservation reservation = reservationRepository.save(Reservation.of(reservationDto, schedule));
    return ReservationInfoDto.of(reservation, schedule, performanceInfo.getName());
  } catch (InterruptedException e) {
    throw new RuntimeException(e);
  } finally {
    lock.unlock();
  }
}
```

## 리팩터링

기존 소스에는 Lock 관련 내용이 예약 서비스에 포함되어 있었다. 이로 인해 예약 서비스의 책임이 증가하게 되었다. 그래서 Lock 관련된 내용은 모두 LockProvider로 옮겼다.

```java
public interface LockProvider {

  /**
   * 키를 사용하여 잠금을 얻는다.
   */
  RLock getLock(String key);

  /**
   * 키에 해당하는 자원에 대한 접근 권할을 얻는 메서드
   *
   * @param key       얻으려는 키
   * @param waitTime  접근 권한을 얻을 최대 시간
   * @param leaseTime 접근 권한을 얻은 후 최대 점유 시간
   * @param unit      waitTime, leaseTime 의 시간 단위
   * @return Lock 이 다른 스레드가 점유하면 false, 아니면 true
   * @throws InterruptedException
   */
  boolean tryLock(String key)
      throws InterruptedException;

  /**
   * 키에 대한 락을 얻을 떄까지 block 대기한다.
   *
   * @param key
   */
  void lock(String key);

  /**
   * 키에 대한 락을 해제한다.
   *
   * @param key
   */
  void unlock(String key);

  <T> T tryLockAndExecute(String key, ErrorMessage errorMessage, Callable<T> task)
      throws InterruptedException;

}
```

그리고 각 의미에 맞는 메서드를 지정하여 가독성을 높이기 위해 수정했다. 그리고 RedissTemplate를 쓰기엔 복잡한 설정을 하지 않기 때문에 Spring Cache로 변경했다.

```java
public ReservationDto createReservation(Long performanceId, Long scheduleId,
      ReservationDto reservationDto) {
    checkReservable(scheduleId);

    PerformanceInfo performanceInfo = performanceInfoRepository.findById(performanceId)
        .orElseThrow(
            () -> new PerformanceInfoNotFoundException(ErrorMessage.PERFORMANCE_INFO_NOT_FOUND,performanceId)
        );

    PerformanceSchedule schedule = findPerformanceSchedule(performanceInfo, scheduleId);
    validationReservation(performanceInfo);

    try {
      lockSeat(performanceId, scheduleId);
      reservationSeat(schedule);
      updateReserveAvailability(scheduleId, schedule.isAvailable());
      
    } catch (InterruptedException e) {
      Thread.currentThread().interrupt();
    } finally {
      unlockSeat(performanceId, scheduleId);
    }

    Reservation reservation = saveReservation(reservationDto,
        schedule);

    publishReservationApplyEvent(reservation);
    return ReservationDto.from(reservation);
  }

@Cacheable(value = "performance-reserve-availability", key = "#scheduleId")
public Boolean getReserveAvailability(Long scheduleId) {
	return performanceScheduleRepository.findById(scheduleId)
		.orElseThrow(() -> new PerformanceScheduleNotFoundException(ErrorMessage.PERFORMANCE_SCHEDULE_NOT_FOUND, scheduleId))
		.isAvailable();
}

@CachePut(value = "performance-reserve-availability", key = "#scheduleId")
public Boolean updateReserveAvailability(Long scheduleId, Boolean isAvailable) {
	return isAvailable;
}
```