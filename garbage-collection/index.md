# Garbage Collection


## Garbage Collection
Garbage Collection, GC는 **JVM 상에서 더 이상 사용되지 않는 데이터가 할당되어있는 메모리를 해제시키는 작업**이다. JVM에서 자동으로 동작하기 때문에 특별한 경우가 아니면 메모리 관리를 개발자가 직접 해줄 필요가 없다.

참조되고 있는지에 대한 개념을 reachability라고 하고, 유효한 참조를 reachable, 유효하지 않은 참조를 unreachable이라고 한다. Garbace Collector는 unreachable 한 객체들을 garbage라고 인식한다.

![image](https://user-images.githubusercontent.com/46465928/158067335-6bddd61d-bb75-45e2-9577-5bba62463893.png)

Heap 영역 내부의 객체들은 Method Area, Stack, Native Stack에서 참조되면 reachable로 판정된다. 이렇게 reachable로 인식되게 만들어주는 JVM Runtime Area들을 root set 이라고 한다. reachable 객체가 이 참조하고 있는 다른 객체는 reachable이 된다. 반면에 root set에의해 참조되고 있지 않은 객체들은 unreachable로 판정이 되어 GC 의 대상이 된다.

## Stop-The-World

JVM 은 GC를 통해 여유 메모리를 확보할 수가 있다. 하지만 빈번한 GC는 프로그램의 성능을 저하시킨다. **GC가 일어나면 GC를 담당하는 쓰레드를 제외한 모든 쓰레드들은 작동이 일시적으로 정지**되게 되며, 이를 Stop-The-World 현상이라고 한다. 따라서 적절한 빈도의 GC가 실행되도록 하여 Stop-The-World 시간을 줄여 쓰레드가 정지되는 시간을 줄이는 것이 중요하다.

## GC 알고리즘
### Weak Generational Hypothesis

Weak Generational Hypothesis 가설은 **대부분의 객체는 빠르게 unreachable 한 상태로 전환이 된다**고 본다. 또한, Heap 메모리 영역 기준으로 **오래된 영역에서 최신 영역으로의 참조는 적게 존재한다**고 가정한다.
실제로 이 가설은 여러 관찰을 통해서 높은 경향성을 가지는 것으로 증명이 되었다.

### Mark And Sweep Algorithm

가장 기본적인 알고리즘이다. root set으로부터 출발하여, 참조되는 객체들에 대해서 마크를 한다. 이 단계를 Mark Phase라고 한다. 이후에는 마크되지 않은 객체들을 추적하여 삭제한다. 삭제하는 단계를 Sweep Phase 라고 한다.

![image](https://user-images.githubusercontent.com/46465928/158067278-02f8668d-cafd-4518-a1e5-da9aac93f0a9.png)

Mark And Sweep 알고리즘은 메모리가 단편화되는 단점이 있다. 메모리에서의 단편화는 데이터가 정렬되지 않은 조각으로 나뉘어져 절대적인 크기는 충분하지만 추가적으로 메모리 할당이 되기 힘든 상태를 의미한다.

### Mark And Compact Algorithm
Mark And Compact 알고리즘은 Mark And Sweep 알고리즘처럼 참조되는 객체들에 대해서 마크를 하고, 참조되지 않으면 삭제한다. 이후에 메모리를 정리하여 **메모리 단편화를 해결**한다. 많은 GC 방식들이 Mark And Compact 알고리즘을 바탕으로 하여 구현되어 있다.

## Major GC 와 Minor GC

### Heap의 세부 영역
Heap은 Eden, Survivor0, Survivor1, Old , Perm 으로 나누어진다.
Young Gen 이라고 불리는 비교적 신생 데이터 부분은 Eden, Survivor0, Survivor1이다. Eden에는 new 키워드를 통해 새롭게 생성된 인스턴스가 위치하며, 이후에는 Survivor로 이동하게 된다.

![image](https://user-images.githubusercontent.com/46465928/158067296-c166b3ca-c115-463e-893e-34b50f214df3.png)

### Minor GC
Minor GC 는 JVM 의 Young 영역에서 일어나는 GC이다. Young 에 위치한 각각의 영역이 가득 차게 되어 더 이상 새로운 객체를 생성할 수 없을 때 마크된 영역이 다음 영역으로 복사가 되면서 이루어진다. 마크가 된 영역만 복사되기 때문에 삭제는 이루어지지 않는다.

![image](https://user-images.githubusercontent.com/46465928/158067308-89a85aaa-518b-4df9-9523-dc66a05bda41.png)

### Major GC
Major GC 는 Old 영역에서 이루어진다. 상당히 긴 시간 Stop-The-World 가 이루어지며, 이는 Java 프로그램에 영향을 준다. 이를 해결하기 위해서 여러 GC 방식들이 선택 및 적용된다.

## G1 GC
G1 GC의 핵심은 Heap을 동일한 크기의 Region으로 나누고, 가비지가 많은 Region에 대해 우선적으로 GC를 수행하는 것이다. 그리고 G1 GC도 다른 가비지 컬렉션과 마찬가지로 2가지 GC(Minor GC, Major GC)로 나누어 수행된다.

### Minor GC
한 지역에 객체를 할당하다가 해당 지역이 꽉 차면 다른 지역에 객체를 할당하고, Minor GC가 실행된다. G1 GC는 각 지역을 추적하고 있기 때문에, 가비지가 가장 많은(Garbage First) 지역을 찾아서 Mark and Sweep를 수행한다.
Eden 지역에서 GC가 수행되면 살아남은 객체를 식별(Mark)하고, 메모리를 회수(Sweep)한다. 그리고 살아남은 객체를 다른 지역으로 이동시키게 된다. 복제되는 지역이 Available/Unused 지역이면 해당 지역은 이제 Survivor 영역이 되고, Eden 영역은 Available/Unused 지역이 된다.

### Major GC(Full GC)
시스템이 계속 운영되다가 객체가 너무 많아 빠르게 메모리를 회수 할 수 없을 때 Major GC(Full GC)가 실행된다. 그리고 여기서 G1 GC와 다른 GC의 차이점이 두각을 보인다.
기존의 다른 GC 알고리즘은 모든 Heap의 영역에서 GC가 수행되었으며, 그에 따라 처리 시간이 상당히 오래 걸렸다. 하지만 G1 GC는 어느 영역에 가비지가 많은지를 알고 있기 때문에 GC를 수행할 지역을 조합하여 해당 지역에 대해서만 GC를 수행한다. 그리고 이러한 작업은 Concurrent하게 수행되기 때문에 애플리케이션의 지연도 최소화할 수 있다.
G1 GC는 어떠한 GC 방식보다 처리 속도가 빠르며 큰 메모리 공간에서 멀티 프로세스 기반으로 운영되는 애플리케이션을 위해 고안되었다. 또한 G1 GC는 다른 GC 방식의 처리속도를 능가하기 때문에 Java9부터 기본 가비지 컬렉터(Default Garbage Collector)로 사용된다.

## 참고
https://tecoble.techcourse.co.kr/post/2021-08-30-jvm-gc/

