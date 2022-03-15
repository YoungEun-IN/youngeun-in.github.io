# Java에서 String을 생성하는 방식


## Java에서 String을 생성하는 방식

Java에서 String을 생성하는 방식은 두가지가 있다.

* new 연산자를 이용하는 방법 
  * String str = new String("Hello");
* 리터럴을 이용하는 방법 
  * String str = "Hello";

new 연산자를 통해서 생성하게 되면 String은 Heap영역에 존재하게 된다.
하지만 리터럴을 이용할 경우 Heap 영역의 String pool이라는 영역에 존재하게 된다.

```java
String a = "apple";
String b = new String("apple");
String c = "apple";
String d = new String("apple");

System.out.println(a==b); // false
System.out.prrintln(a==c); // true
```

첫 번째 줄에서 String a가 생성될 때, 'apple'은 리터럴 문자열이므로 Heap영역 중에서 String pool에 위치하게 된다.

두 번째 줄에서 String b는 new연산자를 통해 객체를 생성합니다. Heap영역에 위치하지만 String pool이 아닌 다른 주소값을 참조한다. '=='연산자는 같은 메모리를 참조하는가를 비교한다. 때문에  a와 b의 비교(a==b)는 false를 반환하게 된다.

세번째 줄에서 c가 가리키는 'apple'을 String pool에 넣으려고 보니 이미 'apple'이 존재한다. 그러면 c는 a와 같은 String pool의 'apple'을 가리키게 된다. a와 c가 같은 값을 참조하므로 a==c는 true를 반환합니다.

이를 그림으로 표현해보면 다음과 같다.

![image](https://user-images.githubusercontent.com/46465928/156321351-4754e772-d537-4190-8bd1-5640c7a45d63.png)

## 리터럴로 선언 시 메모리에서의 동작

String을 리터럴로 선언할 경우 내부적으로 String의 intern()메서드가 호출되게 된다. intern() 메서드는 주어진 문자열이 String pool에 존재하는지 검색하고 있다면 그 주소값을 반환, 없다면 문자열을 String pool에 넣고 그 주소값을 반환한다.

```java
String a = "apple";
String b = new String("apple");
String c = b.intern()

System.out.println(a==b); // false
System.out.prrintln(a==c); // true
```

때문에 c의 경우 b의 값인 'apple'이 String pool에 이미 존재하는가를 찾고, 이미 존재하므로 해당 문자열을 반환해 c='apple'이 되고, a와 c는 결국 같은 주소를 참조하게 되어 a==c는 true가 된다.

## 출처
https://simple-ing.tistory.com/3

