[**Java 가변인자 사용법.**](https://dpdpwl.tistory.com/137)

**가변인자?**

가변인자는 가변인자를 나타내는 기호(...)를 사용한다.

변수 타입뒤에 붙여주고 변수명을 쓰면 끝

**53-1 간단한 가변인수 활용 예)**

```java
static int sum(int... args){
		int sum = 0; 
		for(int arg : args)
				sum += arg; 
		return sum;
}
```

**53-2 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예** 

```java
static int min(int... args){
		if(args.length == 0)
				throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
		int min = args[0];
		for(int i = 1; i<args.length; i++)
				if(args[i] < min)
						min = args[i];
		return min;
}
```

> **다른 파라미터와 가변인자를 같이 사용하는 경우에는 가변인자를 제일 뒤에 위치 시켜야 합니다.**
> 

**53-3 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법.** 

```java
static int min(int firstArg, int... remainingArgs){
		int min = firstArg; 
		for(int arg : remainingArgs)
				if(arg < min)
						min = arg; 
		return min;
}
```

가변 인수는 인수 개수가 정해지지 않았을 때 아주 유용하다.

 

하지만, **가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화 하므로, 성능에 민감한 상황에서는 걸림돌**이 될 수 있다. 비용을 감당할 수 없지만, 가변인수의 유연성이 필요한 경우 다중정의를 통해 해결할 수 있다.

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

만약, 95%가 인수 3개 이하로 사용한다면, 다음과 같이 다중정의를 통해 5%만 배열을 생성하도록 할 수 있다. `EnumSet`
의 정적 팩터리도 위 기법을 사용해 열거 타입의 집합 생성 비용을 최적화하고 있다.

---

### 정리

- 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다.
- 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.