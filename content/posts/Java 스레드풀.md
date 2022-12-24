---
title: Java 스레드풀
date: 2022-04-23T15:12:26+09:00
categories:
  - java
tags: 
  - thread-pool
---

## 스레드풀
### 스레드풀 생성
스레드가 기하급수적으로 늘어난다면 스레드 생성과 스케줄링으로 인해 CPU의 메모리 사용량이 늘어나고 어플리케이션의 성능의 저하가 일어난다. 병렬작업의 급증을 막기 위해서는 특정 스레드 개수만큼 작업 컨베이어 벨트 (작업 큐)를 만들어 놓고 작업을 컨베이어 벨트에 올려놓아 작업이 끝난 스레드가 컨베이어에서 작업을 꺼내 기능을 수행하는 방식으로 작업을 구성할 수 있다. 아무리 작업 처리 요청이 폭주하여도 스레드의 전체 개수가 늘어나지 않기 때문에 어플리케이션의 성능이 급격하게 저하되지 않는다. 

자바는 스레드 풀을 생성하고 사용할 수 있도록 `java.util.concurrent.ExecutorService` 인터페이스와 Executors 클래스 메소드 중 `newCachedThreadPool`과 `newFixedThreadPool` 메소드를 제공하고 있다.

```java
// 스레드 풀 생성 
//1. 자동으로 스레드 수 생성
ExecutorService executorServiceWithCached = Executors.newCachedThreadPool();

//2. 원하는 개수만큼 생성
ExecutorService executorServiceWithNum = Executors.newFixedThreadPool(2);

//3. 최대치로 생성
ExecutorService executorServiceWithMax = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

//4. 완전 수동
// ThreadPoolExecutor( 코어 스레드 수, 최대 스레드 개수, 놀고 있는 시간, 놀고있는 시간 단위, 작업 큐 )
ExecutorService executorServiceWithCustom = new ThreadPoolExecutor(3, 100, 120L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
```

{{< admonition >}}
**Executors.newFixedThreadPool(10)**
* 최대 쓰레드를 10개까지 만드는 풀.  동시에 일어나는 업무의 량이 비교적 일정할때 사용한다.

**Executors.newCachedThreadPool()**
* 쓰레드 수의 제한을 두지 않은 방식의 쓰레드풀 방식으로, 새로운 쓰레드 시작 요청이 들어올때마다 하나씩 쓰레드를 생성한다.
* 업무가 종료된 쓰레드들은 바로 사라지지 않고 1분동안 살아있다가 다른 작업 요청이 없으면 사라진다. 짧고 반복되는 스타일의 작업요청이 들어올 경우 유용하다.
{{< /admonition >}}

### 스레드풀 종료
스레드풀의 스레드는 데몬스레드가 아니므로 main이 종료되더라도 작업을 처리하기 위해 계속 실행상태로 남아있다. 때문에 어플리케이션을 종료할 때는 해당 스레드풀을 종료시켜 스레드풀 안의 스레드를 종료상태가 되도록 처리해야 한다.

```java
// 1. 작업 큐에 대기하고 있는 모든 작업이 끝난 뒤 스레드를 종료한다. 
executorServiceWithCached.shutdown();

// 2. 당장 중지한다. 리턴값은 작업큐에 남아있는 작업의 목록이다.
List<Runnable> runable = executorServiceWithCached.shutdownNow();

// 3. 작업은 대기 하지만 모든 작업처리를 특정 시간안에 하지 못하면 작업중인 스레드를 중지하고 false를 리턴한다. 아래는 100초 설정
try {
    boolean isFinish = executorServiceWithCached.awaitTermination(100, TimeUnit.SECONDS);
} catch (InterruptedException e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
}
```

### 작업의 생성
Runable 또는 Callable 구현 클래스로 작업을 생성한다. 둘의 차이는 작업이 끝난 후 리턴 값이 있냐 없느냐다.

```java
// Runable 구현 객체 (익명객체 사용)
Runnable task1 = ()-> {
    for (int index = 0; index < 10; index++) {
        System.out.println("작업 중입니다.");
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}; 

// Callable 구현 
Callable<Boolean> task2 = () ->{
    Boolean isFinish = true;
    
    for (int index = 0; index < 10; index++) {
        System.out.println("작업 중입니다. Call");
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    return isFinish;
};
```

### 작업처리 요청
task를 컨베이어 벨트에 올리는 작업이다. 즉 작업큐에 위 task객체를 넣는 것이다. 

```java
// 1. 리턴 값이 없는 단순 Runnable를 처리
executorServiceWithCached.execute(task1);

// 2. 리턴 가능한 Callable도 넣을 수 있는 메서드
Future<Boolean> returnBoolean =  executorServiceWithCached.submit(task2);

try {
    System.out.println("종료 : " + returnBoolean.get()); //작업 반환을 기다려 얻어온다.
} catch (Exception e) {
}
```

execute와 submit의 차이는 return을 받느냐 못 받느냐의 차이이다. 추가로 execute()는 작업 도중 오류 발생 시 오류가 난 스레드를 스레드풀에서 제거하지만 submit()은 오류가 발생하여도 해당 스레드를 재사용한다.

### 작업완료 통보
submit()은 task 작업을 작업큐에 넣고 바로 Futrue객체를 리턴한다. 해당 객체의 get()메서드를 호출하면 호출한 순간부터 스레드가 작업을 완료할 때까지 대기하고 있다가 완료된 후 결과를 받아온다.

```java
public static void main(String args[]){
    // 스레드 풀 생성 
    //자동으로 스레드 수 생성
    ExecutorService executorServiceWithCached = Executors.newCachedThreadPool();
    
    // Runable 구현 객체 ( 익명구현객체 사용  )
    Runnable task1 = ()-> {
        for (int index = 0; index < 100; index++) {
            System.out.println("작업 중입니다.");
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }; 
    
    // Callable 구현 
    Callable<Boolean> task2 = () ->{
        Boolean isFinish = true;
        
        for (int index = 0; index < 100; index++) {
            System.out.println("작업 중입니다. Call");
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        return isFinish;
    };
    
    // 1. 리턴 값이 없는 단순 Runnable를 처리
    executorServiceWithCached.execute(task1);
    
    // 2. 리턴 가능한 Callable도 넣을 수 있는 메서드.
    Future<Boolean> returnBoolean =  executorServiceWithCached.submit(task2);
    
    // main스레드의 작업이 멈추지 않기 위해 새로운 스레드로 구성
     executorServiceWithCached.execute(()->{
        try {
            if( returnBoolean.get() ) {
                System.out.println("작업이 완벽히 끝났습니다. ");
            } else {
                System.out.println("작업이 끝나지 못했습니다.");
            }
        } catch (Exception e) {
        } 
    });
    
    // main스레드의 작업이 멈추지 않기 위해 새로운 스레드로 구성
    executorServiceWithCached.execute(()->{
        try {
            // 만약 특정 시간 내에 끝났는지 확인하려는 경우
            if( returnBoolean.get(1,TimeUnit.SECONDS) ) {
                System.out.println("작업이 완벽히 끝났습니다. ");
            } 
        } catch (Exception e) {
            System.out.println("작업이 시간내에 끝나지 못했습니다.");
        } 
    });
    
    // 작업 큐에 대기하고 있는 모든 작업이 끝난 뒤 스레드를 종료
    executorServiceWithCached.shutdown();
}

```
### 작업처리 결과를 외부 객체에 저장
```java
public class FirstThread implements Runnable {
    
    ResultShare resultShare; 
    
    //공유객체 외부에서 주입
    public FirstThread ( ResultShare resultShare ) {
        this.resultShare = resultShare;
    }
    
    @Override
    public void run() {
        int result = 0;
        for (int index = 1; index <= 100; index++) {
            result += index;
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        System.out.println("1~100까지의 합은 " + result +"입니다.");
        resultShare.sum(result);
        resultShare.showSum();
    }

}
```
```java
public class SecondThread implements Runnable {
    
    ResultShare resultShare; 
    
    //공유객체 외부에서 주입
    public SecondThread ( ResultShare resultShare ) {
        this.resultShare = resultShare;
    }
    
    @Override
    public void run() {
        int result = 0;
        for (int index = 101; index <= 200; index++) {
            result += index;
        }
        System.out.println("101~200까지의 합은 " + result +"입니다.");
        resultShare.sum(result);
        resultShare.showSum();
    }

}
```

```java
public class ResultShare {
    private int result;
    
    public int sum( int number ) {
        return result+=number;
    }
    
    public void showSum() {
        System.out.println(" 지금 저장된 값은 : " + result + "입니다.");
    }
    
    public int getResult() {
        return result;
    }
}
```

```java
public static void main(String args[]){
    // 스레드 풀 생성 
    //자동으로 스레드 수 생성
    ExecutorService executorServiceWithCached = Executors.newCachedThreadPool();
    
    ResultShare resultShare = new ResultShare();
    
    Runnable task1 = new FirstThread(resultShare);
    Runnable task2 = new SecondThread(resultShare);
            
    // Runable이라도 작업이 끝난 후 Future<리턴 객체>로 무엇을 받을 지 임의로 정할 수 있다. run()의 리턴값으로 아무 값이 없더라도 종료 후에 아무 원하는 객체를 이런 식으로 리턴할 수 있다.
    Future<ResultShare> future1 =  executorServiceWithCached.submit( task1, resultShare );
    Future<ResultShare> future2 =  executorServiceWithCached.submit( task2, resultShare );
    
    // main스레드의 작업이 멈추지 않기 위해 새로운 스레드로 구성
    executorServiceWithCached.execute(()->{
        try {
            ResultShare temp = future1.get();
            temp = future2.get();
           System.out.println("쓰레드 합산이 끝났습니다. 최종 결과는 : " + resultShare.getResult());
        } catch (Exception e) {
        } 
    });
    
    // 작업 큐에 대기하고 있는 모든 작업이 끝난 뒤 스레드를 종료한다. 
    executorServiceWithCached.shutdown();
}
```
```
// 실행 결과
101~200까지의 합은 15050입니다.
 지금 저장된 값은 : 15050입니다.

1~100까지의 합은 5050입니다.
 지금 저장된 값은 : 20100입니다.
쓰레드 합산이 끝났습니다. 최종 결과는 : 20100
```

### 작업 완료 순으로 통보
일반적으로 작업 순서대로 처리가 완료되는 것이 아니기 때문에 끝나는 것은 랜덤이다. 
여러 개의 작업이 차례대로 처리될 필요가 없고 처리 결과도 차례대로 이용할 필요가 없다면 CompletionService 인터페이스를 구현한 ExecutorCompletionService 클래스를 사용하면 된다.

```java
public static void main(String args[]){
    // 스레드 풀 생성 
    //자동으로 스레드 수 생성
    ExecutorService executorServiceWithCached = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    
    ResultShare resultShare = new ResultShare();
    
    Runnable task1 = new FirstThread(resultShare);
    Runnable task2 = new SecondThread(resultShare);
            
    Future<ResultShare> future1 =  executorServiceWithCached.submit( task1, resultShare );
    Future<ResultShare> future2 =  executorServiceWithCached.submit( task2, resultShare );
    
    // 각각의 테스트들의 리턴 결과의 객체를 <ResultShare> 안에 입력 
    CompletionService<ResultShare> compliCompletionService = new ExecutorCompletionService<ResultShare>(executorServiceWithCached);
    
    executorServiceWithCached.submit(()->{
        // 작업 최종 통보가 올때까지 while 문을 통해 대기
        while(true) {
            try {
                // 작업이 끝난 결과를 받아온다.
                Future<ResultShare> take = compliCompletionService.take();
                ResultShare result = take.get();
                
                System.out.println("지금 까지의 합산은 " + result.getResult() + "입니다.");
            } catch ( Exception e) {
                break;
            } 
        }
    });
    
    // 3초 뒤 스레드 강제 중지 
    try {
        Thread.sleep(3000);
        executorServiceWithCached.shutdownNow();
    } catch (InterruptedException e) {
    }
} 
```

```
// 결과 

101~200까지의 합은 15050입니다.
 지금 저장된 값은 : 15050입니다.
1~100까지의 합은 5050입니다.
 지금 저장된 값은 : 20100입니다. 
```

### 콜백 방식의 통보
```java
public class ResultShare {
    private int result;
    //<결과 타입, 첨부타입>
    private static CompletionHandler<ResultShare, String> completionHandler = new CompletionHandler<ResultShare, String>() {
        
        //실패 시 할 일
        @Override
        public void failed(Throwable exc, String attachment) {
            System.out.println("실패하였습니다.");
        }
        
        //성공 시 할 일
        @Override
        public void completed(ResultShare result, String attachment) {
             System.out.println("지금까지의 저장된 합은 " + result.getResult() + "입니다.");
        }
    };
    
    public int sum( int number ) {
        return result+=number;
    }
    
    public void showSum() {
        System.out.println(" 지금 저장된 값은 : " + result + "입니다.");
    }
    
    public int getResult() {
        return result;
    }
    
    public static CompletionHandler<ResultShare, String> getCompletionHandler() {
        return completionHandler;
    }
}
```

```java
public class FirstThread implements Runnable {
    
    ResultShare resultShare; 
    
    //공유객체 외부에서 주입.
    public FirstThread ( ResultShare resultShare ) {
        this.resultShare = resultShare;
    }
    
    @Override
    public void run() {
        int result = 0;
        for (int index = 1; index <= 100; index++) {
            result += index;
        }
        System.out.println("1~100까지의 합은 " + result +"입니다.");
        resultShare.sum(result);
        
        // 끝난 후 성공 콜백 실행
        resultShare.getCompletionHandler().completed(resultShare, null);
    }
}
```

```java
public class SecondThread implements Runnable {
    
    ResultShare resultShare; 
    
    //공유객체 외부에서 주입.
    public SecondThread ( ResultShare resultShare ) {
        this.resultShare = resultShare;
    }
    
    @Override
    public void run() {
        int result = 0;
        for (int index = 101; index <= 200; index++) {
            result += index;
        }
        System.out.println("101~200까지의 합은 " + result +"입니다.");
        resultShare.sum(result);
        // 끝난 후 콜백 실행
        resultShare.getCompletionHandler().completed(resultShare, null);
    }
}
```
```java
public static void main(String args[]){
    // 스레드 풀 생성 
    //자동으로 스레드 수 생성
    ExecutorService executorServiceWithCached = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    
    ResultShare resultShare = new ResultShare();
    
    Runnable task1 = new FirstThread(resultShare);
    Runnable task2 = new SecondThread(resultShare);
            
    Future<ResultShare> future1 =  executorServiceWithCached.submit( task1, resultShare );
    Future<ResultShare> future2 =  executorServiceWithCached.submit( task2, resultShare );
    
    executorServiceWithCached.shutdown();
}

```
