---
title: ASCII, Unicode
date: 2022-03-25T15:12:26+09:00
categories:
  - etc
tags: 
  - ascii
  - unicode
---

인코딩이란, 사용자가 입력한 문자나 기호를 컴퓨터가 이용할 수 있는 신호로 만드는 것을 말한다. 

## 아스키코드(American Standard Code for Information Interchange)
가장 처음 만들어진 인코딩방식이다. 숫자와 문자를 매칭시키는 국제적 규칙이다. **패리티 비트를 제외하고, 128개의 문자조합을 제공하는 7비트 부호로 구성된다.**(패리티 비트는 데이터의 에러를 탐지하기 위해 사용한다.
일곱자리의 이진수에서 '1'이 홀수개라면 끝에 1을, '1'이 짝수개라면 끝에 0을 덧붙인다. 아주 정밀하진 않지만 패리티 비트를 이용해 어느 정도의 에러를 탐지할 수 있다.)

아스키코드만으로는 영어만 표현 가능하며 각 나라별 언어를 표현할 수 없다는 단점이 있다.

## 유니코드(Unicode)
전 세계 언어의 문자를 정의하기 위한 국제 표준 코드이다. **사용중인 운영체제, 프로그램, 언어에 관계 없이 문자마다 고유한 코드 값을 제공한다.**

### UTF-8 (8-bit Unicode Transformation Format)
유니코드를 사용하는 인코딩 방식 중 하나이며 **표현범위에 따라 2,3,4바이트를 사용한다.** UTF-8은 인코딩 형태를 8비트 단위로 한다는 의미를 가지고 있다.

UTF8은 현재 다양한 플랫폼(Windows, Linux, Unix, Mac OS)이 접속하는 인터넷환경에서 de facto standard(사실상의 표준)으로 자리 잡았다. UTF-8의 강점은 **아스키방식과 완벽하게 호환(영문 영역에서는 100% 호환)** 된다는 것이다. 또한 UTF-8은 **endianness 인코딩으로 little endian, big endian의 구분이 없다.** 그래서 이 기종간의 데이터 교환을 위해 현재 가장 선호되는 인코딩이다.

### UTF-16 (16-bit Unicode Transformation Format)
UTF-16은 아스키와 동일한 범위를 표현할 때도 앞에 0으로 채워서 16 비트가 최소로 이용되며 **2 또는 4 바이트 단위로만 사용된다.** endian 방식에 따라 다르게 읽힌다.

### UTF-32 (32-bit Unicode Transformation Format)
**데이터 표현시 4바이트단위를 사용한다.** 메모리 낭비가 심하다는 단점이 있다.

## 참고
https://dar0m.tistory.com/212

https://nhj12311.tistory.com/59
