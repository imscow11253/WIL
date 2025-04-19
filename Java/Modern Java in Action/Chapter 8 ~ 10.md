
## Chapter 8  컬렉션 API 개선
---

이 장의 목표 : 자바8, 9 에서 개선된 컬렉션 API를 배울 것이다. 

자바 9에서는 ==작은 컬렉션을 쉽게 만들 수 있는 방법==을 제공한다. 

>[!note] 작은 컬렉션이란? 
>적은 요소를 가지는 컬렉션을 말한다. 

### 기존 방식의 문제점

예를 들어서 다음과 같은 경우가 있을 수 있다.
```java
List<String> friends = new ArrayList<>();
friends.add("a");
friends.add("b");
friends.add("c");
```

위의 예시에서 문제점이 뭘까? 요소 3개 넣을 뿐인데 너무 길다. 
Array의 asList() 라는 팩토리 메서드를 사용하면 간단하게 줄일 수 있다. 
```java
List<String> friends
	= Arrays.asList("a", "b", "c");
```

하지만 이건 고정 크기의 리스트라서 요소를 변경할 순 있지만 추가할 수 없다는 문제점이 있다. 


Set은 어떨까? 기존의 방식으로 set을 만들어보자.
```java
Set<String> friends
	= new HashSet<>(Arrays.asList("a", "b", "c"));
```
또는
```java
Set<String> friends
	= Stream.of("a","b","c")
		.collect(Collectors.toSet());
```
으로 표현할 수 있다. 

하지만 두 방법 모두 매끄럽지 못하며, ==내부적으로 불필요한 객체 할당을 한다.== 그리고 set이 변경될 우려도 있다. Map 도 마찬가지이다. 

이런 문제점을 자바 9에서 해결해준다.


### 1. List 팩토리 (자바 9에서 추가)

==List.of== 팩토리 메서드를 사용해서 List를 만들면 고정 크기를 할당하기 때문에 새로운 요소를 추가할 수 없다. UnsupportedOperationException이 발생한다. 만약 변하는 list를 만들고 싶으면 직접 list를 만들면 된다. 또는 stream 의 toList()를 사용하면 된다. 

이런 제약은 장단점이 존재한다. 

장점 
- 컬렉션이 의도치 않게 변하는 것을 막는다.
- null은 금지하므로 데이터 무결성

```java 
List<int> list = List.of(1,2,3);
list.add(4); //UnsupportedOperationException 발생
```


### 2. Set 팩토리 (자바 9에서 추가)

똑같이 ==Set.of== 이라는 팩토리 메서드를 사용하면 된다. List와 똑같이 내부 요소의 개수를 변경할 수 없다. 중복 요소를 지정할 수도 없다. (IllegalArgumentException 발생)

```java
Set<int> set = Set.of(1,2,3);
Set<int> set2 = Set.of(1,1,1); //IllegalArgumentException
set.insert(4); //UnsupportedOperationException
```

### 3. Map 팩토리 (자바 9에서 추가)

==Map.of== 팩토리를 사용하면 변할 수 없는 Map을 만들 수 있다. 다만 key, value를 인자로 번갈아 제공해야 한다.

```java
Map<String, int> map = Map,of("Raphael", 30, "Olivia", 25, "Thibaut", 26)；

```

요소가 10개 이하면 위의 방법이 유용하고, 요소가 좀 많을 경우엔 다음과 같은 방법이 유용하다. 

```java
Map<String, Integer> map= Map.ofEntries(
		entry("Raphael", 30),
		entry("Olivia", 25),
		entry("Thibaut", 26)
	);
```


### List와 Set의 메서드

removeIf 와 replaceAll 메서드가 추가되었다. 

>[!Info] 이런 메서드 왜 추가됨?
>Stream으로 바꿔도 되지만 Stream은 기존 컬렉션은 유지한채, 새로운 컬렉션을 만들어서 반환한다. 상단의 메서드는 기존 컬렉션 자체를 바꾼다. 컬렉션을 바꾸는 행위는 복잡하고 많은 에러를 유발하기 때문에 별도의 메서드로 지원하는 것이다.


1. ==RemoveIf 메서드==

어떤 list에서 특정 요소를 삭제하는 코드를 작성해보자.

```java
for (Transaction transaction : transactions) {
   if(Character.isDigit(transaction.getReferenceCode(),charAt(0))) {
	   transactions.remove(transaction);
	}
)
``` 

   잘 동작할 것 같지만 ==ConcurrentModificationException== 에러가 발생한다. 왜?
   해당 코드는 for-Each 구문을 사용했기 때문에 Iterator를 사용하고, 풀어서 쓰면 다음과 같다.
   
```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
	Transaction transaction = iterator.next() ；
	if(Character.isDigit(transaction.getReferenceCode(),charAt(0))) {
		transactions.remove(transaction);
	}
}
```

여기서 두 개의 개별 객체가 각각 컬렉션을 관리하고 있고, Iterator는 컬렉션 상태와 동기화 하지 않기 때문에 Iterator가 도는 동안 컬렉션의 요소가 변하면 에러가 발생하는 것이다. 

- Iterator 객체
- Collection 객체 자체

그러면 remove 할 때 transactions.remove(transaction); 가 아니라 iterator.remove(); 처럼 iterator를 사용해서 remove하면 된다. 

근데 이러면 코드가 복잡해지고, 개발자가 실수할 확률이 있기에 removeIf 를 사용하면 된다. 
인자로 Predicate 템플릿 메서드를 넣어주면 된다.


2. ==ReplaceAll 메서드==

특정 요소를 조건에 맞으면 변환하고 싶을 때 stream API를 사용할 수 있다. (나였어도 stream 쓸 것 같다. ) 하지만 이는 새로운 컬렉션을 만들어낸다는 단점이 있다. 그래서 Iterator를 직접 써도 된다. 하지만 이렇게 하면 코드가 복잡해진다. 그래서 나온 메서드가 replaceAll() 메서드이다. 


### Map 의 메서드

자바 8에서 map 관련 디폴트 메서드가 추가되었는데, 13장에서 자세하게 다룬다. 여기서는 기능만 보도록 하자. 자주 사용되는 패턴을 메서드로 만든 것이다. 

1. ==forEach 메서드==

for-Each를 외부에서 명시적으로 사용해서 map의 entry를 받을 수 있다. 
```java
for(Map.EntryくString, Integer> entry： map.entrySet()) {
	...
)
```

자바 8에서부터는 forEach 메서드를 사용하면 된다.

```java
map.forEach(
	(key, value) -> ...
)
```


2. ==정렬 메서드==

이건 stream을 쓰는 건데...? 암튼 stream으로 정렬할 때 다음 2개의 유틸리티를 stream().sort() 의 인자로 넘겨줄 수 있다고 한다. 

- Entry.comparingByKey
- Entry.comparingByValue

```java
map.entrySet()
	.stream()
	.sorted(Entry.comparingByKey())
	.forEachOrdered(System.out::println);
```


3. ==getOrDefault 메서드==

map에서 key를 찾을 때 key가 없으면 NULL이 반환된다. NullPointerException을 방지하려면 null 체크를 해줘야 하는데, getOrDefault 를 쓰면 편리하다. 

이 메서드는 첫 번째 인자로 찾으려는 key, 두 번째 인자로, 없을 경우 반환할 default 값을 받는다. 

```java
map.getOrDefault("찾으려는 key", "없을 때 반환할 default 값");
```


4. ==계산 패턴==

Map에 특정 값이 있을 때, 없을 때 동작을 지정할 때 사용한다.

- computeIfAbsent :  제공된 키에 해당하는 값이 없으면 값을 계산하고 맵에 추가한다.
- computeIfPresent :  제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
- compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.

Map<String, List`<`Integer`>`> 이런 형태를 많이 쓴다. (나도 많이 썼음) 이럴 때 key가 없다면 new ArrayList<>() 를 추가해주고 add 해야 한다. 이걸 아래와 같이 간략하게 만들 수 있다. 

```java
map.computeIfAbsent("key", name -> new ArrayList<>())
	.add("value");
```


5. ==삭제 패턴==

기존에도 remove가 존재했는데 key를 인자로 받아서 (key, value) entry를 삭제하는 방식이었다. 이를 오버로딩한 (key, value) 를 인자로 받아서 (인자 2개) 일치하는 entry를 삭제하는 메서드가 추가되었다. 
--> 이걸 기존 remove를 활용해서 value도 일치하는지 확인하고 지우는 코드로 풀어쓰면 복잡하는 것을 쉽게 예측할 수 있다. 


6. ==교체 패턴==

맵의 항목을 바꾸는 2개의 메서드가 추가되었다. 
- replaceAll()
  BiFunction 이라는 템플릿 메서드를 받아서 모든 entry에 대해서 실행한다.
- replace()
  key를 인자로 받아서 key가 존재하면 value를 바꾼다. (key, value) 가 일치하면 교체하는 버전의 오버로딩 메서드도 있다. 


7. ==합침==

두 개의 map을 합칠 때 사용하는 메서드이다. 

map1과 map2에 중복된 키가 없다면 다음과 같이 putAll() 메서드를 사용해서 합칠 수 있다. 
```java
new_map.putAll(map1);
new_map.putAll(map2);
```

중복된 key가 있다면, 중복된 key를 어떻게 합칠지를 BiFunction 템플릿 메서드 함수를 인자로 받는 merge 메서드를 사용하면 된다. 

```java
new_map.putAll(map1);
map2.forEach( (k,v) -> 
		new_map.merge(k, v, (v1, v2) -> {중복된 key의 value 2개를 합칠지})	
	)
```

![[Pasted image 20250123171902.png]]
merge는 key가 없거나 key의 value가 null이면 받은 값을 추가해주기도 해서 computeAbsent 를 안써도 된다.

### ConcurrentHashMap 추가

최신 기술이 반영된, 동시성 친화적인 HashMap 클래스이다. 자료구조에서 특정 부분만 lock 할 수 있어서 동시에 insert 할 수 있다. 

이건 교재를 살펴보자. 


## Chapter 9 : 리팩터링, 테스팅, 디버깅
---

이 장의 목표 : 기존 형식의 자바 코드를 리팩터링 하는 기법들을 설명한다. 

앞서서 람다 표현식과 stream API를 배웠는데 문법만 배웠을 뿐, 어떻게 이들을 활용해서 가독성이 좋고 유연한 코드로 만들 수 있을지 설명하지 않았다. 여기서는 기존 코드를 리팩토링 하면서 차이점을 설명한다.

## Chapter10 : 
---

이 장의 목표 : 도메인 지정언어를 만들어서 가독성 높이기 (리팩토링)