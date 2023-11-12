# 생산자 소비자 문제와 해결


## 생산자 소비자 문제

고정된 크기의 버퍼(데이터를 보유하는 임시 영역)가 있을 때, 생산자는 버퍼에 데이터를 생성하는 기능을 제공하고, 소비자는 버퍼에서 데이터를 소비/제거하는 기능을 제공한다. 

이 때 특정 규칙이 있다.

- 소비자 스레드가 데이터를 소비하는 동안 생산자 스레드는 버퍼에 데이터를 생성할 수 없다.
- 생산자 스레드가 데이터를 생성하는 동안 소비자 스레드는 버퍼의 데이터를 사용할 수 없다.

버퍼가 가득 차면 생산자 스레드는 더 많은 데이터를 생성할 수 없고, 버퍼가 비어 있으면 소비자 스레드는 데이터를 소비할 수 없다. 이처럼 **생산자 소비자 문제는 생산자 스레드는 버퍼가 비었는지 계속 확인해야 하고, 소비자 스레드는 버퍼에 태스크가 있는지 계속 확인해야 하는 문제를 말한다.** 

## WAIT-NOTIFY를 통한 해결

```java
import java.util.*;
import java.lang.*;
import java.io.*;

class Buffer{
    private static final int size = 10;
    private Queue<Integer> buffer = new LinkedList<>();

    public synchronized void produce(int item){
        try{
            if(this.buffer.size() == size){
                this.wait();
            }
    
            System.out.println(Thread.currentThread().getName() + " produces item: " + item);
            this.buffer.add(item);
            this.notifyAll();

        } catch(InterruptedException exc){
            exc.printStackTrace();
        }  
    }

    public synchronized void consume(){
        try{
            if(this.buffer.isEmpty()){
                this.wait();
            }
            System.out.println(Thread.currentThread().getName() + " consumes item: " + buffer.poll());
            this.notifyAll();
 
        } catch(InterruptedException exc){
             exc.printStackTrace();
        }
    }
}


class Producer implements Runnable{
    private Buffer sharedBuffer;
     
    public Producer(Buffer buffer){
        this.sharedBuffer = buffer;
    }

    @Override
    public void run(){
        for(int i=1; i<=10; i++){
            sharedBuffer.produce(i);
        }
    }
}

class Consumer implements Runnable{
    private Buffer sharedBuffer;
    
    public Consumer(Buffer myBuffer){
        this.sharedBuffer = buffer;
    }

    @Override
    public void run(){
        for(int i=1; i<=10; i++){
           sharedBuffer.consume();
        }
    }
}

public class Main{
     public static void main(String[] args) throws java.lang.Exception{
          Buffer sharedBuffer = new Buffer();

          Thread producerThread1 = new Thread(new Producer(sharedBuffer));
          Thread consumerThread2 = new Thread(new Consumer(sharedBuffer));
           
          Thread producerThread2 = new Thread(new Producer(sharedBuffer));
          Thread consumerThread2 = new Thread(new Consumer(sharedBuffer));

          producerThread.start();
          consumerThread.start();
          
     }
}
```

## Semaphore를 통한 해결

```java
import java.util.*;
import java.lang.*;
import java.io.*;

class Buffer{
    private Semaphore Prod = new Semaphore(1);
    private Semaphore Cons = new Semaphore(0);


    private Queue<Integer> buffer = new LinkedList<>();
       
    public void produce(int item){
        try{
            Prod.acquire();
            
            System.out.println(Thread.currentThread().getName() + " produces item: " + item);
            buffer.offer(item);

            Cons.release();
        } catch(InterruptedException exc){
            exc.printStackTrace();
        }  
    }

    public void consume(){
        try{

            Cons.acquire();
            System.out.println(Thread.currentThread().getName() + " consumes item: " + buffer.poll());
            Prod.release();

        } catch(InterruptedException exc){
             exc.printStackTrace();
        }
    }
}

class Producer implements Runnable{
    private Buffer sharedBuffer;
     
    public Producer(Buffer buffer){
        this.sharedBuffer = buffer;
    }

    @Override
    public void run(){
        for(int i=1; i<=10; i++){
            sharedBuffer.produce(i);
        }
    }
}

class Consumer implements Runnable{
    private Buffer sharedBuffer;
    
    public Consumer(Buffer myBuffer){
        this.sharedBuffer = buffer;
    }

    @Override
    public void run(){
        for(int i=1; i<=10; i++){
           sharedBuffer.consume();
        }
    }
}

public class Main{
     public static void main(String[] args) throws java.lang.Exception{
          Buffer sharedBuffer = new Buffer();

          Thread producerThread1 = new Thread(new Producer(sharedBuffer));
          Thread consumerThread2 = new Thread(new Consumer(sharedBuffer));
           
          Thread producerThread2 = new Thread(new Producer(sharedBuffer));
          Thread consumerThread2 = new Thread(new Consumer(sharedBuffer));

          producerThread.start();
          consumerThread.start();
          
     }
}
```

## 참고

https://www.youtube.com/watch?v=Dms1oBmRAlo

https://medium.com/@madhurgupta977/producer-consumer-problem-and-its-solution-626b1e573c50

