**배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다. 식물을 간단히 나타낸 다음 클래스를 예로 살펴보자.** 

```java
class Plant { 
		enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

		final String name; 
		final LifeCycle lifeCycle;
		
		Plant(String name, LifeCycle lifeCycle){
				this.name = name; 
				this.lifeCycle = lifeCycle; 
		}
		@Override public String toString(){
				return name; 
		}
}
```

**37-1. ordinal()을 배열 인덱스로 사용. - 따라 하지 말 것!**

```java
Set<Plant>[] plantsByLifeCycle = 
		(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for(int i=0; i < plantsByLifeCycle.length; i++)
		plantsByLifeCycle[i] = new HashSet<>();

for(Plant p : garden)
		plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for(int i=0; i < plantsByLifeCycle.length; i++){
		System.out.println("%s: %s%n", 
				Plant.LifeCycle.values()[i]. plantsByLifeCycle[i]);
}
```

위 소스의 문제점. 

- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다.
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
- 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 여러분이 직접 보증해야 한다.는 점이다.
- 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다.
- 잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 운이 좋다면 `ArrayIndexOutOfBoundsException`을 던질 것이다.

**37-2. EnumMap을 사용해 데이터와 열거 타입을 매핑한다.** 

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = 
		new EnumMap<>(Plant.LifeCycle.class);
for(Plant.LifeCycle lc : Plant.LifeCycle.values())
		plantsByLifeCycle.put(lc, new HashSet<>());
for(Plant p : garden)
		plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```

- 더 짧고 명로하고 안전하고 성능도 원래 버전과 비등하다.
- 안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다.
- 나이가 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다.
- EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 떄문이다.
- 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다.
- 여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다.

아래와 같이 스트림으로 맵을 관리하면 코드를 더 줄일 수 있다. 

**37-3.  스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다.** 

```java
System.out.println(Arrays.stream(garden)
			.collect(groupingBy(p -> p.lifeCycle)));
```

**37-4. 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.** 

```java
System.out.println(Arrays.stream(garden)
			.collect(groupingBy(p -> p.lifeCycle, 
					() -> new EnumMap<>(LifeCycle.class), toSet())));
```

스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다. 

- EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.
- 예컨대 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서는 맵을 3개 만들고 스트림 버전에서는 2개만 만든다.

**37-5 배열들의 배열의 인덱스에 orinal()을 사용 - 따라하지 말 것!**

```java
public enum Phase { 
		SOLID, LIQUID, GAS; 

		public enum Transition {
				MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

				// 행은 from 의 ordinal을, 열은 to의 ordinal을 인덱스로 본다. 
				private static final Transition[][] TRANSITIONS = {
						{ null, MELT, SUBLIME },
						{ FREEZE, null, BOIL },
						{ DEPOSIT, CONDENSE, null }
				};

				// 한 상태에서 다른 상태로의 전이를 반환한다. 
				public static Transition from(Phase from, Phase to){
						return TRANSITIONS[from.ordinal()][to.ordinal()];
				};
		}

}
```

- 두 열거 타입 값들을 매핑하느라 ordinal을 쓴 배열들의 배열을 본 적이 있을 것이다. 다음은 이 방식을 적용해 두 가지 상태(Phase) 를 전이(Transition)와 매핑하도록 구현한 프로그램이다. 예컨대 액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)가 되고, 액체에서 기체(GAS)로의 전이는 기화(BOIL)가 된다.

앞서 보여준 간단한 정원 예제와 마찬가지로 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없다. 

즉, Phase나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 것이다. 

Array IndexOutOfBoundsException이나 NullPointerException을 던질 수도 있고, 예외도 던지지 않고 이상하게 동작할 수도 있다. 그리고 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것이다. 

다시 이야기하지만 EnumMap을 사용하는 편이 훨씬 낫다. 

전이 하나를 얻으려면 이전 상태(from) 와 이후 상태 (to)가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다. 

안쪽 맵은 이전 상태와 전이를 연결하고 바깥 맵은 이후 상태와 안쪽 맵을 연결한다. 

전이 전후의 두 상태를 전이 열거 타입 Transition의 입력으로 받아, 이 Transition 상수들로 중첩된 EnumMap을 초기화하면 된다. 

**37-6. 중첩 EnumMap으로 데이터와 열거 타입을 쌍을 연결했다.**

```java
public enum Phase { 
		SOLID, LIQUID, GAS; 

		public enum Transition {
				MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), 
				BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID), 
				SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

				private final Phase from;
				private final Phase to;

				Transition(Phase from, Phase to){
						this.from = from; 
						this.to = to;
				}

				//  상전이 맵을 초기화한다. 
				private static final Map<Phase, Map<Phase, Transition>> m 
								= Stream.of(values()).collect(
																									groupingBy(t -> t.from, () -> new EnumMap<>(Phase.class), toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));

				public static Transition from(Phase from, Phase to){
						return m.get(from).get(to);
				}
		}
}
```

상전이 맵을 초기화하는 코드는 제법 복잡하다. 

이 맵의 타입인 Map<Phase, Map<Phase, Transition>> 은 “ 이전 상태에서 ‘이후 상태에서 전이로의 맵’에 대응시키는 맵”이라는 뜻이다. 

이러한 맵의 맵을 초기화하기 위해 수집기(java.util.stream.Collector) 2개를 차례로 사용했다. 

첫번째 수집기인 groupingBy에서는 전이를 이전 사앹를 기준으로 묶고, 두 번째 수집기인 toMap에서는 이후 상태를 전이에 대응 시키는 EnumMap을 생성한다. 

두 번째 수집기의 병합 함수인 (x, y) → y는 선언만 하고 실제로는 쓰이지 않는데, 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리를 제공하기 때문이다. 

**37-7. EnumMap 버전에 새로운 상태 추가하기.**

```java
public enum Phase { 
		SOLID, LIQUID, GAS, PLASMA;
		
		public enum Transition {
				MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
		}

}
```