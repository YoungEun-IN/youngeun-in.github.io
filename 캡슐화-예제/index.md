# 캡슐화 예제


## 객체지향과 캡슐화
객체는 캡슐화된 상태와 외부에 노출되어 있는 행동을 갖고 있으며, 다른 객체와 메시지를 주고 받으면서 협력한다. 객체는 메시지를 받으면 객체 그에 따른 로직(행동)을 수행하게 되고, 필요하다면 객체 스스로 내부의 상태값도 변경한다. 간단히 말해서 객체지향 프로그래밍은 객체가 스스로 일을 하도록 하는 프로그래밍이다.

상태를 가지는 객체를 추가했다면 객체가 제대로 된 역할을 하도록 구현해야 한다. 즉 객체가 로직을 구현하도록 한다.
상태 데이터를 꺼내 로직을 처리하도록 구현하지 말고 객체에 메시지를 보내 일을 하도록 해야한다.

## 캡슐화 예제
### 캡슐화를 위반하는 코드

자동차 경주 게임의 예시 코드를 보자. 
```java
public class Cars {
     public static final String DELIMITER = ",";
     public static final int MINIMUM_TEAM = 2;
     private List<Car> cars;

     public Cars(String inputNames) {
         String[] names = inputNames.split(DELIMITER, -1);
         cars = Arrays.stream(names)
                 .map(name -> new Car(name.trim()))
                 .collect(Collectors.toList());
         validateCarNames();
     }
         ...

    public List<String> findWinners() {
        final int maximum = cars.stream()
                  .map(car -> car.getPosition())	
                  .max(Integer::compareTo)
                  .get();
           
        return cars.stream()
                .filter(car -> car.getPosition() == maximum)
                .map(Car::getName)
                .collect(Collectors.toList());
    } 
         ...
}
```

여러 자동차들 중 position값이 제일 큰 우승 자동차(들)를 구하는 `findWinners()` 메소드를 살펴보자.

Car 객체에서 getPosition() 을 사용해 position 상태값을 직접 꺼내 비교한다. 그러나, Cars에서 position 값을 비교하는 로직을 수행하는 게 맞을까?

Car의 접근 제한자가 private인 멤버변수 position 값 끼리 비교하는 로직이다. 따라서 Car 객체에게 position 값을 비교할 또 다른 Car 객체를 넘겨주고 Car끼리 position을 비교해야 한다. Cars가 아니라 Car에서 해야 하는 일인 것이다.

### 캡슐화를 준수하는 코드

Car 객체 내에서 같은 자동차끼리 position 값을 비교하고, Car 객체 내에서 maximum 과 일치하는지 비교하도록 Cars의 로직을 Car 안으로 옮기도록 하자.

즉, Car 객체에게 position 값을 비교할 수 있도록 메시지를 보내고, Car 객체에게 maximum 값과 자신의 position 값이 같은지 물어보는 메시지를 보내 getPosition() 을 사용하지 않도록 리팩토링하면 결과는 다음과 같다.

```java
public class Car implements Comparable<Car> {
         ...
    public boolean isSamePosition(Car other) {
        return other.position == this.position;
 	}
 	
    @Override
    public int compareTo(Car other) {
        return this.position - other.position;
    }
         ...
}

public class Cars {
         ...
    public List<String> findWinners() {
        final Car maxPositionCar = findMaxPositionCar();
        return findSamePositionCars(maxPositionCar);
    }
    
    private Car findMaxPositionCar() {
        Car maxPositionCar = cars.stream()
            .max(Car::compareTo)
            .orElseThrow(() -> new IllegalArgumentException("차량 리스트가 비었습니다."));
    }

    private List<String> findSamePositionCar(Car maxPositionCar) {
        return cars.stream()
            .filter(maxPositionCar::isSamePosition)
            .map(Car::getName)
            .collect(Collectors.toList());
    }
}
```

`getPosition()` 을 없애는 방향으로 리팩토링 한 코드이다. Car에서 Comparable을 상속받아 `compareTo()`를 구현해 Car내에서 자동차끼리 비교를 해준다. max를 통해 cars 중, 최대 길이의 position을 가진 Car를 찾을 수 있다. 또, `isSamePosition()`을 구현해 Car 내에서 직접 position 값을 비교할 수 있게 된다.

## 참조
https://tecoble.techcourse.co.kr/post/2020-04-28-ask-instead-of-getter/

