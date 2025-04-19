
시작하기 전에 하나 짚고 넘어가자.

우리가 contract를 생성하면 블록체인 네트워크의 모든 노드 (컴퓨터)에 내가 작성한 contract 노드가 생긴다. 
누군가 우리 contract의 external 함수를 호출한다는 것은 블록체인 내에 아무 node 하나에 쿼리를 보낸다는 뜻이다. 

## JSON-RPC

이더리움의 노드는 JSON-RPC 라는 언어를 사용한다. 즉 특정 노드에게 contract의 함수를 호출하는 형식은 JSON-RPC 형식을 따르고 다음과 같다. 
```JSON-RPC
// Yeah... Good luck writing all your function calls this way! 
// Scroll right ==> 

{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}
```

우리는 클라이언트를 ==Web3.js== 라는 자바 스크립트 라이브러리를 사용할 건데
다행인 점은 Web3.js에서는 이 복잡한 JSON-RPC 코드를 추상화해서 제공한다. 

```js
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔") .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })
```


Web3.js 에서는 Web3 Provider를 설정해주면 우리가 contract의 함수를 호출하기 위해서 
어떤 노드와 통신하면 되는지 주소를 반환해준다. 

만약 내가 contract를 생성했다.
그리고 이 contract를 사용하기 위해서는 자체적으로 이더리움 노드를 가지고 있어야 할까? (= 내 컴퓨터를 이더리움 노드로 등록해야 하는 걸까?)

Infura 라는 서비스를 사용하면 자체 이더리움 노드를 사용하지 않더라도 DApp(내가 개발한 contract)을 사용할 수 있다.

## Infura

Infura는 자체 이더리움 노드를 가지고 있지 않아도 이더리움 노드를 유지할 수 있도록 한다. 

Web3 provider에서는 다음과 같이 Infura를 지정해주면 된다.
```js
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```


## 디지털 서명

DApp은 많은 사용자가 접근 가능해야 하고, 읽기만 하는 것이 아니라 쓰기 작업도 해야 하므로,
개인 키로 서명하는 방법이 필요하다. 

즉 우리가 contract에 특정 함수를 호출해서 쓰기 작업을 하려면 개인키를 통해 서명을 해야 한다. 

하지만 이런 개인키를 클라이언트가 직접 관리한다면 안전할까?

MetaMask라는 서비스는 클라이언트가 개인키를 잘 관리할 수 있도록 돕는 서비스다. 

## MetaMask

MetaMask는 Chrome과 FireFox의 확장 프로그램이다. 
Web3.js와 상호작용할 수 있고, 사용자가 이더리움 계정과 개인 키를 안전하게 관리할 수 있도록 한다.

이더리움 contract와 통신하는 Web3.js 코드를 작성하려면 MetaMask를 깔아야 한다. 

MetaMask는 브라우저에서 Web3의 존재를 파악하면, 
js 전역 변수인 web3.currentProvider에 클라이언트가 이더리움에 접근하면 되는 node 주소를 넣어주기 때문에 Web3에서는 해당 변수를 가져와서 그냥 사용하면 된다.

## Web3 Provider 설정

js 코드에서 해당 변수를 가져오는 템플릿 코드는 다음과 같다. 
```js
window.addEventListener('load', function() { 

	// Checking if Web3 has been injected by the browser (Mist/MetaMask) 
	if (typeof web3 !== 'undefined') { 
		// Use Mist/MetaMask's provider 
		web3js = new Web3(web3.currentProvider); 
	} else { 
		// Handle the case where the user doesn't have web3. Probably 
		// show them a message telling them to install Metamask in 
		// order to use our app. 
	} 
	// Now you can start your app & access web3js freely:
	startApp() 
})
```


이제 Web3 Provider 설정이 되었으므로 Web3.js 를 사용해서 contract와 통신하도록 하자.

## Web3.js에서 contract와 통신

contract의 주소와 ABI를 알아야 한다.

contract를 이더리움에 배포하면 영구적인 주소를 얻게 된다.

ABI는 Application Binary Interface의 약자로, Web3.js가 contract의 함수를 호출하는 JSON 포맷을 알려주는 것이다. (contract는 JSON-RPC로 통신하기 때문이다.)
ABI도 contract를 컴파일할 때 알려준다. 

주소랑 ABI를 알게 되면 contract를 객체화 할 수 있다. 
```js
// Instantiate myContract 
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```


객체를 생성하면 call과 send 메서드를 사용해서 contract와 통신할 수 있다. 

call 은 view, pure 함수를 호출하는데만 사용된다. 
view와 pure는 로컬 노드에서만 실행되며, 블록체인에 transaction을 생성하지 않는다. 
다음과 같이 사용한다. 
```js
myContract.methods.myMethod(123).call();
```

send는 데이터를 변경하는 메서드를 사용할 때 사용한다. 
즉 블록체인에 transaction을 생성하여 gas를 필요로 한다. 
send 메서드를 사용하는 contract의 메서드를 호출하면, gas를 지불해야 하므로 서명하라는 MetaMask 팝업창이 뜬다. 
다음과 같이 사용한다. 
```js
myContract.methods.myMethod(123).send();
```


여담 : solidity 에서는 변수를 public 으로 선언하면 getter 함수가 자동으로 제공된다. 

web3.js 에서는 다음과 같이 contract와 통신할 수 있다. 
```js
function getZombieDetails(id) { 
	return cryptoZombies.methods.zombies(id).call() 
} 

// Call the function and do something with the result: 
getZombieDetails(15) 
.then(function(result) { 
	console.log("Zombie 15: " + JSON.stringify(result)); 
});
```

getZombieDetails 함수 호출은 비동기 방식으로 이루어지고,
promise를 반환한다. result에는 solidity에서 Zombie 구조체로 정의 되었던 것이 받아지는데 json 형식으로 받아진다 
```json 
{ 
	"name": "H4XF13LD MORRIS'S COOLER OLDER BROTHER", 
	"dna": "1337133713371337", 
	"level": "9999", 
	"readyTime": "1522498671", 
	"winCount": "999999999", 
	"lossCount": "0" // Obviously. 
}
```


## 현재 내 계정 알기

solidity 문법을 작성할 때 `_`from 같이 사용자의 주소를 전달받는 경우가 있었다. 
Web3.js에서 자신의 주소를 어떻게 알 수 있을끼?

다음과 같은 코드를 통해 알 수 있다. (배열인 것을 보면 사용자 계정을 여러개 관리할 수 있다.)
```js
var userAccount = web3.eth.accounts[0];
```


## 그외

- 이더리움에 transaction을 생성하는 send 메서드를 호출하는 경우엔 오래 걸리는 작업이므로 비동기 처리를 해주어야 한다. (모든 node가 검증을 하기 때문에 컴퓨팅 리소스가 많이 듦)
- 이더리움 transaction 생성 작업이 완료되면 receipt 가 fire 된다. 

## Wei 

Ether를 잘게 쪼갠 단위가 wei이다.  ==10^18 `*` wei  = 1 Ether== 이다. 
Web3.js 에선 다음과 같이 쓸 수 있다. 
```js
// This will convert 1 ETH to Wei 
web3js.utils.toWei("1");
```


## event listener

contract에서 발생되는 event를 Web3.js에서 수신할 수 있다. 

```js
cryptoZombies.events.NewZombie() 
.on("data", function(event) { 
	let zombie = event.returnValues; 
	// We can access this event's 3 return values on the `event.returnValues` object: 
	console.log("A new zombie was born!", zombie.zombieId, zombie.name, zombie.dna); 
}).on("error", console.error);
```

다음 코드는 contract에서 발생한 과거 event를 수신하는 코드다. 
과거 event도 가져올 수 있다. 
block 범위를 지정해서 특정 시간에 발생한 event만 필터링할 수 있다. (block은 이더리움에서 생성된 block을 의미한다.)
```js
cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: "latest" })
.then(function(events) { 
	// `events` is an array of `event` objects that we can iterate, like we did above 
	// This code will get us a list of every zombie that was ever created 
});
```


