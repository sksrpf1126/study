# **_HTTP VERSION_**

- ## **_HTTP란?_**
  - **_H_** yper **_T_** ext **_T_** ransfer **_P_** rotocol(하이퍼텍스트 전송 프로토콜)로, 초기에는 HTML문서만을 전송하기 위한 프로토콜 이었지만, 현재는 다른 문서뿐만 아니라 인터넷에서 통신을 하기 위한 수단으로 확장되서 사용되고 있다.

</br>

---

- ## **_HTTP 특징_**

  - **_Client - Server 구조_**  
     클라이언트는 서비스(리소스)를 사용하는 컴퓨터, 서버는 서비스(리소스)를 제공하는 컴퓨터  
    </br>
  - **_비 연결성_**  
    HTTP는 TCP/IP 연결을 우선적으로 맺고 나서 통신을 하는데, HTTP 0.9 ~ 1.0 까지는 하나의 커넥션에 하나의 요청과 응답만을 처리하고 나서 연결을 끊는다.  
    그렇기에 새로운 요청마다 계속 커넥션을 해주어야 했음(과거)  
    연결을 유지하는데에 리소스가 발생하는데, 대량의 트래픽이 발생했을 때에는 그만큼의 큰 비용이 발생
    HTTP 1.1 이상부터는 개선을 통하여 극복함  
    </br>
  - **_무상태성 (stateless)_**  
    HTTP통신에서 서버는 클라이언트의 상태를 기억하지 않는다.  
    그 이유는 서버의 확장성을 고려하였기 때문

  </br>
  <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/185851027-f2757d09-d5fa-4b82-bd9d-847bd7b7eac1.png" height = 200px></p>

  서버가 클라이언트의 상태를 기억을 해야했다면 위와 같이 처음의 클라이언트 요청에 대하여 다른 서버들도 해당 클라이언트에 대한 상태를 전부 기억해야 클라이언트가 이후 요청에 대해 다른 서버로 접근했을 때에도 문제가 발생하지 않음  
  하지만, 이러한 상태 값이 커지면 커질수록 서버에 많은 부담이 가게되기 때문에 서버가 기억하는 것이 아닌 클라이언트 측에서 이전의 상태 값을 저장해 놓았다가 다음 요청에서 같이 보내는 방식으로 해결

</br>

---

- ## **_HTTP 버전에 대해_**

  - **_HTTP/0.9_**  
    HTTP 초기 버전에는 버전 번호가 없었으며, HTTP/0.9는 이후 차후 버전과 구별하기 위해 0.9라는 버전을 붙임  
    HTTP/0.9는 HTTP Method 가 GET만 존재했으며, 응답으로는 HTML 파일만을 전송해주는 정도여서 **_"원라인 프로토콜"_** 이라 하기도 한다.

    ```
    <요청>
    GET /myHomePage.html

    <응답>
    <HTML> myHomePage HTML </HTML>
    ```

    ***

  </br>

  - **_HTTP/1.0_**  
    HTTP/1.0 부터는 헤더가 생겼으며, 헤더 안의 content-type에 따른 HTML 이외의 다른 문서들이 전송이 가능해졌으며, 응답에 상태 코드도 생겨서 실패와 성공에 대한 구분이 확실해 졌음  
    </br>

    ```
    # 요청 메세지
    GET /myHomePage.html HTTP/1.0                   # start-line
    User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)       # 요청 해더


    # 응답 메세지
    200 OK --> 상태 코드
    Date: Tue, 15 Nov 1994 08:12:31 GMT     # 일반 헤더
    Server: CERN/3.0 libwww/2.17        # 응답 헤더
    Content-Type: text/html         # 앤터티/개체 해더(보통 응답 헤어에 포함)

    <HTML>              # 응답된 데이터
    myHomePage with an image
      <IMG SRC="/myimage.gif">
    </HTML>
    ```

    <span style="color : skyblue">HTTP/0.9 ~ HTTP/1.0 TCP/IP 연결에 대한 커넥션 하나당 하나의 요청과 응답만을 처리</span>

    1. 요청마다 새로운 커넥션을 맺어줘야 하다보니 성능 저하
    2. 서버 부하 비용 증가

    ***

    </br>

  - **_HTTP/1.1_**  
     위 버전들의 커넥션 문제를 1.1로 넘어오면서 해결  
     -> Persistent Connection 개념을 도입하였는데, 이는 지정한 timeout 동안은 커넥션을 닫지 않는 것으로 지속적인 통신이 가능  
     </br>
    파이프라이닝(Pipelining) 기법 추가
    </br>
    <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/185858847-7bfffa79-fdba-4cfe-a9b9-547358f98375.png" height = 350px></p>

    맨 왼쪽은 1.1 이전의 버전으로 요청과 응답 하나당 새로운 커넥션을 맺는다.  
     가운데는 Persistent Connection 으로, HTTP/1.1에서 등장한 개념이며, 그림에서는 하나의 요청에 대해 응답을 받고, 응답을 받은 후에 새로운 요청을 하고... 이러한 식으로 동작하다가 최종적으로 연결을 끊는 것이다.  
     하지만 이러면 1번의 요청에 대해 응답이 오기 전까지는 이후의 요청을 할 수 없다는 것이다.  
     이러한 문제로 지연시간이 늘어나게 되는데 이를 해결한 기법이 맨 오른쪽의 파이프라이닝 기법이다.  
     파이프라이닝 기법은 응답을 기다리지 않고, 연속적으로 요청을 보내며, 요청을 보낸 순서대로 응답을 기다린다.  
     하지만, 파이프라이닝 기법 또한 문제가 있는데, 요청은 연속적으로 하지만 응답은 요청한 순서대로 받아야 하기 때문에 처음에 보낸 요청에 대한 처리가 늦어지면 이후의 응답들은 대기를 해야하는 Head Of Line Blocking (HOL Blocking)의 문제가 발생한다.  
     </br>

    HTTP/1.1에서는 HOL Blocking 외에 **_"헤더 중복"_** 이라는 문제가 존재한다.  
     </br>
      <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/185858862-7ed95c38-bc2b-493b-a957-b176f5cbd693.png" height = 200px></p>

    헤더에는 많은 정보가 담겨 있는데, 위와 같이 요청에 따른 헤더 구조가 중복되는 부분이 많아도 그대로 전송하게 된다.  
     그러다 보니 불필요하게 데이터가 커지게 되는 것이다.

    ***

    </br>

  - **_HTTP/2.0_**  
    HTTP/1.1 버전을 15(?)년 정도 사용이 되어오다가 2015년 HTTP/2.0이 등장하게 되었다.  
    </br>
    <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/185869389-03b12678-41f2-4e94-9456-de4860e5dcec.png" height = 200px></p>

    HTTP/2.0은 응용계층 내부에 바이너리 프레이밍 계층이 생겼다.  
    이전 버전들은 데이터 전송 단위가 message로, message는 헤더와 데이터가 같이 묶여있는 구조이며, 하나의 요청 또는 하나의 응답의 단위로 사용이 되었다.  
    하지만 HTTP/2.0 에서는 헤더와 데이터 두 부분을 Frame 단위로 나누었으며, 이를 바이너리(이진)로 인코딩을 하여, Stream이라는 단위 내에서 송수신을 한다. 바이너리로 인코딩을 하기 때문에 파싱과 전송 속도가 더욱 빨라지게 되었다.

    ```
    Stream(스트림) : TCP 커넥션 내에서 전달되는 바이트의 "양방향" 흐름이며, 하나 이상의 메시지를 전달하는데 사용

    Message(메시지) : 하나의 요청 또는 응답의 HTTP 메시지이며, 다수의 프레임으로 구성

    Frame(프레임) : HTTP/2.0의 통신 최소 단위, 메시지 내의 최소 단위는 하나의 HEADER Frame 또는 DATA Frame
    ```

    </br>
      <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/185871381-fe22dbca-f037-417a-8422-41794014420d.png" height = 300px></p>

    하나의 TCP 커넥션에 스트림으로 양방향 통신하며, 스트림의 수는 제한이 없음 (클라이언트, 서버 양쪽 다 서로의 의사 없이 스트림을 만들 수 있음)

    위의 그림에서 Stream1에서는 요청으로 하나의 메시지에 하나의 HEADER Frame이 담겨서 서버 측으로 보내주며, 응답으로는 하나의 메시지에 HEADER Frame, DATA Frame이 담겨서 전송이 된다.

    Stream N은 메시지 여러개의 송수신 과정을 간략히 보여준다.

    위와 같이 스트림은 병렬적으로 메시지(데이터)를 송수신이 가능하며, 요청 순서에 상관없이 처리한 순서대로 전달이 가능하기 때문에 HTTP/1.1에서의 응답 순서에 따른 HOL Blocking 문제를 해결하였으며, 이와 같이 하나의 TCP 연결상에서 클라이언트의 다수의 요청과 응답을 비동기 방식으로 처리하는 기술을 **_멀티플렉싱(MultiPlexing)_** 이라 한다.

    하지만 응답의 순서를 지정해야 하는 경우가 있다. 예를 들어 리소스 A와 리소스 B가 있는데 리소스 A가 리소스B에 의존적인 경우 리소스A를 먼저 받더라도 리소스B가 없기에 활용할 수 없게되는 문제가 발생한다.  
    이를 해결할 수단으로 HTTP/2.0은 스트림간에 가중치를 두어서 우선순위를 둘 수 있으며, 또한 스트림간에 종속관계를 설정할 수 있어 이를 통해 순서 제어가 가능하다.

    ```
    https://ibocon.tistory.com/257
    -> 스트림에 가중치와 종속관계를 맺어준다 하여도 그 순서를 최종적으로 결정하는건 서버한테 있다.
    그 이유는 해당 요청들에 대해서 처리하는데 어느정도의 시간이 소요되는지는 서버측에서 알 수 있으며, 서버는 우선순위가 높다하여도 해당 요청에 처리하는데에 시간이 오래걸리면 이후의 우선순위라도 미리 응답을 보내놓는게 성능상으로 더 유리하기 때문이다.
    ```

    ### **_Sever Push_**

    </br>

      <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/185876039-fa8600c2-bb6f-4c22-8d13-2dd1ef251d3f.png" height = 200px></p>

    단일 요청에 대해 복수개의 응답 또는 요청하지 않아도 서버에서 응답을 보내주는 방식이다.  
    이전 버전에서는 하나의 요청이 먼저 들어와야 하나의 응답만을 보내주었지만, HTTP/2.0 으로 오면서 방식이 바뀌었다.  
    위와 같이 클라이언트에서 HTML만을 요청했을 때 서버는 그에 필요한 JS와 CSS파일을 같이 보내주는 것이다.

    ### **_헤더 압축(Header Compression)_**

    </br>

      <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/185877585-b4443e9f-fd91-4ddb-9ac1-ef47808b3c52.png" height = 300px></p>

    중복되는 헤더의 크기를 줄여서 로딩 시간을 줄이는 기술로써, HTTP/1.1에서의 문제를 해결  
    HPACK 알고리즘을 통해 압축하는 방식이며, 각각의 Header Field들을 Static Huffman Code로 encoding 하여 압축을 한다.  
    클라이언트와 서버가 서로 주고받은 헤더 필드에 대한 인덱스를 생성하고 유지를 한다. 이 인덱스를 이용해서 이미 보낸 헤더라면 요청을 보낼 때 중복은 배제하고 갱신된 헤더들만 담는다.  
    HPACK은 static Table, dynamic Table로 이루어져 있으며, static Table은 HTTP/2.0 스펙에서 정의하고 있는 헤더들을 관리하고, dynamic Table은 사용자에 의해 정의된 헤더들을 저장하고 관리한다.

    ### **_TCP 에서의 HOL Blocking_**

    TCP는 패킷(데이터)을 전송할 때, 문제가 생겼을 시에는 해당 패킷을 재전송을 하고, 후속 패킷은 재전송이 이루어질 때까지 대기를 하는데, 이러한 부분이 TCP의 HOL Blocking이다.  
    HTTP/2.0이 스트림을 통해 양방향으로 순서와 상관없이 데이터를 주고받지만, 결국 해당 데이터들은 TCP상에서 패킷으로 변환되어 TCP의 방식대로 데이터를 주기 때문.
    HTTP/2.0을 통해 HTTP 프로토콜 내에서의 HOL Blocking의 문제점을 극복하였지만, TCP의 HOL Blocking은 해결하지 못했음.

  </br>

  ***

  - ## **_QUIC_**

    구글은 TCP에서의 HOL Blocking에서의 문제점과 TCP의 연결을 맺고 끊어야 하는 경우마다 3-way handshaking, 4-way handshaking 과정을 거치는 것에 대한 지연시간을 QUIC 프로토콜을 통해 해결하였다.

    QUIC 프로토콜은 UDP 기반으로 만들어진 프로토콜로, UDP 자체는 연결을 맺고 끊는 동작도 없을 뿐더러, 데이터를 전달해주기만 할 뿐 그에 대한 제어는 따로 하지 않는다. 복잡한 일을 하지 않기에 UDP 프로토콜은 따로 커스텀하기에 적합하였고, 이를 통해 구글이 만든 프로토콜이 QUIC 이다.

    기존의 TCP와는 달리 QUIC는 처음 연결에서 연결에 필요한 정보들을 데이터와 함께 넘겨주고, 이후 새로운 연결이 필요할 떄는 Connection UUID라는 고유한 식별자를 통해서 커넥션을 다시 맺을 필요 없이 바로 데이터 송수신이 가능하다.

    QUIC는 TCP와 달리 문제가 생긴 패킷이 있을경우, 해당 패킷만 중단하 뿐 나머지 패킷들은 그대로 전송이 이루어진다. 그렇기에 TCP의 HOL Blocking의 문제를 해결하였음

    ### **_HTTP/2.0은 TCP를 이용하지만 HTTP/3.0에서는 구글이 만든 QUIC 프로토콜에서 동작이 되며, 구글은 실제로 사용중이다!_**

</br>

---

# **_참고_**

- https://kotlinworld.com/97 [HTTP 특징]
- https://bentist.tistory.com/36 [HTTP 버전별 특징과 QUIC]
- https://www.youtube.com/watch?v=xcrjamphIp4 [우아한테크코스 HTTP 영상]
- https://velog.io/@dnr6054/HOL-Blocking [Head Of Line Blocking]
- https://ibocon.tistory.com/257 [HTTP 2.0 자세히 알아보기]
- https://shortstories.gitbooks.io/studybook/content/http2.html [HTTP2.0/ 헤더 압축 내용
