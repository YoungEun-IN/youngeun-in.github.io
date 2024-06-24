# Garbage Collection


## Garbage Collection
Garbage Collection, GC는 **JVM 상에서 더 이상 사용되지 않는 데이터가 할당되어있는 메모리를 해제시키는 작업**이다. JVM에서 자동으로 동작하기 때문에 특별한 경우가 아니면 메모리 관리를 개발자가 직접 해줄 필요가 없다.

참조되고 있는지에 대한 개념을 reachability라고 하고, 유효한 참조를 reachable, 유효하지 않은 참조를 unreachable이라고 한다. Garbace Collector는 unreachable 한 객체들을 garbage라고 인식한다.

![image](https://user-images.githubusercontent.com/46465928/158065049-ea462718-a2dc-4382-8ede-f8eae6b6c89a.png)

Heap 영역 내부의 객체들은 Method Area, Stack, Native Stack에서 참조되면 reachable로 판정된다. 이렇게 reachable로 인식되게 만들어주는 JVM Runtime Area들을 root set 이라고 한다. reachable 객체가 이 참조하고 있는 다른 객체는 reachable이 된다. 반면에 root set에의해 참조되고 있지 않은 객체들은 unreachable로 판정이 되어 GC 의 대상이 된다.

{{< admonition note "Stop-The-World" true >}}
JVM 은 GC를 통해 여유 메모리를 확보할 수가 있다. 하지만 빈번한 GC는 프로그램의 성능을 저하시킨다. **GC가 일어나면 GC를 담당하는 쓰레드를 제외한 모든 쓰레드들은 작동이 일시적으로 정지되게 되며, 이를 Stop-The-World 현상이라고 한다.** 따라서 적절한 빈도의 GC가 실행되도록 하여 Stop-The-World 시간을 줄여 쓰레드가 정지되는 시간을 줄이는 것이 중요하다.
{{< /admonition >}}

## GC 기본 동작

JVM GC는 알고리즘에 관계 없이 매커니즘은 동일하다. JVM 힙 메모리의 사용가능한 모든 Object를 트래킹해서 참조되지 않는 것들을 폐기한다.

### Weak Generational Hypothesis

Weak Generational Hypothesis 가설은 **대부분의 객체는 빠르게 unreachable 한 상태로 전환이 된다고 본다. 또한, Heap 메모리 영역 기준으로 오래된 영역에서 최신 영역으로의 참조는 적게 존재한다고 가정한다.**
실제로 이 가설은 여러 관찰을 통해서 높은 경향성을 가지는 것으로 증명이 되었다.

### Mark And Sweep Algorithm

가장 기본적인 알고리즘이다. **root set으로부터 출발하여, 참조되는 객체들에 대해서 마크를 한다. 이 단계를 Mark Phase라고 한다. 이후에는 마크되지 않은 객체들을 추적하여 삭제한다. 삭제하는 단계를 Sweep Phase라고 한다.**

![image](https://user-images.githubusercontent.com/46465928/158065455-e8bdb38c-ae69-49f5-bce6-222622c2da42.png)

Mark And Sweep 알고리즘은 메모리가 단편화되는 단점이 있다. 메모리에서의 단편화는 데이터가 정렬되지 않은 조각으로 나뉘어져 절대적인 크기는 충분하지만 추가적으로 메모리 할당이 되기 힘든 상태를 의미한다.

### Mark And Compact Algorithm

Mark And Compact 알고리즘은 Mark And Sweep 알고리즘처럼 참조되는 객체들에 대해서 마크를 하고, 참조되지 않으면 삭제한다. 이후에 **메모리를 정리하여 메모리 단편화를 해결**한다. 많은 GC 방식들이 Mark And Compact 알고리즘을 바탕으로 하여 구현되어 있다.

## Heap의 세부 영역
**Heap은 Eden, Survivor0, Survivor1, Old , Perm 으로 나누어진다.** Young Gen 이라고 불리는 비교적 신생 데이터 부분은 Eden, Survivor0, Survivor1이다. Eden에는 new 키워드를 통해 새롭게 생성된 인스턴스가 위치하며, 이후에는 Survivor로 이동하게 된다.

![image](https://user-images.githubusercontent.com/46465928/158065379-3b8f98e7-1090-430f-b9d0-6efe86e70e80.png)

## GC 종류

GC의 종류는 아래와 같이 나눌 수 있다.
- Major GC - Old, Perm 영역에서 발생하는 GC
- Minor GC - Young 영역에서 발생하는 GC
- Full GC - 메모리 전체를 대상으로 하는 GC

### Minor GC
Minor GC 는 JVM 의 Young 영역에서 일어나는 GC이다. Young 에 위치한 각각의 영역이 가득 차게 되어 더 이상 새로운 객체를 생성할 수 없을 때 마크된 영역이 다음 영역으로 복사가 되면서 이루어진다. 마크가 된 영역만 복사되기 때문에 삭제는 이루어지지 않는다.

![image](https://user-images.githubusercontent.com/46465928/158065701-b27a5c82-6b9f-4ff0-9ad4-62490fc44fa0.png)

### Major GC
Major GC 는 Old 영역에서 이루어진다. 상당히 긴 시간 Stop-The-World 가 이루어지며, 이는 Java 프로그램에 영향을 준다. 이를 해결하기 위해서 여러 GC 방식들이 선택 및 적용된다.

## GC 알고리즘

각 GC의 종류에 따라 성능에 크게 영향을 주며, GC의 알고리즘 종류는 크게 아래 4가지가 있다.

- **Serial GC**(Java 1.3 및 이전 default GC)
   - 작동 방식:
     - Mark (표시): 살아있는 객체를 식별한다.
     - Sweep (제거): 죽은 객체를 제거한다.
     - Compact (압축): 남아있는 객체를 힙의 한쪽으로 이동시켜 단편화를 줄인다.
    - 장점:
      - 간단한 구현: GC 알고리즘이 단순하여 이해하고 디버깅하기 쉽다.
      - 낮은 오버헤드: 추가적인 스레드를 사용하지 않아 메모리와 CPU 오버헤드가 적다.
    - 단점:
      - 긴 멈춤 시간: 단일 스레드로 동작하기 때문에 GC가 실행되는 동안 애플리케이션이 멈춘다.
        - 단일 스레드로 동작하기 때문에, 가비지 컬렉션 작업이 진행되는 동안 멀티코어 프로세서 환경에서도 나머지 코어들은 유휴 상태로 남게 된다. 이는 시스템 자원을 비효율적으로 사용하는 결과를 초래한다.
    - 적용 사례:
      - 작은 힙 메모리를 사용하는 애플리케이션.
      - 단일 코어 또는 적은 수의 코어를 가진 시스템.
- **Parallel GC**(Java 1.4 ~ Java 8 default GC)
  - 작동 방식:
    - 여러 스레드를 사용하여 Mark, Sweep, Compact 단계를 병렬로 수행한다.
  - 장점:
    - 향상된 처리량: 멀티코어 CPU 환경에서 더 많은 객체를 동시에 처리하여 GC 시간을 단축한다.
  - 단점:
    - 응답 시간 증가: GC 수행 시 멈춤 시간이 길어질 수 있어 실시간 응답이 중요한 애플리케이션에는 적합하지 않다.
  - 적용 사례:
    - 처리량(throughput)을 중시하는 대규모 애플리케이션.
    - 멀티코어 시스템.
- **G1 (Garbage-First) GC**(Java 9 및 이후 default GC)
  - 작동 방식:
    - 힙을 여러 개의 고정 크기 영역(region)으로 나눈다.
    - 각 영역을 개별적으로 관리하며, GC는 우선순위가 높은 영역부터 수집한다.
    - Young Generation과 Old Generation을 관리하며, 필요 시 Mixed GC를 통해 Old Generation도 수집한다.
  - 장점:
    - 예측 가능한 멈춤 시간: 목표 멈춤 시간에 맞춰 GC 작업을 조정할 수 있다.
    - 효율적인 메모리 관리: 메모리 단편화를 줄이고, 큰 힙에서도 효율적으로 동작한다.
  - 단점:
    - 설정의 복잡성: 최적의 성능을 위해 다양한 파라미터를 조정해야 할 수 있다.
    - 초기 튜닝 필요: 최적의 성능을 얻기 위해 초기 튜닝이 필요할 수 있다.
  - 적용 사례:
    - 대규모 힙 메모리를 사용하는 애플리케이션.
    - 예측 가능한 멈춤 시간이 중요한 시스템.
- **Z GC**(Java 11부터 적용 가능)
  - 작동 방식:
    - Region-Based: 힙을 작은 영역으로 나누어 관리한다.
    - Concurrent: 대부분의 GC 작업을 애플리케이션 스레드와 병행하여 수행한다.
    - Load Barrier: 객체 참조 시 추가 작업을 수행하여 낮은 멈춤 시간을 유지한다.
  - 장점:
    - 매우 낮은 지연 시간: 10ms 이하의 매우 낮은 멈춤 시간을 제공한다.
    - 대규모 힙 지원: 수 TB 크기의 힙에서도 효율적으로 동작한다.
  - 단점:
    - 제한된 JVM 지원: 특정 버전의 JVM에서만 지원된다.
    - 상대적으로 새로운 기술: 비교적 최신 기술로, 사용 경험이 제한적일 수 있다.
  - 적용 사례:
    - 매우 낮은 지연 시간이 중요한 대규모 애플리케이션.
    - 실시간 응답이 중요한 시스템.

## G1 GC

### Minor GC
한 지역에 객체를 할당하다가 해당 지역이 꽉 차면 다른 지역에 객체를 할당하고, Minor GC가 실행된다. G1 GC는 각 지역을 추적하고 있기 때문에, 가비지가 가장 많은(Garbage First) 지역을 찾아서 Mark and Sweep를 수행한다.
Eden 지역에서 GC가 수행되면 살아남은 객체를 식별(Mark)하고, 메모리를 회수(Sweep)한다. 그리고 살아남은 객체를 다른 지역으로 이동시키게 된다. 복제되는 지역이 Available/Unused 지역이면 해당 지역은 이제 Survivor 영역이 되고, Eden 영역은 Available/Unused 지역이 된다.

### Major GC(Full GC)
시스템이 계속 운영되다가 객체가 너무 많아 빠르게 메모리를 회수 할 수 없을 때 Major GC(Full GC)가 실행된다. 그리고 여기서 G1 GC와 다른 GC의 차이점이 두각을 보인다.
기존의 다른 GC 알고리즘은 모든 Heap의 영역에서 GC가 수행되었으며, 그에 따라 처리 시간이 상당히 오래 걸렸다. 하지만 G1 GC는 어느 영역에 가비지가 많은지를 알고 있기 때문에 GC를 수행할 지역을 조합하여 해당 지역에 대해서만 GC를 수행한다. 그리고 이러한 작업은 Concurrent하게 수행되기 때문에 애플리케이션의 지연도 최소화할 수 있다.
G1 GC는 어떠한 GC 방식보다 처리 속도가 빠르며 큰 메모리 공간에서 멀티 프로세스 기반으로 운영되는 애플리케이션을 위해 고안되었다. 또한 G1 GC는 다른 GC 방식의 처리속도를 능가하기 때문에 Java9부터 기본 가비지 컬렉터(Default Garbage Collector)로 사용된다.

## 참고
https://tecoble.techcourse.co.kr/post/2021-08-30-jvm-gc/

