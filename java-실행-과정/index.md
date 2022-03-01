# Java 실행 과정

![Java 실행 과정](https://user-images.githubusercontent.com/52314663/99345263-3ce94e00-28d5-11eb-9ffd-d1832f985e9a.png)

## Java 컴파일 과정

자바 소스 파일(`*.java`)을해당 파일을 Java Compiler가 JVM이 해석할 수 있는 파일인 Java ByteCode (`*.class`)파일로 변환한다. Java Compiler는 Java 설치 시  Javac.exe라는 실행 파일 형태로 존재한다.

### Java 바이트 코드란?

Java 바이트 코드란 Java Source File을 Java Compiler가 컴파일해 만든 파일이다. 
Java Compiler에 의해 변환되는 코드의 명령어 크기가 1byte이기 때문에 Java 바이트코드라고 불린다.
Java 바이트 코드는 JVM이 해석할 수 있는 언어이며 *.class 확장자를 가진다.

---

## Java 실행 과정

1. Java Compiler를 통해 만들어진 Java Bytecode (*.class)를 Class Loader를 통해 가져온 후 Runtime Data Area에 배치한다. 
2. Execution Engine에서 바이트코드를 기계어로 해석한다. Execution Engine은 **인터프리터 방식** 혹은 **JIT(Just-In-Time) Compiler 방식**이 있다.
    - 인터프리터 방식은 소스 코드를 한 문장씩 읽으며 해당 문장을 기계어 코드로 변환시켜준다.
    - JIT Compiler는 실행 시점에 기계어 코드로 변환시켜준다.
3. 프로세서는 기계어에 따라 동작을 수행한다.

### JIT 컴파일러

**JIT(Just-In-Time) 컴파일러**는 JVM의 Execution Engine이 바이트코드를 사용하는 운영체제에 맞는 기계어로 변환시켜주는 방식중 하나이다.

JIT 컴파일러는 프로그램을 실제 실행하는 시점에 기계어를 번역한다. 이렇게만 보면 인터프리터 방식과의 차이점이 없어보이지만 JIT 컴파일러는 같은 코드를 매번 해석하지 않고 기계어로 번역하면서 **캐싱**한다. 해당 파일이 다시 기계어로 컴파일 된다면 **이전 파일과 비교해서 바뀐 부분은 새롭게 컴파일하고 변동 사항이 없는 부분은 캐싱된 코드를 사용**한다.

이러한 방식은 **인터프리터 속도가 느린 점을 JIT 컴파일러로 개선할 수 있는 장점**이 된다.

