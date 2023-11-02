---
title: Java에서 String을 생성하는 방식
date: 2023-02-16T15:12:26+09:00
categories:
  - java
tags: 
  - string-pool
  - intern()
---

## Java에서 String을 생성하는 방식

String 클래스의 경우 다른 참조 자료형과 달리 불변의 원칙이 적용되며, 참조 클래스 중 유일하게 + 연산을 수행할 수 있다.

Java에서 String을 생성하는 방식은 두가지가 있다.

* 생성자를 이용하는 방법 
  * String str = new String("Hello");
* 리터럴을 이용하는 방법 
  * String str = "Hello";

```java
String a = "apple";
String b = new String("apple");
String c = "apple";
String d = new String("apple");

System.out.println(a==b); // false
System.out.prrintln(a==c); // true
```

첫 번째 줄에서 String a가 생성될 때, 'apple'은 리터럴 문자열이므로 Heap영역 중에서 String pool에 위치하게 된다.

두 번째 줄에서 String b는 new연산자를 통해 객체를 생성한다. Heap영역에 위치하지만 String pool이 아닌 다른 주소값을 참조한다. '=='연산자는 같은 메모리를 참조하는가를 비교한다. 때문에  a와 b의 비교(a==b)는 false를 반환하게 된다.

세번째 줄에서 c가 가리키는 'apple'을 String pool에 넣으려고 보니 이미 'apple'이 존재한다. 그러면 c는 a와 같은 String pool의 'apple'을 가리키게 된다. a와 c가 같은 값을 참조하므로 a==c는 true를 반환한다.

이를 그림으로 표현해보면 다음과 같다.

![image](https://user-images.githubusercontent.com/46465928/156321351-4754e772-d537-4190-8bd1-5640c7a45d63.png)

### 생성자로 초기화하는 방식

**String 클래스는 생성자를 통해서 초기화하면 `Heap 메모리`에 해당 값을 할당한다**
생성자를 통해 초기화된 String 변수들은 똑같은 문자열이라고 해도,
Heap 메모리에 할당된 영역이 각 변수마다 다르기 때문에 동등비교연산자(==)로 비교했을 때 false가 리턴된다. 즉, 똑같은 문자열이 Heap 메모리에 여러 개 있다는 의미이다.
이는 동등비교연산자가 객체를 비교할 때 객체의 메모리 주소를 비교하기 때문인데, 문자열 자체를 비교하기 위해서는 String 클래스에서 재정의된 equals() 메소드를 호출 해야한다.

### 리터럴로 초기화하는 방식

**String을 리터럴로 선언할 경우 내부적으로 String의 `intern()`메서드가 호출되게 된다. intern() 메서드는 주어진 문자열이 `String pool`에 존재하는지 검색하고 있다면 그 주소값을 반환, 없다면 문자열을 `String pool`에 넣고 그 주소값을 반환한다.** Heap 영역은 GC의 대상이기 때문에 String Constant Pool에서 참조를 잃은 문자열 객체들은 다시 메모리로 반환된다.

```java
String a = "apple";
String b = new String("apple");
String c = b.intern()

System.out.println(a==b); // false
System.out.prrintln(a==c); // true
```

c의 경우 b의 값인 'apple'이 String pool에 이미 존재하는가를 찾고, 이미 존재하므로 해당 문자열을 반환해 c='apple'이 된다. a와 c는 결국 같은 주소를 참조하게 되어 a==c는 true가 된다.

## 참고
https://simple-ing.tistory.com/3

https://tecoble.techcourse.co.kr/post/2020-09-07-dive-into-java-string/
