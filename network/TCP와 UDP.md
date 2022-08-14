# TCP와 UDP

- ## TCP (Transmission Control Protocol)

  - ### TCP란?

    - 전송을 제어하는 프로토콜 (인터넷상에서 데이터를 메세지의 형태로 보내기 위해 IP와 함께 사용하는 프로토콜)
    - IP가 라우팅(목적지IP에 도달하기 위한 행위)을 하기위해 사용이 된다면, TCP는 데이터(패킷)을 추적 및 관리를 함

    </br>

  - ### TCP 특징

    - 연결형 서비스로 가상 회선 방식을 제공
    - 데이터의 전송 순서 보장 (TCP 헤더의 Sequence Number를 통해)
    - 데이터에 대한 신뢰성을 보장 (3-way handshaking, 4-way handshaking 방식)
    - 데이터 흐름 제어(수신자 버퍼 오버플로우 방지) 및 혼잡 제어(패킷 수가 과도하게 증가하는 현상 방지)
    - 1 : 1 통신만 가능하며, UDP 보다 속도가 느림

    </br>

  - ### 3-way handshaking, 4-way handshaking

    - TCP 연결을 맺고, 끊는 과정을 의미
    - TCP는 이 과정을 통하여 송신지와 수신지의 연결을 맺고나서, 데이터 송수신이 이루어지기 때문에 신뢰성 보장과 연결지향적인 특징을 지님

    </br>

    <p align ="center"><img src="https://user-images.githubusercontent.com/62879192/184525851-bc360674-bb51-419c-83a8-0cc611321196.png"></p>

    - 위 이미지는 3-way handshaking(TCP 연결을 맺는 과정)이며, HOST P를 클라이언트, HOST Q를 서버라고 가정  
      </br>

    1. 클라이언트는 서버어게 연결을 맺고 싶다는 의사표시로 SYN이라는 난수 값을 전송 (client -> server로 SYN 값 전송)
    2. 서버가 SYN값을 받음에 따라 클라이언트의 요청을 잘 받았다는 의미로 새로운 난수 SYN값과, 요청에 수락한다는 의미로 ACK(받은 SYN + 1)값을 전달 (server -> client로 SYN + ACK 값 전송)
    3. 클라이언트는 연결 요청에 대한 서버의 응답을 확인했다는 표시로 ACK(서버한테 받은 SYN + 1)값을 다시 전달 (client -> server로 ACK값 전송)
    4. 서버는 최종적으로 client에게 ACK값을 받고나서, 연결을 맺음  
       </br>

    ```
    2-way handshaking은 안되는 이유
    -> TCP는 양방향 연결이기 떄문에 양쪽에서 각각 자신이 보낸 요청한 값에 대한 응답이 와야한다. 
    그래서 처음에 클라이언트가 SYN값을 "요청" 하고 이후 서버가 SYN값에 대한 응답으로 ACK값으로 "응답"을 하고 동시에 서버 측 또한 SYN 값을 만들어 client 측에 "요청"을 한다.
    여기까지가 2-way이며, 아직 서버는 자신이 보낸 "요청"에 대한 응답을 받지 못한 상황이므로 양방향 연결이 되었다고 볼 수 없는 것이다. 
    그래서 마지막에 클라이언트가 서버로 ACK값을 전달해 줌으로써 최종적으로 양방향 연결이 이루어 지는 것
    ```

      <p align ="center"><img src="https://user-images.githubusercontent.com/62879192/184527187-f5ac4ae1-7772-4a63-afdf-b44df248252f.png"></p>

    - 위 이미지는 연결 이후, 모든 통신이 끊났을 경우 TCP연결을 끊는 과정  
      </br>

    1. 클라이언트가 서버로 통신을 끊기위한 요청으로 FIN 값을 보냄 (client -> sever로 FIN 전송)
    2. 서버는 클라이언트의 FIN값을 받음을 통해 연결 종료 요청에 대한 응답으로 ACK값 전송 (server -> client로 ACK 전송)
    3. 이후, 서버는 연결 종료를 준비하며, 다시 클라이언트로 종료를 하겠다는 의사 표시로 FIN 값을 전송 (server -> client로 FIN 전송)
    4. 클라이언트는 서버에서 온 FIN값을 통해 서버의 종료 의사표시를에 대한 응답으로 ACK값을 보냄 (client -> sever로 ACK 전송)
    5. 서버는 클라이언트에서 온 ACK값을 통해 TCP 연결을 끊음

    </br>

  - ### 흐름제어와 혼잡제어란?

    - 데이터의 수신측이 송신측보다 데이터 처리하는 속도가 빠르다면, 데이터 송신하는 속도를 제어할 필요가 없지만, 이 반대의 경우에는 문제가 생긴다. 그래서 이러한 속도를 제어하는 것이 **"흐름제어"** 이며, 제어하는 방법으로는 Stop and Wait(전송한 패킷에 대한 확인 응답을 받아야만 다음 패킷 전송)와 Sliding Window(송신 측 window size와 수신 측 window size를 맞추어 window size만큼 전송을 하고 응답을 받은 size만큼 window size를 옆으로 옮겨서 다시 전송하는 방식)이 존재

    - 송신측의 데이터는 라우팅을 통하여 인터넷 네트워크를 돌아다니다가 최종적으로 목적지IP로 도착하게 된다. 하지만 인터넷 네트워크에서 하나의 급격하게 데이터가 몰릴 수 있고, 해당 라우터가 모든 데이터를 처리하기에는 무리가 생김 결국 속도에 문제가 생겨 송신측은 해당 데이터를 재전송을 하게되고, 결국 혼잡해지며, 오버플로우나 데이터 손실이 발생하는데 이를 해결하기 위해 송신측에서 보내는 데이터의 전송속도를 제어함에 따라 해결을 하고 이를 **"혼잡제어"** 라 한다.

    </br>

- ## UDP (User Datagram Protocol)

  - ### UDP란?

    - 데이터를 데이터그램 단위로 처리하는 프로토콜
    - 데이터그램이란 독립적인 관계를 지니는 패킷을 의미하며, TCP와 달리 UDP는 연결을 맺지 않기 떄문에 각 패킷은 다른 경로로 전송이 됨

    </br>

  - ### UDP 특징

    - 비연결형 서비스로 데이터그램 방식을 제공
    - 연결을 맺지 않기 때문에 신뢰성이 낮음
    - sequence number와 같은 전송 순서를 보장하는 방법이 없기 때문에, 순서가 바뀔 수 있음
    - 흐름제어, 혼잡제어, 연결 맺는 과정 등이 없기 때문에 TCP보다 속도가 빠름
    - 1 : 1, 1 : n, n : n 통신 가능

    </br>

- # TCP와 UDP 비교

    <p align ="center"><img src="https://user-images.githubusercontent.com/62879192/184528000-b677dabf-6168-46a8-ace5-212e7567ed11.png"></p>

    </br>

---

# 참고

- https://github.com/gyoogle/tech-interview-for-developer/blob/master/Computer%20Science/Network/TCP%20(%ED%9D%90%EB%A6%84%EC%A0%9C%EC%96%B4%ED%98%BC%EC%9E%A1%EC%A0%9C%EC%96%B4).md#tcp-%ED%9D%90%EB%A6%84%EC%A0%9C%EC%96%B4%ED%98%BC%EC%9E%A1%EC%A0%9C%EC%96%B4 [TCP/흐름제어/혼잡제어]

- https://mangkyu.tistory.com/15 [TCP와 UDP -망나니개발자]

- https://github.com/gyoogle/tech-interview-for-developer/blob/master/Computer%20Science/Network/TCP%203%20way%20handshake%20%26%204%20way%20handshake.md [3-way handshaking, 4-way handshaking]
