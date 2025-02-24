#http 
## 목차
[[1_모든 것이 HTTP - 인터넷 통신과 프로토콜]]
[[2_모든 것이 HTTP - URI와 요청 흐름]]
[[3_모든 것이 HTTP - HTTP 기본]]
[[4_모든것이 HTTP - HTTP 메서드]]

# URI
네트워크 통신에서, 정보를 요청하는 입장을 클라이언트 , 정보를 제공하는 입장을 서버라고 말한다. 서버 컴퓨터에는 수백개, 수만개, 많으면은 수억개의 데이터 리소스들이 저장되어 있을것이다. 그러면 클라이언트와 서버는 통신과정에서 어떻게 서로 원하는 정보를 완벽하게 주고 받을 수 있을까 ? 
**답은 식별자를 만드는 것이다 . 정보를 식별하는 역할을 하는 식별자, 이를 URI 라고 부른다. **

> URI(Uniform Resource Identifier)


• Uniform: 리소스 식별하는 통일된 방식
• Resource: 자원, URI로 식별할 수 있는 모든 것(제한 없음)
• Identifier: 다른 항목과 구분하는데 필요한 정보



![](https://images.velog.io/images/kkj5158/post/570644e1-d5cc-459f-ac2c-2bba9c6f7571/3-1.URI.PNG)

**"URI는 로케이터(locator), 이름(name) 또는 둘다 추가로 분류될 수 있다"**

리소스의 위치를 나타낸다 -> URL Uniform Resource Locator
리소스 그 자체의 이름을 나타낸다 -> URN Uniform Resource Name

미네랄이라는 리소스가 구글이라는 사이트내에 있다. 
-> URN(리소스 이름)  : 미네랄 , URL(리소스 위치)  : 구글 


URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화 되지 않음
(실제로우리가 다루는 URI의 개념은 대부분 위치를 나타내는 URL의 개념이다. ) 
-> 앞으로 URI를 URL과 같은 의미로 이야기하겠음


> **URI는 리소스를 식별하는 정보이다 **

# 요청흐름

지금까지 배운 모드 내용을 바탕으로 http를 기반으로 서버와 클라이언트가 어떻게 서로 메세지를 주고받는지 정리해보자. 
![](https://images.velog.io/images/kkj5158/post/bea87f5f-9f09-4eb6-a8b2-e06b43eead4e/3-2%20http%20%EC%9A%94%EC%B2%AD%EB%A9%94%EC%8B%9C%EC%A7%80(%EC%B5%9C%EC%A2%85).PNG)

첫번째로, 클라이언트는 http 메시지를 tcp/ip패킷에 포장하여서 구글서버에 전달한다 . 
이 http 메시지 안에는 특정 정보를 식별하는 URI의 정보가 담겨져 있다 . 

![](https://images.velog.io/images/kkj5158/post/31b6c0f8-d506-4fd0-ab52-b4f8d0eaa1f1/3-2%20http%20%EC%9D%91%EB%8B%B5%EB%A9%94%EC%84%B8%EC%A7%80.PNG)

두번째로, 구글서버는 전달된 패킷을 뜯어서 , http 메세지를 확인한다. http 메세지 안에는 요청하는 **리소스의 정보를 알려주는 식별자인 URI ( URN , URL ) 의 정보가 담겨져 있을것이다**. 구글 서버는 이 URI를 바탕으로 해당 리소스를 찾는다. (여기서는 구글서버에서 hello라는 query 질의문을 한 결과가 리소스 일것이다. ) 

**서버는 해당 리소스를 찾은 후에 , 다시 http메세지를 TCP/IP 패킷에 포장하여서 클라이언트에게 보낸다. 
**


![](https://images.velog.io/images/kkj5158/post/29ac57bc-ee9b-44af-9da7-93a0249879e5/3-2%20http%20%EB%A0%8C%EB%8D%94%EB%A7%81.PNG)

세번째로,
**서버에서 받은 정보를 바탕으로 웹브라우저는 해당 정보를 사용자가 보기 편한 화면으로 구성해주는 렌더링 작업을 해준 다음에 화면에 띄어준다. **

이 모든 과정이 우리가, google에서 hello라는 검색어를 입력할때 발생하는 백엔드 단에서의 네트워크 통신과정이다. 

[[3_모든 것이 HTTP - HTTP 기본]]

