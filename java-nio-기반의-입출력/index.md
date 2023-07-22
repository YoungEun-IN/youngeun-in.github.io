# JAVA NIO 기반의 입출력


## NIO의 채널(Channel)과 버퍼(Buffer)
- 스트림과 채널의 공통점
  - 스트림도 채널도 데이터의 입력 및 출력을 위한 통로가 된다.

- 스트림과 채널의 차이점
  - 스트림은 한 방향으로만 데이터가 이동하지만 채널은 양방향으로 데이터 이동이 가능하다.

- 채널에만 존재하는 제약사항
  - 채널은 반드시 버퍼에 연결해서 사용해야 한다.

- 채널 기반 데이터 입출력 경로
  - 출력 경로: 데이터 ⇨ 버퍼 ⇨ 채널 ⇨ 파일
  - 입력 경로: 데이터 ⇦ 버퍼 ⇦ 채널 ⇦ 파일

채널과 버퍼는 독립적이다. 버퍼와 채널은 일대일 일대다 다대일 다대다 모두 가능하다.

## NIO 기반 파일 복사 예제

```java
public static void main(String[] args) {
   Path src = 복사할 대상 파일
   Path dst = 사본 이름 

   // 하나의 버퍼 생성 - 인스턴스 생성
   ByteBuffer buf = ByteBuffer.allocate(1024);

   // try에서 두 개의 채널 생성 - 버퍼와 채널은 별개
   try(FileChannel ifc = FileChannel.open(src, StandardOpenOption.READ);
          FileChannel ofc = FileChannel.open(dst, StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
      int num;
      while(true) {
         num = ifc.read(buf);         // 채널 ifc에서 버퍼로 읽어 들임
         if(num == -1)         // 읽어 들인 데이터가 없다면
            break;

         buf.flip();         // 버퍼 모드 변환!
         ofc.write(buf);         // 버퍼에서 채널 ofc로 데이터 전송
         buf.clear();         // 버퍼 비우기
      }
   }
   catch(IOException e) {
      e.printStackTrace();
   }
}
```

## 성능 향상 포인트

- 효율적인 버퍼링
  - 기본 IO 스트림 기반의 복사 프로그램과 달리 하나의 버퍼만을 사용하였다.
  - 버퍼의 수가 줄은 것이 핵심이 아니라, 이로 인해 버퍼에서 버퍼로의 데이터 이동이 불필요해진 부분이 핵심

### Non-direct 버퍼를 대신하는 Direct 버퍼의 생성

|ByteBuffer buf = ByteBuffer.allocate(1024);|ByteBuffer.allocateDirect(1024);|
|---|---|
|Non-direct 버퍼|Direct 버퍼|
|파일 ⇨ 운영체제 버퍼 ⇨ 가상머신 버퍼 ⇨ 실행 중인 자바 프로그램|파일 ⇨ 운영체제 버퍼 ⇨ 실행 중인 자바 프로그램|
|I/O스트림의 구조적인 문제 가상머신을 거치는 부분은 해결하지 못했지만 각각의 버퍼에 받아 복사하고 전송하지 않고 한 버퍼에서 받고 보내기 할 수있어 성능향상은 이루어졌다고 볼 수 있다.|가상머신 부분 생략 가능|

