---
title: RuntimeException & IOException
date: 2022-10-17T15:12:26+09:00
categories:
  - java
tags: 
  - RuntimeException
  - IOException
---

![image](https://user-images.githubusercontent.com/46465928/196196649-21dc80e2-8293-49b3-976c-a15f9a606160.png)

## RuntimeException : Unchecked Exception
프로그램 실행 도중 발생하는 예외로, 프로그래머의 잘못으로 발생한다고 볼 수 있다.

배열의 범위를 넘어선 접근, Null 객채에 대한 접근, 잘못된 형 변환 등을 예로 들 수가 있다,

이러한 예외는 처리에 있어서 강제성을 띄지 않는다.

## IOException : Checked Exception

런타임 예외는 언제 에러가 발생하는지 알 수 없기 때문에 처리를 하지 않아도 되지만,

IOException은 예외가 발생할 시점과 종류를 알 수 있기 때문에 반드시 처리를 해주어야 하는 강제성을 띄게 된다.

## 참고
https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=bestheroz&logNo=95526984
