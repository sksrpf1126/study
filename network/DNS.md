# **_DNS_**

- ## **_DNS란_**

  - **_DNS(Domain Name System)_** 란 주소창에 Host Domain Name을 입력했을 때 해당 주소에 일치하는 IP를 찾아서 변환해 주는 시스템
  - "www.naver.com" 은 네이버의 Host Domain Name이며, 주소창에 해당 도메인 명을 입력하면 네이버 페이지가 나타나는데 이것은 실제로 DNS를 통해 도메인 명으로 네이버 IP를 가져와서 해당 IP로 네이버 페이지와 관련된 데이터를 요청한 결과이다.

      <p align = "center">
    <img src="https://user-images.githubusercontent.com/62879192/185735948-e7a96fb3-c17f-48dc-946d-519e9dc4d45f.png" height = 150px>
    </p>

  - 위 223.130.200.107 또는 223.130.195.200으로 주소창에 입력하면 네이버 페이지가 나타난다. 하지만 이렇게 IP로 접근하면 사용자 입장에서는 접근성이 떨어지고 IP가 변경이 된다면 문제가 생기게 된다. 하지만 DNS를 통하여 IP와 도메인 명을 매칭시켜주기만 한다면, IP가 변경 되어도 사용자는 해당 도메인 명으로만 접근하면 된다.

</br>

---

- ## **_Local DNS Server_**

  - 도메인 명을 찾을 때 가장 먼저 들리는 곳이 Local DNS Server이고, 보통 가정집에서 이미 해당 DNS 서버가 등록이 되어있다. (Local DNS Server보다 우선적으로 브라우저에 도메인 명이 캐싱되어 있는지를 찾는다)
  - ISP 즉, 인터넷 제공 업체 마다 자신들의 Local DNS Server를 가지고 있으며, 가정집에서 인터넷 제공 업체에 따라서 Local DNS Server가 등록이 된다.
  - 예를들어 www.google.com 에 대한 IP를 찾고자 한다면 Local DNS Server로 가서 해당 도메인에 대한 IP가 있는지 물어보고 있다면 해당 IP를 반환할 것이고 없다면, Local DNS Server는 Root DNS 서버한테 물어본다.

</br>

---

- ## **_Root DNS Server_**

  - ICANN(국제인터넷주소관리기구)이 관리하는 서버이며, TLD(최상위 도메인) DNS Server 들의 IP주소를 저장을 해두는 서버

  </br>

    <p align = "center">
    <img src="https://user-images.githubusercontent.com/62879192/185778873-c2984f8f-13a8-47c8-b14e-6e109d3eecda.png" height = 250px>

  - 도메인 명은 위와 같이 트리구조로 이루어져 있으며, 자식 노드들(.com, .net, .org 등등)을 top-level 도메인이라 한다. 그 자식은 Second-level, 그 자식은 Sub...

    모든 DNS 서버는 Root DNS Server 주소를 알고 있으며, 자신이 모르는 IP주소는 Root DNS Server에 요청을 하고, Root DNS Sever는 Top-level 도메인 명을 통해 자식의 TLD DNS Server의 주소를 반환해준다.  
    "www.naver.com", "www.google.com"과 같은 주소는 .com을 관리하는 TLD DNS Server의 IP주소를 반환

    ```
      전세계 Root DNS Server의 수는 13개였지만, 2020년 이후로 1,034개로 늘었다.
    ```

</br>

---

- ## **_TLD DNS Server_**

  - Authoritative DNS Server의 IP를 저장해두는 서버로, .com의 도메인들을 관리하는 TLD DNS Server의 경우 xxx.xxxxx.com 의 주소를 보고서, 어떤 Authoritative DNS Server의 IP를 줘야할지 판단하고 반환한다.

</br>

---

- ## **_Authoritative DNS Server_**
  - 실제 개인 도메인과 IP의 관계를 저장하는 서버로, 일반적으로 도메인/호스팅 업체의 DNS 서버를 의미하며, Root DNS Server, TLD DNS Server를 통해 최종적으로 도메인의 IP를 반환해주는 서버

</br>

---

- ## **_DNS 동작 과정_**

</br>
    <p align = "center">
    <img src="https://user-images.githubusercontent.com/62879192/185779449-57377fb7-6ac6-4129-9e0a-aff94754ca52.png">

**_1)_** beta.example.com 도메인명에 대한 IP주소를 알아오기 위하여, 위에는 없지만 우선 브라우저에 캐싱되어 있는지 확인을 한다.

**_2)_** 존재하지 않는다면, Local DNS Server로 가서 해당 도메인에 대한 IP주소가 캐싱되어 있는지 확인하고, 있으면 반환을 해주고 없으면 Root DNS Server로 이동

**_3)_** Root DNS Server 는 beta.example.com 의 Top-level 도메인인 .com을 통해 .com의 도메인을 관리하는 최상위 도메인 서버(TLD DNS Server)의 IP를 반환

**_4)_** Local DNS Server 는 다시 TLD DNS Server로 가서 요청을 하며, TLD DNS Server는 example.com의 도메인을 관리하는 Authoritative DNS Server의 IP 주소를 반환 (바로 beta.example.com의 도메인을 관리하는 Authoritative DNS Server의 IP 주소를 반환할 수 있음)

**_5)_** Local DNS Server 는 다시 반환받은 Authoritative DNS Server의 IP주소를 통해 해당 서버로 요청을 보내고, Authoritative DNS Server는 최종적으로 beta.example.com의 IP주소를 반환

**_6)_** Local DNS Server는 반환받은 IP를 캐싱해두고, 요청한 브라우저에게 IP주소를 반환해 준다.

</br>

---

- ## **_DNS는 보통 UDP를 사용_**

  - DNS Server에 요청을 할 때 53번 포트만을 사용하며, UDP 프로토콜을 사용한다. 단, 512Byte 이상의 패킷을 전송해야 하는 경우에는 TCP 프로토콜을 사용

  - UDP를 사용하는 이유는 DNS는 그저 도메인에 따른 IP만을 반환해주는 역할이므로, 데이터의 신뢰성보다는 신속성을 중요시 한다.(제대로 받지 못했으면 다시 전송하면 될 뿐)  
    그렇기에 연결을 맺고 끊고를 반복(3-way handshaking, 4-way handshaking)하며, 데이터를 주고받는 TCP보다는 그러한 과정 없이 빠르게 데이터를 주고받을 수 있는 UDP 프로토콜을 사용하는 것

</br>

---

# **_참고_**

https://hwan-shell.tistory.com/320 [DNS]  
https://steady-coding.tistory.com/523 [DNS 정리]
