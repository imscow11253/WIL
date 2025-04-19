
25.1.20 ~ ing

### OpenStack 시작 전 
---
![[Pasted image 20250120215828.png]]

- OpenStack의 각 컴포넌트가 서로 통신할 때 AMQP라는 프로토콜을 사용해서 통신한다. 
- deploy 할 때는 message broker, DB 같은 건 아무거나 써도 된다. RabbitMQ, MySQL, MariaDB, and SQLite 등등
- 유저는 웹 인터페이스, CLI 환경으로 OpenStack cloud에 접근할 수 있다. 
- OpenStack 내부에서 어떤 아키텍처를 정하고 구축할지는 천차만별이다. 상황에 맞게 적절한 아키텍쳐를 선정해서 구축하면 된다. 물리적 환경이든 논리적 환경이든

### OpenStack 물리적 요소들 (하드웨어 자원)
---
![[Pasted image 20250120222212.png]]

- Controller 
  하는 일이 많다. ID 서비스, Image 서비스, 배치 서비스, 네트워킹, 대시보드 등 다양한 서비스를 운영하기 위한 하나의 컴퓨터이다. 
  최소 2개의 NIC가 필요하다.

- Compute
  VM을 생성/관리하기 위한 hypervisor를 구동할 하나의 컴퓨터이다. 
  hypervisor로는 다양한 툴들이 있는데 default로는 KVM(kernel-based VM)을 사용한다.
  VM과 VLAN을 연결하기 위한 Networking Service Agent 가 필요하다. 
  최소 2개의 NIC가 필요하다.

- Block Storage (optional)
  VM을 위한 storage를 제공해주는 하드웨어 자원이다. block 기반으로 데이터를 다루기 때문에 block storage라고 부른다. 물리적 구성의 단순화를 위해서 compute 노드랑 block storage는 management network를 사용하는데, 실제 배포할 때는 별도의 storage network를 구성해주는 것이 좋다. 

- Object Storage (optional)
  계정, 컨테이너, 그리고 객체를 저장하기 위한 disk가 있는 하드웨어 자원이다. 이것도 block storage랑 마찬가지로 단순하게 구현하기 위해 compute 노드와 통신할 때 management network를 사용하는데 배포할 때는 별도의 storage networ를 구성해주는 것이 좋다.
  최소 2개의 노드가 필요하고, 각각 NIC가 하나 필요하다. 

- Provider networks
  구축해놓은 OpenStack 인프라를 L2 switch와 VLAN 만으로 배포할 수 있게 해주는 network이다. DHCP 프로토콜을 사용해서 VM에 ip를 할당해준다. 
  DHCP 란? https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-DHCP-%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80

- Self-service networks
  NAT를 사용해서 가상 네트워크를 물리 네트워크로 라우팅한다. --> 이해 안감. 나중에 상세히 설명





