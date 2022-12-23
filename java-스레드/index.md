# Java 스레드


## 스레드
모든 자바 어플리케이션은 Main Thread가 main() 메소드를 실행하면서 시작된다.
Main Thread 흐름 안에서 싱글 스레드가 아닌 **멀티 스레드 어플리케이션은 필요에 따라 작업 쓰레드를 만들어 병렬로 코드를 실행할 수 있다.**
싱글 스레드 같은 경우 메인 스레드가 종료되면 프로세스도 종료되지만, 멀티 스레드는 메인 스레드가 종료되더라도 실행 중인 스레드가 하나라도 있다면 프로세스는 종료되지 않는다. 프로세스 내부에서의 멀티 스레드는 공유되는 자원이 있어 하나의 스레드에서 예외가 발생한다면 프로세스 자체가 종료될 수 있다.

## Thread 생성

### Thread 클래스로부터 직접 생성
new를 통해 Thread 클래스 객체를 생성 후 start 메서드를 통해 다른 스레드에서 할 작업을 할당하는 방법이다. 
Thread객체를 생성할 때는 Runnable 인터페이스를 구현한 클래스 객체를 매개변수로 받는다.

```java
public class Task implements Runnable {

    @Override
    public void run() {
        int sum = 0;
        for (int index = 0; index < 10; index++) {
            sum += index;
            System.out.println(sum);
        }
        System.out.println( Thread.currentThread() + "최종 합 : " + sum);
    }

}

```
```java
public static void main(String args[]){
    Runnable task = new Task();
    Thread subTread1 = new Thread(task);
    Thread subTread2 = new Thread(task);
    subTread1.start();
    subTread2.start();
}

```
```
// 결과 (스레드가 끝나는 순서는 랜덥)
0
1
3
6
10
15
21
28
36
45
0
1
3
Thread[Thread-0,5,main]최종 합 : 45
6
10
15
21
28
36
45
Thread[Thread-1,5,main]최종 합 : 45
```

### 익명 객체 생성

```java
public static void main(String args[]){
    Runnable task = new Runnable() {
        public void run() {
            int sum = 0;
            for (int index = 0; index < 10; index++) {
                sum += index;
                System.out.println(sum);
            }
            System.out.println( Thread.currentThread() + "최종 합 : " + sum);
        }
    };
    Thread subTread1 = new Thread(task);
    Thread subTread2 = new Thread(task);
    subTread1.start();
    subTread2.start();
}

```

### 람다식을 통해 익명 객체를 구현

```java
public static void main(String args[]){
    Runnable task = ()-> {
        int sum = 0;
        for (int index = 0; index < 10; index++) {
            sum += index;
            System.out.println(sum);
        }
        System.out.println( Thread.currentThread() + "최종 합 : " + sum);
    };
    
    Thread subTread1 = new Thread(task);
    Thread subTread2 = new Thread(task);
    subTread1.start();
    subTread2.start();
}

```

### Thread 하위 클래스로부터 생성
스레드가 실행할 작업을 Runable 구현클래스 대신 Thread를 상속한 새로운 클래스를 정의하여 run 메소드를 Overriding 하는 방법이다. 
혹은 코드를 단순히 하기 위해 Thead 익명 객체로 작업 스레드 객체를 생성할 수 있다.

```java
// Thread클래스를 상속한 클래스
public class CustomThread extends Thread {
    
    @Override
    public void run() {
        int sum = 0;
        for (int index = 0; index < 10; index++) {
            sum += index;
            System.out.println(sum);
        }
        System.out.println( Thread.currentThread() + "최종 합 : " + sum);

    }
}

```
```java
public static void main(String args[]){
    Thread subTread1 = new CustomThread();
    
    // 익명 객체 생성
    Thread subTread2 = new Thread() {
        public void run() {
            int sum = 0;
            for (int index = 0; index < 10; index++) {
                sum += index;
                System.out.println(sum);
            }
            System.out.println( Thread.currentThread() + "최종 합 : " + sum);
        }
    };
    
    subTread1.start();
    subTread2.start();
}

```

## 동기화 메소드 또는 동기화 블록 
멀티스레드 환경에서의 문제는 스레드들이 객체를 공유하며 작업하는 경우가 생기기 때문에 A스레드와 B스레드가 공유하는 객체가 서로의 작업에 영향을 미치면 안 된다.

동기화 메소드와 동기화 블록 방법은 스레드가 사용 중인 객체를 Lock을 걸어 해당 작업을 진행하는 Thread가 작업을 마칠 때까지 다른 쓰레드가 작업을 하지 못하게 하는 방법이다.

```java
// 동기화 메소드 
public synchronized void setValue( int value ) {
    this.value = value;
    try {
        Thread.sleep(2000);
    } catch (Exception e) {
    }
    System.out.println(Thread.currentThread().getName() + "의 Value 값은 " + this.value +"입니다.");  
}

// 동기화 블록 
public  void setValue( int value ) {
    synchronized ( this ) {
        this.value = value;
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
        }
        System.out.println(Thread.currentThread().getName() + "의 Value 값은 " + this.value +"입니다.");
    }
}
```
하지만 Synchronized 키워드를 너무 남발하면 오히려 프로그램 성능저하를 일으킬 수 있기 때문에 적재적소에 잘 사용해야 한다.

## 참고
https://honbabzone.com/java/java-thread/

