
## 토큰에 대해서

토큰은 공통의 규칙을 가지는 하나의 ==smart contract==일 뿐이다. 
smart contract은 내부적으로 mapping (address => uint256) balance 라는 자료구조를 가진다. 

토큰이라는 contract도 하나의 contract이기 때문에 내부에 balance 정보를 가진다. 
기본적으로 토큰이라는 contract는 누가 얼마나 많은 토큰을 가지고 있는지를 추적하는 contract이다.
그리고 특정 토큰을 이동시키고 하는 로직을 포함한다. 

ERC20 이라는 것은 토큰의 표준 중 하나로, 대체 가능하다는 특징을 가진다. 이는 이더리움에서 사용된다. 
우리가 사용할 ERC721는 똑같이 토큰의 표준 중 하나인데, 대체 불가능하다는 특징이 있다. 

ERC20이라는 토큰을 우리 contract에서 사용한다면 다른 contract와도 해당 토큰을 공유할 수 있다는 장점이 있다. Ether와의 차이점은 Ether는 이더리움이라는 시스템에서만 유효한데, ERC20 토큰은 비트코인, 도지코인 등 다른 시스템과도 공유가 가능하다는 장점이 있다. 

우리는 ERC721 이라는 토큰 표준을 사용할 것이다. 

Why?
우린 좀비를 표현하는데 토큰을 사용할 것인데, ERC20은 0.237 처럼 나눌 수 있다. 
하지만 0.237 좀비 라는 것은 존재하지 않기 때문에 ERC20 토큰을 사용할 수는 없다.
그리고 ERC20은 대체 가능하기 때문에 좀비를 표현하는데 적합하지 않다. 

ERC721 토큰의 interface는 다음과 같다. 
```solidity
contract ERC721 { 

	event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId); 
	event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId); 
	
	function balanceOf(address _owner) external view returns (uint256); 
	function ownerOf(uint256 _tokenId) external view returns (address); 
	function transferFrom(address _from, address _to, uint256 _tokenId) external payable; 
	function approve(address _approved, uint256 _tokenId) external payable; 
}
```

이걸 사용하려면 우리 contract에서 import 하고 ERC721를 상속하면 된다.

각 함수를 설명하면 다음과 같다. 
- ==balanceOf==
  주소를 받아서, 해당 주소에 토큰이 몇 개 있는지를 반환한다. 
- ==ownerOf==
  토큰을 받아서, 해당 토큰을 소유하고 있는 주소를 반환한다.
- ==transferFrom, approve==
  토큰을 전달하는 방법으로는 2가지가 있다. 
  1. The first way is the token's owner calls `transferFrom` with his `address` as the `_from` parameter, the `address` he wants to transfer to as the `_to` parameter, and the `_tokenId` of the token he wants to transfer.
  2. The second way is the token's owner first calls `approve` with the address he wants to transfer to, and the `_tokenID` . The contract then stores who is approved to take a token, usually in a `mapping (uint256 => address)`. Then, when the owner or the approved address calls `transferFrom`, the contract checks if that `msg.sender` is the owner or is approved by the owner to take the token, and if so it transfers the token to him.
  
  크립토 좀비에서는 transfer 이라는 함수를 만들어서 이를 추상화 한다.
  그리고 해당 transfer 함수 마지막에 ERC721에서 정의되어 있는 Transfer 라는 event 를 emit 한다.


정리하자면 이더리움에서 토큰이란 contract에서 발행하는 문자열인 것이다. 대체 가능/불가능 토큰으로 나뉜다. contract에서 토큰을 발행하기 위해서는 특정 양식 ERC20 이나 ERC721 같은 contract의 인터페이스를 구현하면 된다. 
Ether는 이더리움 내에서 통용되는 화폐의 느낌이면, token은 특정 양식을 따르기만 하면 통용되는 화폐? 문자열? 의 느낌이 강하다. 

이거 구현해보기 전에 토큰이 뭔가에 대해서 블로그 포스팅을 많이 찾아봤는데..
역시 제일 이해하기 빠른건 직접 구현해보는 것이다. 
우리 예제에서는 좀비를 구분짓고 거래하기 위해 좀비에다가 token을 부여하는 것이다. 


## SafeMath

OpenZepplin에서는 오버/언더 플로우를 방지하기 위한 라이브러리인 safeMath를 제공하고 있다.
블록체인에서 라이브러란 특수한 형태의 contract를 의미한다. 

다음과 같은 연산을 활용한다. 
```solidity
using SafeMath for uint256; 

uint256 a = 5; 
uint256 b = a.add(3); // 5 + 3 = 8 
uint256 c = a.mul(2); // 5 * 2 = 10
```

SafeMath는 contract 키워드가 아니라 library 키워드를 사용한다. 
library라는 키워드로 선언된 contract는 using 이라는 키워드로 메소드들을 가져올 수 있다.

오버/언더 플로우가 감지되면 내부적으로 require이 아니라 assert를 사용하는데, 
차이점은 require은 오류가 발생하면 gas를 반환해주지만 assert는 gas를 반환해주지 않는다. 

근데 SafeMath는 uint256으로 타입캐스팅 하기 때문에 다른 type은 지원하지 않는다.
그래서 직접 구현해주면 된다. 


## 컨벤션

solidity 커뮤니티에서는 주석으로 contract의 기능을 표현한다. 
각 주석마다 @ 기호를 붙여서 주석의 설명을 보조한다. 

```solidity
/// @title A contract for basic math operations
/// @author H4XF13LD MORRIS 💯💯😎💯💯 
/// @notice For now, this contract just adds a multiply function 

contract Math { 
	/// @notice Multiplies 2 numbers together
	/// @param x the first uint. 
	/// @param y the second uint. 
	/// @return z the product of (x * y) 
	/// @dev This function does not currently check for overflows 
	function multiply(uint x, uint y) returns (uint z) { 
		// This is just a normal comment, and won't get picked up by natspec 
		z = x * y; 
	} 
}
```

@title은 제목
@author는 만든 사람
@notice는 정보
@dev는 개발자를 위한 설명
@param은 함수 파라미터 정보
@return은 리턴 값 정보