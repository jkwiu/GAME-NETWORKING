# GAME-NETWORKING

[원본 강의 동영상](https://www.youtube.com/watch?v=lAhAdnsIN6I&list=PLy-g2fnSzUTDsS7kCzmFYn4BJK6nCs0_r&ab_channel=Tucker%EC%9D%98GoLang%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)

1. 네트워킹이란?
   1. 데이터를 주고 받는 것, 컴퓨터안의 cpu, ram 간 데이터를 주고 받는 것도 네트워킹이다.
   2. 컴퓨터 내부와 외부 네트워킹의 차이점
      1. 지연시간(latency)
         1. 지구 한바퀴를 빛이 도는 시간(133ms = 0.13sec)
            1. 굉장히 빠른 것같지만, 컴퓨터 입장에서는 굉장히 느리다.
         2. **게임에서 얼마나 ``Latency``를 눈속임하느냐가 관건이다.**
            1. 아무리 빨라도 물리적인 한계 때문에 속도를 줄이는 것은 안된다.
      2. 연결의 불안정성
         1. 인터넷은 여러 거점을 거쳐가는데 이 경로 중 하나가 문제가 생기면 연결이 끊어진다.
            1. 연결이 취약하므로 언제든지 끊김이 발생할 수 있다.
            2. 근래는 모바일 무선네트워크를 많이 쓰므로 더 빈도가 높다.
            3. **이러한 끊김에 대한 안정성을 커버하는 것이 관건이다.** 
      3. 순서 비보장
         1. 내부
            1. 순서대로
         2. 외부
            1. 랜덤
            2. 모든 path를 살포하여 가장 빠른 쪽을 캐쉬에 기억했다가 전송한다.
         3.  **패킷 순서를 보장하는 것이 관건이다.**
2.  Protocol TCP/UDP
    1.  TCP
        1.  Hardware에서 다음 3가지를 보장해주자.
            1.  연결은 추상적 개념
            2.  연결 유무확인
                1.  ``3 way handshake``
                    1.  req
                    2.  res
                    3.  res에 대한 res
                2.  Connect / Disconnect 를 알 수 있게 프로그래밍
            3.  순서보장
            4.  송신보장
        2.  단점
            1. 지연시간이 느리다(3번 주고 받아야 해서 느림)
        3. 느려도 되는 게임
           1. MMORPG, 보드게임
    2.  UDP
        1. 연결 유무x
        2. 순서 보장x 
        3. 송신 보장x
        4. 장점
           1. 지연시간이 빠르다(1번 줘서 빠름)
        5. 단점
           1. 데이터가 유실될 수 있다.
        6. 빠른 반응 속도 필요
           1. 격투게임, 슈팅게임, LOL, 오버워치, 스타크래프트
    3. Reliable UDP
       1. 빠른 UDP를 쓰기 위해 프로그래밍된 **software**
       2. UDP에서의 순서보장, 송신 보장을 해주기 위해 만든 **software**
       3. 만들기 쉽지 않다.
          1. 과거에는 블리자드 같은 대형회사에서 자체적으로 만들었으나 요즘에는Unity, Unreal에서 제공해주고 Open Source도 많아졌다.
3. Deterministic 방식
   1. 같은 ``input``이면 같은 ``result``가 나온다는 가정하에 시작
      1. delay
      2. rollback
      3. relay server
   2. 데이터를 다루는 방식(TCP/UDP는 전송 방식)
   3. 같은 입력을 같은 시간에 어떻게 넣느냐가 관건
   4. Deterministic
      1. UDP
      2. a와 b의 input에 대한 output을 지연시킴으로써 a와 b의 output시간을 맞춰주는 개념
      3. ``desync``상황을 잡기 힘들다.
         1. 언제 어디서 발생할지도 예측불가
   5. Server Authority
      1. TCP
4. Delay와 Rollback
   1. ``latency``만큼 ``delay``를 줘서 sync를 맞춰주는 방법(보통 ``100ms`` 줌)
   2. 단점
      1. delay가 생김(랙)
   3. 도달시간이 150ms라치면 100ms의 delay를 줘도 문제가 생긴다. 이럴 때 ``Rollback``
   4. Rollback
      1. 지연된 시간만큼 Rollback하여 다시 앞감기를 하여 sync를 맞춘다.
      2. 단점
         1. 프레임이 튄다.(플레이어가 갑자기 순간이동)
         2. 구현이 매우 어렵다.
            1. 뒤감기 -> 앞감기
               1. 10명일 때 모든 data를 기억하고 있다가 다시 넣어줘야함
            2. 입력을 중간에 집어넣기
            3. 구현이 어렵기 때문에 ``input``이 오는 것과 상관없이 매 프레임 롤백하는 방식을 사용함.
      3. ``fps``에서 많이 씀.
   5. 그렇기 때문에 Delay와 Rollback을 섞어 쓴다.
      1. Ex)
         1. 오버워치가 잘 Delay & Rollback이 잘 구현되어 있다.
            1. 트레이서의 ``되돌리기`` 스킬
         2. LoL Replay
            1. 시작부터 끝까지 ``state``의 값이 아닌 ``input``의 값을 모두 기록해야 한다.
5. 다른 유용한 방식들
   1. 중계서버(Relay Server / Broadcast Server)
      1. 모든 클라이언트는 똑같은 ``input``을 받는다.
      2. 타임슬롯에 해당 ``input``들을 넣는다.(``turn제`와 같다)
         1. 각 ``player``는 서버에 자신의 행동을 보낸다.
         2. ``server``가 취합한다.
      3. 장점
         1. 구현이 쉽고 ``desync``도 잘 일어나지 않는다.
      4. 단점
         1. 반응성이 떨어진다. 
      5. ``AOS``, ``스포츠``에서 많이 씀
6. P2P, Relay Server 방식
   1. P2P
      1. Peer To Peer
      2. 방식
         1. 방장을 하나 정해서 방장에게 ``connect``하는 방식
            1. 장점
               1. 각 ``client``가 관리해야하는 ``connection``수가 적다.
               2. 방장이 심판의 역할을 한다.
            2. 단점
               1. 방장이 겜을 나가버린경우 방이 폭파된다.
                  1. 나갔을 경우 새로 방장을 뽑는 방법도 있다.(복잡함)
         2. Fully Connection 방식
            1. 서로가 연결
            2. 장점
               1. 중간에 한명이 나가도 게임이 진행된다.
            3. 단점
               1. 관리해야하는 ``connection``수가 많다.
   2. Relay
      1. ``client``가 ``server``에 접속하는 방식
      2. 장점
         1. 연결이 안정적이다.
         2. 공정한 플레이가 가능
      3. 단점
         1. 서버가 죽었을 때, 게임자체가 아예 플레이가 안된다.
   3. P2P vs Relay
      1. P2P에 아주 큰 문제점이 있기 때문에 Relay를 사용하고 있다.
         1. Internet이 만들어질 때, 소련과 미국이 냉전 중에 있을 때, 두 국가 다 핵무기를 많이 보유하고 있었다. 관건은 핵미사일 발사를 감지하고 최대한 빠르게 대응하여 발사하는 것이다.
         2. 요청을 전송했을 때 선이 하나가 아닌 여러 경로에서 최단경로를 찾아서 패킷이 전송되는 시스템을 원했다. (선이 하나면 끊겼을 때 발사를 요청할 수 없다. 애초에 전세계에 인터넷이 생길 것을 가정하고 만든 것이 아니다.)
         3. IPv4와 IPv6
            1. IPv4로 갯수가 모자라서 IPv6을 만들었으나, IPv4로도 많은 주소를 할당할 수 있는 기술이 나와서 IPv6는 잘 안쓰게 됨.
               1. 고정IP(Public IP)가 받아서 유동IP(Private IP)로 할당해준다.
         4. 홀 펀칭(hole punching)
7. 홀 펀칭
   1. 개인pc에서 naver에 req를 날렸을 때, naver는 개인pc의 ip를 알 수 없다. NAT가 받으면 나갈 때 할당되었던 개인pc의 ip를 기억해뒀다가 바인딩해준다.
   2. NAT
      1. ``port forwarding``을 해서 ``binding``해주는 것
   3. Web RTC Protocol
      1. P2P에서 c1과 c2가 server에 접속했을 때 서로에게 바인딩 된 포트 번호를 알려준다. 이 때 서버를 ``STUN SERVER``라고 한다. 알려주면 c1에서 c2, 혹은 c2에서 c1으로 알려준 ip로 날려보는데(홀펀칭을 시도), Gateway가 이 패킷을 통과시켜줄지 말지는 지 맘대로다.
      2. STUN Server로 홀 펀칭을 시도해보고 안되면 turn server로 돌린다.
8. Deterministic과 해킹
   1. Deterministic은 input에 의존하기 때문에 해킹에 취약하다.
   2. 해킹
      1. 외부해킹
         1. 막는 방법
            1. 외부 tool 블랙아웃(``nProtect``: 검증되지 않은 툴 종료)
      2. 내부해킹
         1. 공격자가 유리하다(client가 인질로 잡혀있기 때문에)
         2. 막는 방법
            1. 파일 사이즈가 변경되었는지 확인
            2. Checksum
               1. 조금만 변경되어도 Hash가 많이 바뀜. 바뀌면 종료
            3. 코드 암호화  
9. Server Authority
   1. 질의, 응답, 알림
   2. 예전엔 질의 후 응답하면 바로 끊었으나, 요즘은 Long polling방식으로 일정시간 connection을 유지한다. 
   3. MMORPG에서 많이 사용
      1. 동접수가 많다.(10K 서버: 동시접속자 만명을 핸들링 할 수 있는 서버)
      2. lol은 한 session의 동접 10명
   4. 단점
      1. 서버가 모든 클라이언트를 관리해야 한다.(로직이 필요)
10. Socket Programming
    1. 예전에는 프로그래머들이 hardware에 직접 붙었다.(windows, mac 이전 시대)
       1. hardware별로 다르게 프로그래밍해야 했다.
          1. HAL
             1. OS에서 제공하는 추상화된 layer
             2. adapting 패턴
       2. hardware와 os의 연결을 프로그래밍하는 것이 Socket Programming
    2. Socket I/O 
       1. Read & Write  
          1. 동기식
             1. 응답이 올 때까지 thread가 멈추기 때문에 서버에서 잘 쓰지 않는다.
          2. 비동기식
             1. Select 방식(pre-request 방식)
                1. 일의 주체가 나
                2. 간단하게 구현 가능(connection수가 100개 이하일 때 좋다)
             2. IOCP 방식(post-request 방식)
                1. 일의 주체는 대리인
                2. 성능이 좋다.(connection수가 100개 이상일 때 좋다)
          3. 그런데 요즘은 언어자체에서 네트워크 기능을 잘 지원해주기 때문에 IOCP인지 select인지 알 필요 없다.
11. MMORPG 서버 구조1
    1. 유저가 많다. -> 작업량이 많다.(Actor -> Player, NPC, Control Zone)
    2. 조건
       1. 패킷 처리가 필요하다.(Read/Write)
       2. Tick 작업(매초 player의 이동 처리) 
       3. 그 외 시스템
    3. 그래서 멀티 쓰레드가 필수다. 코어 당 100ms, 4코어면 400ms를 확보
    4. 좋은 서버 구조로 만들어져있으면 multi threading하기 좋다.
    5. 서버 구조
       1. 싱글 스레드
          1. Network I/O -> Queue -> WorkThread
          2. Network I/O(요청 queue를 쌓고)
             1. Select
             2. IOCP
          3. WorkThread(요청을 처리하고 actor의 tick을 돌고 다음 처리)
          4. 장점
             1. 단순하다.
          5. 단점
             1. 시간을 확보하기 어렵다.
             2. 많은 Actor를 처리하기 어렵다
             3. 3000 ~ 4000명까지의 동접
          6. 과거에 MMORPG가 많이 사용했다.
       2. 싱글 스레드 + Dedicated Thread(상점일, 거래소 일, 결투장 등)
          1. 기존 싱글 스레드 방식에 특정한 일을 하는 스레드들을 추가
       3. 멀티 스레드
          1. Network I/O -> Queue -> WorkThread(MultiThread)
    6. 멀티 쓰레드 vs 멀티 프로세스
       1. 멀티 쓰레드
          1. 하나의 프로세스에서 쓰레드를 여러개 나눠서 각각을 따로 동작하는 것
          2. 같은 프로세스기 때문에 서로의 메모리를 공유하기 때문에 서로의 메모리를 망칠 수 있다.
       2. 멀티 프로세스
          1. 하나의 컴퓨터에 여러 프로세스를 띄우는 것
          2. 프로세스마다 독립된 메모리를 갖고 있어 서로 망치지 않는다.
          3. Process가 다르면 서로 상호작용할 수 없다.
12. MMORPG 서버구조2(싱글프로세스 vs 멀티프로세스)
    1. 와우에서 각 월드가 하나의 게임 서버로 돌아간다. -> 싱글 프로세스 
    2. 여러 서버들이 하나의 서버를 구성한다. -> 멀티 프로세스
    3. 싱글 프로세스
       1. Scale Up
          1. 하나의 Machine의 성능을 올린다.
          2. 비용이 성능에 따라 y=ax<sup>2^</sup>으로 증가(비용⬆)
       2. But 공짜 점심은 끝났다.
       3. Availability(가용성)⬇
          1. 서버가 죽으면 서비스가 종료
       4. 관리가 쉽다
    4. 멀티 프로세스
       1. Scale Out
          1. Machine을 늘린다.
          2. 비용이 성능에 따라 y=ax로 증가(비용⬇)
       2. Availability(가용성)⬆
          1. 서버가 죽어도 서비스가 종료되지 않는다.
       3. 관리가 어렵다
    5. 서버를 나누는 방법
       1. Zone 방식(와우에서 대륙과 대륙)
          1. 대륙 넘어갈 때 로딩
       2. 채널 방식 
          1. 대도시
             1. 채널
       3. Zone 방식 + 채널 방식
    6. 프로세스 관리가 중요하다.
13. MMORPG 서버구조3
    1. 멀티 쓰레드의 구획화
       1. Actor 기준(Akka: scala에서 제공하는 framework) 
          1. Actor별로 따로 스레드.
          2. Actor간의 통신은 Message를 통해서 
       2. System 기준(ECS구조: Entity Component System)
          1. Entity(Component를 갖고 있다.)
             1. 케릭터
          2. Component(상태)
             1. 상태를 갖고 있으나 기능을 갖고 있지 않다.(Class가 아니다)  
             2. Flying(data)
             3. Attack(data)
             4. Moving(data)
          3. System(기능)
             1. Flying(function)
          4. 장점
             1. 확장성이 좋다.
             2. Locallity가 좋다.(data가 모여있다.)