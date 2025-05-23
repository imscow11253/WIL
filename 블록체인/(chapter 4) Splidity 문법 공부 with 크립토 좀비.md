
## payable modifier

우리가 단순하게 web API를 호출하는 형태로 돈을 송금할 수는 없다. 
근데 이더리움에서는 특정 함수에 payable이라는 modifier를 붙이는 것만으로도 Ether를 전달 받을 수 있다. 
이를 통해 내 contract에서 특정 기능을 사용하려면 Ether를 지불하라는 로직을 만들 수 있다.
이렇게 전달받은 Ether는 내 contract balance에 저장되게 된다. 

msg.value는 해당 함수를 호출할 때 전달된 Ether를 반환하는 것이다. 
보통 다음과 같은 형식으로 Ether 지불 여부를 확인한다.

```solidity
contract OnlineStore { 
	function buySomething() external payable { 
		
		// Check to make sure 0.001 ether was sent to the function call:
		require(msg.value == 0.001 ether); 
		
		// If so, some logic to transfer the digital item to the caller of the function: 
		transferThing(msg.sender); 
	}
}
```

내 contract에서 Ether를 인출하고 싶으면 다음과 같은 코드를 짜면 된다.
```solidity
contract GetPaid is Ownable { 
	function withdraw() external onlyOwner { 
		address payable _owner = address(uint160(owner()));
		_owner.transfer(address(this).balance);
	} 
}
```


## 랜덤 생성 숫자에 대한 보안

우리는 이 chapter에서 랜덤 숫자 생성을 다음과 같은 방식으로 진행하고 있다. 
```solidity 
// Generate a random number between 1 and 100: 
uint randNonce = 0; 

uint random = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100; 
randNonce++; uint random2 = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;
```

keccak256이라는 라이브러리 함수를 사용해서 만들고 있다.
근데 이러면 다음과 같은 문제가 발생할 수 있다고 알려주는데 이해가 잘 안되어서 GPT한테 쉽게 설명해달라고 해서 다음과 같은 결과를 뱉었다. 


<details>
	<summary>
		GPT 답변
	</summary>
	이 내용을 쉽게 설명해 드릴게요.

1. 이더리움에서 계약 실행은 어떻게 이루어지나요? (일반적인 과정)

- 요청 보내기: 당신이 이더리움 스마트 계약의 특정 기능(예: 게임 참여, 송금)을 사용하고 싶을 때, 이 요청을 '거래(Transaction)'라는 형태로 네트워크에 보냅니다. 마치 은행에 "송금해주세요"라고 요청하는 것과 같아요.
- 거래 모으기: 네트워크의 컴퓨터들(노드, 특히 채굴자 노드)은 이렇게 들어온 여러 사람들의 거래 요청들을 모읍니다. 은행 창구에서 여러 사람의 요청서를 모으는 것과 비슷해요.
- 블록 만들기 경쟁 (작업 증명, PoW): 이제 노드들은 이 모은 거래들을 하나의 묶음('블록')으로 만들고, 이 블록을 공식적으로 인정받기 위해 아주 어려운 수학 문제를 푸는 경쟁을 시작합니다. 이걸 '작업 증명(Proof of Work, PoW)'이라고 해요. 가장 먼저 문제를 푼 노드가 이번 블록을 만들 권한을 얻습니다.
- 블록 전파 및 검증: 문제를 가장 먼저 푼 노드는 자기가 만든 거래 묶음(블록)과 문제 풀이 증명(PoW)을 네트워크의 다른 모든 노드에게 "내가 먼저 풀었어! 이게 정답이고 거래 목록이야!"라고 알립니다.
- 합의 및 다음 단계: 다른 노드들은 이 블록 안의 거래들이 유효한지, 그리고 문제 풀이가 정말 맞는지 검증합니다. 모두 "맞네!"라고 동의하면, 이 블록을 공식 기록(블록체인)에 추가하고, 다음 블록을 만들기 위한 새로운 문제 풀이 경쟁을 시작합니다.

1. 왜 이게 랜덤 숫자 생성에 문제를 일으킬까요?

위 과정 때문에, 블록을 만드는 **채굴자(노드 운영자)**는 어떤 거래를 포함시킬지 **선택할 힘**을 가집니다.

3. 동전 던지기 게임 예시로 살펴보기:

- 게임 규칙: 50 이상이면 앞면(승리, 돈 2배), 50 미만이면 뒷면(패배, 돈 잃음). 결과는 블록 정보 등을 이용한 '랜덤' 함수로 결정된다고 가정합시다.
- 채굴자의 악용 방법:
    1.  나쁜 마음을 먹은 채굴자 A가 이 동전 던지기 게임에 참여하고 싶어 합니다.
    2.  A는 "동전 던지기 게임 참여"라는 거래 요청을 만들지만, 다른 사람들에게 알리지 않고 자기 컴퓨터에만 몰래 보관합니다.
    3.  A는 자기 컴퓨터에서만 미리 동전 던지기 결과를 계산해 봅니다 (이때 사용되는 '랜덤' 값은 A가 만들 블록 정보에 의존하므로 예측 가능).
    4.  결과 확인:
        - 만약 졌다면 (뒷면): A는 이 거래 요청을 그냥 버립니다. 아무도 A가 게임에 참여하려 했는지조차 모릅니다. 그리고 A는 계속해서 다음 블록 만들기 경쟁(수학 문제 풀이)을 합니다.
        - 만약 이겼다면 (앞면): A는 "앗싸!" 외치며 이 이긴 거래 요청을 자기가 만들고 있는 블록에 포함 시킵니다.
    5.  A는 자신이 이길 때까지 이 과정을 반복합니다. 그러다가 드디어 동전 던지기에서 이기고 + 동시에 어려운 수학 문제(PoW)도 가장 먼저 풀어서 블록을 만들 권한을 얻으면, 그제서야 자신이 이긴 거래가 포함된 블록을 네트워크에 공개합니다.

결론:

이런 방식 때문에 채굴자는 자신이 불리한 결과(패배)는 무시하고, 유리한 결과(승리)만 선택적으로 블록에 포함시킬 수 있습니다. 따라서 블록 정보에 기반한 단순한 랜덤 함수는 예측 가능하고 조작될 수 있어, 공정한 게임 등에 사용하기에는 매우 위험하다는 뜻입니다. 채굴자가 항상 이길 수 있게 되는 거죠!
</details>


이 크립토 좀비 튜토리얼은 이런 난수 생성 방식이 위험하다는 것을 알지만 확률이 극히 낮을 뿐더러 우린 단순 튜토리얼이라서 그냥 하기로 한다. 

