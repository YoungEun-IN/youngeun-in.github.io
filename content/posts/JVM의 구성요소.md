---
title: JVM의 구성요소
date: 2022-03-01T15:12:26+09:00
categories:
  - java
tags: 
  - jvm
---

## JVM의 구성요소

JVM은 크게 **Class loader, Runtime Data Areas, Excution Engine** 3가지로 구성된다.

![jvm](https://user-images.githubusercontent.com/52314663/99345185-15928100-28d5-11eb-985c-75c751468169.png)

### Class Loader(클래스 로더)

Java 컴파일러로 컴파일된 Java 바이트 코드를 **Runtime 시점에** **JVM내로 가져오고 Runtime Data Area(JVM 메모리 영역)에 배치해주는 역할**을 한다. 

### Runtime Data Area

JVM의 메모리 영역에 해당하며 Method Area, Heap Area, Stack Area, PC Register, Native Method Stack으로 세분화된다.

- Method Area : Class, Interface, Method, Field, Static, Final 변수 등을 보관하는 영역
- Heap Area : new 키워드로 생성된 객체나 배열 인스턴스를 보관하는 영역, Runtime 시 동적으로 생성되고 JVM이 사용여부를 판단 후 Garbage Collection을 하는 영역
- Stack Area : Heap Area에 동적으로 생성되어 보관되있는 객체(참조 타입)의 래퍼런스를 보관하는 영역, 기본 타입의 경우 직접 Stack Area에 보관됨
- PC Register : Thread가 생성될 때 생성되는 영역으로서 Thread마다 하나의 영역을 가진다. JVM이 현재 수행중인 명령의 주소를 가짐
- Native Method Stack : Java외 다른 언어로 작성된 Native Code를 위한 Stack 영역

### Execution Engine

Class Loader를 통해 Runtime Data Area영역에 배치된 바이트 코드를 실행하는 역할을 한다.
