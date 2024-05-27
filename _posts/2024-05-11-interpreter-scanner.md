---
title: 2. 인터프리터 구현 - Scanner
author: gyuhwan
date: 2024-05-11 14:40:00 +0800
categories: [Interpreter, Java]
tags: [Interpreter]
---

이번은 [로버트 나이스트롬의 인터프리터 록스](https://product.kyobobook.co.kr/detail/S000211521372)의 구현의 첫 단계인 스캐닝(scanning)에 대한 글이다. 

스캐닝은 컴파일러 또는 인터프리터가 소스 코드를 분석하는 첫 단계로,원시 소스 코드를 입력받아 **토큰(token)** 이라는 의미 있는 청크(Chunk)로 변환한다. 이 과정에서, 문자열은 **렉심(lexeme)** 이라고 하는 기본 단위로 나뉘며, 이들은 나중에 토큰으로 조합한다.

예를 들어, Java 언어에서 변수를 선언하는 코드를 살펴보자.

```java
var language = "lox"
```

여기서 **`var`**는 변수를 선언하는 **키워드**로 사용된다. 단독으로 보면 **`v`**, **`a`**, **`r`**은 각각 하나의 문자에 불과하지만, 이들이 합쳐질 때 **`var`**라는 키워드로서 의미를 갖게 된다. 스캐닝 과정은 이러한 문자들을 연속적으로 읽어들여  **렉심**으로 구분하고, 이를 통해 **토큰**을 생성하는 작업이다.

스캐닝은 다음과 같은 요소들을 식별한다

- **식별자(Identifier)**: 변수나 함수와 같은 사용자 정의 이름
- **키워드(Keyword)**: **`var`**와 같이 언어에 의미가 정의된 예약어
- **연산자(Operator)**: **`=`**와 같이 연산을 수행하는 기호
- **리터럴(Literal)**: **`"lox"`**와 같이 고정된 값
- **구분자(Delimiter)**: **`;`**와 같이 문장의 끝을 나타내는 기호

이러한 렉심들은 **토큰**으로 묶이며, 토큰은 렉심과 함께 해당 렉심의 유형을 나타내는 토큰 타입(token type)을 포함한다.

스캐닝 과정은 다음과 같은 단계로 이루어진다

1. **입력**: 원시 소스 코드를 문자열로 입력받습니다.
2. **렉심 분리**: 문자열을 읽어가며 렉심으로 분리합니다.
3. **토큰 생성**: 분리된 렉심과 토큰 타입을 결합하여 토큰을 생성합니다.
4. **출력**: 생성된 토큰을 다음 분석 단계로 전달합니다

## 렉심과 토큰

### **토큰 타입(TokenType) 정의하기**

먼저, 우리가 다루게 될 모든 종류의 토큰 타입을 `enum`으로 정의해보자. 이것은 우리의 인터프리터가 이해할 수 있는 '어휘'의 기본 단위가 된다.

```java
public enum TokenType {
  // 단일 문자 토큰들
  LEFT_PAREN, RIGHT_PAREN, LEFT_BRACE, RIGHT_BRACE,
  COMMA, DOT, MINUS, PLUS, SEMICOLON, SLASH, STAR,
  
  // 하나 또는 두 개 문자 토큰들
  BANG, BANG_EQUAL,
  EQUAL, EQUAL_EQUAL,
  GREATER, GREATER_EQUAL,
  LESS, LESS_EQUAL,
  
  // 리터럴
  IDENTIFIER, STRING, NUMBER,
  
  // 키워드
  AND, CLASS, ELSE, FALSE, FUN, FOR, IF, NIL, OR,
  PRINT, RETURN, SUPER, THIS, TRUE, VAR, WHILE,
  
  EOF // 파일의 끝을 나타냄
}
```

### **토큰(Token) 클래스 정의하기**

토큰을 더 자세히 살펴보면, 각 토큰은 자신의 타입, 렉심(토큰의 문자열), 리터럴 값, 그리고 어디에 위치해 있는지(줄 정보)를 가지고 있습니다. **에러 추적**을 위해 이런 정보가 필요합니다. 필요하다면 열 정보나 길이 정보도 추가할 수 있다.

```java
class Token {

  final TokenType type;
  final String lexeme;
  final Object literal;
  final int line;

  public Token(TokenType type, String lexeme, Object literal, int line) {
    this.type = type;
    this.lexeme = lexeme;
    this.literal = literal;
    this.line = line;
  }

  public String toString() {
    return type + " " + lexeme + " " + literal;
  }
}
```

## **스캐너(Scanner) 구현하기**

**이제 본격적으로 소스 코드를 읽어들이며 토큰을 생성하는 스캐너의 구현에 들어간다. 아래는 스캐너 클래스의 기본 구조다. 원시 소스 코드를 받아서 `scanTokens()` 메서드를 통해 분석을 시작한다.**

```java
class Scanner {
  private final String source;
  private int start = 0;
  private final List<Token> tokens = new ArrayList<>();
  private int current = 0;
  private int line = 1;
  
  public Scanner(String source) {
    this.source = source;
  }

  public List<Token> scanTokens() {
    // 토큰을 스캔하는 로직이 여기에 들어갑니다.
  }
  ...
}
```

### **렉심 식별하기**

소스 코드를 한 문자씩 읽어가며 토큰을 식별하는 과정은 다음과 같다. 각 문자를 읽을 때마다 해당 문자가 어떤 토큰에 해당하는지 판단하여 토큰 리스트에 추가한다.

```java
private void scanToken() {
  char c = advance();
  switch (c) {
    case '(' -> addToken(LEFT_PAREN);
    case ')' -> addToken(RIGHT_PAREN);
    case '{' -> addToken(LEFT_BRACE);
    case '}' -> addToken(RIGHT_BRACE);
    case ';' -> addToken(SEMICOLON);
    case ',' -> addToken(COMMA);
    case '.' -> addToken(DOT);
    case '-' -> addToken(MINUS);
    case '+' -> addToken(PLUS);
    case '*' -> addToken(STAR);
    // 기타 문자에 대한 처리도 여기에 추가...
  }
}

private char advance() {
  return source.charAt(current++);
}
```

current는 source에서 현재 읽어들이는 위치를 나타내며 하나의 문자를 읽어들이며 switch를 사용하여 토큰을 추가한다.

### 연산자

연산자는 한 글자짜리일 수도 있고, 때에 따라서는 두 글자짜리일 수도 있다. 이를 위해, 현재 문자와 다음 문자를 확인하여 올바른 토큰을 생성해야 한다. 예를 들어, `=`, `>=`, `!=`는 각각 다른 의미를 가지며, 이를 정확히 구분하는 것이 중요하다.

match 메서드는 인수로 들어온 글자가 다음 문자에 포함되어 있는지 확인한다.

```java
private void scanToken() {
  ...
	case '!' -> addToken(match('=') ? BANG_EQUAL : BANG);
	case '=' -> addToken(match('=') ? EQUAL_EQUAL : EQUAL);
	case '<' -> addToken(match('=') ? LESS_EQUAL : LESS);
	case '>' -> addToken(match('=') ? GREATER_EQUAL : GREATER);
	...
}
```

### 주석

연산자를 살펴보면 나눗셈 연산자(`/`)가 빠져있는 것을 확인할 수 있다. 주석도 `/`로 시작하기 때문에 이를 위해서 추가적인 로직이 필요하다. 만약 `/`가 나온다면 다음 글자도 `/` 인지 확인한다. 만약 다음 글자도 동일하다면 주석이라고 판단하고 줄 끝까지 이동한다. 주석은 **토큰으로 만들 필요가 없다.**

```java
case '/' -> {
  if (match('/')) {
    while (peek() != '\n' && !isAtEnd()) {
      advance();
    }
  } else {
    addToken(SLASH);
  }
}

private char peek() {
  if (isAtEnd()) {
    return '\0';
  }
  return source.charAt(current);
}
```

### 예약어와 식별자

프로그래밍 언어를 분석할 때, '**예약어**'와 '**식별자**'를 정확히 구분하는 건 정말 중요하다. 예를 들어, `orchid`라는 변수명을 사용한다고 가정보자. 만약 스캐너가 이 변수명의 처음 두 글자 `or`만 보고 바로 예약어로 판단해버린다면, 이건 분명히 문제가 된다. 이런 문제를 방지하기 위해서는, **가능한 모든 문자 부분을 읽어서 예약어인지 아닌지를 판단하는 것이 좋다**

변수명 같은 식별자는 보통 알파벳(a-z, A-Z) 또는 밑줄(_)로 시작하고, 그 뒤에는 숫자(0-9)가 올 수도 있다. 따라서 식별자를 정확히 식별하기 위해선, 첫 글자가 알파벳이나 밑줄인지 확인하고, 다음 글자들이 알파벳이나 숫자가 아닐 때까지 계속 읽어나가면 됩니다. 이렇게 해서 최종적으로 얻어진 문자열이 예약어인지 아닌지를 확인할 수 있다.

```java
// 식별자 확인하는 메소드
private void checkIdentifier() {
  // 알파벳이나 숫자가 아닐 때까지 읽어나가기
  while (!isAlphaNumeric(peek())) {
    advance();
  }

  // 최종적으로 얻은 문자열
  String identifier = source.substring(start, current);
  // 미리 정의해 둔 예약어 Map에서 확인
  TokenType type = keywords.get(identifier);
  if (type == null) {
    // 예약어가 아니라면 식별자로 처리
    type = IDENTIFIER;
  }
  // 토큰 추가
  addToken(type);
}

// 문자인지 확인하는 메소드
private boolean isAlpha(char character) {
  return (character >= 'a' && character <= 'z') ||
         (character >= 'A' && character <= 'Z') ||
         character == '_';
}

// 문자 또는 숫자인지 확인하는 메소드
private boolean isAlphaNumeric(char character) {
  return isAlpha(character) || isDigit(character);
}

// 숫자인지 확인하는 메소드
private boolean isDigit(char character) {
  return character >= '0' && character <= '9';
}

```

동시에 예약어인지 빠른 확인을 위해 미리 Map에 예약어를 저장해놓는다. 

```java
// 예약어를 저장하는 Map 초기화하기
private static final Map<String, TokenType> keywords;

static {
  keywords = new HashMap<>();
  // 여러 예약어들을 Map에 저장
  keywords.put("and", AND);
  keywords.put("class", CLASS);
  keywords.put("else", ELSE);
  keywords.put("false", FALSE);
  keywords.put("for", FOR);
  keywords.put("fun", FUN);
  keywords.put("if", IF);
  keywords.put("nil", NIL);
  keywords.put("or", OR);
  keywords.put("print", PRINT);
  keywords.put("return", RETURN);
  keywords.put("super", SUPER);
  keywords.put("this", THIS);
  keywords.put("var", VAR);
  keywords.put("while", WHILE);
}
```