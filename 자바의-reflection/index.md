# 자바의 Reflection


## Reflection API란?
아래와 같이 Car 클래스가 존재한다.

```java
public class Car {
    private final String name;
    private int position;

    public Car(String name, int position) {
        this.name = name;
        this.position = position;
    }

    public void move() {
        this.position++;
    }

    public int getPosition() {
        return position;
    }
}
```

자바의 특징 중 하나인 다형성 덕분에 아래와 같이 객체 생성이 가능하다.

```java
public static void main(String[] args) {
    Object obj = new Car("foo", 0);
}
```

자바는 컴파일러를 사용한다. 즉 컴파일 타임에 타입이 결정된다. obj라는 이름의 객체는 컴파일 타임에 Object로 타입이 결정됐기 때문에 Object 클래스의 인스턴스 변수와 메서드만 사용할 수 있다. 따라서 obj라는 이름의 객체는 Car 클래스의 move 메서드를 사용할 수 없다.

그러므로 아래와 같은 코드는 필연적으로 컴파일 에러가 난다.

```java
public static void main(String[] args) {
    Object obj = new Car("foo", 0);
    obj.move();    // 컴파일 에러 발생 java: cannot find symbol
}
```

생성된 obj라는 객체는 Object 클래스라는 타입만 알 뿐, Car 클래스라는 구체적인 타입은 모른다. 결국 컴파일러가 있는 자바는 구체적인 클래스를 모르면 해당 클래스의 정보에 접근할 수 없다는 것을 알 수 있다. Reflection API는 이렇게 불가능한 일을 마법처럼 가능하게 해준다.

---

위에서 봤던 예제와 똑같은 상황에서 Reflection API를 활용해 Car 클래스의 move 메서드를 호출해보자.

```java
public static void main(String[] args) throws Exception {
    Object obj = new Car("foo", 0);
    Class carClass = Car.class;
    Method move = carClass.getMethod("move");

    // move 메서드 실행, invoke(메서드를 실행시킬 객체, 해당 메서드에 넘길 인자)
    move.invoke(obj, null);

    Method getPosition = carClass.getMethod("getPosition");
    int position = (int)getPosition.invoke(obj, null);
    System.out.println(position);
    // 출력 결과: 1
}
```

move 메서드가 실행되고 0으로 초기화했던 Car 클래스 인스턴스 변수 position이 1로 출력되는 걸 확인할 수 있다. 구체적인 클래스 Car 타입을 알지 못해도 Reflection API를 통해 move 메서드에 접근한 것이다.

```java
Class carClass2 = Class.forName("Car");
```

위의 예제처럼 클래스의 이름만으로도 해당 클래스의 정보를 가져올 수 있다.
다시 말해서 **Reflection API를 통해 클래스의 이름만 가지고도 생성자, 필드, 메서드 등등 해당 클래스에 대한 거의 모든 정보를 가져올 수 있다.**

## Reflection은 어떻게 가능할까?
JVM이 실행되면 사용자가 작성한 자바 코드가 컴파일러를 거쳐 바이트 코드로 변환되어 Runtime Data Area에 저장된다. **Reflection API는 Runtime Data Area에 저장된 정보를 활용한다.** 그래서 클래스 이름만 알고 있다면 언제든 정보를 가져올 수 있는 것이다.

## Reflection을 어디에 활용할 수 있을까?
위에서 살펴봤던 예제 코드를 보면 멀쩡한 Car 객체를 Object 타입으로 생성하고 있다. 실제로 우리가 코드를 작성할 때는 예제와 같이 작성하지 않는다. 그러므로 우리가 코드를 작성하면서 Reflection을 활용할 일은 거의 없다. 구체적인 클래스를 모를 일이 거의 없기 때문이다.

게다가 Reflection은 마법 같은 힘을 가지고 있는 만큼 치명적인 단점들을 가지고 있다. 당연히 사용하지 않을 수 있다면 대부분의 경우 사용하지 않는 게 좋다.

치명적인 단점 중 대표적으로 성능 오버헤드가 있다. 컴파일 타임이 아닌 런타임에 동적으로 타입을 분석하고 정보를 가져오므로 JVM을 최적화할 수 없기 때문이다. 뿐만 아니라 직접 접근할 수 없는 private 인스턴스 변수, 메서드에 접근하기 때문에 내부를 노출하면서 추상화가 깨진다. 이로 인해 예기치 못한 부작용이 발생할 수 있다.

결론적으로 **Reflection은 애플리케이션 개발보다는 프레임워크나 라이브러리에서 많이 사용된다.** 프레임워크나 라이브러리는 사용자가 어떤 클래스를 만들지 예측할 수 없기 때문에 동적으로 해결해주기 위해 Reflection을 사용한다.

실제로 intellij의 자동완성, jackson 라이브러리, Hibernate 등등 많은 프레임워크나 라이브러리에서 Reflection을 사용하고 있다.

Spring Framework에서도 Reflection API를 사용하는데 대표적으로 Spring Container의 BeanFactory가 있다. Bean은 애플리케이션이 실행한 후 런타임에 객체가 호출될 때 동적으로 객체의 인스턴스를 생성하는데 이때 Spring Container의 BeanFactory에서 리플렉션을 사용한다.

Spring Data JPA 에서 Entity에 기본 생성자가 필요한 이유도 동적으로 객체 생성 시 Reflection API를 활용하기 때문이다. Reflection API로 가져올 수 없는 정보 중 하나가 생성자의 인자 정보이다. 그래서 기본 생성자가 반드시 있어야 객체를 생성할 수 있는 것이다. 기본 생성자로 객체를 생성만 하면 필드 값 등은 Reflection API로 넣어줄 수 있다.

## 참고
https://tecoble.techcourse.co.kr/post/2020-07-16-reflection-api/

