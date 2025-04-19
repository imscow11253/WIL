
## 이더리움 개념

이더리움 블록체인 시스템은 계좌(account)를 만들어 준다. 
그리고 각 계좌에는 잔액(balance)이 존재하고, Ether 라는 단위를 사용한다. 
각 계좌에는 주소(address)가 존재한다. 

이 계좌는 특정 사용자 또는 스마트 컨트랙트마다 생성될 수 있다. 

그럼 앞서서 생성한 좀비 id에 우리 주소를 매핑하면 우리가 생성한 좀비가 누군지 알 수 있다. 


## 자료형

- mapping이라는 이름으로 map 자료형을 제공한다. 
```solidity
// For a financial app, storing a uint that holds the user's account balance: 
mapping (address => uint) public accountBalance; 
// Or could be used to store / lookup usernames based on userId 
mapping (uint => string) userIdToName;
```

- msg.sender 
  해당 contract의 함수를 호출한 유저 또는 스마트 컨트랙트의 주소를 반환한다.
  보통 블록체인에서는 msg.sender를 활용해서, 현재 호출한 주소와 인자로 전달받은 개인키 간의 매칭이 맞는지 확인하는 방식으로 보안성을 높인다. 

- 메모리 저장 vs 디스크 저장
  블록체인 내에 영구적으로 저장하는 것을 디스크 저장, ram처럼 동작하는 것을 메모리 저장이라고 한다. 
  보통 contract에서 필드 값으로 선언된 것은 디스크 저장, 함수에서 local로 선언된 변수를 메모리 저장이다. solidity 가 알아서 해준다. 
  근데 명시적으로 지정할 수 있다. memery, storage 라는 키워드를 사용해서..

## Require

함수에서 특정 조건을 만족하는지 확인하고 싶을 때 사용하는 문법이다. 
```solidity
require(keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked("Vitalik")));
```

require 키워드 안에 조건문이 들어가게 되고, 만족하지 않으면 오류를 반환한다. 


## 상속

contract 간 상속이 가능하다. 
```solidity 
contract Doge { 
	function catchphrase() public returns (string memory) { 
		return "So Wow CryptoDoge"; 
	} 
} 

contract BabyDoge is Doge { 
	function anotherCatchphrase() public returns (string memory) { 
		return "Such Moon BabyDoge"; 
	} 
}
```

internal 키워드를 특정 변수나 함수에 선언하면, 상속 받은 contract에서 쓸 수 있다는 것이고
external 키워드는 상속받은 contract나 자체적으로 사용 불가, 외부에서만 호출할 수 있는 함수를 의미한다.


## 블록체인 내 다른 contract와 통신하기

인터페이스에 따라 함수를 호출해야 한다.
해당 인터페이스를 외부에 공개해서 내 contract의 함수를 호출하게 하여 통신하도록 할 수 있다. 

호출하고자 하는 interface를 작성하고, 해당 인터페이스를 생성할 때, address를 인자로 넘겨주면 객체로 받아올 수 있다. 

## 그외

- import 키워드로 다른 파일의 contract 코드를 가져올 수 있다.