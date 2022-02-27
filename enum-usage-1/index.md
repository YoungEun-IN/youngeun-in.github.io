# Enum 사용 (개념)


## 상태와 행위를 한곳에서 관리
예를 들어 DB에 저장된 code의 값이 "CALC_A"일 경우엔 값 그대로, "CALC_B"일 경우엔 10 한 값을, "CALC_C"일 경우엔 3을 계산하여 전달하는 경우를 생각해보자.
이 때 "DB의 테이블에서 뽑은 특정 값은 지정된 메소드와 관계가 있다."는 사실을 코드로 나타내기 위해 Enum을 사용할 수 있다.
즉 JAVA에서 Enum을 통해 상태와 행위를 한곳에서 관리할 수 있다.

```java
import java.util.function.Function;

public enum CalculatorType {
    CALC_A(value -> value),
    CALC_B(value -> value * 10),
    CALC_C(value -> value * 30),
    CALC_ETC(value -> 0L);

    private Function<Long, Long> expression;

    CalculatorType(Function<Long, Long> expression) {
        this.expression = expression;
    }

    public long calculate(long value) {
        return expression.apply(value);
    }
}
```

```java
import javax.persistence.Column;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;

public class Entity {
    @Column
    //@Enumerated(EnumType.STRING)를 선언하면 Enum 필드가 테이블에 저장시 숫자형인 1,2,3이 아닌, Enum의 name이 저장된다.
    @Enumerated(EnumType.STRING)
    private CalculatorType calculatorType;
}
```

```java
import org.testng.annotations.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class CalculatorTypeTest {

    private CalculatorType selectType() {
        return CalculatorType.CALC_B;
    }

    @Test
    public void 코드에_따라_서로다른_계산하기_Enum() {
        CalculatorType code = selectType();
        long originValue = 10000L;
        long result = code.calculate(originValue);

        assertEquals(result, 100000L);
    }
}
```
값(상태)과 메소드(행위)가 어떤 관계가 있는지에 대해 더이상 다른 곳을 찾을 필요가 없게 되었다.
코드내에 전부 표현되어 있고, Enum 상수에게 직접 물어보면 되기 때문이다.

