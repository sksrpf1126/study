# TCP-IP 4계층 데이터 흐름

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/183290379-f46d1f28-7f28-4d50-9a9c-28e00e21d087.png" width=50% height=250px>
<img src="https://user-images.githubusercontent.com/62879192/183293670-74506643-8890-4854-bf63-b391699de965.png" width=40%>
</p>

- 위 두 이미지는 TCP-IP 4계층에서 데이터 송수신이 이루어질 때의 흐름이며, 각 계층을 거치면서 데이터에 여러 헤더가 씌여지거나 벗겨지는지를 보여준다.
- 송신측에서의 데이터는 각각의 계층을 지나오며 헤더들이 데이터에 "캡슐화"가 되고, 수신측에서는 반대로 "역캡슐화"를 통해 각 계층에 맞는 헤더를 가져온다.

---

# 헤더 정보

- ## 응용 계층

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/184473252-22d7910b-8d95-4813-ab74-1d6b3d6e6909.png" width = 70%>
</p>

- 위 이미지는 www.naver.com의 url로 요청했을 때의 요청 헤더로 응용 계층에서는 데이터에 http 헤더가 추가된다.

</br>

- ## 전송 계층

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/184473511-bd79b6b9-707d-414a-8e23-618a19b45027.png" width = 80%>
</p>

- 위 이미지는 TCP헤더 구조로, 상위 계층(응용 게층) 데이터에 TCP 헤더를 붙인다.
- 데이터의 크기가 크면 MTU 단위로 데이터를 쪼개는데 이를 세그먼트라 한다.
- 네트워크 상황에 따라 목적지 측에 도착하는 데이터의 순서가 바뀔 수 있기 때문에 TCP헤더에는 Sequence Number를 통해 데이터 순서를 제어한다.
- TCP헤더에는 출발지 포트, 목적지 포트에 대한 정보도 담겨있으며, 이를 통해 어떠한 응용 프로세스와 연결해야 할지를 정한다.

</br>

- ## 인터넷 계층

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/184473777-04a37f24-e926-49a9-8539-7f9533c70e2e.png" width = 80%>
</p>

- 위 이미지는 IP헤더 구조로, 세그먼트에 IP헤더를 붙인다.
- IP헤더를 붙여서 만들어진 데이터를 "패킷"이라고 부른다.
- IP헤더에는 출발지 IP, 목적지 IP에 대한 정보가 들어있으며, 이를 통해 라우팅을 하여 목적지IP(목적지의 3계층 장비)까지 도달하는데 사용이 된다.

</br>

- ## 네트워크 인터페이스 계층

- 네트워크 인터페이스 계층에서는 "패킷"에 이더넷 헤더와 Tails(FCS) 이 2개가 붙는다.
- 이더넷 헤더에는 출발지 MAC주소와 목적지 MAC주소가 담겨 있으며, Tails에 있는 FCS는 데이터 전송 도중 에러 여부에 대해 판별할 때 사용된다.

</br>

---

# 참고

- https://icarus8050.tistory.com/103#recentComments [TCP/IP 패킷 전송 과정]
- https://junu0516.github.io/posts/tcp_ip_4%EA%B3%84%EC%B8%B5/ [TCP/IP 4계층의 이해]
