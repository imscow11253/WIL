2020/03/01 초판 2쇄

## chapter 1 : 개요
---

java 8은 1. 간결한 코드 2. 멀티코어 cpu 의 활용을 목표로 이전 자바 버전과는 달리 크게 달라졌다. 분기점이라고 할 수 있다. (9, 10도 큰 변화는 x)
그리고 가장 큰 변화는 '===함수형 프로그래밍==='이라는 개념이 도입 되었다는 점이다.

### 추가된 기능
-  stream api 
-  메서드 인자로 코드 전달 (람다식) --> 동작 파라미터화
-  인터페이스의 디폴트 메서드

### stream API 
stream API를 사용하게 되면 멀티코어 기능을 내부적으로 자동 병렬처리 작업이 수행된다. 이 API가 없을 때는 네프에서 배웠던 것처럼 synchronized 키워드로 스레드를 수행해야 하는데 안 해도 알아서 해준다. 어떤 의미에서 stream API 가 동작을 병렬적으로 처리한다는 것일까? 
다음과 같은 쉘 명령어가 있다고 하자. 
```shell
cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3
```
따로따로 코드를 작성한다면 cat 명령어가 모든 데이터에 대해서 다 끝나고 tr 적용 시작하고, 또 tr이 모든 데이터에 대해서 다 끝나고 sort, tail이 순서대로 적용된다. 하지만 위에서처럼 pipe로 연결지으면, 뒤의 명령어는 앞의 명령어가 끝난 데이터에 대해서 미리 실행될 수 있다. 
a,b,c 데이터가 있으면 따로 하면 cat(a), cat(b), cat(c) tr(a) tr(b) tr(c) ... 이런 식의 순서로 진행되는데 pipe로 연결지으면 cat(a), {tr(a), cat(b)}, {sort(a), tr(b), cat(c)} .... 이런 식으로 각 작업이 병렬적으로 처리될 수 있다. --> ===근데 sort 같은건 모든 데이터가 다 있어야 되는 것 아닌가?===

코드 상에서의 장점을 살펴보자. 
다음과 같은 문제가 주어졌을 때 Collection API로 해결하는 코드와 Stream API로 해결하는 코드를 봐보자. 
리스트에서 고가의 트랜잭션만 필터링한 다음, 통화에 따라 결과를 그룹핑 해라. 
```java 
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>();

for(Transaction transaciton : transactions){
	if(transaction.getPrice > 1000){
		Currency currency = transaciton.getCurrency();
		List<Transaction> transactionForCurrency = transactionsByCurrencies.get(currency);
		if(transactionForCurrency == null){
			transactionsForCurrency = new ArrayList<>();
			transactionByCurrencies.put(currency, transactionForCurrency);
		}
		transactionForCurrency.add(transaction);
	}
}
```

stream을 쓰면 다음과 같이 간단하게 변한다. 
```java
Map<Currency, List<Transaction>> transactionByCurrencies = transactions.stream()
	.filter((Transaction t) -> t.getPrice() > 1000)
	.collect(groupingBy(Transaction::getCurrency));
```


컬렉션은 어떻게 데이터를 저장하고 접근할지에 중점을 두는 반면, 스트림은 데이터에 어떤 계산을 할 것인지 묘사하는 것에 중점을 둔다. 

stream api는 병렬처리를 제공하기 때문에,  컬렉션을 필터링할 수 있는 가장 빠른 방법은 컬렉션을 스트림으로 바꾸고, 병렬로 처리한 다음에, 리스트로 다시 복원하는 것이다. 
--> 앞으로 list를 sort할 때 이렇게 하자. ===그럼 컬렉션의 sort 메서드는 어떻게 동작하지?===
https://chatgpt.com/share/67763e48-ec58-8013-911e-cf1ffac37f75 --> GPT 한테 물어본 결과

### 동작 파라미터
자바 8 이전에서는 메서드를 인자로 전달하는 것이 안됐다. 메서드로 전달할 수 있는 것을 일급 시민 또는 일급 값이라고 한다. 자바 8 이전에서 메서드는 이급 값이다. 메서드를 전달할 수 없을 때는 어떻게 했냐? 다음 코드와 같이 해당 메서드를 가지고 있는 클래스의 인스턴스를 만들어서 넘겨주었다. `
```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter(){
	public boolean accept(File file){
		return file.isHidden();
	}
})
```
즉 listFiles() 라는 메서드의 인자로 isHidden이라는 메서드를 넘겨주기 위해서 isHidden 메서드를 가지고 있는 FileFilter라는 클래스의 인스턴스를 만들어서 인자로 넘겨준다. 
하지만 자바 8 이후부터는 메서드를 일급값으로 취급가능해지면서 넘겨주는 것이 가능해졌다. 
```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

익명 함수도 일급값으로 취급할 수 있는데 이를 람다 함수라고 한다. 한 두번만 사용하는 메서드를 인자로 넘겨주기 위해서 정의해두는 것은 비효율적이므로 람다 함수를 쓴다. 
```java
(int x) -> x+1;
```


## chapter2  : 동작파라미터
---

동작 파라미터화는 메서드의 인자로 함수를 전달하는 것이다. 이렇게함으로써 유연한 코드를 작성할 수 있다. 

사과 리스트에서 녹색인 것만 출력하는 코드를 살펴보자. 
~~~ java 
public static List<Apple> filterGreenApples(List<Apple> inventory){
	
	List<Apple> result = new ArrayList<>();
	
	for (Apple apple : inventory){
		if(GREEN.equals(apple.getColor())){
			result.add(apple);
		}
	}
	return result;
}
~~~

만약  여기서 다른 색깔을 필터링하고 싶다면? Color를 파라미터화 하면 된다.

~~~java
public static List<Apple> filterGreenApples(List<Apple> inventory, Color color){
	
	List<Apple> result = new ArrayList<>();
	
	for (Apple apple : inventory){
		if(apple.getColor().equals(color)){
			result.add(apple);
		}
	}
	return result;
}
~~~

무게로 필터링하고 싶으면? 다른 속성들로 필터링 하고 싶으면? 단순하게 생각해서 각 속성에 대해서 파라미터화를 시키면 된다. 그리고 flag 값도 인자로 받아서 어떤 속성으로 필터링 할지 판단하면 된다. 

~~~java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag){

	List<Apple> result = new ArrayList<>();

	for(Apple apple : inventory){
		if(
			(flag && apple.getColor()).equals(color) || 
			(!flag && apple.getWeight() > weight)
		){
			result.add(apple);
		}
	}
	return result;
}
~~~

이 코드를 처음 봤을 때, 뜨끔했다. 내가 정말 자주 사용하는 패턴이다. 그리고 좀 놀랐다. 누구한테도 이렇게 하라고 들은 적 없고 내가 생각해서 이런 방식으로 그동안 짜왔기에 나만 쓰는 방식인 줄 알았는데, 모두가 생각하는 방식이 똑같구나 라는 것을 느꼈다. 

책에서는 형편없는 코드라고 한다. 요구사항이 바뀌면 유연하게 대처할 수도 없을 뿐더러 가독성도 매우 떨어진다. flag가 true인게 어떤 속성을 나타내는지 바로 알 수 없기 때문이다. 즉 지금까지는 나는 java 8 이전 버전 기능만을 써서 코딩을 해오고 있었던 것이다.

그럼 동작 파라미터, 즉 함수 로직을 인자로 어떻게 넘겨줄 수 있을까? 
단순하게 생각해서 해당 로직 메서드를 가지는 클래스를 선언하고 인스턴스를 넘겨주면 된다. 

~~~java
public interface ApplePredicate{   // 인터페이스
	boolean test(Apple apple);
}

public class AppleHeavyWeightPredicate implements ApplePredicate{   // 구현클래스1
	public boolean test(Apple apple){ 
		return apple.getWeight() > 150;
	}
}

public class AppleGreenColorPredicate implements ApplePredicate{    // 구현클래스2
	public boolean test(Apple apple){
		return GREEN.equals(apple.getColor());
	}
}

public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){

	List<Apple> result = new ArrayList<>();

	for(Apple apple : inventory){
		if(p.test(apple)){
			result.add(apple);
		}
	}
	return result;
}
~~~

위의 방식이 런타임 시에 함수의 동작을 제어할 수 있다는 점에서 유연한 코드 작성이 가능하다. 하지만 인터페이스도 구현해야 하고, 해당 인스턴스를 구현하는 구현체 클래스도 정의해야 한다. 또 해당 클래스의 인스턴스를 생성해서 넘겨주어야 한다. 

이때 사용할 수 있는 것이 익명 클래스이다. 

~~~java 
List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
	public boolean test(Apple apple){
		return RED.equals(apple.getColor());
	}
});
~~~

그래도 코드가 한 눈에 알아보기 힘들고, 명확하지 못하다. 
이때 람다 표현식을 사용하면 좋다. 

~~~java
List<Apple> redApples = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
~~~

![[Behavior Parameterization.png]]

이 그림이 이해하기 쉽다.지금껏 내가 자주 사용하던 방식은 값 파라미터화 였다. java 버전은 21을 쓰는 놈이 8에서 제공하는 문법도 제대로 못쓴다는 것이 말도 안된다. 나도 진화하자. 

추가적으로 동작 파라미터화를 한다는 것은 '전략패턴'을 자연스레 사용하게 된다는 뜻이다. 

## chapter 3 : 람다 표현식
---

람다 표현식은 ***메서드로 전달할 수 있는 익명함수를 단순화한 것*** 이다. 
사실 익명 클래스는 자바 8 이전에도 있었다. 그럼 람다 함수가 가지는 의의는 뭘까?

동작 파라미터를 이용할 때 익명클래스 등 판에 박힌 코드를 구현할 필요가 없어진다는 점에서 의의를 가진다. 

#### 함수형 인터페이스

앞선 2장에서 람다형 함수를 사용할 수 있다고만 하고 인자로 어떻게 명시하는 지를 알려주지 않았다. 어떻게 명시하면 될까? 이때 사용하는 것이 함수형 인터페이스이다. 

함수형 인터페이스는 하나의 추상메서드만을 가지는 인터페이스를 의미한다. 대표적으로 Predicate<>, Comparator<> 가 있다. 

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

```java
public interface Predicate<T>{
	boolean test(T t);  //이러면 아무 타입이나 인자로 받아서 boolean 값을 반환하기만 하는 람다함수이면 다 okay이다. 
}
```

위 두 함수형 인터페이스는 보통 자바에서 정해진 것이고 직접 함수형 인터페이스를 정의해서 사용해도 된다. 이 인터페이스를 메서드의 인자로 명시해두면 람다형 표현식을 인자로 전달가능하다. 

@FunctionalInterface 라는 어노테이션을 사용해도 된다. 

함수형 인터페이스 내부에 선언된 하나의 메서드를 함수 디스크립터라고 한다. 

() -> {}  이런 구조로 람다 표현식을 사용하는데 괄호 안에 들어오는 인자의 타입을 굳이 명시하지 않아도 추론 가능하다. 타입을 명시하지 않는게 좋은지, 명시하는게 좋은지에 대한 정답은 없다. 클린코드 책에서 한 번 알아보자. 

#### 메서드 참조

인텔리제이에서 자바로 코딩하다가 stream.map 에서 람다 형식을 쓰다보면 간략한 문법의 경우 메서드 참조를 쓰라고 추천을 해준다. 

```java
list.stream()
	.map( it -> it.getString())
	.toList();
```

```java
list.stream()
	.map(Person::getString)
	.toList();
```

이런 식으로 추천해준다. 

이렇게 간단한 경우에는 메서드 참조법을 사용하면 된다. 


## 정리

함수형 프로그래밍 왜 쓰는데?
--> 많은 곳에서 도입되어서 장점이 입증되었고, 메서드를 일급 시민으로 만들어서 인자로 넘겨주기 위함.

stream api 왜 생겼는데?
병렬처리하기 좋고, 반복문 로직을 외부에서 처리하던 것을 내부에서 알아서 수행하므로 코드도 무척 간결해진다.

void sort(Comparator<\? super E> c )

1. Comparator 를 implements 하는 인터페이스를 선언하고 이걸 구현한 구현체를 생성 -> 인자로 전달
2. 익명 클래스 사용 --> 람다 표현식 사용
3. 메서드 참조 사용 




