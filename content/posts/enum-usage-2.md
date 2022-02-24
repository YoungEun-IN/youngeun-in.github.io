---
title: Enum 사용 (활용)
date: 2022-02-04T15:12:26+09:00
categories:
  - java
tags: 
  - enum
---

## 데이터 그룹관리
결제를 예로 들어 Enum의 활용법을 알아보고자 한다.
결제라는 데이터는 결제 종류와 결제 수단이라는 2가지 형태로 표현된다.
예를 들어 신용카드 결제는 신용카드 결제라는 결제 수단이며, 카드라는 결제 종류에 포함된다.
이 카드 결제는 페이코, 카카오페이등 여러 결제 수단이 포함되어 있다.

결제종류, 결제수단등의 관계를 명확히 표현하며,
각 타입은 **본인이 수행해야할 기능과 책임**만 가질 수 있도록 하기 위해 Enum을 사용할 수 있다.

이 때 DB 테이블의 결제수단 컬럼에 잘못된 값을 등록하거나,
파라미터로 전달된 값이 잘못되었을 경우에도 관리할 수 있도록 결제수단 역시 Enum으로 등록할 수 있다.

```java
public enum PayType {

    ACCOUNT_TRANSFER("계좌이체"),
    REMITTANCE("무통장입금"),
    ON_SITE_PAYMENT("현장결제"),
    TOSS("토스"),
    PAYCO("페이코"),
    CARD("신용카드"),
    KAKAO_PAY("카카오페이"),
    BAEMIN_PAY("배민페이"),
    POINT("포인트"),
    COUPON("쿠폰");

    private final String title;

    PayType(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }
}
```

```java
import lombok.Getter;

import java.util.Arrays;
import java.util.Collections;
import java.util.List;

@Getter
public enum PayGroup {

    CASH("현금", Arrays.asList(PayType.ACCOUNT_TRANSFER, PayType.REMITTANCE, PayType.ON_SITE_PAYMENT, PayType.TOSS)),
    CARD("카드", Arrays.asList(PayType.PAYCO, PayType.CARD, PayType.KAKAO_PAY, PayType.BAEMIN_PAY)),
    ETC("기타", Arrays.asList(PayType.POINT, PayType.COUPON)),
    EMPTY("없음", Collections.EMPTY_LIST);

    private final String title;
    private final List<PayType> payList;

    PayGroup(String title, List<PayType> payList) {
        this.title = title;
        this.payList = payList;
    }

    public static PayGroup findByPayType(PayType payType){
        return Arrays.stream(PayGroup.values())
                .filter(payGroup -> payGroup.hasPayType(payType))
                .findAny()
                .orElse(EMPTY);
    }

    private boolean hasPayType(PayType payType){
        return payList.stream()
                .anyMatch(pay -> pay == payType);
    }
}
```

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class PayGroupTest {

    @Test
    public void PayGroup에게_직접_결제종류_물어보기_PayType () {
        PayType payType = selectPayType();
        PayGroup payGroup = PayGroup.findByPayType(payType);

        assertEquals(payGroup.name(), "CARD");
        assertEquals(payGroup.getTitle(), "카드");
    }

    private PayType selectPayType(){
        return PayType.BAEMIN_PAY;
    }
}
```

DB 혹은 API에서 `PayType`으로 데이터를 받아, **타입 안전성**까지 확보하여 `PayGroup` 관련된 처리를 진행할 수 있게 되었다.

## 참고
https://github.com/jojoldu/blog-code/tree/master/enum-settler
