---
title: JVM 의 메모리 영역
date: 2022-03-01T15:12:26+09:00
categories:
  - java
tags: 
  - jvm
---

![image](https://user-images.githubusercontent.com/46465928/157824081-ea59f679-f103-4baa-81d5-5d30fc93e5de.png)

JVM의 메모리 영역은 Method Area, Heap Area, Stack Area, PC Register, Native Method Stack으로 세분화된다.

## Method Area

Method Area 에는 **클래스**, **인터페이스**, **생성자**, **메소드**, **Runtime Constant Pool**과 **static 변수** 등이 저장된다. 이 영역은 JVM 당 하나만 생성이 되며 JVM 의 모든 Thread 들이 Method Area 을 공유한다. JVM 구동 시작 시에 생성이 되며, 종료 시까지 유지된다.

## Heap

Heap 영역에는 문자열에 대한 정보를 가진 **String Pool** 뿐만 아니라 실제 데이터를 가진 **인스턴스**, **배열** 등이 저장된다. JVM 당 하나만 생성이 되고, 해당 영역이 가진 데이터는 모든 Java Stack 영역에서 참조되어 Thread 간 공유가 되므로 동기화를 신경써야 한다. Heap 영역이 가득 차게 되면 OutOfMemoryError이 발생한다. Heap 영역은 GC의 주 대상이다.

## Stack Area

각 Thread 별로 따로 할당되는 영역이다. 각각의 Thread 별로 메모리를 따로 할당하기 때문에 동시성 문제에서 자유롭다. 각 Thread 들은 메소드를 호출할 때마다 Stack에 Frame 이라는 단위를 추가(push)한다. 메소드가 결과를 반환하면 해당 Frame 은 Stack 으로부터 제거(pop)가 된다. Frame 은 메소드에 대한 정보를 가지고 있는 **Local Variable**, **Operand Stack** 그리고 **Constant Pool Reference**로 구성이 되어 있다. Local Variable 은 메소드 안의 지역 변수들을 가지고 있다. Operand Stack은 메소드 내 연산을 위해서, 바이트 코드 명령문들이 들어있는 공간이다. Constant Pool Reference 는 Constant Pool 참조를 위한 공간이다. Java Stack 영역이 가득 차게 되면 StackOverflowError이 발생한다.

## PC(Program Counter) Register

Java 에서 Thread 는 각자의 메소드를 실행한다. 이때, Thread 별로 동시에 실행하는 환경이 보장되어야 하므로 **최근에 실행한 명령어 주소값**을 저장할 공간이 필요한데, 이 부분을 PC Registers 영역이 관리하여 추적한다. Thread 들은 각각 자신만의 PC Registers 를 가지고 있다. 만약 실행했던 메소드가 네이티브하다면 undefined가 기록이 된다.

## Native Method Stack

Java 로 작성된 프로그램을 실행하면서, 순수하게 Java 로 구성된 코드만을 사용할 수 없는 시스템의 자원이나 API 가 존재한다. 다른 프로그래밍 언어로 작성된 메소드들을 Native Method 라고 한다. Native Method Stack은 **Java로 작성되지 않은 메소드**를 다루는 영역이다. C Stacks 라고 부르기도 한다. Native Method가 실행될 경우 Stack에 해당 프레임이 쌓이게 된다. Thread가 생성되면 Native Method Stack도 동일하게 생성된다.

## 참고
https://tecoble.techcourse.co.kr/post/2021-08-09-jvm-memory/
