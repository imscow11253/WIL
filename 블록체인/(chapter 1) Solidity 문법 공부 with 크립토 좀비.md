

크립토 좀비를 생성하는 factory를 만드는 실습을 하면서 solidity 문법을 공부해보자.
https://cryptozombies.io/ko/

solidity로 작성된 코드는 contract로 감싸진다. (코드 상에나 실제로나) (그냥 contract를 만드는 문법이 solidity이다. 이더리움에서 제공)
contract란 block chain 의 기본 단위로, chain 상에 저장할 (이어서 붙일) 정보를 담은 느낌이다. 

```solidity
pragma solidity >=0.5.0 <0.6.0; 

contract HelloWorld { 

}
```

pragma solidity {버전} 키워드를 통해 허용 가능한 solidity 컴파일러를 지정해둔다. 이렇게 해당 contract를 컴파일할 수 있는 컴파일러 버전을 미리 명시해두어서 contract가 깨지지 않는 것을 목표로 한다. 

그리고 contract body 부분을 작성하면 된다.

문법 자체는 java와 비슷하다.

## 자료형

- uint - 256 bit를 가진 unsigned int (uint8, uint16, uint36 등이 있다. default는 256)
- 구조체 생성 가능 (c와 동일)
- 배열은 고정, 가변 둘 다 가능, 삽입 연산은 push
  uint`[`5`]` arr; 
  uint`[` `]` arr;
  uint`[`5`]` public arr; 
  uint`[` `]` public arr;
- typecasting은 c랑 동일하게 uint8 c = a `*` uint8(b);
  
  
## 사칙연산

- 더하기, 빼기, 곱하기, 나누기, 나머지(%) 연산 가능
  제곱 연산은 ** 로 하면 된다. 


## 함수

```solidity 
function eatHamburgers(string memory _name, uint _amount) public returns (string memory) {
	
}
```

- memory 키워드 붙이면 call by reference로 동작, default는 call by value
- 컨벤션 : 함수 인자 앞에 언더바 붙이기 `_`
- 컨벤션 : private으로 선언하면 함수명 앞에 `_`
- getter 느낌의 함수면 view 라는 키워드, 필드 값을 쓰지 않으면 pure라는 키워드 사용
  function sayHello() public view returns (string memory)
  function _multiply(uint a, uint b) private pure returns (uint)
- return 값으로 여러 개를 반환할 수 있다. 
  여러개를 받을 때는 (, , , , ,) = 이런 식으로 받으면 된다.

## 이벤트

contract에서 특정 상황이 될 때마다 event를 발생시켜서 front-end가 눈치채게 할 수 있다. 
```solidity
// declare the event 
event IntegersAdded(uint x, uint y, uint result); 

function add(uint _x, uint _y) public returns (uint) { 
	uint result = _x + _y; 
	// fire an event to let the app know the function was called: 
	emit IntegersAdded(_x, _y, result); return result; 
}
```

그럼 java script는 다음과 같이 event를 수신할 수 있다.
```javascript
YourContract.IntegersAdded(function(error, result) { 
	// do something with the result 
})
```

## 그외 

- 랜덤한 숫자 생성하는 법 (16진수 표현의 256bit)
```solidity
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5 keccak256(abi.encodePacked("aaaab")); //b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9 keccak256(abi.encodePacked("aaaac"));
```
