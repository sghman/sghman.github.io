---
title: Intellij 단축키 Tip
author: gyuhwan
date: 2023-10-15 12:50:00 +0800
categories: [Backend, Tips]
tags: [Intellij]
---

# **윈도우 단축키** 

## 1. 실행

- **shft +  f10/f9** : 이전 실행했던 것 실행 / 디버그 (우측 상단 실행문 목록 확인)
- **ctrl + shft  + f10** : 현재 커서가 있는 곳 실행

## 2. 추가

- **alt +  insert** : 생성자, 메서드, 테스트 추가
- **ctrl + shft + t** : 테스트 클래스 추가
- **f5** : 현재 클래스 복제
- **f6** : 현재 클래스 이동

## 3. 자동 

- **ctrl + alt + l** : 정렬
- **ctrl+ shft + enter** : ( 이후 자동완성
- **ctrl +  space** : 자동완성 창 열기
- **ctrl + alt + v** : 변수 추출
- **ctrl + alt + p** = 매개변수 추출
- **ctrl + alt + f** = 필드 추출
- **ctrl + alt + c** = 상수 추출
- **ctrl + P** :  매개변수 정보 출력
- **alt + enter**  :  필요한 import문 자동으로 import
- **sout** :      System.out.println() 자동 완성
- **soutv** :      System.out.println(“최근 변수 = “ + 최근 변수); 자동 완성
- **iter** : 향상된      for문 자동 입력

## 4. 체크 

* **ctrl + f8** : 커서 위치 디버깅 포인트 체크
* **f11** : 북마크 체크
* **alt + 2** : 북마크, Breakpoints 표시

## 5. 수정 

- **ctrl + alt  + shft + t** : Refactor 단축키
- **shft + F6** :      같은 코드 한 번에 수정 및 클래스명 변경
- **ctrl + 방향키**      : 멀티커서
- **ctrl + shft  + u** : 드래그 한 문자 대문자로 변환
- **ctrl + alt + n** : inline 적용
- **ctrl + shft + alt + j** : 같은 단어 찾기
- **ctrl + alt + o** : 사용하지 않는 import문 정리
- **ctrl + w** : 현재 커서 위치 드래그

## 6. 검색 

- **ctrl + alt  + S** : setting창 열기
- **ctrl + alt + shft + s** : project structer 창 열기
- **shft 2번** :      파일, 클래스, 성정 등 키워드에 관련된 모든 것 검색
- **ctrl + e** :      Recent file로 작업했던 것들을 검색
- **ctrl + e** :      해당 클래스와 연관된 다른 것들 검색
- **ctrl + alt + b** : 인터페이스면 구현체로 이동
- **ctrl + shft  + f** : 전체에서 찾기
- **ctrl + f12** : 현재 클래스의 메서드 목록
- **f2** : 오류 위치로 바로 이동

## 7. 이동 

- **alt + tab** :  최근 창으로 이동
- **ctrl + shft + 위, 아래** : 메서드 이동
- **alt + shft + 위, 아래** : 한 줄 이동
- **ctrl + shft      + F12** : 전체 화면 만들기
- **ctrl + [, ]** : 해당 범위 안에 끝과 끝 이동

## 8. 복사
- **ctrl + shft      + C** : 파일 경로 복사

# Mac 단축키 
ctrl -> command, alt -> option 

## 1. 실행

- **ctrl + R / D** : 이전 실행했던 것 실행 / 디버그 (우측 상단 실행문 목록 확인)
- **ctrl + option + R / D** : 파일 선택해서 실행 / 디버그
- **ctrl + shft + R** : 현재 편집창 실행

## 1. 추가 
- **command + N** : 파일, 메소드추가

## 2. 자동 
- **option +  command + L** : 정렬
- **command+ shft + enter** : ( 이후 자동완성
- **ctrl + space** : 현재 작성하고 있는 것 자동완성 창 열기
- **command + option + V** : 현재 작성문의 반환값을 ‘타입 변수 = 작성문’ 으로 자동 완성, 반환타입 알고자 할 때 용이
- **command + P** : 매개인자에 대한 정보 , 매개인자자리에서 사용 시 (a, b) a에는 어떤값이, b에는 어떤값이 와야하는지 알려줌
- **option + enter** : 재정의 자동으로 가져오기, 필요한 import문 자동으로 import
- **command + shft + T** : create new test를 누르면 테스트껍떼기를 한 번에 생성
- **sout** :      System.out.println() 자동 완성
- **soutv** :      System.out.println(“최근 변수 = “ + 최근 변수); 자동 완성
- **iter** : 향상된      for문 자동 입력

## 3. 수정 
- **ctrl + T** :      Refactor 단축키
- **command + option + M** : extract method
- **shft + F6** :      같은 코드 한 번에 수정
- **command + option + P** : 함수 안에서 값들을 파라미터로 꺼내기
- **option + 방향키** : 멀티커서
- **command +  shft + U** : 드래그 한 문자 대문자로 변환
- **command + option + N** : inline, return문으로 한 번에 합친다.
- **ctrl + command + G** : 같은 단어 찾기
- **Ctrl + option + O** : 사용하지 않는 import문 정리

## 4. 검색 
- **command + ,(콤마)** : setting창 열기
- **shft 2번** : 파일, 클래스, 성정 등 키워드에 관련된 모든 것 검색
- **command + E**  : Recent file로 작업했던 것들을 검색
- **command + B** : 해당 클래스와 연관된 다른 것들 검색
- **command + shft + F** : 전체에서 찾기

## 5. 이동 
- **command + 1**  : 프로젝트 탭으로 커서 옮기기
- **command + shft + <- , ->** : 커서를 프로젝트 탭으로 옮긴 상태에서 프로젝트 탭 크기 조절
- **command + shft + F12 또는 command + 1** : 전체 화면 만들기,
- **command + shft + 방향키** : 메서드 위치 통채로 위 아래로 옮기기

## 6. 복사
- **command + shft + C** : 파일 경로 복사