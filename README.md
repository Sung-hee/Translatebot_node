## 카카오톡 자동응답 API + 파파고 API를 이용한 자동번역 챗봇!

### 1. 챗봇이란 ?

- 대화형 인터페이스 상에서 규칙 또는 AI(인공지능)으로 유저와의 인터랙션을 하는 서비스 

- 쉽게 말하면, 대화를 하는 로보트

- 전화로 하는 ARS 자동응답시스템이 메신저 안으로 들어왔다고 생각하면 된다 !

  ​

> 아마존의 '알렉사', KT의 '기가지니', SK텔레콤의 '누구' 등의 서비스가 요즘 미디어에 자주 등장합니다.
>
> 일상 생활에서 음성인식 기능을 통해 서비스를 이용 할 수 있게 도와 줍니다.
>
>  
>
> "지니야, 오늘 날씨를 알려줘"
>
> "지니야, 트와이스 노래 틀어줘"
>
>  
>
> '자비스' 처럼 돈될(?) 만한 일을 해주지는 않지만 과거에 리모콘을 가지고 했던 작업을 음성인식 기술을 통해 생각보다 영리하게 처리합니다.
>
>  
>
> 이러한 "인공지능 비서" 역시 큰틀에서 보면 챗봇이라 할 수 있습니다. 직접 타이핑했던 텍스트가 음성으로 바뀌었을 뿐 결국 자연어 처리와 딥러닝 과정을 통해 사용자의 의도를 파악하고 서비스(스킬)을 수행하는 원리가 동일합니다.
>
> 출처: 
>
> http://jincrom.tistory.com/61



### 2. 채널 / 플랫폼

서비스의 품질이 아무리 좋아도 어떤 장소/플랫폼에서 서비스하는 가는 중요한 문제입니다.

아래는 "3.3 어떤 플랫폼에서 서비스를 제공 할 것인가"에 대한 표입니다. 

 

| 채널/플랫폼           | 설명                                       | 관련 링크                                    |
| ---------------- | ---------------------------------------- | ---------------------------------------- |
| 카카오톡(Kakao Talk) | 옐로아이디(비즈니스 아이디)를 통해서만 서비스 개발 가능          | [https://yellowid.kakao.com/](https://yellowid.kakao.com/login) |
| 라인(Line)         | Line Business Center의 Messanging API 를 통해서 개발 가능 | <https://business.line.me/ja/services/bot> |
| 페이스북(Facebook)   | Messenger 플랫폼의 API를 통해 개발 가능             | <https://developers.facebook.com/docs/messenger-platform> |

 

위 표는 각 플랫폼별 챗봇 개발을 위한 API를 제공해주는 링크 모음입니다.

 

서비스 실질적으로 활용해야 하는 기술은 각 플랫폼 사들이 제공하는 서비스의 특성과 API에 대한 이해입니다.



### 3. 구축환경

- Atom
- 네이버 PAPAGO SMT API 
- 카카오톡 자동응답 API
- C9 Node.js
- Iphone 6 (플러스 친구 테스트)



### 4. 구조 설계

> ```flow
> st=>start: Start
> op1=>operation: KaKao API
> cond=>condition: C9 Server
> op2=>operation: PAPAGO API
> op3=>operation: Iphone
> e=>end : End
>
> st->op1->cond->op2->op1
> cond(yes)->op2
> ```
>
> #### 순서도는 대충 이해하자!
>
> 1. 먼저 Kakao API에 Text가 입력
> 2. C9 Server를 통해 PAPAGO API 번역을 요청
> 3. 번역 정보를 JSON으로 C9 Server로 전달
> 4. C9 Server은 Kakao API 를 통해서 정보를 출력 !

### 5. node.js 패키지 설치
>```
>$ npm init -y
>$ npm install request
>$ npm install body-parser
>$ npm install express
>```



## 자동번역 챗봇 + 정규표현식(Regular Expression)

#### 텍스트가 한글인지 영어인지 구분하기 !

이번에는 카카오톡을 통해 텍스트가 입력이 된다면 ! 텍스트가 한글인지 영어인지 구분하여  

한글 -> 영어 / 영어 -> 한글 번역을 지원하자 !



#### 1. 정규표현식 (Regular Expression) 이란?

> 사전적인 의미로는 특정한 규칙을 가진 문자열특정한 규칙을 가진 문자열의 집합을 표현하는 데 사용하는 [형식 언어](https://ko.wikipedia.org/wiki/%ED%98%95%EC%8B%9D_%EC%96%B8%EC%96%B4)이다. 정규 표현식은 많은 [텍스트 편집기](https://ko.wikipedia.org/wiki/%ED%85%8D%EC%8A%A4%ED%8A%B8_%ED%8E%B8%EC%A7%91%EA%B8%B0)와 [프로그래밍 언어](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D_%EC%96%B8%EC%96%B4)에서 문자열의 검색과 치환을 위해 지원하고 있다.![img](http://www.nextree.co.kr/content/images/2016/09/jhkim-140117-RegularExpression-151-1.png)Regular Expression UML



#### 2. 한글 구분

> ```js
> var check = _obj.content
> var languageCheck = /[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]/;
>   //language가 ture이면 ko -> en 으로 번역
>   if(languageCheck.test(check)){
>     // 네이버 번역기 전송할 데이터 만들기
>     var options = {
>       url: api_url,
>       //한국어(source : ko) > 영어 (target : en ), 카톡에서 받은 메시지(text)
>       form: {'source':'ko', 'target':'en', 'text':req.body.content},
>       headers: {'X-Naver-Client-Id':client_id, 'X-Naver-Client-Secret': client_passkey}
>     };
>
>     translatepost(options);
>   }
>
>   //language가 false면 en -> ko 으로 번역
>   else {
>     var options = {
>       url: api_url,
>       //영어(source : en) > 한국어 (target : ko ), 카톡에서 받은 메시지(text)
>       form: {'source':'en', 'target':'ko', 'text':req.body.content},
>       headers: {'X-Naver-Client-Id':client_id, 'X-Naver-Client-Secret': client_passkey}
>     };
>
>     translatepost(options);
>   }
> ```


