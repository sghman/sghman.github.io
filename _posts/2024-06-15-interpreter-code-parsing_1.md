---
title: 주석
author: gyuhwan
date: 2024-09-02 14:40:00 +0800
categories: [Document, Java]
tags: [Java]
---

프로젝트를 새로 개발하거나 새로운 요구 사항을 처리할 때마다 느끼는 것은 ‘문서화의 중요성’이다.  그리고 주석은 코드에서 문서화의 중요한 도구로 활용될 수 있다. 각 회사의 컨벤션에 따라 다를 수 있지만,  일반적으로 사용되는 주석 작성 틀을 아래와 같이 정리해 보았다. 이 글은 ‘코드 작성 가이드’ 와 Spring Framework를 참고하여 두고두고 보려는 목적으로 작성했다. 예시로 사용된 ModelAndView는 이해를 위해 수정한 예시이다.

## 문서화 주석(클래스)

문서화 주석은 `/** **/`형식으로 작성할 수 있다.  클래스 혹은 메서드에 대한 상세한 정보와 요약, 기본적인 사용법, 제약 사항등을 설명하는 사용된다.

```java
<ModelAndView in SpringFramework>

/**
 * 웹 MVC 프레임워크에서 Model과 View를 모두 보유하는 클래스.       
 *
 * 이 둘은 완전히 별개임을 유의하라. 이 클래스는 단지 컨트롤러가
 * 하나의 반환 값으로 Model과 View를 모두 반환할 수 있도록 하기 위해
 * 둘 다 소유합니다.
 
 * <p>handler에 의해 반환된 Model과 View를 나타내며, DispatcherServlet에 의해
 * 해결됩니다. View는 ViewResolver 객체에 의해 해결되어야 ...
 * 모델은 키값으로 다양한 객체를 Map으로 관리한다.
 *
 * <p>예시:
 * <pre>
 *     ModelAndView modelAndView = new ModelAndView("viewName");
 *     modelAndView.addObject("attributeName", attributeValue);
 *     // Controller에서 반환
 *     return modelAndView;
 * </pre>
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @author Rob Harrop
 * @author Rossen Stoyanchev
 * @see DispatcherServlet
 * @see ViewResolver
 * @see HandlerAdapter#handle
 * @see org.springframework.web.servlet.mvc.Controller#handleRequest
 */
public class ModelAndView {
```

### 주석 구성 요소

1. **요약** : 제일 처음 **마침표**까지를 클래스의 요약으로 본다. 클래스의 기능을 간결하게 설명한다.
    
    ```java
    /**
     * 웹 MVC 프레임워크에서 Model과 View를 모두 보유하는 클래스.      
     *
    ```
    
2. **상세 설명** :  클래스의 동작,  제약 사항, 기본적인 사용법 예시를 작성한다.
    
    ```java
     *
     * 이 둘은 완전히 별개임을 유의하라. 이 클래스는 단지 컨트롤러가
     * 하나의 반환 값으로 Model과 View를 모두 반환할 수 있도록 하기 위해
     * 둘 다 소유한다
     *
     * <p>handler에 의해 반환된 Model과 View를 나타내며, DispatcherServlet에 의해
     * 해결됩니다. View는 ViewResolver 객체에 의해 해결되어야 ...
     * 모델은 키값으로 다양한 객체를 Map으로 관리한다.
     *
    ```
    
3. **작성자, 참조** : 작성자에 대한 정보나 관련 메서드, 클래스에 대한 참조를 작성한다.
    
    ```java
     * @author Rod Johnson
     * @author Juergen Hoeller
     * @author Rob Harrop
     * @author Rossen Stoyanchev
     * @see DispatcherServlet
     * @see ViewResolver
     * @see HandlerAdapter#handle
     * @see org.springframework.web.servlet.mvc.Controller#handleRequest
     */
    ```
    
4. **예시** :  ****사용 예시를 통해 추가적인 정보를 제공한다.
    
    ```java
     *
     * <p>예시:
     * <pre>
     *     ModelAndView modelAndView = new ModelAndView("viewName");
     *     modelAndView.addObject("attributeName", attributeValue);
     *     // Controller에서 반환
     *     return modelAndView;
     * </pre>
     *
    ```
    

## 문서화 주석(메서드)

메서드에 대한 주석은 반환값, 매개변수, 예외 상황 등을 설명하여 사용자가 이해하기 쉽게 만드는 것이 목적이다.

### 1. 반환값 주석

메서드가 반환하는 값에 대한 설명을 제공한다. 반환값이 무엇이고 어떤 상황에서 반환되는지 `@return`을 사용하여 작성한다. 

- 일반적으로 model map을 반환하고 모델이 존재하지 않으면 null을 반환한다.

```java
/**
 * 모델 맵을 반환한다. {@code null}을 반환할 수 있다.
 * DispatcherServlet에 의해 모델 평가를 위해 호출된다.
 *
 * @return 모델 맵, 또는 모델이 없을 경우 {@code null}
 */
@Nullable
protected Map<String, Object> getModelInternal() {
    return this.model;
}
```

### 2. 매개변수 주석

메서드의 매개변수에 대한 설명을 제공한다. 각 매개변수가 어떤 역할을 하는지, `@param`을 사용하여 작성한다.

- 매개변수로 들어온 item을 추가하는데, 반드시 null이 되면 안된다고 명시한다. 만약 null 값이 들어오면 **NullPointerException**이 발생한다.

```java
/**
 * 리스트에 새로운 항목을 추가한다.
 *
 * @param item 추가할 항목, {@code null}이 아니어야 한다
 * @throws NullPointerException 항목이 {@code null}일 경우
 */
public void addItem(@NonNull String item) {
    if (item == null) {
        throw new NullPointerException("Item must not be null");
    }
    this.items.add(item);
}

```

### 3. 예외 주석

메서드가 던질 수 있는 예외에 대해 설명한다. 어떤 상황에서 예외가 발생하는지, 어떤 예외를 던지는지 `@throw`를 사용하여 작성한다. 

- 만약 인덱스가 범위를 벗어난다면 **IndexOutOfBoundsException** 이 발생한다.

```java
/**
 * 인덱스로 리스트에서 항목을 가져온다.
 *
 * @param index 가져올 항목의 인덱스
 * @return 지정된 인덱스의 항목
 * @throws IndexOutOfBoundsException 인덱스가 범위를 벗어날 경우
 */
public String getItem(int index) {
    if (index < 0 || index >= this.items.size()) {
        throw new IndexOutOfBoundsException("Index out of range");
    }
    return this.items.get(index);
}

```

## 비형식 주석

비형식 주석은 `//` 또는 `/* … */` 형식으로 작성된다. 코드의 가독성을 높이고 추가적인 로직의 의도를 설명할 때 사용된다.

```java
/** View instance or view name String. */
@Nullable
private Object view;

/** Model Map. */
@Nullable
private ModelMap model;

/** Optional HTTP status for the response. */
@Nullable
private HttpStatusCode status;
```

## 잘못된 주석

주석의 내용은 코드 레벨보다 높은 수준의 추상화로 작성해야 한다. 같은 수준의 추상도로 작성된 주석은 오히려 혼란을 초래할 수 있다. 다음은 잘못된 주석의 예시이다.

```java
/**
 * 주문을 처리하는 메서드다.
 * isValidOrder가 true면 saveOrder, processPaymemt를 호출한다.
 * 그렇지 않다면 오류 메시지를 반환한다.
 */
public String processOrder(Order order) {
    if (isValidOrder(order)) { // 주문이 유효한지 확인
        saveOrder(order); // 주문을 저장
        processPayment(order); // 결제 진행
        return "주문이 성공적으로 처리되었습니다.";
    } else {
        return "유효하지 않은 주문입니다."; // 오류 메시지 반환
   
}
```

주석을 작성할 때는 읽는 사람이 이해할 수 있도록 추상도를 높여서 작성하자.

```java
/**
 * 주문을 처리하는 메서드다.
 * 만약 주문이 유효하면 주문을 저장하고 결제를 진행한다.
 * 그렇지 않다면 오류 메시지를 반환한다.
 */
public String processOrder(Order order) {
    if (isValidOrder(order)) { 
     ...
```