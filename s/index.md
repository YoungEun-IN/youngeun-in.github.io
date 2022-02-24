# equals와 hashCode


## equals 메소드
Object 클래스에 정의된 equals 메소드는 다음과 같다. 

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```    
단순히 Object의 ==로 비교하는 것을 확인할 수 있다.

두 객체의 내용이 같은지 확인하려면 equals 메소드를 Override하면 된다.

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    AttachFile that = (AttachFile) o;
    return Objects.equals(id, that.id) && Objects.equals(fileName, that.fileName) && Objects.equals(minutes, that.minutes);
}
```

## hashCode 메소드
Object 클래스에 정의된 hashCode 메소드는 인스턴스가 다르면 구성 내용에 상관없이 전혀 다른 해시 값을 반환하도로 정의되어 있다.

```java
public static int hashCode(Object o) {
    return o != null ? o.hashCode() : 0;
}

단순히 Object의 ==로 비교하는 것을 확인할 수 있다.

두 객체의 내용이 같은지 확인하려면 equals 메소드를 Override하면 된다.

```java
    @Override
        return Objects.hash(id, fileName, minutes);
    }
```

## equals와 hashCode는 왜 같이 재정의해야 할까?

hash 값을 사용하는 Collection(HashMap, HashSet, HashTable)은 객체가 논리적으로 같은지 비교할 때 아래 그림과 같은 과정을 거친다.

hashCode 메소드의 반환값을 이용해서 검색의 범위를 확 줄여버리고, 해당 부류 내에 존재하는 데이터의 내용 비교는 equals 메소드를 통해서 진행한다.

Object 클래스의 hashCode 메서드는 객체의 고유한 주소 값을 int 값으로 변환하기 때문에 객체마다 다른 값을 리턴한다. 두 개의 Car 객체는 equals로 비교도 하기 전에 서로 다른 hashCode 메서드의 리턴 값으로 인해 다른 객체로 판단된 것이다.

성능에 아주 민감하지 않은 대부분의 프로그램은 간편하게 Objects.hash 메서드를 사용해서 hashCode 메서드를 재정의해도 문제없다. 민감한 경우에는 직접 재정의해주는 게 좋다.


