# 자바 제네릭(Generics)


제네릭이란 JDK 1.5부터 도입한 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법이다. 제네릭이 갖는 의미는 ‘일반화’이다. 그리고 자바에서 그 일반화의 대상은 자료형이다.

## 용어
- **Box\<T\> 클래스에서 사용된 T를 가리켜 ‘타입 매개변수(Type Parameter)’라 한다.** 메소드의 매개변수와 유사하게 자료형 정보를 인자로 전달받는 형태이기 때문이다. 
- **\<\>안에 전달되는 것을 가리켜 ‘타입 인자(Type Argument)’라 한다.** 타입 매개변수 T에 전달되는 인자로 바라보고 그렇게 이름을 지어준 것이다.
- Box\<Apple\> aBox = new Box\<Apple\>(); -> **Box\<Apple\>을 가리켜 ‘매개변수화 타입(Parameterized Type)’이라 한다.** 자료형 Apple이 타입 매개변수 T에 전달되어 Box< Apple>이라는 새로운 자료형이 완성된 것이기 때문에 ‘매개변수화 타입’이라 부른다.

## 제네릭 필요성
### 불필요한 타입 변경을 없애준다.

다음 예제를 보자.

```java
public static void main(String[] args) {
    List numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    int sum = 0;
    for (Object number : numbers) {
        sum += (int) number;
    }
}
```

List에 있는 모든 숫자를 더하는 로직이다. List에 타입 지정을 안 했기 때문에 Object로 타입이 지정되고 더하는 부분에서 형 변환을 직접 해줘야 하는 번거로움이 있다. 위 예제에서는 형 변환을 한 번밖에 안 했지만 만약 타입 지정을 안 한 List가 사용되는 곳이 1000군데가 넘는다면 1000군데서 전부 예제처럼 직접 형변환을 해줘야 하는 번거로움이 있다.

아래와 같이 제네릭을 사용한다면

```java
public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    int sum = 0;
    for (Integer number : numbers) {
        sum += number;
    }
}
```

불필요한 형 변환을 안 해도 되고 코드도 더 깔끔해진다.

### 타입 안전성이 있다.

```java
public static void main(String[] args) {
    List numbers = Arrays.asList("1", "2", "3", "4", "5", "6");
    int sum = 0;
    for (Object number : numbers) {
        sum += (int) number;
    }
}
```

int형으로 형 변환을 해주며 더해주지만 List의 요소가 int형이라는 보장이 없다. 위 예제와 같이 List에 문자열을 넣어주어도 컴파일 에러가 발생하지 않고 런타임에 ClassCastException 이 터지게 된다. 컴파일 시 타입을 체크하고 에러를 찾아낼 수 있는 컴파일 언어의 장점을 발휘하지 못하는 셈이다.

제네릭을 사용했다면 아래와 같이 컴파일 시에 의도하지 않은 타입이 들어오는 걸 막을 수 있다.

![image](https://user-images.githubusercontent.com/46465928/160351882-44fcae90-6cab-47b2-8456-5f530aca7c54.png)

## 제한된 제네릭

제네릭은 원하는 타입이 있을 때도 모든 타입이 들어올 수 있는 문제가 있다.

```java
public class Car<T> {
    private final T name;

    public Car(T name) {
        this.name = name;
    }
    ...
}
```

위와 같은 Car 클래스 인스턴스 변수 name은 문자 관련 타입 지정되기를 원해도 아래와 같이 아무 타입이나 들어올 수 있다.

```java
public static void main(String[] args) {
    Car<Integer> car = new Car<>(1);
}
```

이럴 때 타입 뒤에 extends 키워드를 사용해 타입을 제한시킬 수 있다.

```java
public class Car<T extends CharSequence> {
    private final T name;

    public Car(T name) {
        this.name = name;
    }
    ...
}
```

CharSequence 인터페이스 하위 객체들만 데이터 타입으로 지정할 수 있게 함으로써 인스턴스 변수 name에 문자열 관련 타입만 지정할 수 있게 되었다.

```java
public static void main(String[] args) {
    Car<String> car = new Car<>("sports");
}
```

Collections.sort에서도 제한된 제네릭을 사용하고 있다.

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    list.sort(null);
}
```

정렬하기 위해서 List에 지정될 타입 T를 Comparable 인터페이스 하위 타입으로 제한시킨 것이다.

## 참고
https://tecoble.techcourse.co.kr/post/2020-11-09-generics-basic/

