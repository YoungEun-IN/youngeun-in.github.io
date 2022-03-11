# Java 실행 과정


![image](https://user-images.githubusercontent.com/46465928/157829030-f5dd6848-f6e9-4154-9828-067acf258038.png)

## 1. 컴파일
자바 소스 파일(\*.java)을 Java Compiler가 JVM이 해석할 수 있는 파일인 Java ByteCode (\*.class)파일로 변환한다. Java Compiler는 Java 설치 시  Javac.exe라는 실행 파일 형태로 존재한다.

## 2. Class Loader를 통해 가져온 후 Runtime Data Area에 배치
ClassLoader는 크게 Loading, Linking, 그리고 Initialization 3가지 역할을 한다.

### Loading
.class 확장자를 가진 클래스 파일은 각 디렉터리에 흩어져 있다. 또한, 기본적인 라이브러리의 클래스 파일들은 $JAVAHOME_ 내부 경로에 존재한다. **각각의 클래스 파일들을 찾아서 JVM 의 메모리에 탑재**해주는 역할을 하는 것이 ClassLoader의 역할이다.

클래스 로더는 .class 파일을 읽어 바이트 코드를 메소드 영역(Method Area)에 저장한다.

각 .class 파일은 JVM에 의해 메소드 영역에 다음의 정보들을 저장한다.
* 로드된 클래스를 비롯한 그의 부모 클래스의 정보
* class 파일이 Class, Interface, Enum와 관련 여부
* 변수나 메소드의 정보 등

.class 파일이 로딩된 후에는, JVM은 힙 메모리 영역에 이 파일이 나타내는 클래스 유형의 객체를 생성한다.

### Linking
Linking 은 **로드된 클래스 파일들을 검증하고, 사용할 수 있게 준비**하는 과정을 의미한다. Verification, Preparation, 그리고 Resolution이라는 세 가지 단계로 이루어진다.
* Verification : 클래스 파일이 유효한지를 확인하는 과정이다. 클래스 파일이 JVM 의 구동 조건대로 구현되지 않았을 경우에는 VerifyError를 던지게 된다.
* Preparation : 클래스 및 인터페이스에 필요한 static field 메모리를 할당하고, 이를 기본값으로 초기화한다. 기본값으로 초기화된 static field 값들은 이후 Initialization 과정에서 코드에 작성한 초기값으로 변경된다.
* Resolution : Symbolic Reference 값을 JVM 의 메모리 구성 요소인 Method Area 의 런타임 환경 풀을 통하여 Direct Reference 라는 메모리 주소 값으로 바꾼다. 

### Initialization
Initialization 단계에서는 **클래스 파일의 코드를 읽고 Java 코드에서의 `class`와 `interface`의 값들을 지정한 값들로 초기화**한다. 여기에 `static field`도 포함된다. 이때, JVM 은 멀티 쓰레딩으로 작동을 하며, 같은 시간에 한 번에 초기화를 하는 경우가 있기 때문에 초기화 단계에서도 동시성을 고려해주어야 한다. 

## 3. 실행 시점에 Execution Engine에서 바이트코드를 기계어로 해석
`Java Native Interface(JNI)`, `Native Method Libraries`와 상호작용하며 해석한다.
* Java Native Interface(JNI) : Native Method Libraries와 상호작용하고 실행에 필요한 네이티브 라이브러리(C, C++)을 제공하는 인터페이스이다. JVM은 C/C++ 라이브러리를 통해 호출할 수 있고, 특정 하드웨어와 관련된 C/C++ 라이브러리로 호출 될 수도 있다.
* Native Method Libraries : Execution Engine으로 인해 요구되는 네이티브 라이브러리(C,C++)의 모음이다.

해석 방식은 `인터프리터 방식` 혹은 `JIT(Just-In-Time) Compiler 방식`이 있다.

* 인터프리터(Interpreter) : 바이트코드를 한줄씩 해석한다. 단점은 여러번 하나의 메소드를 호출할 경우 매번 해석을 요청해야하기 때문에 비효율적이다.
* Just-In-Time(JIT) Compiler : 같은 코드를 매번 해석하지 않고 기계어로 번역하면서 캐싱한다. 해당 파일이 다시 기계어로 컴파일 된다면 **이전 파일과 비교해서 바뀐 부분은 새롭게 컴파일하고 변동 사항이 없는 부분은 캐싱된 코드를 사용**한다. 메소드의 반복 호출을 확인할 때마다 직접 네이티브 코드로 제공함으로써 효율성이 증가된다.

## 참고
https://velog.io/@zayson/%EB%B0%B1%EA%B8%B0%EC%84%A0%EB%8B%98%EA%B3%BC-%ED%95%A8%EA%BB%98%ED%95%98%EB%8A%94-live-study-1%EC%A3%BC%EC%B0%A8-JVM%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EB%A9%B0-%EC%9E%90%EB%B0%94-%EC%BD%94%EB%93%9C%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%8B%A4%ED%96%89%ED%95%98%EB%8A%94-%EA%B2%83%EC%9D%B8%EA%B0%80

https://mygumi.tistory.com/115

