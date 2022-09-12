[HashMap, HashSet 과 EnumMap EnumSet과의 차이점.](https://yeti.tistory.com/287)

[enum 정의하는 방법, enum이 제공하는 메서드, java.lang.Enum, EnumSet](https://velog.io/@dion/%EB%B0%B1%EA%B8%B0%EC%84%A0%EB%8B%98-%EC%98%A8%EB%9D%BC%EC%9D%B8-%EC%8A%A4%ED%84%B0%EB%94%94-11%EC%A3%BC%EC%B0%A8-Enum)

---

**열거한 값들이 주로 집합으로 사용될 경우. 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.** 

**비트필드는 아래와 같이 사용한다.** 

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

**36-1. 비트 필드 열거 상수 - 구닥다리 기법**

```java
public class Text {
		public static final int STYLE_BOLD = 1 << 0; // 1
		public static final int STYLE_ITALIC = 1 << 1; // 2
		public static final int STYLE_UNDERLINE = 1 << 2; // 4
		public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

		// 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다. 
		public void applyStyles(int styles){ ... }
}
```

### 비트 필드의 장점.

비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다. 

### 비트 필드의 단점.

- 비트 필드는 정수 열거 상수의 단점을 그대로 가지고 있다.
- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
- 마지막으로 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통은 Int나 long)을 선택해야 한다.
- API를 수정하지 않고는 비트 수(32비트 or 64비트)를 더 늘릴 수 없기 때문이다.

### EnumSet 클래스

- java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
- Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
- 하지만 EnumSet의 내부는 비트 벡터로 구현되었다.
- 원소가 총 64개 이하라면, 즉 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
- removeAll 과 retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

위의 코드를 EnumSet 을 사용해 수정해보았다.

**36-2. EnumSet 비트필드를 대체하는 현대적 기법.** 

```java
public class Text { 
		public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGHT }

		// 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다. 
		public void applyStyles(Set<Style> styles) { ... }

}
```

applyStyles 메서드에 EnumSet 인스턴스를 건네는 클라이언트 코드다. 

( EnumSet 은 집합 생성 등 다양한 기능의 정적 팩터리를 제공하는데, 다음 코드에서는 그중  of 메서드를 사용했다. 

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC)); 
```

applyStyles 메서드가 EnumSet<Style> 이 아닌 Set<Style>을 받은 이유를 생각해보자. 

모든 클라리언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는게 일반적으로 좋은 습관이다. 

---

### 정리

- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.
- EnumSet의 유일한 단점이라면 불변 EnumSet을 만들 수 없다는 것이다.