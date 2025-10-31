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
<img width="564" height="130" alt="image" src="https://github.com/user-attachments/assets/cd9ba2f4-9d75-4a81-99f7-f2a8f5c3de5a" />

- 사용 기술 : CompletableFuture, JPA, Virtual Thread
- 기능 설명 : 사용자가 번역 요청 시 아래의 3단계를 걸쳐 작동
1. 클라이언트 내부 캐시를 조회 (실패 시 2단계 진행)
2. 서버 내부 캐시 조회 & DB 조회가 비동기 방식으로 동시에 실행 (모두 실패 시 3단계 진행)
3. 외부 번역 API 호출 후 결과 전달 (실패 시 에러 전달)

2단계의 서버 캐시 조회와 DB 조회는 I/O 대기가 발생하는 작업입니다. **CompletableFuture**와 **Virtual Thread**를 활용해 이 두 작업을 동시에(concurrently) 실행하도록 구현했습니다.

이를 통해, 순차적으로 실행했을 때(캐시 대기 + DB 대기) 발생할 지연 시간을 **둘 중 더 오래 걸리는 작업의 시간**만큼으로 단축하여, 사용자의 응답 지연 시간을 최소화했습니다.

[I/O pool 소스 코드 보기](https://github.com/FinalMonstu/FINAL_MONSTU_back/blob/main/src/main/java/com/icetea/MonStu/async/AsyncConfig.java)

[CompletableFuture 사 소스 코드 보기](https://github.com/FinalMonstu/FINAL_MONSTU_back/blob/main/src/main/java/com/icetea/MonStu/service/TranslationService.java)

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

### 🛠️ 사용 기술 스택

- **Frontend**: React, Material-UI (MUI)
- **Backend**: Spring Boot 3.4.4
- **DB**: MySQL 8.3
- **AI API**: Google Translation API
- **Auth & Security**: JWT, Spring Security
- **ORM & Query**: JPA, QueryDSL

### ⚙️ 배포 환경

- OS: Ubuntu Linux
- Domain : Cloudflare
- Hardware: Laptop

---

### ⭐ 성과

- RESTful API 설계 및 JPA 기반 CRUD 기능 구현
- QueryDSL을 활용한 동적 필터링 검색 기능 개발
- 캐싱 적용으로 AI 번역 속도 개선 (평균 140ms → 129ms)
- CompletableFuture와 I/O pool을 활용한 비동기 방식으로 외부 번역 API 기능 속도 개선 (380ms → 180ms)

---

### ✅ 성능·신뢰성 개선

- 3단계 조회 파이프라인: 캐시 → DB → 외부 API, 동시 실행 후 ‘첫 성공 결과’ 반환(캐시/DB 경쟁 실행으로 지연 최소화)

- 결과 캐싱 & 영속화: 외부 API 결과를 캐시 저장 + DB 저장으로 재사용·비용 절감

### ✅ 서버 스레드 효율성 분석

**사용 패키지 :** Vitual VM + JMeter

**시나리오 :** 30명의 사용자가 '동시에' '동일한' 단어를 요청

<img width="440" height="249" alt="image" src="https://github.com/user-attachments/assets/302488ba-7bb8-402b-ade5-10654061b02a" />
<img width="440" height="249" alt="image" src="https://github.com/user-attachments/assets/2e01de15-c0e3-4ab5-8bc5-133126426a6c" />

결과 : 실험 결과, 제안하는 비동기 방식은 동기 방식 대비 약 15% 적은 스레드 자원을 사용

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
