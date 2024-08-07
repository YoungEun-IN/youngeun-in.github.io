---
title: 자바의 Nested Class
date: 2022-02-17T10:36:46+09:00
categories:
  - java
tags: 
  - nested-class
---

**Nested Class란 클래스 내부에 선언한 클래스를 말한다.** Nested 클래스는 Static Nested 클래스와 Inner 클래스로 나뉘며, Inner 클래스는 다시 멤버클래스, 지역 클래스와 익명 클래스로 나뉜다. 

Nested 클래스는 클래스 내부에서만 사용되는 경우 논리적으로 묶기 위해서 사용되거나 캡슐화를 통해 구현되어야 할 때 사용된다. 이와 같은 이유로 내부 클래스는 캡술화의 특징을 가지게 되는데, 클래스 내부를 숨기거나 은닉하는데 유용하다.

Nested Class의 구조는 다음과 같다.

![image](https://user-images.githubusercontent.com/46465928/154427384-255b07f7-2cef-48e8-8687-ae01e6a49fcb.png)

## Static Nested Class
**Static Nested Class는 클래스 내부에 구현된 클래스에 static 예약어를 붙여 논리적으로 내부 클래스와 내부 클래스를 구현한 클래스 관계를 묶어준다.**

Static Nested Class는 Static 선언이 갖는 특성이 반영된 클래스이다. 따라서 자신을 감싸는 외부 클래스의 인스턴스와 상관없이 static 네스티드 클래스의 인스턴스 생성이 가능하다. 클래스 내부에서만 공유하고자 하는 변수가 있을 때 private static 키워드로 선언하고 static nest class로 선언하여 캡슐화할 수 있다.

## Member Class
Member Class는 인스턴스 변수, 인스턴스 메소드와 동일한 위치에 정의한다. **멤버 클래스의 인스턴스는 외부 클래스의 인스턴스에 종속적이며, 클래스의 정의를 감추어야 할 때 사용된다.** 멤버클래스의 예시는 반복자가 있다.

![image](https://user-images.githubusercontent.com/46465928/154427580-45475f92-f409-4275-94be-0cfa57e77f90.png)

## Local Class
**Local Class는 중괄호 내에, 특히 메소드 내에 정의하는 클래스이다.**

![image](https://user-images.githubusercontent.com/46465928/154427604-67dce8d4-4900-45b3-9df0-3142f987c1ab.png)
 
## Anonymous Class
**Anonymous Class는 이름이 없는 클래스이다.** Anonymous Class를 객체로 선언하기 위해서는 생성자 혹은 메소드 내부에서 생성자를 호출하고, 이 때 중괄호를 만들어 그 내부에 해당 클래스를 정의하여 구현한다. 

익명 클래스는 런타임 시, JVM이 읽어야 하는 클래스 파일이 줄어 실행 속도가 향상될 수 있다는 장점이 있지만, 남발할 경우 코드 가독성과 유지보수가 어렵다는 단점이 있다.

![image](https://user-images.githubusercontent.com/46465928/154427624-f45ea302-e92c-446e-9a56-9b53991d59bf.png)

## 참고
윤성우의 열혈 JAVA 프로그래밍, 오렌지미디어
