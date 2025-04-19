25.1.16

## 전체 주제 : 클라우드 서비스의 밑바닥 (EC2가 어떻게 만들어졌는지)

### OpenStack의 역사
---
컴공 툴의 역사를 보면 항상 기업의 제품이 나오면 거기에 대응하는 오프소스 라이브러리가 등장한다. 클라우드에서는 대표적으로 AWS Cloud랑 OpenStack이 있다. 

Nasa에서 웹 서비스를 통일된 포맷으로 만들기 위한 Nasa.net 프로젝트에서 발전된 형태다. Nasa.net에서는 코드만 웹 사이트에 올리면 알아서 infra 구축해주고, 웹 서버 띄우고 하는 과정을 통일된 방식으로 해주는데, 이 기술 기반으로 클라우드 서비스로 발전된 형태가 Nasa의 Nebula이고, 이를 RacSpace 라는 기업이 같이 진행하면서 OpenStack이 등장했다. 
https://spinoff.nasa.gov/Spinoff2012/it_2.html (Nasa 포스트)

OpenStack은 아직까지도 100% python으로 동작한다. 

### OpenStack 이란
---
공식 정의
Cloud Infrastructure for Virtual Machines, Bare Metals(물리적인 머신에 어떤 SW도 없는 상태) and Containers.

Cloud infrastructure는 IT 인프라 자원 (물리적인 자원)을 클라우드 서비스로 제공하는 형태 (= IaaS, Infrastructure as a Service) 
물리적인 IT 자원을 가상화하고 추상화된 인터페이스를 만들어 API로 제공한다. 

Cloud가 뭔데? (면접에서 물어본다면?)
내가 원하는 자원을 언제 어디서든 인터넷을 통해서 제공해주는 것

### OpenStack 의 구조
---
![[Pasted image 20250117165507.png]]

간략하게만 설명했다. 빨간 박스가 핵심 컴포넌트이고, 초록 박스는 만든 infra 자원을 다채롭게 쓰는 도구들 like container, auto scale, provisioning 등등 오른쪽 별도의 박스는 오케스트레이션 툴이다. 보라 박스는 사용자 인터페이스 제공 툴이나 API이다. 
다만 EC2 API가 제공되는데 이건 뭐냐? 라고 하면, 요즘은 private, public 클라우드를 섞어쓰는 하이브리드 클라우드가 많이 쓰이니까 이때 기존 AWS를 쓰던 사람들이 같은 API를 써도 openstack에서 쓸 수 있게끔 변형 시켜주는 API이다. 

OpenStack의 6가지 기본 컴포넌트는 다음과 같다.
![[Pasted image 20250118021625.png]]


오픈스택은 자원(서버, 스토리지, 네트워크)의 상태를 관리할 뿐이다. 자원의 실체는 이미 만들어진 다른 서비스에게 위임한다. 이제 VM(virtual machine)을 제공하기 위한 요소 3개를 기준으로 openstack이 어떻게 관리하는지를 살펴보자. 요소 3개는 Computing resource, Storage, Network 이다. 

### OpenStack에서 Computing Resource 관리

![[Pasted image 20250118022112.png]]
Nova라는 컴포넌트로 관리한다. 하나의 머신에서 여러 개의 VM을 띄울 수 있게 해주는 SW를 Hypervisor라고 하는데 Nova에서는 여러 개의 Hypervisor 중 하나와 연결하여 openstack이 computing resource를 관리할 수 있게 해준다.  아래 그림은 Hypervisor 종류들이다. 
![[Pasted image 20250118022417.png]]


### OpenStack에서 OS Image Resource 관리

![[Pasted image 20250118022622.png]]
VM을 생성할 때마다 OS를 소프트웨어로 받아와서 설치하면 적어도 30-40분 정도 걸린다. 만약 우리가 EC2 하나 생성하는데 30-40분 뒤에 쓸 수 있다고 하면 굉장히 불편할 것이다. 그래서 OS를 설치하는 것이 아니라 바로 부팅 가능한 형태인 image로 만들어서 관리할 수 있게 하면 좋을 것이다.(docker image 마냥) 이것을 가능하게 하는 openstack 컴포넌트가 glance이다. 

OS를 처음 설치하면 계정 정보, ip 등과 같은 설정 정보를 받아야 하는데 같은 image를 사용하면 VM 마다 같아지게 된다. 이를 방지하기 위해서 설정 정보를 자동화 해주는 툴인 cloud-init 이 사용되고 포함되어 있다. 

image disk format은 다양하다. 
- raw
- vhd
- vmdk 
- iso
- qcow2

만약 ubuntu 새로운 버전이 release 되었다고 해보자. 그럼 ubuntu에서 cloud를 위한 image를 함께 release 한다. 그럼 우리는 그걸 다운 받아서 glance에 등록해두면 된다. Windows는 오픈소스가 아니므로 MS와 계약을 맺어야 한다. 


### OpenStack에서 Storage Resource 관리

![[Pasted image 20250118023629.png]]

우선 Block Storage란? 운영체제, 데이터베이스 수업에서 배웠던 것처럼 Block 단위로 disk를 관리하는 Storage라고 생각하면 된다.  
클라우드에서 제공하는 Storage는 우리가 일반적으로 사용하는 SSD, HDD, NAS 이런 거랑은 차원이 다르다. 고스펙, 고사양, 고용량이어야 한다. 그래서 Vender 사의 스토리지 전용 장비를 쓴다. 
이런 것들을 쓰면 Cinder랑 어떻게 연결하냐? 두가지 방법이 있다. 
1. Vender 사에서 제공하는 드라이버를 사용한다. 
2. Ceph 과 같은 오픈소스 툴을 사용한다. 
3. LVM (리눅스가 스토리지 관리하는 방법)

1번 방법은 쉽지만 모든 기능 지원이 힘들고, Vender 종속적이게 된다는 단점이 있다. 2번은 어렵지만 기능 확장이 쉽고, 유연하다는 장점이 있다. 네이버, 카카오, 삼성 같은 큰 대기업은 전문 Ceph 팀이 있어서 운영되지만 대표님은 NHN Cloud 같은 비교적 작은 회사는 Ceph을 다루기 어려우므로 Vender 사 드라이버를 사용한다고 한다. 

자 그럼 Cinder가 어떻게 인스턴스에게 스토리지를 제공해주는 흐름을 살펴보자. 
![[Pasted image 20250118032011.png]]
![[Pasted image 20250118032229.png]]

여기선 Storage로 NAS를 쓴다고 가정한다. Storage의 A존에 100GB 디스크의 ubuntu 22.04 인스턴스를 만들어달라는 요청을 들어오게 되면 Cinder는 우선 OpenStack에서 관리하는 Storage 정보에서 A존에 대한 정보를 조회한다. 그리고 Glance에 저장된 압축된 OS image를 가져와서 압축을 푼다(raw 파일로 변환). 그리고 NAS의 A존에 100GB 크기의 OS를 띄운다. 그리고 Hypervisor가 동작하는 서버 디바이스에서 실행된 VM에 해당 공간을 Mount 해준다.
Hypervisor가 동작하는 서버의 디스크에서 OS를 띄우고 VM에 mount 해줘도 되는데 이렇게 하는 것의 장점의 데이터 보호이다. Hypervisor가 동작하는 서버는 의외로 쉽게 죽는다. 실제로 AWS에서 데이터 베이스의 에어컨이 고장나서 고열로 VM이 동작하는 서버들이 죽는 경우가 발생했고, AWS를 사용하던 쿠키런 킹덤이 서비스 장애를 겪었다. 하지만 위 그림과 같은 구조로 되어있어 데이터는 영향이 없었고 쉽게 복구할 수 있었다고 한다.


### OpenStack에서 Network Resource 관리

![[Pasted image 20250118033310.png]]

openstack에서 ==통신이 가능한 Layer 2 환경을 만들어주는 컴포넌트==이다.
물리적인 네트워크를 구성해두고 여기에 사용되는 switch나 기기들은 packet을 다루기만 하도록 설정해두고 각 packet의 논리적인 제어는 Neutron이 하는 것이다. 즉 SDN (software defined network)라고 할 수 있다. -> open v switch 같은 것을 제어하는 역할을 맡는다. 

openstack에서 네트워크를 구성할 때 가장 중요한 안건은 ==어떤 형태의 가상 네트워크를 제공할 것인가==이다. 네트워크를 잘 알아야 한다. 가장 간단한 네트워크 분리 기술이 VLAN (Virtual Local Area Networ) 이다. [[VLAN 톺아보기 (조성수 대표님 유튜브)]]

openstack은 고객에게 cloud를 제공하는 것이 목표다. 자세하게 말하면 고객 별로 VM을 제공하고 동일한 물리적 네트워크 환경에서 고객 별로 분리된 VM과 Storage에 접근할 수 있는 가상의 네트워크를 제공하는 것이 목표이다. 근데 물리적 네트워크를 분리할 수 없으니 VLAN을 사용하는 것이고, VPC (virtual private cloud) 마다 VLAN 번호를 할당해주어야 한다. 이 할당 작업은 Neutron에서 해주는 것이다. 

![[Pasted image 20250118041335.png]]

neutron agent에 새로 생성된 VM에 B 네트워크를 위한 VLAN 200번을 할당하라는 요청이 들어오면, neutron은 OpenVSwitch라는 SW로 구현된 switch에게 명령을 내린다. OpenVSwitch는 가상의 port를 생성해서 B 네트워크를 사용하는 VM의 eth0와 연결하고(VM의 NIC가 OpenVSwitch와 연결되는 거임), 해당 eth0에서 나오는 packet에는 VLAN 200 이라는 정보를 추가하고, 물리 인터페이스로 내보내는 것이다. 

 ![[Pasted image 20250118041703.png]]

위의 그림은 여러 개의 VM이 OpenVSwitch에 연결된 형태를 보여준다. 

만약 VLAN이 아니라 VxLAN이라는 확장된 가상 네트워크 분리 기술을 사용한다면 형태는 더욱 복잡해진다. 


### openstack 전체 흐름 정리
---
정리하자면 openstack을 하려면 neutron을 쓰기 위해서 network 기술도 알아야 하고, nova를 쓰기 위해서 가상화 하는 방법도 알아야 한다. 이렇게 openstack은 뭔가 새로운 기술을 만들었다기 보단 기존에 있는 기술들을 모아서 control 할 수 있는 집합체 느낌이다. 


openstack이 인스턴스를 생성하는 흐름 정리 
![[Pasted image 20250118043828.png]]

Nova API로 VM 생성 요청 (인스턴스 생성 요청)이 오면 Nova Scheduler를 호출해서 Hypervisor 중에 리소스가 넉넉하고 여유로운 놈을 골라서 정보를 전달해준다. 그러면 Nova API가 해당 Hypervisor의 Nova Compute agent 에게 VM 생성 요청을 한다. Nova Compute agent 비동기로 Neutron Server, Cinder API에게 각각 생성할 VM을 위한 네트워크, 스토리지 작업을 맡기고 VM을 생성한다. 그리고 완료되면 VM 생성 정보를 Nova API에게 반환하는 방식으로 동작한다. 

openstack의 모든 컴포넌트들 (nova, neutron 등)은 외부로 요청을 받는 ==API 서버==와 자원을 다루는 ==Agent 서버==로 이루어져 있다. 
이때 API 서버가 요청을 받으면 Agent 서버에게 명령을 해야 하는데 이때 어떤 방식을 사용하냐면 메시지 큐를 사용하는 RCP 방식(Remote Procedure Call)으로 진행한다. openstack의 모든 API는 비동기 방식으로 처리 된다. 자원을 요청하면 자원의 UUID와 함께 202 Accepted 를 받을 뿐, 실제로 작업이 완료되었는지는 알 수 없다. 그래서 자원을 요청한 쪽은 자원 생성이 완료되었는지 주기적으로 UUID 를 인자로 보내면서 해당 자원의 상태를 확인한다. timeout이 지나면 요청 실패 처리한다. 


### openstack 실습 
---
openstack을 실습해보려고 하는데 실제 물리 디바이스를 다 구축하기에는 돈이 많이 든다. 그래서 쉽게 실습할 수 있는 환경을 제공해주는데 그게 바로 DevStack이라고 한다. 
결론은 openstack을 실습해보기 위해서 VirtualBox VM 4 core 8GB 만 띄우면 되는데 이것도 DevStack을 쓰면 단 3줄만에 해결할 수 있다. 

```shell
git clone https://opendev.org/openstack/devstack
cd devstack
./stack.sh
```

