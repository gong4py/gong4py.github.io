
# Watson Conversation을 이용한 Chatbot 소개

2017-12-4, 명현기

---
1. 챗봇이란?
2. Chatbot을 만들수 있는 Tool 및 연동가능 메신저
3. Watson Conversation으로 챗봇만들기
4. (Application으로의 제작)
5. 시사점

---
# 1. 챗봇이란?

> 챗봇은 텍스트나 음성으로 인간과 대화하는 소프트웨어를 뜻함.
> 1950년대 앨런튜링이 제안
> 1960년대 엘리자라는 초기단계 챗봇등장
> 과거의 챗봇은 사전에 구축된 데이터베이스에서 대화 패턴을 찾아 기계적인 반응을 하는 수준에 불과
> 하지만 최근 들어 빅데이터 및 머신러닝 기반의 인공지능이 크게 발전하면서 챗봇도 유의미한 수준으로 진화 

---
# 2. Chatbot을 만들수 있는 Tool 및 연동가능 메신저
### Tool
1. Chatfuel
>가장 간단한 챗봇 툴 중 하나

2. Watson Conversation
>[Watson Conversation으로 회의실예약 챗봇만들기](https://developer.ibm.com/kr/cloud/bluemix/watsonservice/2017/01/13/watsonchatbot-1-watson-conversation/)

3. Python
> [Python](http://static.wooridle.net/lectures/chatbot/)

### 메신저
1. 카카오톡
> 한국인들이 가장 많이 사용
> 챗봇이 먼저 대화를 할때 건당 17원부과

2. 페이스북 메신저
> 자유로운 메시지 발송가능

---

# 3. Watson Conversation으로 챗봇만들기

###1. IBM Watson™ Conversation 서비스
> 자연어를 이해하고 기계 학습(Machine learning)을 사용하여 고객과 소통할 때 사람이 하는 대화 방식으로 응답하는 어플리케이션을 만들 수 있도록 해줌
![](https://developer.ibm.com/kr/wp-content/uploads/sites/98/conversation_arch_overview.png)

###2. Intent, Entity, Dialog
####Intent
> 사용자의 메시지에서 목적/의도를 분류해 내는 항목
> 인사, 일정 추가, 삭제 등 다양한 항목
> ![](https://i.imgur.com/gptxCnm.png)

####Entity
> 사용자가 한 말에서 얻어낼 수 있는 정보들, 장소, 시간과 같은 요소를 추출해냄
> 동의어를 일일이 추가해야 하는 번거로움 존재 -> 베타버전이지만 패턴이 존재
> ![](https://i.imgur.com/AM40LRh.png)
> 
> 기본적인 날짜, 시간과 같은  Watson conversation에서 제공하는 system entities 존재
> ![Imgur](https://i.imgur.com/S5Xsf6D.png)

####Dialog
> 메시지를 바탕으로 어떤 작업을 수행할지 logic tree를 설계하는 것
> 실제 동작하는 알고리즘
> 사용자의 입력에서 Intent를 인식하고 이에 따른 작업 수행
> ![Imgur](https://i.imgur.com/p73gbAf.png)
> json 형식 사용
> ![Imgur](https://i.imgur.com/J9DaCq1.png)

---
# (4. Application으로의 제작)

>  이 파일은 작동하는 환경을 지정해주는 파일입니다. 대화를 담당하는 Conversation항목과 데이터의 저장을 담당하는 Cloudant 항목이 존재합니다.

> 이 어플리케이션은 /api/message api를 완성하여 웹 클라이언트를 통해 Conversation 서비스를 노출합니다. 웹 클라이언트는 ./public에 개발합니다.

> Public폴더에는 웹에 ui를 개발하고, app.js를 통해 나타냅니다. App.js코드를 보면 ./api 밑에 REST API를 생성하여 노출하는 구조로 되어 있습니다.

> Api폴더는 Conversation API을 관리하고 호출합니다.
>  index.js는 api폴더의 하위의 api 목록을 관리하고 초기화하기 위한 index 파일입니다.
message.js는 message api를 노출함과 동시에 Conversation API를 호출하는 모듈입니다.

> 인풋을 담당하는 getConversationResponse 함수는 사용자의 message와 context 오브젝트를 input으로 받아 Conversation API를 호출하고 Promise 객체를 리턴합니다.

이후 카카오톡 플러스친구 개설 및 자동응답을 API형태로 바꿔줌
![Imgur](https://i.imgur.com/OP01pM1.png)

---
# 시사점
챗봇의 툴에 따른 배움의 깊이 - 파이썬은?
알고리즘만으로 추가된 일정을 확인하고 수정할 수 없다. application 개발을 좀 더 진행해야 이를 수행할 수 있다.
챗봇의 비즈니스성

---
