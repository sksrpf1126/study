# **_redirect와 forward_**

해당 글은 **_김영한님의 모든 개발자를 위한 HTTP 웹 기본 지식 및 여러 참고 글_** 들을 읽고, 정리한 내용이다.

---

Redirect란, Client에서 요청이 왔을 때 서버측은 응답을 하면서 http 헤더에 Location 정보에 다른 페이지(URL)를 넣어서 Client에게 다른 페이지로 이동하라고 명령을 내리는 것이다. (Redirect는 300번대 코드를 보낸다)

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/197325207-a822f3a6-eead-4b97-b913-e27a38f385fa.png" width = 70%>
</p>

위 이미지를 통해 순서를 정리하면

1. 요청(Client -> Server)  
   -> Client 가 "/event" url로 Server에 요청

2. 응답(Server -> Client)  
   -> 응답 헤더에 300번대 코드를 전송하며, Location에 Redirection할 URL 주소를 실어서 응답한다.

3. 자동 리다이렉트  
   -> Client는 Location 정보를 토대로, 새로운 URL(페이지)로 이동한다.

4. 요청(Client -> Server)  
   -> 자동 리다이렉트를 통해 새로운 URL에 맞게 Server로 새로운 요청을 보낸다.

5. 응답(Server -> Client)  
   -> Server는 요청에 맞는 처리를 하고나서 Client에 응답을 한다.

</br>

---

## **_3XX 상태코드에 따른 리다이렉션_**

Redirect는 상태코드에 따라 동작하는 방식을 정의해 놓았다. 즉, 같은 Redirect라 해도 동작방식이 다를 수 있다는 것이다.

• 300 Multiple Choices  
• 301 Moved Permanently  
• 302 Found  
• 303 See Other  
• 304 Not Modified  
• 307 Temporary Redirect  
• 308 Permanent Redirect

300번은 현재 사용하지 않으며, 304번은 캐시와 관련이 되어 있어 설명을 제외하고, 나머지를 설명.

### **_영구 리다이렉션_**

영구 리다이렉션은 301, 308 상태코드를 사용한다.

리소스의 URI가 영구적으로 이동하며, 보통 영구 리다이렉션의 용도는 사용하던 URL이 변경되었는데, 클라이언트가 이전의 URL로 요청할 시에 바뀐 URL로 다시 안내(Redirect)를 할 때 사용이 된다.

</br>

### **_301_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/197326410-8c543705-2c6b-46bc-9aa3-83f334b28772.png" width = 70%>
</p>

301은 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있다.  
본문이 제거가 되는게 아니고 될 수 있다는 것에 주의.

보통 이전에 어떠한 요청 메서드에 상관없이 Redirect 할 때의 메서드는 GET으로 요청한다.

</br>

### **_308_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/197326414-704b292d-782f-435f-8a09-4cbf67813727.png" width = 70%>
</p>

308의 경우에는 Redirect시 요청 메서드와 본문 유지(처음 POST를 보내면 리다이렉트도 POST 유지)를 한다.

</br>

## **_일시적인 리다이렉션_**

302, 303, 307이 일시적인 리다이렉션에 속하며, 리소스의 URI가 일시적으로 변경 할 때에 사용이 된다. 보통의 용도는 상품을 주문한 후에 새로고침을 하게 되면 그대로 POST 요청이 되기 때문에 중복 주문이 이루어지는데, 이를 방지하기 위해서 주문의 결과 페이지로 리다이렉션을 하는 것이다. 다른 에시로는 게시판 글쓰기가 있다.

302 Found  
• 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음(MAY)

307 Temporary Redirect  
• 302와 기능은 같음  
• 리다이렉트시 요청 메서드와 본문 유지(요청 메서드를 변경하면 안된다. MUST NOT)

303 See Other  
• 302와 기능은 같음  
• 리다이렉트시 요청 메서드가 GET으로 변경

</br>

---

## **_PRG(POST - Redirect - GET)_**

일시적인 리다이렉션의 용도 예시에서 상품 중복 주문을 들었다. 이를 해결하기 위해서 사용하는 패턴이 바로 PRG 이다.

## **_PRG 사용 전_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/197326409-597c801c-d7b7-420b-9a14-e3e2f913dadd.png" width = 70%>
</p>

</br>

## **_PRG 사용 후_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/197326408-4bf9345f-7a9e-4f5a-8629-535a8bc5beea.png" width = 70%>
</p>

위와 같이 POST 요청을 Redirect를 통해 GET요청으로 변환 하여 조회화면을 보여준다. 그러면 클라이언트 측에서 새로고침으로 마지막 요청을 계속 날려도 GET요청이 가기 때문에 상품 중복 주문을 방지할 수 있게 된다.

보통 명확한 303과 307을 권장하지만 이미 실무에서는 302를 기본값으로 많이 사용하기 때문에 자동 리다이렉션시에 GET요청으로 변한다면 302를 사용해도 문제가 없다.

</br>

---

## **_Forward_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/197327526-2c5bb87f-9521-4760-ad9c-9dcc2f85d456.png" width = 50%>
</p>

출처 : https://mangkyu.tistory.com/51

Redirect의 경우에는 2번의 요청과 2번의 응답을 서로 주고받고 하는 방식이다. 즉, 새로운 요청을 하여 다시 서버로 들어온다는 것이다.

하지만 Forward는 위 이미지와 같이 Client는 단 한번의 요청을 Server로 보내고, 서버 내부에서의 페이지(URL 변경)이동만 한다.

그렇기에 Client측은 페이지가 이동이 된 것을 알 수 없다.

보통 로그인이 필요한 페이지의 경우에 Forward를 사용한다.  
Redierect로 2번의 요청을 보내서 2번 응답하는 것보다는 서버 내부에서 바로 로그인 페이지를 띄워주면 되기에 더 효율적이다.
