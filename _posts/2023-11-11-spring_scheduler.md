---
title: 스프링 스케줄러 사소한 팁
author: gyuhwan
date: 2023-11-11 15:50:00 +0800
categories: [Backend, Spring]
tags: [Scheduler]
---

# Spring Scheduler Tip

## 1. 스케줄러의 사용이유

스케줄러는 일정한 주기에 특정한 작업을 반복시키기 위해 사용된다. 예를 들어, 가게 정산을 밤 11시 마다 하고 싶다면 이를 하나의 스케줄의 작업으로 묶어서 설정할 수 있다.

스프링에서는 이러한 스케줄 처리를 아주 간단하게 설정할 수 있게 지원해준다.

## 2. 스케줄러 사용방법

기본적으로 `spring-boot-starter` 를 의존성으로 가지면 Scheduler 도 포함되어 빌드된다.

**1) `@EnableScheduling`**

스프링 스케줄러를 사용하려면 EnableScheduling 애너테이션을 추가해줘야 한다.

1. 사용하려는 **작업 단위**에 `@Scheduled` 애너테이션을 추가해주자. (Bean 객체)

```java
@Scheduled(fixedDelay = 1000)
public void fixedDelayTest() throws InterruptedException {
    log.info("fixedDelay started..");
    log.info("fixedDelay finished..");
}
```

- flxedDelay : 작업이 끝나는 시점을 기점으로 시작된다.
- flxedRate : 작업이 시작되는 기점으로 시작된다.
- initialDelay : 초기 시작 대기시간, 이 시간 이후 스케줄링이 시작된다.

## 3. 주의 사항

### 1. 싱글 스레드

기본적으로 스레드는 싱글 스레드로 구성되어 있다. 만약 작업 시간이 예상 시간보다 오래 걸려서 다음 스케줄 실행시간을 지나가면 FlxedRate의 경우, 바로 실행이 된다.

여기서 **문제**는 스케줄이 여러 개일 때 이다. 스케줄이 여러 개 실행이 된다면 이러한 작업 딜레이가 다른 스케줄에도 영향을 미치게 된다.

이를 해결하기 위해 두 가지 방법을 같이 쓰는 것이 좋다. 바로 **스레드 개수 증가**와 **비동기 처리**이다.

- **Thread 개수 증가**

  ```java
  @EnableScheduling
  @Configuration
  public class SchedulerConfig implements SchedulingConfigurer {
  		// 증가하려는 스레드 갯수
      private final int POOL_SIZE = 5;
  
      @Bean
      public Executor taskExecutor() {
          return Executors.newSingleThreadExecutor();
      }
  
      @Override
      public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
          final ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
  
          threadPoolTaskScheduler.setPoolSize(POOL_SIZE);
          threadPoolTaskScheduler.setThreadNamePrefix("test");
          threadPoolTaskScheduler.initialize();
  
  				// 스케줄 정보 등록
          scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
      }
  }
  ```

  **ScheduledTaskRegister** : 이름 그대로 스케줄 작업을 등록하는 객체이다.

  **ThreadPoolTaskScheduler** : 등록할 스케줄 설정 정보이다. 이곳에서 관리할 스레드 크기와 기타 정보를 설정한다.

- **비동기 처리**

  스케줄은 기본적으로 싱글 스레드에 **동기** 방식으로 처리된다. 이는 작업이 오래 걸리거나 에러가 발생하면 연관된 흐름에 영향을 끼쳐 성능상 문제를 일으키게 된다.

  그렇기 때문에 비동기로 실행을 하면 이전 스케줄러의 작업이 끝나지 않더라도 기다리지 않고 실행할 수 있게 하여 성능 하락을 막을 수 있다.

  간단하게 root Application or Configure 에 `@EnableAsync` 애너테이션을 추가하고 사용하려는 메서드에 `@Async` 애너테이션을 추가하면 된다.