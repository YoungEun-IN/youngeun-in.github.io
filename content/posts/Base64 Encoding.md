---
title: Base64 Encoding
date: 2022-04-17T15:12:26+09:00
categories:
  - etc
tags: 
  - base64
  - ascii
---

Base64를 글자 그대로 직역하면 64진법이라는 뜻이다. 컴퓨터 분야에서 쓰이는 Base 64(베이스 육십사)란 **8비트 이진 데이터(예를 들어 실행 파일이나, ZIP 파일 등)를 문자 코드에 영향을 받지 않는 공통 ASCII 영역의 문자들로만 이루어진 일련의 문자열로 바꾸는 인코딩 방식**을 가리키는 개념이다.

인코딩(encoding)은 파일에 저장된 정보의 형태나 형식을 데이터 표준화, 보안, 처리 속도 향상, 저장 공간 절약 등을 위해서 다른 형태로 변환하는 처리 혹은 그 처리 방식을 말한다. 이메일 등의 전송, 동영상이나 이미지 영역에서 많이 사용되며, 반대말은 디코딩(decoding)이다.

기존 ASCII 코드는 시스템간 데이터를 전달하기에 안전하지 않다. 모든 binary 데이터가 ASCII 코드에 포함되지 않으므로 제대로 읽지 못한다. 반면 Base64는 ASCII 중 제어문자와 일부 특수문자를 제외한 안전한 출력 문자만 이용하므로 데이터 전달에 더 적합하다.

Base64는 어떤 문자와 기호를 쓰느냐에 따라 다양한 변종이 있지만, 대부분 처음 62개는 알파벳 A-Z, a-z와 0-9를 사용하며 마지막 두 개를 어떤 기호를 쓰느냐의 차이만 있다. Base64 표준에서는 마지막 62, 63번 글자가 +, / 이다. 이 문자들을 전송하게 되면 정상적으로 전송되지 않는 문제가 발생한다. 따라서 이러한 원문의 오류를 막기 위해서 등장한것이 RFC 4648의 base64url이다. base64url에서는 62번, 63번 글자가 각각 -, \_이다.

![image](https://user-images.githubusercontent.com/46465928/163704797-575000ec-82ca-4680-9079-f6361a510c8e.png)

## Base64 변환과정
base64 인코딩 과정은 먼저 24bit의 buffer를 생성하여 위쪽(MSB)부터 바이트 데이터를 넣은 뒤, 버퍼의 오른쪽부터 6bit 단위로 잘라 Base64 테이블의 ASCII 문자로 변환한다. 

다시 말해,  원본 문자열> ASCII binary> 6bit로 cut> base64 encodeing  순서가 된다.

예를 들어, Man이라는 세 글자 단어를 Base64로 인코딩한다면 Man > 77 97 110 > 01001101   01100001 01101110 > TWFu로 변환이 된다.

![image](https://user-images.githubusercontent.com/46465928/163704824-c6586bb8-d228-4af4-a100-e8e428311421.png)

## Base64 사용 이유

**Base64는 HTML 또는 Email과 같이 문자를 위한 Media에 Binary Data를 포함해야 될 필요가 있을 때, 포함된 Binary Data가 시스템 독립적으로 동일하게 전송 또는 저장되는 것을 보장하기 위해 사용한다.**
 
Base64로 인코딩을 하게 되면  6bit당 2bit의 Overhead가 발생하여 전송해야 될 데이터의 크기가 약 33% 정도 늘어난다. 33%나 데이터의 크기가 증가하고, 인코딩과 디코딩의 추가 연산까지 필요한데 Base64 인코딩을 사용하는 이유는 통신과정에서 바이너리 데이터의 손실을 막기 위함이다.

플랫폼 독립적으로 Binary Data(이미지나 오디오)를 전송할 필요가 있을 때, ASCII로 Encoding 하여 전송하게 되면 여러 가지 문제가 발생할 수 있다. 대표적인 문제는 다음과 같다.

- ASCII는 7 bits Encoding인데 나머지 1bit를 처리하는 방식이 시스템 별로 상이하다.
- 일부 제어 문자 (e.g. Line ending)의 경우 시스템 별로 다른 코드값을 가진다.

위와 같은 문제로 ASCII는 시스템 간 데이터를 전달하기에 안전하지 않다. 

Base64는 ASCII 중 제어 문자와 일부 특수문자를 제외한 64개의 안전한 출력 문자만 사용한다. 안전한 출력 문자란 문자 코드에 영향을 받지 않는 공통 ASCII를 의미한다.

## 참고

https://devuna.tistory.com/41
