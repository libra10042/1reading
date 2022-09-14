- **타입 안전 열거 패턴 (typesafe enum pattern)    vs   열거 패턴**
    - **타입 안전 열거 패턴**
        - 확장할 수 있다.
        - 열거한 값들을 그대로 가져온 다음 값을 더 추가하여 다른 목적으로 쓸 수 있다.
    - **열거타입**
        - 확장할 수 없다.
        - 값을 더 추가하여 다른 목적으로 쓸 수 없다.
        

> **대부분 상황에서 열거 타입을 확장하는 것은 좋지 않다.**
> 

**why ?**

확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 복잡해진다.

**열거 타입.** 

```java
public enum Week {
		//열거 상수
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY
}
```

**인터페이스를 이용해 확장 기능 열거 타입을 흉내 냈다.**

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    private final String symbol;
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

**연산(EXP)과 나머지 연산(REMAINDER)추가.**

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

**main 메서드는 test 메서드에 ExtendedOperation의 class 리터널을 넘겨 확장된 연산들이 무엇인지 알려준다.**

```java
// 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
private static <T extends Enum<T> & Operation> void test(
	  Class<T> opEnumType, double x, double y) {
	  for (Operation op : opEnumType.getEnumConstants())
	    System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

```java
public static void main(String[] args) {    
  double x = Double.parseDouble(args[0]);    
  double y = Double.parseDouble(args[1]);   
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet, double x, double y) {    
  for (Operation op : opSet)        
  System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

`<T extends Enum<T> & Operation>` : Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다.

- **인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식의 문제점.**
    - **열거 타입끼리 구현을 상속할 수 없다.**

---

열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.