# 스핀락 vs 뮤텍스 vs 세마포어 vs 모니터


동기화(Synchronization)란 여러 프로세스/스레드를 동시에 실행해도 공유 데이터의 일관성을 유지하는 것을 말한다.

상호 배제(Mutual Exclusion)란 동시 프로그래밍에서 공유 불가능한 자원의 동시 사용을 피하기 위해, 즉 동기화를 위해 사용되는 알고리즘을 말한다. 상호 배제는 락(Lock)을 사용해서 달성할 수 있다.

```java
do {
    acquire lock // 여러 프로세스/스레드가 lock을 획득하기 위해 경합
    	[critical section] // lock을 획득한 프로세스/스레드만 임계 영역에서 실행함
    release lock // 작업을 끝내고, lock을 반환함
    	remainder section
} while(true)
```

## 스핀락(Spinlock)

```c
volatile int lock = 0; // global

void critical() {
  while(test_and_set(&lock) == 1); // lock을 획득하려는 시도를 함
  [... critical section] // lock을 얻었다면, 임계 영역으로 진입
  lock = 0; // 작업이 끝난 후, lock을 반환
}

int test_and_set(int* lockPtr) {
    int oldLock = *lockPtr;
    *lockPtr = 1; // 반환하기 직전에 lock의 값을 1로 바꿔줌
    return oldLock;
}
```

위 코드에서는 test_and_set(&lock)을 통해 임계 영역에서의 동시 작업을 방어하고 있다. test_and_set은 CPU의 atomic 명령어로, 하드웨어의 도움을 받아 수행된다. 이것을 활용하면 상호 배제 등을 편리하게 구현할 수 있다.

이와 같이 **while loop를 빠져나가기 위해 계속해서 lock을 확인하여 획득하는 방식을 Spinlock(스핀락) 이라고 한다.** 다시 말해 '락을 가질 수 있을 때 까지 반복해서 시도하는 방식'이다.

**이 방식은 쉽게 Mutual Exclusion을 구현할 수 있다는 장점이 존재하지만, 락이 존재하는지를 계속해서 확인해야 하므로 CPU를 낭비한다는 단점이 있다.** 특히, 디스크 I/O나 네트워크 통신과 같은 상대적으로 느린 작업을 수행하는 임계 영역이 존재한다면 그 동안 다른 스레드는 락을 획득하려고 계속해서 CPU를 점유하게 된다. 따라서 전체 시스템의 성능이 저하될 수 있다.

## 뮤텍스(Mutex)

```c++
class Mutex{
    int value = 1; // 임계 영역에 진입하기 위해 이 value를 취득해야 함
    int guard = 0;
}

Mutex::lock() {
    while(test_and_set(&guard));
    if(value == 0) { // 이미 누가 value를 취득한 상태라면 (lock을 획득할 수 없음)
        ... 현재 스레드를 큐에 넣음; // lock을 획득할 수 있을 때 깨워주기 위해 선입선출 구조인 Queue에 넣음
        guard = 0; & go to sleep
    } else { // lock을 획득할 수 있다면
        value = 0; // lock을 취득함
        guard = 0;
    }
}

Mutex::unlock() {
    while(test_and_set(&guard));
    if(큐에 하나라도 대기중이라면) { // ex) queue.isNotEmpty()
    	그 중에 하나를 깨운다; // ex) queue.poll()
    } else {
    	value = 1;
    }
    guard = 0;
}
```

**뮤텍스에서는 당장 lock을 획득할 수 없다면 FIFO구조의 queue에 현재 스레드를 넣고 쉬러 간다(sleep). 그리고 queue에 대기중인 스레드가 존재한다면 스레드를 깨워서 lock을 부여한다.**

Mutex의 value라는 값도 결국은 여러 프로세스/스레드가 서로 취득하기 위해 경쟁하는 공유 데이터이다. 그렇다면, value라는 값 자체도 그 값을 바꿔줄 때 마다 임계 영역에서 안전하게 바꿔주어야 한다. 그렇지 않으면, 경쟁 상태가 발생할 수 있기 때문이다. 그래서 이 **value값을 임계 영역에서 안전하게 바꿔주기 위한 장치가 필요한데 그것이 guard이다.** value 값을 바꿔주는 로직을 보호하기 위해 guard라는 장치를 통해 보호하고 있다.

### Spinlock vs Mutex

**Spinlock과 다르게 Mutex는 lock을 획득할 수 없으면 스레드가 sleep 상태가 되는데, 이로 인해 Cpu Cycle이 낭비되는 것을 최소화할 수 있다.** 이러한 이유로 Spinlock보다 Mutex Lock이 사용된다.

그러나 멀티 코어 환경이고, 임계 영역에서의 작업이 문맥 교환보다 더 빨리 끝난다면 Spinlock이 Mutex보다 더 이점이 있다.

멀티 코어라는 조건이 붙는 이유는 싱글 코어 환경에서는 Spinlock을 사용한다고 하더라도, lock을 취득하려면 누군가가 쥐고있는 lock을 해제해야 하는데 이러한 과정이 결국 문맥교환을 필요로 하기 때문이다. 반면 **한 코어에서 다른 코어의 락을 획득하기 위해 Spinlock 방식으로 확인하고 있다면, lock을 획득하는 과정에서 문맥교환이 발생하지 않기 때문에 성능 상 이점이 존재한다.**


Mutex는 lock을 획득할 수 없다면 스레드들이 sleep 상태로 대기하다가 lock을 획득할 수 있다면 깨어나서 lock을 획득하여 작업을 이어가는 방식이다. 반대로 Spinlock은 계속해서 lock을 획득할 수 있는지 확인하는 작업이다. **임계 영역에서의 작업이 문맥교환 시간보다 짧다면 Spinlock이 더 우세할 수 있다.**

## 세마포어(Semaphore)

**세마포어는 임계 영역에 프로세스/스레드가 하나 이상 들어가도록 하는 장치이다.** 세마포어는 Signal Mechanism을 가지는데, 이는 Semaphore의 값이 변경될 때 다른 프로세스/스레드에 그 사실을 알리는 방법을 의미한다. 이러한 Signal Mechanism을 통해 실행 순서도 정해줄 수 있게 된다.

```c++
class Semaphore {
    int value = 1; // mutex와 달리 0, 1, 2... 의 값을 가질 수 있음
    int guard = 0;
}

Semaphore::wait() {
    while(test_and_set(&guard));
    if(value == 0) { 
        ... 현재 스레드를 큐에 넣음;
        guard = 0; & go to sleep
    } else {
        value -= 1; // mutex에서는 0으로 바꾸었지만, semaphore에서는 1씩 차감하는 형태
        guard = 0;
    }
}

Semaphore::signal() {
    while(test_and_set(&guard));
    if(큐에 하나라도 대기중이라면) { 
    	그 중에 하나를 깨운다;
    } else {
    	value += 1; // mutex에서는 1로 바꾸었지만, semaphore에서는 1씩 증가하는 형태
    }
    guard = 0;
}
```

전체적으로 Mutex의 코드와 비슷하나, **차이점으로는 value(lock)을 Semaphore는 1 이상 가질 수 있는 부분과 락을 획득했을 때와 해제했을 때 단순히 value를 0, 1로 바꾸는 것이 아니라 현재 값에서 차감하고 증가하는 형태로 바뀐 부분이 있다.**

### Mutex vs Semaphore

**Mutual Exclusion(상호 배제)만 필요하다면 Mutex를, 작업 간의 실행 순서 동기화가 필요하다면 Semaphore가 권장된다.**

세마포어를 통해 작업순서를 동기화하는 예시는 다음과 같다.

```swift
//DispatchSemaphore 초기값 0으로 설정
let semaphore = DispatchSemaphore(value: 0)
print("task A가 끝나길 기다림")
// 다른 스레드에서 task A 실행
DispatchQueue.global(qos: .background).async {
 //task A
 print("task A 시작!")
 print("task A 진행중..")
 print("task A 끝!")
 //task A 끝났다고 알려줌
 semaphore.signal()
}
// task A 끝날때까지는 value 가 0이라, task A 종료까지 block
semaphore.wait()        
print("task A 완료됨")
```

### Mutex vs Binary Semaphore

**value값으로 1을 가지는 Semaphore를 binary Semaphore, 혹은 이진 세마포어라고 하고, value값으로 1이 아닌 값을 가지는 Semaphore를 Counting Semaphore라고 한다.**

차이점은 다음과 같다.

1. Mutex는 lock을 가진 스레드만 lock을 해제 할 수 있지만, Semaphore는 그렇지 않다.
   - Mutex는 lock을 가진 프로세스/스레드만이 lock을 해제할 수 있기 때문에, 누가 lock을 해제할지 예상할 수 있다. 그러나 Semaphore는 wait를 호출하는 존재와 signal을 호출하는 존재가 다를 수 있다. 
   - mutex
     - ```
        mutex->lock(); // mutex의 lock을 가지기 위해서 경합
        [critical section] // lock을 획득했다면, 임계 영역에 진입
        mutex->unlock(); // 작업이 끝났다면, lock을 반환
       ```
    - semaphore
      - ```
        semaphore -> wait(); // mutex에서의 lock
        [critical section]
        semaphore -> signal(); // mutex에서의 unlock
        ```
       

2. Mutex는 Priority Inheritance 속성을 가지지만, Semaphore는 그렇지 않다.
   - P2의 우선순위를 P1만큼 올려주는 작업을 Priority Inheritance라고 부른다. Semaphore는 누가 lock을 해제할지(signal)알 수 없기 때문에 뮤텍스와 달리 우선순위를 상속하는 속성이 존재하지 않는다.

### 모니터(Monitor)

**Monitor는 임계 구역을 지켜내기 위한 방법인 상호 배제를 프로그램으로 구현한 것이다.** 모니터는 프로그래밍 언어 수준에서 제공되며, 순차적으로 사용할  수 있는 공유 자원 혹은 공유 자원 그룹을 할당하는 데 사용된다. 모니터는 이진 세마포어만 가능하다.

공유 자원에 점유 중인 프로세스(스레드)는 Lock을 가지고 있다. 공유 자원을 점유 중인 프로세스(스레드)가 있는 상황에서 다른 프로세스(스레드)가 공유 자원에 접근하려고 하면 외부 모니터 준비 큐에서 진입을 `wait` 한다. Monitor 는 Semaphore 처럼 signal 연산을 보내는 것이 아니라 조건 변수를 사용하여 특정 조건에 대해 대기 큐에 signal 을 보내 작업을 시작시킨다.

#### 모니터의 구성 요소

- mutex lock: critical section에서 mutual exclusion을 보장하는 장치
  - mutex lock을 취득해야 critical section에 진입할 수 있음
  - mutex lock을 취득하지 못한 쓰레드는 큐에 들어간 후 대기(wating) 상태로 전환
- condition variable
  - waiting queue를 가짐
  - 조건이 충족되길 기다리는 쓰레드들이 대기상태로 머무는 곳

#### 자바에서 Monitor 구현

Java 의 `synchronized` 키워드는 스레드 동기화를 할 때 사용하는 대표적인 기법이다. 자바의 모든 인스턴스는 Monitor를 가지고 있으며 Monitor 를 통해 Thread 동기화를 수행한다. 자바의 모니터는 condition variable를 하나만 가지며, `synchronized` 키워드가 붙은 메서드를 사용하려면 Lock을 가지고 있어야 한다.

`synchronized`를 사용하는 방법으로는 메서드 앞에 키워드 명시, 인스턴스로 사용하기가 있다. 동기화가 필요한 메서드 앞에 `synchronized` 키워드만 붙여주면 편리하게 동기화를 적용할 수 있다. 인스턴스로 사용하려면 메서드 내부에서 `synchronized (메서드) { 구현 }` 으로 사용할 수 있다.

Monitor의 Condition Variable을 통해 `wait()`, `notify()` 메서드가 구현되어 있다. Lock 을 가진 스레드가 다른 스레드에 Lock 을 넘겨준 이후에 대기해야 한다면 `wait()` 를 사용하면 된다. 그리고 대기 중인 임의의 스레드를 깨우려면 `notify()` 를 통해 깨울 수 있다. 대기 중인 모든 스레드를 깨우려면 `notifyAll()` 을 통해 깨울 수 있는데, 이 경우에는 하나의 스레드만 Lock 을 획득하고 나머지 스레드는 다시 대기 상태에 들어간다.

## 참고

https://www.youtube.com/watch?v=gTkvX2Awj6g

https://tecoble.techcourse.co.kr/post/2021-10-23-java-synchronize/

