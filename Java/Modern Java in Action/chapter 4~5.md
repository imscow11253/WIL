()
## chapter 4 : 스트림 소개
---

stream 이란 ==데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소== 로 정의할 수 있다. 
==stream은 Collection에서 데이터 처리 질의를 표현하는 강력한 도구이다.== 마치 데이터 베이스 같은 연산을 수행할 수 있다. 

### collection API VS stream API 

1. 복잡한 로직 직접 구현 vs 만들어진 기능 가져와서 쓰면서 선언형으로 구현
2. 직접 구현 vs 파이프라이닝 가능
3. 쓰레드 직접 구현 vs 자동 병렬처리

</br>

4. 모든 데이터를 로드 해야만 함 vs 요청할 때만 해당하는 요소를 계산
		ex) DVD는 모든 영화 내용이 전부 저장되어 있는 상태에서 보여진다는 점에서 collection 에 비유 가능하고 internet streaming 은 먼저 로딩된 프레임들을 순차적으로 보여준다는 점에서 stream에 비유 가능하다. 

</br>

5. 외부 반복 vs 내부 반복
		collection 에서는 사용자가 직접 반복문을 사용해서 요소를 방문 했어야 했다. (= 외부 반복) stream은 반복을 알아서 처리하고 결과 값을 어딘가에 저장해준다. (= 내부 반복) 
		내부 반복의 장점은? 
		외부 반복을 사용하게 되면 직접 동작을 지정해주어야 하고 지정된 동작대로만 작업하기 때문에 최적화를 할 수 없다. 예를 들어서 아이에게 장난감을 정리하라고 시킨다고 하면 공 들어서 상자에 넣어. 인형 들어서 상자에 넣어. 이런 외부 반복으로 하면 두 손을 다 활용하지 못한다.
		하지만 장난감 정리해. 라고만 하면 아이가 알아서 두 손을 사용해서 장난감 정리를 할 수 있는 것이다. 


stream의 연산 과정을 확인해보기 위해서 각 연산마다 print 시켜볼 수 있다. 

```java 
List<String> names = 
	menu.stream()
		.filter(dish -> {
			System.out.println("filtering : " + dish.getName());
			return dish.getCalories() > 300;
		})
		.map(dish -> {
			System.out.println("mapping : " + dish.getName());
			return dish.getName();
		})
		.limit(3)
		.collect(toList());
```

> filtering : pork
> mapping : pork
> filtering : beef
> mapping : beef
> filtering : chicken
> mapping : chicken

이러한 결과가 나온 이유는 ==쇼트서킷== 과 ==루프 퓨전== 기법 덕분이다. 이건 5장에서 나온다.
==게으른 연산==

## chapter 5 : 스트림 활용
---

이 장에서는 stream API 가 제공하는 기능을 살펴보자. 

#### 필터링
1. .filter()
	Predicate 를 입력 받아서 사용되며, 필터링을 한다. 
2. .distinct()
	결과 stream에 중복을 제거한다. 
#### 슬라이싱
3. .takeWhile()
	Predicate 를 입력으로 받으며, 첫 번째 요소부터 Predicate 를 적용하면서 false일 때까지의 값만 다음 stream에 포함시킨다. 주로 정렬된 요소에서 사용하기 좋다. 
4. .dropWhile()
	Predicate 를 입력으로 받으며, 첫 번째 요소부터 Predicate 를 적용하면서 true일 때까지의 값은 무시하다가 true인 값부터 다음 stream 에 포함시킨다. 주로 정렬된 요소에서 사용하기 좋다. 
5. .limit()
	다음 stream에 들어갈 요소의 개수를 제한한다. 인자로 숫자를 받는다.
6. .skip()
	건너뛸 요소의 개수를 인자로 받는다. 
#### 매핑
7. .map()
	함수를 인자로 받으며, stream의 각 요소에 인자로 전달된 함수가 적용된다. 이 과정은 '기존의 값을 고친다' 라는 개념보다는 =='새로운 버전을 만든다'== 라는 개념에 가깝다. 그래서 인자로 오는 함수는 return 값이 꼭 있어야 한다. 
	map 에서 어떤 type을 return 하느냐에 따라 다음 stream의 내부 요소 type이 바뀐다. 
8. ==.flatMap()==
	flatMap은 하나의 평면화된 stream을 만들어준다. 
	이게 무슨 뜻이냐면 만약 각 요소가 stream으로 되어 있다면 Stream<Stream<>> 이런 형태가 아니라 각 요소 stream을 풀어서 하나의 stream에 합쳐서 반환한다. 그래서 flat 이라는 뜻이 붙는다. 

	예를 들어서 ["Hello", "World"]  이런 배열이 들어 왔을 때, 중복을 제거한 알파벳 배열을 가지고 싶다고 할 때가 있다면 flatMap을 사용하면 된다. ["H", "e", "l", "o", "W", "r", "d"]
	
	```java 
	String[] words = ["Hello", "World"];
	List<String> list =
		words.stream()
			.map(word -> word.split(""))
			.flatmap(Array::stream)
			.distinct()
			.collect(toList());
	```
	우선 String 배열의 각 요소를 알파벳 하나 별로 쪼갠 배열로 만든다. Array::stream 메소드 에서 각 배열 요소를 stream으로 만들고 이를 flatMap을 통해 하나의 stream으로 통합시킨다. 
	그리고 중복 제거하고 list  형태로 만든다. 

	예를 또 들어보면 [1,2,3] [3,4] 이런 식으로 리스트가 주어지면 각 요소들의 모든 조합인 [(1,3), (1,4), (2,3), (2,4), (3,3), (3,4)] 를 반환하는 경우가 있다고 해보자. 
	```java 
	List<Integer> number1 = Arrays.asList(1,2,3);
	List<Integer> number2 = Arrays.asList(3,4);

	List<int[]> pairs = number1.stream()
							.flatMap(i -> 
								number2.stream()
									.map(j -> new int[]{i,j})
							)
							.collect(toList());
	
	```
#### 검색과 매칭
9. .anyMatch()
	Predicate를 인자로 받고 true가 하나라도 있으면 true를 반환한다. 최종연산이다. 
10. .allMatch()
	Predicate를 인자로 받고 모든 요소에 대해 true이면 true를 반환한다. 최종연산이다. 
11. .noneMatch()
	allMatch와 완전 반대로, 모든 요소에 대해 false이면 true를 반환한다. 최종연산이다.

위 세 메소드는 모두 ==쇼트서킷 연산==이다. 전체 stream 연산을 처리하지 않아도 쇼트서킷 연산 중에 결론이 나면 결과를 반환할 수 있다. like (and, or 연산) 

12. .findAny()
	현재 stream에서 임의의 요소를 반환한다. 최종연산이다. 

> ==Optional 이란?==
> java.util.Optional 에 정의되어 있고, 값의 존재나 부재 여부를 표현할 수 있는 컨테이너 클래스이다. java 에서 null은 NullPointerException을 일으키기 때문에 java 8부터는 Optional<> 을 만든 것이다. 
> 
> Optional 에 대한 자세한 설명은 10장에서 한다. 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공한다. `isPresent()`, `ifPresent()`, `get()`, `orElse()`

13. .findFirst()
	논리적인 아이템 순서가 정해져 있을 때, 첫 번째 요소를 반환하는 메서드이다. 최종연산이다. 
	findAny() 는 병렬 stream에서 사용하기 좋지만, findFirst()는 병렬 실행을 제한할 수 있다. 

#### 리듀싱
14. .reduce()
	stream에서 각 요소에 주어진 함수를 적용해서 하나의 값을 도출하는 메서드이다. 최종연산이다. 
	```java
	int sum = numbers.stream()
					.reduce(0, (a,b) -> a+b);
	```
	이런 식으로 코드를 작성한다. 0은 초기 값이고 두 번째 인자는 람다 표현식으로 표현된 함수가 전달된다. a에는 초기 값이 전달되고 b에는 stream의 각 요소가 전달된다. 
	
	초기 값을 안 받을 수도 있다. 
	> 그냥 반복문으로 합계를 구하는 것과 reduce로 만드는 것 중 뭐가 더 좋을까?
	> 반복문으로 구하면 병렬처리가 안된다. sum 변수를 공유해야 하므로 mutex를 쓰다보면 결국 병렬성이 떨어지기 때문이다. reduce는 병렬처리가 가능하다. 
	
#### 그 외
15. .sorted()
	Comparator<> 를 인자로 받아서 정럴된 stream을 반환한다.
16. .forEach()
	Consumer<> 를 인자로 받아서 각 요소에 적용한다. 반환은 void이다. stream의 각 요소에 함수를 적용만 한다. 즉 각 요소를 변화시키는 목적이다. 
17. .collect()
	Collector<> 를 인자로 받는다. 6장에서 살펴보자. 
18. .count()
	stream의 요소 개수를 반환한다. 

#### 숫자형 스트림
위에서 stream 요소의 합을 계산할 때나 최대/최소를 구할 때  reduce를 사용했다. 왜 .sum .max .min 같은 메서드를 제공하지 않는 것일까? 

모든 stream의 요소가 int가 아닐 수도 있기 때문이다. 객체이거나 문자열 같은 타입에 sum, max, min 같은 개념은 없기 때문이다. 하지만 이런 것들을 구하기 위해서 매번 reduce 메서드를 사용하는 것은 불편한다. 그래서 ==기본형 특화 스트림==을 제공한다.
IntStream, DoubleStream, LongStream 등이 있다. 이런 기본형 특화 스트림에서는 해당 기본형에 유용한 메서드들을 제공해준다. 

기본 stream을 기본형 특화 stream으로 mapping 하는 메서드가 있다. 
```java 
int calories = menu.stream()
					.mapToInt(Dish::getCalories)
					.sum();
```
바로 mapToInt 메서드이다. 

다시 기본 stream으로 복원할 수도 있다. 
```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);

Stream<Integer> stream = intStream.boxed();
```
boxed() 메서드를 사용하면 된다. 

이때 IntStream에서 0 이 아닌 null 을 허용하고 싶으면 Optional 클래스를 허용하는 stream인 OptionalInt, OptionalDouble, OptionalLong 을 사용할 수 있다. 

</br></br>

숫자 범위 stream으로 표현하기
```java
IntStream evenNumber = IntStream.rangeClosed(1,100)
								.filter(n -> n % 2 == 0);
```

피타고라스 수만 표현하기 
```java
Stream<int[]> pythagoreamTriples = 
	IntStream.rangeClosed(1,100).boxed()
			.flatMap( a -> 
				IntStream.rangeClosed(a, 100)
							.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
						.mapToObj(b -> 
							new int[]{a, b, (int)Math.sqrt(a*a + b*b)}
						)
			);
```

</br>

==stream은 Collection에서 데이터 처리 질의를 표현하는 강력한 도구이다.== 마치 데이터 베이스 같은 연산을 수행할 수 있다. 


