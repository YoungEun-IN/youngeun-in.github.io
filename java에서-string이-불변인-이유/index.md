# Java에서 String이 불변인 이유


## Java에서 String이 불변인 이유

String은 기본 타입(Primitive Type)이 아닌 참조 타입(Reference Type)이다. 즉, String은 클래스이다.

Java에서 String 객체를 리터럴로 선언하면 특별히 스택 메모리에 직접 저장되는 것이 아니라 Heap 영역 중에서 String constant pool이라는 곳에 메모리를 할당받아 거기에 값을 저장한다. 그리고 String 변수는 그 주소 값을 참조하게 된다. 따라서 Java에서 String은 불변성을 띄게 된다.

Java에서 String이 불변인 이유는 다음과 같다.

1. String pool을 통해 String을 관리함으로써 Java는 Runtime에서 Heap 영역의 많은 메모리를 절약할 수 있다. 왜냐하면 같은 값을 갖는 String에 대해 같은 메모리를 참조하게 할 수 있기 때문이다.
2. String이 불변이 아니라면 보안상의 문제를 야기할 수 있다. 예를 들어, DB의 username과 password 라던가, 소켓 통신에서 host와 port에 대한 정보가 String으로 다루어지기 때문에 String이 불변이라야 해커의 공격으로부터 값이 변경되는 것을 예방할 수 있다.
3. String이 불변이기 때문에 멀티 쓰레딩 환경에서 안전(thread-safe)한다. 값의 변경 가능성이 없기 때문에 멀티 쓰레딩 환경에서 동기화 문제를 걱정하지 않아도 된다.
4. Java는 String의 hashcode를 생성 단계에서부터 캐싱한다. 따라서 String의 hashcode는 쓰일 때마다 매번 계산되지 않는다. 이 특징은 특히 객체의 hashCode를 Key로 사용하는 HashMap의 경우에 효과를 발휘한다. 다른 객체는 키로 쓰일 때마다 hashCode를 계산하는데 비해 String은 캐싱을 하고 있기 때문에 다른 객체를 Key로 했을 때보다 String을 Key로 했을 때 더 빠른 속도로 사용할 수 있다.

## 참고

https://readystory.tistory.com/139

