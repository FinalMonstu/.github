# MonStu 다국어 학습 플랫폼
요약: AI 보조 영어 학습 웹 애플리케이션

개발 기간: "2025.03 ~ 08"

핵심 기술 스택: 
&nbsp; Java+Spring Boot, MySQL, React, JPA


---

### 💿 Demo Video:
[![YouTube Video Thumbnail](https://img.youtube.com/vi/CH2E0r3U4CA/hqdefault.jpg)](https://www.youtube.com/watch?v=CH2E0r3U4CA)

### 🚩 Develop Log Blog: [tistory.com](https://code-is-code.tistory.com/)

---

### 📖 소개

웹에서 복사해 온 텍스트를 붙여넣고 모르는 단어·문장을 드래그만 하면 즉시 번역·저장되는 AI 보조 영어 학습 웹 애플리케이션입니다. 

사용자가 선택한 표현을 실시간 번역으로 보여주고, 학습 기록을 제공하여 반복 학습을 도와줍니다. 
 
비회원도 핵심 기능을 이용할 수 있으며, 회원은 게시물 및 번역 기록을 저장하고 관리 할 수 있어 재미있고 가벼운 학습 경험을 제공합니다.

---

### 🛠️ 사용 기술 스택

- **Frontend**: React, Material-UI (MUI)
- **Backend**: Spring Boot 3.4.4
- **DB**: MySQL 8.3
- **AI API**: Google Translation API
- **Auth & Security**: JWT, Spring Security
- **ORM & Query**: JPA, QueryDSL

---

### 📌 [핵심 성과] 번역 API "Thundering Herd" 문제 해결

[문제 인지]

프로젝트의 핵심 기능인 번역 API에서, **동일한 단어**에 대한 **동시 요청**이 몰릴 경우(Thundering Herd) 심각한 성능 및 데이터 정합성 문제가 발생함을 JMeter 테스트로 확인



[해결]

"Key별 락" 패턴과 **"가상 스레드(Virtual Threads)"**를 도입하여 해결


<br/>
[문제 상황 : 중복 호출 및 DB 중복 저장 (Duplicate)]

JMeter를 통해 "100명의 사용자가 'apple'을 동시에 요청"하는 시나리오로 테스트한 결과, 단순 동기 방식(Semaphore만 사용)은 DuplicateKeyException을 발생시키며 실패

원인: Semaphore를 통과한 20개의 스레드가 동시에 "apple" API를 중복 호출하고, DB에 20번 중복 저장을 시도

<img width="1280" height="61" alt="image" src="https://github.com/user-attachments/assets/8bac8464-93b3-4a54-8847-65699bbb734b" />
<img width="1280" height="35" alt="image" src="https://github.com/user-attachments/assets/f921549a-fd6d-4c13-8c39-7f1332bbc6ad" />

<br/>
<br/>
[해결 상세 : "Key별 락(Per-Key Lock)"을 통한 요청 병합]

1. "Global Lock" 대신, **ConcurrentMap**과 **CompletableFuture**를 "작업 티켓"으로 활용하는 "Key별 락" 패턴을 도입
2. Virtual_thread 적용, 스레드 블로킹 비용 절약


<br/>
[해결 후 코드 테스트 결과 - 성공]

<img width="1280" height="53" alt="image" src="https://github.com/user-attachments/assets/ece97d28-d0c0-483e-909e-e0a18c730bce" />

<br/>
<br/>
[해결 원리]

<img width="1280" height="649" alt="image" src="https://github.com/user-attachments/assets/09cd9a8e-d6a9-4dd6-8c87-70f4cc108142" />


1. "깃발 세우기" (computeIfAbsent):
 
- 500개의 "apple" 요청이 오면 선두 스레드가 ConcurrentMap에 진입

2. "진동벨 남기기" (supplyAsync):
   
- 선두 스레드가 외부 API호출 작업을 Virtual Thread에게 비동기로 넘기고 진동벨(CompletableFuture)을 앱에 저장 후 락을 해제

3. 499개의 스레드 모두 동일한 진동벨을 들고  future.get()에서 대기, "선두 스레드"가 결과를 .complete()하여 대기 중인 스레드에게 동일한 결과를 공유


추가 효과 (회복탄력성):

- .orTimeout()을 설정하여, 외부 API 지연 시 "연쇄 장애(Cascading Failure)"를 방지

[소스 코드 보기](https://github.com/FinalMonstu/FINAL_MONSTU_back/blob/main/src/main/java/com/icetea/MonStu/service/TranslationService.java)

---

### 📖 구성
<img width="996" height="812" alt="image" src="https://github.com/user-attachments/assets/97ffd599-aff8-4c2c-b1dc-7194addbe86f" />

---

### 📖 ERD 설계
<img width="1760" height="781" alt="image" src="https://github.com/user-attachments/assets/3714d35a-4cc9-4521-8615-0af6057c80e0" />

---

### 🚀 주요 기능

- 실시간 단어 / 문장 번역
- 회원 관리, 게시글 관리

---

### 📌핵심기술 - 번역 기능   
<img width="564" height="550" alt="image" src="https://github.com/user-attachments/assets/227f205f-492a-4fc9-a197-a57d92b9a2e8" />
<img width="564" height="198" alt="화면 캡처 2025-11-03 222545" src="https://github.com/user-attachments/assets/9f5b1217-395f-434a-bc8a-c73bfc2fede2" />

- 사용 기술 : CompletableFuture, JPA, Virtual Thread, ConcurrentMap
- 기능 설명 : 사용자가 번역 요청 시 아래의 4단계를 걸쳐 작동
1. 클라이언트 내부 캐시를 조회 (실패 시 2단계 진행)
2. 서버 내부 캐시 조회 (실패 시 3단계 진행)
3. DB 조회 (실패 시 4단계 진행)
4. 외부 번역 API 호출 후 결과 전달 (실패 시 에러 전달)

Semaphore로 외부 API 요청을 조절하고
CompletableFuture와 ConcurrentMap를 활용하여 **중복 호출** 문제를 해결

---

### 📌핵심기술 - 관리자 게시물, 사용자 관리 기능  
<img width="432" height="246" alt="image" src="https://github.com/user-attachments/assets/cd68a90e-363c-47ca-bba1-966b5f91f967" />
<img width="428" height="248" alt="image" src="https://github.com/user-attachments/assets/cc9db796-471f-439d-9813-7c702c49b6de" />
<img width="860" height="336" alt="image" src="https://github.com/user-attachments/assets/f8d8b7b2-fd74-4ac1-8967-18938170b88f" />

- 사용 기술 : JPA, QueryDSL, React MUI
- 기능 설명 : 정보를 필터링 조회 후 화면에 표시
1. 요청 받은 조건을 조합하여 QueryDSL로 쿼리를 생성
2. 쿼리를 전달 받아 JPA를 통해 DB 조회
3. 조회된 정보를 사용자에게 전달, MUI를 사용하여 깔끔한 UI제공
   
[소스 코드 보기](https://github.com/FinalMonstu/FINAL_MONSTU_back/blob/main/src/main/java/com/icetea/MonStu/manager/FilterPredicateManager.java)

---

### ⚙️ 배포 환경

- OS: Ubuntu Linux
- Domain : Cloudflare
- Hardware: 로컬 개발 서버

---

### ⭐ 성과

- RESTful API 설계 및 JPA 기반 CRUD 기능 구현
- QueryDSL을 활용한 동적 필터링 검색 기능 개발
- 캐싱 적용으로 AI 번역 속도 개선 
- CompletableFuture와 ConcurrentMap를 활용하여 **중복 호출** 방지

---

### 🙆‍♂️ 개발자 포트폴리오 [notion.com](https://www.notion.so/248303eae1f280949123e25ee9ad7d04)

---

🚀 함께 사용된 소규모 프로젝트

Frontend

[React MultiSnackBar (@Mui) 팝업 메시지 컴포넌트](https://www.notion.so/React-MultiSnackBar-Mui-221303eae1f280e0b55ed9d5d8957111?pvs=21) 

[React Custom Route 설정 개선](https://www.notion.so/React-Custom-Route-1a0303eae1f280bea23efb4c4830e718?pvs=21) 

[React HandleState – 서버 응답 구조 통일화](https://www.notion.so/React-HandleState-1a0303eae1f280c1940be96afbf81ca0?pvs=21) 


Backend

[QueryDSL 기반 동적 필터링 검색](https://www.notion.so/QueryDSL-221303eae1f280c8b79feff1eeee8a34?pvs=21)

---
