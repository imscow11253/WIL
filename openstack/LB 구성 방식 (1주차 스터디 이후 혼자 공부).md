25.1.14

보통 우리가 서버 배포 환경을 만들면 그냥 EC2 하나에 서버 띄워 놓고 EC2의 IP의 8080 port로 오는 모든 트래픽을 오게 만든다. 일반적인 웹 서비스는 이와 달리 서버를 여러 개 두고 LB(Load Balencer)를 두어서 트래픽이 분산되게 하는 것이 일반적이고, 이 LB도 죽을 수 있으니 2개를 두어서 서로 health check 하면서 VIP를 서로 가지도록 하는 형태가 일반적이다. 
보통 Server가 Client와의 연결을 유지하거나 기억하고 싶을 때 사용하는 방식 중에 Session이 있다. 근데 중간에 LB가 끼여 있다. 즉 LB가 끼여있는 형태에서 Session을 유지하고 싶으면 LB가 Client의 요청이 이전 것과 같은 Client인지 Client를 구분할 필요가 있고, 이를 packet의 Source IP, Source Port로 구분한다. 이 정보를 알기 위해서는 packet을 OSI의 Transport Layer까지는 뜯어봐야 하므로 LB는 L4 Switch 인 것이 일반적이다. 
(번외로 L7 은 HW가 아니라 SW로 구현된 형태가 많다. like nGinx)

시작하기 앞서서 LB가 Load Balancing 하는 방식에는 여러가지가 있다. 
1. Round Robin (번갈아가면서 순차적으로)
2. Least Connection (현재 Connection 수가 가장 적은 Server)
3. RTT (Round Trip Time) 응답이 오기까지의 시간이 가장 적은 Server
4. Priority (Server마다 우선순위가 존재)

LB가 Server의 health check 하는 방식은 3가지가 있다. 
1. ICMP (ping 보내서 pong 받기)
2. 


그럼 서버로 요청이 왔을 때 응답하는 방식에는 어떤 것들이 있을까? 크게 3가지로 나뉜다. 
1. Proxy (One Arm)
2. DSR
3. Inline (Two Arm)

## Proxy
Proxy 방식은 Client와 Server 중간에 LB가 껴있는 형태로, Client는 Server로 요청을 보낼 때 LB의 VIP(Virtual IP)로 요청을 보낸다. 원래는 도메인 주소로 요청을 보내고 DNS의 record에 등록된 LB의 VIP를 가져오는데 여기선 그냥 Client가 VIP를 알고 있고 바로 요청을 보낸다고 하자. 
Client가 VIP로 요청을 보내면 LB가 받는다. 그리고 LB는 packet의 source IP를 LB의 PIP(proxy IP) 로 바꾸고 Destination IP를 Server의 IP로 바꾼다.(이렇게 IP를 바꾸는 것을 NAT, Network Address Translation이라고 부른다.) Server에서는 응답을 보낼 때 packet의 Source IP와 Destination IP를 바꾸서 보내기 때문에 Server 응답의 목적지는 Client가 아니라 LB가 된다. LB는 이를 받아서 똑같이 Source IP와 Destination IP를 바꾸어서 Client에게 응답을 보낸다. 

장점 
- Client 입장에서는 자기가 요청을 보냈던 주소에서 그대로 요청이 돌아오니까 정상적으로 인식 (뒤에 DSR 방식은 Client가 요청을 보낸 주소와는 다른 곳에서 응답이 오기 때문에 이를 정상 응답으로 인식하기 위한 많은 방법이 사용됨)

단점
- LB의 부담이 커진다. 만약 동영상 스트리밍 같이 Server의 응답의 용량이 크다면, 모든 데이터가 LB를 거쳐야 하므로 LB의 부담이 커진다. 
- Server가 Client 의 IP를 알 수 없기 때문에 필요한 경우에는 사용 불가? 


## DSR
Direct Server Return의 약자로, Client로부터 Server로 응답이 들어올 때는 흐름이 같지만 Server가 Client에게 응답을 줄 때는 LB를 거치지 않고 바로 전달하는 방식이다. 근데 이럴 경우 Client가 요청을 보낸 주소는 VIP인데 응답을 받은 Source의 주소는 LB의 VIP가 아니라 Server의 실제 주소가 된다. 이때 Server에서 응답을 보낼 때 사용되는 것이 바로 LoopBack Interface이다. LoopBack Interface는 VIP 주소를 가지고 있어서 Server가 Client에게 직접적인 통신을 할 때는 Loopback interface에 있는 VIP를 참조하여 Src IP를 NAT하고 전달한다. 

이때도 LB가 NAT 작업을 하는데 이 부분에서 어느 정보를 수정할지 정도에 따라 L2DSR, L3DSR 등의 방식으로 나뉜다. 

### 1. L2DSR
Client에서 Server로 전송되는 packet (이걸 Inbound packet이라 하고, 반대는 Outbound packet이라 한다.) 의 Dst Mac Address를 Server의 Mac Address로 바꾸는 기법이다. 
다만 LB는 Inbound packet의 Mac Address 만 바꾸기 때문에 LB와 Server는 반드시 같은 Network에 속해 있어야 한다. 

### 2. L3DSR
LB가 Inbound packet의 Dst IP를 바꾸어서 Server로 전달하는 방식이다. 즉 L2DSR과 달리 LB와 Server가 같은 network에 없어도 된다. 
Server가 응답을 할 때 VIP를 Src IP로 하기 위해서 L3DSR은 DSCP field를 사용하거나 Packet을 tunneling 한다. --> 추후에 알아보자. 


## Inline
인라인 방식은 Proxy 방식과 같은 흐름은 같다. Proxy에서는 Server가 Client의 IP를 알 수 없다는 단점이 있었지만 인라인에서는 이걸 해결한다. 
LB에서 Inbound Packet을 NAT 할 때 Destination의 IP만 NAT해서 Server로 전달하기 때문에 Server는 Src IP에 있는 Client IP를 알 수 있다. 그러면 Server가 응답을 보낼 때는 Src IP와 Dst IP를 바꿔서 보내기 때문에 DSR이 되는 것 아닌가? 할 수 있다. 
인라인 방식에서 Server의 Default Gateway는 LB로 설정되어 있기 때문에 Outbound packet은 LB로 전달된다. 이때 LB는 다시 Outbound packet의 Src IP를 VIP로 NAT해서 응답을 한다. 


https://blog.naver.com/voice45/221322291590
