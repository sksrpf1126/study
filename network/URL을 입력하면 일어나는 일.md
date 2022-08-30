# **_URL을 입력하면 일어나는 일_**

## **_주소창에 www.naver.com을 입력했을 때 동작과정_**

1. DNS를 통해 URL에 맞는 IP를 가져온다.
2. 가져온 IP를 통해 TCP Socket을 연결한다.
   ##### https인 경우 TLS handshaking이 추가
3. HTTP 프로토콜로 요청
4. 요청에 대하여 서버가 응답
5. 웹 브라우저가 그린다. (HTML,CSS, JS 렌더링)

</br>

---

# **_참고_**

- https://owlgwang.tistory.com/1
- https://velog.io/@khy226/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EC%97%90-url%EC%9D%84-%EC%9E%85%EB%A0%A5%ED%95%98%EB%A9%B4-%EC%96%B4%EB%96%A4%EC%9D%BC%EC%9D%B4-%EB%B2%8C%EC%96%B4%EC%A7%88%EA%B9%8C
