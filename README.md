![logo](https://user-images.githubusercontent.com/38099203/121180185-87471200-c89b-11eb-84a8-51b2419acd7d.jpg)

# 숙소예약(AirBnB)

본 예제는 airbnb_project 팀 과제에 대한 개인 평가 자료입니다.
- 팀 과제 :  https://github.com/indie2k/airbnb_project


# Table of contents

- [예제 - 개인 평가 자료](#---)
  - [서비스 시나리오](#서비스-시나리오)
  - [팀 과제에 추가된 서비스 시나리오](#서비스-시나리오)
  - [체크포인트](#체크포인트)
  - [분석/설계](#분석설계)
  - [구현:](#구현-)
    - [DDD 의 적용](#ddd-의-적용)
    - [폴리글랏 퍼시스턴스](#폴리글랏-퍼시스턴스)
    - [폴리글랏 프로그래밍](#폴리글랏-프로그래밍)
    - [동기식 호출 과 Fallback 처리](#동기식-호출-과-Fallback-처리)
    - [비동기식 호출 과 Eventual Consistency](#비동기식-호출-과-Eventual-Consistency)
  - [운영](#운영)
    - [CI/CD 설정](#cicd설정)
    - [동기식 호출 / 서킷 브레이킹 / 장애격리](#동기식-호출-서킷-브레이킹-장애격리)
    - [오토스케일 아웃](#오토스케일-아웃)
    - [무정지 재배포](#무정지-재배포)
  - [신규 개발 조직의 추가](#신규-개발-조직의-추가)


# 서비스 시나리오

AirBnB 커버하기

기능적 요구사항
1. 호스트가 임대할 숙소를 등록/수정/삭제한다.
2. 고객이 숙소를 선택하여 예약한다.
3. 예약과 동시에 결제가 진행된다.
4. 예약이 되면 예약 내역(Message)이 전달된다.
5. 고객이 예약을 취소할 수 있다.
6. 예약 사항이 취소될 경우 취소 내역(Message)이 전달된다.
7. 숙소에 후기(review)를 남길 수 있다.
8. 전체적인 숙소에 대한 정보 및 예약 상태 등을 한 화면에서 확인 할 수 있다.(viewpage)

비기능적 요구사항
1. 트랜잭션
    1. 결제가 되지 않은 예약 건은 성립되지 않아야 한다.  (Sync 호출)
1. 장애격리
    1. 숙소 등록 및 메시지 전송 기능이 수행되지 않더라도 예약은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    1. 예약 시스템이 과중되면 사용자를 잠시동안 받지 않고 잠시 후에 하도록 유도한다  Circuit breaker, fallback
1. 성능
    1. 모든 방에 대한 정보 및 예약 상태 등을 한번에 확인할 수 있어야 한다  (CQRS)
    1. 예약의 상태가 바뀔 때마다 메시지로 알림을 줄 수 있어야 한다  (Event driven)

# 팀 과제에 추가된 서비스 시나리오

Mileage 결제 및 제휴상품 구매 기능 추가하기

추가된 기능적 요구사항
1. 회원 가입 시 최조 mileage를 부여하고 Review를 남길 경우 추가로 mileage를 부여한다.
2. 임대 숙소 예약 결제 시 mileage를 사용할수 있다.
3. 예약 지역 근처의 제휴상품 (activity, 맛집 이용권 등)을 등록하고 판매할수 있다.
5. 제휴상품 결제 시 mileage를 사용할수 있다.
6. 예약 및 제휴상품 구입 내역에 대해 Message 전달을 한다.
7. mileage 및 제휴상품 구매 등을 한 화면에서 확인 할 수 있다.(viewpage)

추가된 비기능적 요구사항
1. 트랜잭션
    1. 결제시 mileage 차감이 같이 진행되어야 한다.  (Sync 호출)
	2. 결제가 되지 않은 제휴상품 구매 건은 성립되지 않아야 한다.  (Sync 호출)
1. 장애격리
    1. 제휴상품 등록 및 리뷰장성 시 mileage 추가 부여, 메시지 전송 기능이 수행되지 않더라도 예약 및 제휴상품 구매 등은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    1. 제휴상품 주문 시스템이 과중되면 사용자를 잠시동안 받지 않고 잠시 후에 하도록 유도한다  Circuit breaker, fallback
1. 성능
    1. 회원 가입, mileage, 제휴상품 구매 등을 한번에 확인할 수 있어야 한다  (CQRS)
    1. 상태가 바뀔 때마다 메시지로 알림을 줄 수 있어야 한다  (Event driven)


# 체크포인트

- 분석 설계

  - 이벤트스토밍: 
    - 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
    - 각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
    - 어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
    - 기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?    

  - 서브 도메인, 바운디드 컨텍스트 분리
    - 팀별 KPI 와 관심사, 상이한 배포주기 등에 따른  Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
      - 적어도 3개 이상 서비스 분리
    - 폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
    - 서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?
  - 컨텍스트 매핑 / 이벤트 드리븐 아키텍처 
    - 업무 중요성과  도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
    - Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
    - 장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
    - 신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
    - 이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?

  - 헥사고날 아키텍처
    - 설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?
    
- 구현
  - [DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?
    - Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
    - [헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를 적응시킬 수 있는가?
    - 분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?
  - Request-Response 방식의 서비스 중심 아키텍처 구현
    - 마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
    - 서킷브레이커를 통하여  장애를 격리시킬 수 있는가?
  - 이벤트 드리븐 아키텍처의 구현
    - 카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
    - Correlation-key:  각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
    - Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
    - Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
    - CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가 가능한가?

  - 폴리글랏 플로그래밍
    - 각 마이크로 서비스들이 하나이상의 각자의 기술 Stack 으로 구성되었는가?
    - 각 마이크로 서비스들이 각자의 저장소 구조를 자율적으로 채택하고 각자의 저장소 유형 (RDB, NoSQL, File System 등)을 선택하여 구현하였는가?
  - API 게이트웨이
    - API GW를 통하여 마이크로 서비스들의 집입점을 통일할 수 있는가?
    - 게이트웨이와 인증서버(OAuth), JWT 토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?
- 운영
  - SLA 준수
    - 셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
    - 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
    - 오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
    - 모니터링, 앨럿팅: 
  - 무정지 운영 CI/CD (10)
    - Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명 
    - Contract Test :  자동화된 경계 테스트를 통하여 구현 오류나 API 계약위반를 미리 차단 가능한가?


# 분석/설계

## AS-IS 조직 (Horizontally-Aligned)
  ![image](https://user-images.githubusercontent.com/77129832/119316165-96ca3680-bcb1-11eb-9a91-f2b627890bab.png)

## TO-BE 조직 (Vertically-Aligned)  
  ![TO-BE조직](https://user-images.githubusercontent.com/38099203/121192817-44d80200-c8a8-11eb-9fb7-2679ce77005e.png)


## Event Storming 결과
* MSAEz 로 모델링한 이벤트스토밍 결과:  http://www.msaez.io/#/storming/Dln1uw3DPPRk4Av4kdEG49YjZZ23/mine/5bb5e6b5f6167c87380b1b5b65c45217


### mileage 및 제휴상품 이벤트 신규 도출
![신규 개인과제 이벤트 도출](https://user-images.githubusercontent.com/38099203/121212087-00ecf900-c8b8-11eb-8570-e01fcc503338.PNG)

### mileage 및 제휴상품 적용을 위한 기존 이벤트 수정
![기존 팀과제 이벤트 수정](https://user-images.githubusercontent.com/38099203/121205022-4c9ca400-c8b2-11eb-95f6-574bfb4e89c5.PNG)

    - 결제 시스템 : 
      - mlieage 사용할수 있도록 memId와 mileage 사용 정보 저장
	  - 제휴상품 및 주문정보 저장
    - 숙소 예약 시스템 : 
      - mlieage 사용할수 있도록 memId와 mileage 사용 정보 저장
    - 리뷰 시스템 : 
      - 리뷰작성 시 mileage 부여를 위한 정보 저장

### mileage 및 제휴상품 부적격 이벤트 탈락
![신규 개인과제 부적격 이벤트 퇴출](https://user-images.githubusercontent.com/38099203/121212289-2b3eb680-c8b8-11eb-8fd1-d1dba4b3b3a1.PNG)


### 신규 개인과제 액터, 커맨드 부착하여 읽기 좋게
![신규 개인과제 액터, 커맨드 부착하여 읽기 좋게](https://user-images.githubusercontent.com/38099203/121214558-2f6bd380-c8ba-11eb-8d26-1218fe73638f.PNG)


### 신규 개인과제 이벤트 어그리게잇으로 묶기
![신규 개인과제 이벤트 어그리게잇으로 묶기](https://user-images.githubusercontent.com/38099203/121215146-b8830a80-c8ba-11eb-9410-b6354d95a82b.PNG)

    - Member, Affiliateproduct, Order 등은 그와 연결된 command 와 event 들에 의하여 트랜잭션이 유지되어야 하는 단위로 그들 끼리 묶어줌

### 신규 개인과제 이벤트 바운디드 컨텍스트로 묶기

![신규 개인과제 이벤트 바운디드 컨텍스트로 묶기](https://user-images.githubusercontent.com/38099203/121215886-5c6cb600-c8bb-11eb-8d6f-3017309df19e.PNG)

 
### 기존 팀과제 이벤트와 신규 개인과제 이벤트 폴리시 부착

![기존 팀과제 이벤트와 신규 개인과제 이벤트 폴리시 부착](https://user-images.githubusercontent.com/38099203/121220414-8cb65380-c8bf-11eb-9408-0fe9b7bfa754.PNG)


### 기존 팀과제 이벤트와 신규 개인과제 이벤트 간 폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)

![기존 팀과제 이벤트와 신규 개인과제 이벤트 폴리시의 이동과 컨텍스트 매핑](https://user-images.githubusercontent.com/38099203/121222228-4f52c580-c8c1-11eb-85f2-b45053da983d.PNG)


### 기능적/비기능적 요구사항을 커버하는지 1차 검증

![기존 팀과제 이벤트와 신규 개인과제 이벤트 1차 검증](https://user-images.githubusercontent.com/38099203/121446338-40a60480-c9ce-11eb-93b1-9bc343f5ad33.PNG)

    - 회원 가입 시 최조 mileage를 부여하고 Review를 남길 경우 추가로 mileage를 부여한다.(ok)
    - 임대 숙소 예약 결제 시 mileage를 사용할수 있다.(ok)
    - 예약 지역 근처의 제휴상품 (activity, 맛집 이용권 등)을 등록하고 판매할수 있다.(ok)
    - 제휴상품 결제 시 mileage를 사용할수 있다.(ok)
    - 예약 및 제휴상품 구입 내역에 대해 Message 전달을 한다.(ok)
    - mileage 및 제휴상품 구매 등을 한 화면에서 확인 할 수 있다.(viewpage)(ok)

### 모델 수정

![모델 수정](https://user-images.githubusercontent.com/38099203/121224805-e15bcd80-c8c3-11eb-91ae-34d087a97e59.PNG)

### 비기능 요구사항에 대한 검증

![모델 수정_비기능 요구사항에 대한 검증](https://user-images.githubusercontent.com/38099203/121225928-f84eef80-c8c4-11eb-871f-04b3fd026e35.PNG)

    - 1) 결제시 mileage 차감이 같이 진행되어야 한다.  (Sync 호출)
    - 2) 결제가 되지 않은 제휴상품 구매 건은 성립되지 않아야 한다.  (Sync 호출)
    - 3) 제휴상품 등록 및 리뷰장성 시 mileage 추가 부여, 메시지 전송 기능이 수행되지 않더라도 예약 및 제휴상품 구매 등은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    - 4) 회원 가입, mileage, 제휴상품 구매 등을 한번에 확인할 수 있어야 한다  (CQRS)
    - 5) 상태가 바뀔 때마다 메시지로 알림을 줄 수 있어야 한다  (Event driven)


## 헥사고날 아키텍처 다이어그램 도출

![image](https://user-images.githubusercontent.com/80744273/119319091-fc6bf200-bcb4-11eb-9dac-0995c84a82e0.png)


    - Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
    - 호출관계에서 PubSub 과 Req/Resp 를 구분함
    - 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐


# 구현:

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
   mvn spring-boot:run
```

## CQRS

### 기존 팀과제 
숙소(Room) 의 사용가능 여부, 리뷰 및 예약/결재 등 총 Status 에 대하여 고객(Customer)이 조회 할 수 있도록 CQRS 로 구현
- room, review, reservation, payment 개별 Aggregate Status 를 통합 조회하여 성능 Issue 를 사전에 예방할 수 있다.
- 비동기식으로 처리되어 발행된 이벤트 기반 Kafka 를 통해 수신/처리 되어 별도 Table 에 관리한다


### 신규 개인과제
- Table 모델링 (ROOMVIEW)
  기존 팀 과제때와 같이 사용할수 있도록 PK를 roomViewId로 변경하고 자동 생성되게 함

  ![CQRS_Table 모델링 (ROOMVIEW)](https://user-images.githubusercontent.com/38099203/121352503-e7a38580-c967-11eb-83c7-84ec0e23ddbe.PNG)
  
- 회원정보, 제휴상품 주문 정보 생성 변경 시 roomView에 같이 저장되록 함

  회원 가입 시 처리 부분
  ![CQRS_회원가입이 되었을 때](https://user-images.githubusercontent.com/38099203/121353400-ca22eb80-c968-11eb-83bf-65aa163358c4.PNG)

  회원 가입 시 roomView 메시지 부분
  ![CQRS_회원가입시 viewpage 데이터](https://user-images.githubusercontent.com/38099203/121357288-a6fa3b00-c96c-11eb-9392-8778a9381073.PNG)

  제휴 상품 주문 확정 시 처리부분
  ![CQRS_제휴상품 주문이 되었을 때](https://user-images.githubusercontent.com/38099203/121353943-5cc38a80-c969-11eb-9106-f1323f6703cd.PNG)
  
  제휴 상품 주문 확정 시 roomView 메시지 부분
  ![CQRS_주문 확정 시 viewpage 데이터](https://user-images.githubusercontent.com/38099203/121357012-60a4dc00-c96c-11eb-8000-555b59b77f6a.PNG)

  
## 신규 개인과제 기능 소개
- 회원 등록, 변경 기능 (신규)
  
  회원 등록
  ![신규개인과제개요_회원등록](https://user-images.githubusercontent.com/38099203/121359693-b24e6600-c96e-11eb-9536-d66b1814a898.PNG)
  
  회원 수정
  ![신규개인과제개요_회원수정](https://user-images.githubusercontent.com/38099203/121360033-048f8700-c96f-11eb-868b-659c354ee097.PNG)
  
- 제휴상품 등록, 변경 기능 (신규)

  제휴상품 등록
  ![신규개인과제개요_제휴상품등록](https://user-images.githubusercontent.com/38099203/121360526-7667d080-c96f-11eb-87ae-3eea3eba01a2.PNG)
  
  제휴상품 변경
  ![신규개인과제개요_제휴상품변경](https://user-images.githubusercontent.com/38099203/121360810-b3cc5e00-c96f-11eb-9a6f-91d946810742.PNG)

- 제휴 상품 주문 기능 (주문 시 마일리지 사용 가능) (신규 + 수정 (팀 과제인 Payment에 수정))
  ![신규개인과제개요_제휴상품주문](https://user-images.githubusercontent.com/38099203/121361410-39500e00-c970-11eb-8181-5f88e66c7e22.PNG)

- 리뷰 작성 시 마일리지 제공 (수정 (팀 과제인 review에 수정) )
  ![신규개인과제개요_리뷰작성 시 마일리지 부여](https://user-images.githubusercontent.com/38099203/121361931-ab285780-c970-11eb-9b49-667ec6a3ed7c.PNG)

- 회원 정보 변경 및 주문 시 메시지 알람 기능 (수정 (팀 과제인 message에 수정) )
  ![신규개인과제개요_메시지알람기능](https://user-images.githubusercontent.com/38099203/121362226-ec206c00-c970-11eb-851c-8121a0f57fe3.PNG)
  
 

## API 게이트웨이
      1. gateway 스프링부트 App을 추가 후 application.yaml내에 각 마이크로 서비스의 routes 를 추가하고 gateway 서버의 포트를 8080 으로 설정함
         member, order, affiliateproduct 신규 추가
          - application.yaml 예시
            ```
			  profiles: docker
			  cloud:
				gateway:
				  routes:
					- id: payment
					  uri: http://payment:8080
					  predicates:
						- Path=/payments/** 
					- id: room
					  uri: http://room:8080
					  predicates:
						- Path=/rooms/**, /reviews/**, /check/**
					- id: reservation
					  uri: http://reservation:8080
					  predicates:
						- Path=/reservations/**
					- id: message
					  uri: http://message:8080
					  predicates:
						- Path=/messages/** 
					- id: viewpage
					  uri: http://viewpage:8080
					  predicates:
						- Path= /roomviews/**
					- id: member
					  uri: http://member:8080
					  predicates:
						- Path=/members/** 
					- id: affiliateproduct
					  uri: http://affiliateproduct:8080
					  predicates:
						- Path=/affiliateproducts/**
					- id: order
					  uri: http://order:8080
					  predicates:
						- Path=/orders/**
				  globalcors:
					corsConfigurations:
					  '[/**]':
						allowedOrigins:
						  - "*"
						allowedMethods:
						  - "*"
						allowedHeaders:
						  - "*"
						allowCredentials: true           
            ```

         
      2. Kubernetes용 Deployment.yaml 을 작성하고 Kubernetes에 Deploy를 생성함
          - Deployment.yaml 예시
          

            ```
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: gateway
              namespace: airbnb
              labels:
                app: gateway
            spec:
              replicas: 1
              selector:
                matchLabels:
                  app: gateway
              template:
                metadata:
                  labels:
                    app: gateway
                spec:
                  containers:
                    - name: gateway
                      image: 985702435631.dkr.ecr.ap-northeast-2.amazonaws.com/gateway:latest
                      ports:
                        - containerPort: 8080
            ```               
            

            ```
            Deploy 생성
            kubectl apply -f deployment.yaml
            ```     
          - Kubernetes에 생성된 Deploy. 확인
            
![gateway_deploy](https://user-images.githubusercontent.com/38099203/121364568-dd3ab900-c972-11eb-9a76-8fdc727c1eb7.png)
	    
            
      3. Kubernetes용 Service.yaml을 작성하고 Kubernetes에 Service/LoadBalancer을 생성하여 Gateway 엔드포인트를 확인함. 
          - Service.yaml 예시
          
            ```
            apiVersion: v1
              kind: Service
              metadata:
                name: gateway
                namespace: airbnb
                labels:
                  app: gateway
              spec:
                ports:
                  - port: 8080
                    targetPort: 8080
                selector:
                  app: gateway
                type:
                  LoadBalancer           
            ```             

           
            ```
            Service 생성
            kubectl apply -f service.yaml            
            ```             
            
            
          - API Gateay 엔드포인트 확인
           
            ```
            Service  및 엔드포인트 확인 
            kubectl get svc -n airbnb           
            ```                 
![gateway_endpoint](https://user-images.githubusercontent.com/38099203/121364917-2ab72600-c973-11eb-8e9b-9eda0ebdaf5e.png)

# Correlation

팀 프로젝트에서는 예약(Reservation)을 하면 동시에 연관된 방(Room), 결제(Payment) 등으로 구성

예약등록
![image](https://user-images.githubusercontent.com/31723044/119320227-54572880-bcb6-11eb-973b-a9a5cd1f7e21.png)
예약 후 - 방 상태
![image](https://user-images.githubusercontent.com/31723044/119320300-689b2580-bcb6-11eb-933e-98be5aadca61.png)
예약 후 - 예약 상태
![image](https://user-images.githubusercontent.com/31723044/119320390-810b4000-bcb6-11eb-8c62-48f6765c570a.png)
예약 후 - 결제 상태
![image](https://user-images.githubusercontent.com/31723044/119320524-a39d5900-bcb6-11eb-864b-173711eb9e94.png)
예약 취소
![image](https://user-images.githubusercontent.com/31723044/119320595-b6b02900-bcb6-11eb-8d8d-0d5c59603c72.png)
취소 후 - 방 상태
![image](https://user-images.githubusercontent.com/31723044/119320680-ccbde980-bcb6-11eb-8b7c-66315329aafe.png)
취소 후 - 예약 상태
![image](https://user-images.githubusercontent.com/31723044/119320747-dcd5c900-bcb6-11eb-9c44-fd3781c7c55f.png)
취소 후 - 결제 상태
![image](https://user-images.githubusercontent.com/31723044/119320806-ee1ed580-bcb6-11eb-8ccf-8c81385cc8ba.png)


개인 과제에서는 제휴상품 주문(Order)을 하면 동시에 제휴상품 (Affiliateproduct), 결제(Payment) 등으로 구성

제휴상품 주문
![Correlation_제휴상품 주문](https://user-images.githubusercontent.com/38099203/121373209-d9f6fb80-c979-11eb-8e41-9471953a3be2.PNG)

제휴상품 주문 후 제휴상품 상태
![Correlation_제휴상품 주문 후 제휴상품 상태](https://user-images.githubusercontent.com/38099203/121373505-162a5c00-c97a-11eb-8ce2-af728c332788.PNG)

제휴상품 주문 후 주문 상태
![Correlation_제휴상품 주문 후 주문 상태](https://user-images.githubusercontent.com/38099203/121373868-630e3280-c97a-11eb-8618-8debf4a2f5f0.PNG)

제휴상품 주문 후 결제 상태
![Correlation_제휴상품 주문 후 결제 상태](https://user-images.githubusercontent.com/38099203/121374130-9b157580-c97a-11eb-9a32-3ab3621291f3.PNG)

제휴상품 주문 취소
![Correlation_제휴상품 주문 취소](https://user-images.githubusercontent.com/38099203/121374875-39a1d680-c97b-11eb-9944-7ea36e91e3af.PNG)

제휴상품 주문 취소 후 제휴상품 상태
![Correlation_제휴상품 주문 취소 후 제휴상품 상태](https://user-images.githubusercontent.com/38099203/121377588-6ce56500-c97d-11eb-868f-c5dd65c4d2a9.PNG)

제휴상품 주문 취소 후 주문 상태
![Correlation_제휴상품 주문 취소 후 주문 상태](https://user-images.githubusercontent.com/38099203/121377843-a4eca800-c97d-11eb-9f06-a354d93af4bc.PNG)

제휴상품 주문 취소 후 결제 상태
![Correlation_제휴상품 주문 취소 후 결제 상태](https://user-images.githubusercontent.com/38099203/121378067-d49bb000-c97d-11eb-87fd-3a1a3d84b598.PNG)



## DDD 의 적용

- 팀 프로젝트 때와 동일하게 신규로 추가된 회원 (Member), 제휴상품 주문 (Order), 제휴상품 (Affiliateproduct) 등을 Entity 로 선언하였다. (예시는 회원 (Member) 마이크로 서비스). 
  이때 가능한 현업에서 사용하는 언어 (유비쿼터스 랭귀지)를 그대로 사용하려고 노력했다. 
  현실에서 발생가는한 이벤트에 의하여 마이크로 서비스들이 상호 작용하기 좋은 모델링으로 구현을 하였다.

```
package airbnb;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import java.util.List;
import java.util.Date;

@Entity
@Table(name="Member_table")
public class Member {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long memId;
    private Long mileage;
    private String status;

    @PostPersist
    public void onPostPersist(){
        MemberJoined memberJoined = new MemberJoined();
        BeanUtils.copyProperties(this, memberJoined);
        memberJoined.publishAfterCommit();
    }

    @PostUpdate
    public void onPostUpdate(){
        if(this.getStatus().equals("modifyMember")) {
            MemberModified memberModified = new MemberModified();
            BeanUtils.copyProperties(this, memberModified);
            memberModified.publishAfterCommit();
        }

        if(this.getStatus().equals("deleteMember")) {
            MemberDeleted memberDeleted = new MemberDeleted();
            BeanUtils.copyProperties(this, memberDeleted);
            memberDeleted.publishAfterCommit();
        }

        if(this.getStatus().equals("useMileage")) {
            MileageUsed mileageUsed = new MileageUsed();
            BeanUtils.copyProperties(this, mileageUsed);
            mileageUsed.publishAfterCommit();
        }

        if(this.getStatus().equals("restoreMileage")) {
            MileageRestored mileageRestored = new MileageRestored();
            BeanUtils.copyProperties(this, mileageRestored);
            mileageRestored.publishAfterCommit();
        }

        if(this.getStatus().equals("addMileage")) {
            MileageAdded mileageAdded = new MileageAdded();
            BeanUtils.copyProperties(this, mileageAdded);
            mileageAdded.publishAfterCommit();
        }
    }


    public Long getMemId() {
        return memId;
    }

    public void setMemId(Long memId) {
        this.memId = memId;
    }
    public Long getMileage() {
        return mileage;
    }

    public void setMileage(Long mileage) {
        this.mileage = mileage;
    }
    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }
}

```
- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다
```
package airbnb;

import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel="members", path="members")
public interface MemberRepository extends PagingAndSortingRepository<Member, Long>{

}
```
- 적용 후 REST API 의 테스트
```
# 회원 (member) 서비스의 회원 등록
http POST http://a3db54e965c174fd1b51680316cebdda-58724595.ap-northeast-2.elb.amazonaws.com:8080/members mileage=10000 status="New Registered"

# 제휴상품 (affiliateproducts) 서비스의 제휴상품 등록
http POST http://a3db54e965c174fd1b51680316cebdda-58724595.ap-northeast-2.elb.amazonaws.com:8080/affiliateproducts prdNm="Acyivity ##1" qty=1000 desc="스노클링"

# 제휴상품 주문(order) 서비스의 제휴상품 주문 등록
http POST http://a3db54e965c174fd1b51680316cebdda-58724595.ap-northeast-2.elb.amazonaws.com:8080/orders prdId=1 qty=10 memId=1 mileageUsed=100 status=reqOrdered  

```

## 동기식 호출(Sync) 과 Fallback 처리

### 개인 과제
  제휴상품 주문 (Order) -> 결제(payment) 서비스에 대해 동기식으로 처리
  
  결제(payment) -> 회원 (Member)에 마일리지 사용 적용에 대해 동기식으로 처리
![동기식 호출(Sync) 과 Fallback 처리_마일리지 사용 적용](https://user-images.githubusercontent.com/38099203/121385738-568ed780-c984-11eb-999f-2cfb965b4b5e.png)

  결제(payment) 취소 시 -> 회원 (Member)에 마일리지 사용 적용 취소에 대해 동기식으로 처리
![동기식 호출(Sync) 과 Fallback 처리_마일리지 사용 취소 적용](https://user-images.githubusercontent.com/38099203/121386288-d4eb7980-c984-11eb-88b5-b3eaa52c09bc.png)


```
결제(payment)에서 마일리지 사용 적용 취소 예문

# MemberService.java

package airbnb.external;

<import문 생략>

@FeignClient(name="member", url="${prop.room.url}")
public interface MemberService {

    @RequestMapping(method= RequestMethod.GET, path="/members/chkMileage")
    public long chkMileage(@RequestParam("memId") long memId);

}

# Payment.java

package airbnb.external;

<import문 생략>

    @PostPersist
    public void onPostPersist(){
        ////////////////////////////
        // 결제 승인 된 경우
        ////////////////////////////

        //////////////////////////////
        // mileage 차감 진행 (POST방식)
        //////////////////////////////

        long mileage = PaymentApplication.applicationContext.getBean(airbnb.external.MemberService.class).chkMileage(this.getMemId());

        airbnb.external.Member member = new airbnb.external.Member();
        member.setMemId(this.getMemId());
        member.setMileage(mileage - this.getMileageUsed()); // mileage 감소
        member.setStatus("useMileage");
        PaymentApplication.applicationContext.getBean(airbnb.external.MemberService.class).useMileage(member);

        // 이벤트 발행 -> PaymentApproved
        PaymentApproved paymentApproved = new PaymentApproved();
        BeanUtils.copyProperties(this, paymentApproved);
        paymentApproved.publishAfterCommit();


    }

    @PostUpdate
    public void onPostUpdate(){

        //////////////////////
        // 결제 취소 된 경우
        //////////////////////

        //////////////////////////////
        // mileage 원복 진행 (POST방식)
        //////////////////////////////
        long mileage = PaymentApplication.applicationContext.getBean(airbnb.external.MemberService.class).chkMileage(this.getMemId());

        airbnb.external.Member member = new airbnb.external.Member();
        member.setMemId(this.getMemId());
        member.setMileage(mileage + this.getMileageUsed()); // mileage 증가
        member.setStatus("restoreMileage");
        PaymentApplication.applicationContext.getBean(airbnb.external.MemberService.class).restoreMileage(member);

        // 이벤트 발행 -> PaymentCancelled
        PaymentCancelled paymentCancelled = new PaymentCancelled();
        BeanUtils.copyProperties(this, paymentCancelled);
        paymentCancelled.publishAfterCommit();

    }


```


### 팀 프로젝트

분석 단계에서의 조건 중 하나로 예약 시 숙소(room) 간의 예약 가능 상태 확인 호출은 동기식 일관성을 유지하는 트랜잭션으로 처리하기로 하였다. 호출 프로토콜은 이미 앞서 Rest Repository 에 의해 노출되어있는 REST 서비스를 FeignClient 를 이용하여 호출하도록 한다. 또한 예약(reservation) -> 결제(payment) 서비스도 동기식으로 처리하기로 하였다.

- 룸, 결제 서비스를 호출하기 위하여 Stub과 (FeignClient) 를 이용하여 Service 대행 인터페이스 (Proxy) 를 구현 

```
# PaymentService.java

package airbnb.external;

<import문 생략>

@FeignClient(name="Payment", url="${prop.room.url}")
public interface PaymentService {

    @RequestMapping(method= RequestMethod.POST, path="/payments")
    public void approvePayment(@RequestBody Payment payment);

}

# RoomService.java

package airbnb.external;

<import문 생략>

@FeignClient(name="Room", url="${prop.room.url}")
public interface RoomService {

    @RequestMapping(method= RequestMethod.GET, path="/check/chkAndReqReserve")
    public boolean chkAndReqReserve(@RequestParam("roomId") long roomId);

}


```

- 예약 요청을 받은 직후(@PostPersist) 가능상태 확인 및 결제를 동기(Sync)로 요청하도록 처리
```
# Reservation.java (Entity)

    @PostPersist
    public void onPostPersist(){

        ////////////////////////////////
        // RESERVATION에 INSERT 된 경우 
        ////////////////////////////////

        ////////////////////////////////////
        // 예약 요청(reqReserve) 들어온 경우
        ////////////////////////////////////

        // 해당 ROOM이 Available한 상태인지 체크
        boolean result = ReservationApplication.applicationContext.getBean(airbnb.external.RoomService.class)
                        .chkAndReqReserve(this.getRoomId());
        System.out.println("######## Check Result : " + result);

        if(result) { 

            // 예약 가능한 상태인 경우(Available)

            //////////////////////////////
            // PAYMENT 결제 진행 (POST방식) - SYNC 호출
            //////////////////////////////
            airbnb.external.Payment payment = new airbnb.external.Payment();
            payment.setRsvId(this.getRsvId());
            payment.setRoomId(this.getRoomId());
            payment.setStatus("paid");
            ReservationApplication.applicationContext.getBean(airbnb.external.PaymentService.class)
                .approvePayment(payment);

            /////////////////////////////////////
            // 이벤트 발행 --> ReservationCreated
            /////////////////////////////////////
            ReservationCreated reservationCreated = new ReservationCreated();
            BeanUtils.copyProperties(this, reservationCreated);
            reservationCreated.publishAfterCommit();
        }
    }
```

- 동기식 호출에서는 호출 시간에 따른 타임 커플링이 발생하며, 결제 시스템이 장애가 나면 주문도 못받는다는 것을 확인:


```
# 결제 (pay) 서비스를 잠시 내려놓음 (ctrl+c)

# 예약 요청
http POST http://localhost:8088/reservations roomId=1 status=reqReserve   #Fail

# 결제서비스 재기동
cd payment
mvn spring-boot:run

# 예약 요청
http POST http://localhost:8088/reservations roomId=1 status=reqReserve   #Success
```

- 또한 과도한 요청시에 서비스 장애가 도미노 처럼 벌어질 수 있다. (서킷브레이커, 폴백 처리는 운영단계에서 설명한다.)




## 비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트

### 개인 과제

제휴상품에 대해 결제가 이루어진 후에 제휴상품 시스템 (Affiliateproducts)의 상태가 업데이트
![비동기처리_주문](https://user-images.githubusercontent.com/38099203/121388782-bab29b00-c986-11eb-9a10-30762d96f105.png)
![비동기처리_상품](https://user-images.githubusercontent.com/38099203/121388976-e5045880-c986-11eb-9a01-24489a9e808b.png)

제휴상품 주문 시스템 (Order)의 상태 업데이트
![비동기처리_주문 상태 업데이트](https://user-images.githubusercontent.com/38099203/121389171-1a10ab00-c987-11eb-86db-a8bd99aa53ef.png)

주문 및 취소 메시지가 전송되는 시스템과의 통신 행위
![비동기처리_주문 메시지](https://user-images.githubusercontent.com/38099203/121392410-5396e580-c98a-11eb-8e2c-9a975854c850.png)
![비동기처리_주문 취소 메시지](https://user-images.githubusercontent.com/38099203/121395962-de2d1400-c98d-11eb-8680-537348c5313e.png)


리뷰 작성 시 마일리지 부여 
![비동기처리_리뷰 작성 시 마일리지 부여 ](https://user-images.githubusercontent.com/38099203/121396471-5e537980-c98e-11eb-9505-12a6c0434c35.png)


### 팀 프로젝트

결제가 이루어진 후에 숙소 시스템의 상태가 업데이트 되고, 예약 시스템의 상태가 업데이트 되며, 예약 및 취소 메시지가 전송되는 시스템과의 통신 행위는 비동기식으로 처리한다.
 
- 이를 위하여 결제가 승인되면 결제가 승인 되었다는 이벤트를 카프카로 송출한다. (Publish)
 
```
# Payment.java

package airbnb;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;

@Entity
@Table(name="Payment_table")
public class Payment {

    ....

    @PostPersist
    public void onPostPersist(){
        ////////////////////////////
        // 결제 승인 된 경우
        ////////////////////////////

        // 이벤트 발행 -> PaymentApproved
        PaymentApproved paymentApproved = new PaymentApproved();
        BeanUtils.copyProperties(this, paymentApproved);
        paymentApproved.publishAfterCommit();
    }
    
    ....
}
```

- 예약 시스템에서는 결제 승인 이벤트에 대해서 이를 수신하여 자신의 정책을 처리하도록 PolicyHandler 를 구현한다:

```
# Reservation.java

package airbnb;

    @PostUpdate
    public void onPostUpdate(){
    
        ....

        if(this.getStatus().equals("reserved")) {

            ////////////////////
            // 예약 확정된 경우
            ////////////////////

            // 이벤트 발생 --> ReservationConfirmed
            ReservationConfirmed reservationConfirmed = new ReservationConfirmed();
            BeanUtils.copyProperties(this, reservationConfirmed);
            reservationConfirmed.publishAfterCommit();
        }
        
        ....
        
    }

```

그 외 메시지 서비스는 예약/결제와 완전히 분리되어있으며, 이벤트 수신에 따라 처리되기 때문에, 메시지 서비스가 유지보수로 인해 잠시 내려간 상태 라도 예약을 받는데 문제가 없다.

```
# 메시지 서비스 (message) 를 잠시 내려놓음 (ctrl+c)

# 예약 요청
http POST http://localhost:8088/reservations roomId=1 status=reqReserve   #Success

# 예약 상태 확인
http GET localhost:8088/reservations    #메시지 서비스와 상관없이 예약 상태는 정상 확인

```

# 운영


## CI/CD 설정

### 개인 과제
- 회원(Member) 제휴상품 주문 (Order), 제휴상품 (Affiliateproduct)에 대해 CodeBuild 프로젝트를 생성

```
    CodeBuild 프로젝트 생성
	AWS_ACCOUNT_ID 환경 변수 세팅 (ERC 레파지토리 room에 대한 푸시 명령)
	KUBE_URL 환경 변수 세팅 (EKS 클러스터 API 서버 엔드포인트)
	KUBE_TOKEN 환경 변수 세팅
		kubectl apply -f eks-admin-service-account.yml
		kubectl apply -f eks-admin-cluster-role-binding.yml
		kubectl -n kube-system get secret
			eks-admin-token-rjpmq                            kubernetes.io/service-account-token   3      2m5s
		kubectl -n kube-system describe secret eks-admin-token-rjpmq

```
- codebuild 실행
```
codebuild 프로젝트 및 빌드 이력
```
![codebuild](https://user-images.githubusercontent.com/38099203/121400497-9fe62380-c992-11eb-9163-afb564a24c3b.PNG)

- 회원(Member) codebuild 빌드 내역
![Member 빌드기록](https://user-images.githubusercontent.com/38099203/121402253-9e1d5f80-c994-11eb-95f5-e2315dc5ab20.PNG)
![Member 빌드로그](https://user-images.githubusercontent.com/38099203/121404296-cc03a380-c996-11eb-88b3-ffc0da6dcce1.PNG)

- 제휴상품 주문 (Order) codebuild 빌드 내역
![Order 빌드기록](https://user-images.githubusercontent.com/38099203/121404633-15ec8980-c997-11eb-889f-22adb00b10b7.PNG)
![Order 빌드로그](https://user-images.githubusercontent.com/38099203/121404771-3caac000-c997-11eb-91ad-40c889530d0e.PNG)

- 제휴상품 (Affiliateproduct) codebuild 빌드 내역
![Product 빌드기록](https://user-images.githubusercontent.com/38099203/121405665-48e34d00-c998-11eb-9d43-998fbb1dc5e3.PNG)
![Product 빌드로그](https://user-images.githubusercontent.com/38099203/121405698-526cb500-c998-11eb-9058-ba19cff1b9cb.PNG)


### 팀 프로젝트
각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD는 buildspec.yml을 이용한 AWS codebuild를 사용하였습니다.

- CodeBuild 프로젝트를 생성하고 AWS_ACCOUNT_ID, KUBE_URL, KUBE_TOKEN 환경 변수 세팅을 한다
```
SA 생성
kubectl apply -f eks-admin-service-account.yml
```
![codebuild(sa)](https://user-images.githubusercontent.com/38099203/119293259-ff52ec80-bc8c-11eb-8671-b9a226811762.PNG)
```
Role 생성
kubectl apply -f eks-admin-cluster-role-binding.yml
```
![codebuild(role)](https://user-images.githubusercontent.com/38099203/119293300-1abdf780-bc8d-11eb-9b07-ad173237efb1.PNG)
```
Token 확인
kubectl -n kube-system get secret
kubectl -n kube-system describe secret eks-admin-token-rjpmq
```
![codebuild(token)](https://user-images.githubusercontent.com/38099203/119293511-84d69c80-bc8d-11eb-99c7-e8929e6a41e4.PNG)
```
buildspec.yml 파일 
마이크로 서비스 room의 yml 파일 이용하도록 세팅
```
![codebuild(buildspec)](https://user-images.githubusercontent.com/38099203/119283849-30292680-bc79-11eb-9f86-cbb715e74846.PNG)

- codebuild 실행
```
codebuild 프로젝트 및 빌드 이력
```
![codebuild(프로젝트)](https://user-images.githubusercontent.com/38099203/119283851-315a5380-bc79-11eb-9b2a-b4522d22d009.PNG)
![codebuild(로그)](https://user-images.githubusercontent.com/38099203/119283850-30c1bd00-bc79-11eb-9547-1ff1f62e48a4.PNG)

- codebuild 빌드 내역 (Message 서비스 세부)

![image](https://user-images.githubusercontent.com/31723044/119385500-2b0fba00-bd01-11eb-861b-cc31910ff945.png)

- codebuild 빌드 내역 (전체 이력 조회)

![image](https://user-images.githubusercontent.com/31723044/119385401-087da100-bd01-11eb-8b69-ce222e6bb71e.png)




## 동기식 호출 / 서킷 브레이킹 / 장애격리

* 서킷 브레이킹 프레임워크의 선택: istio 사용하여 구현함

### 개인 과제
- CB 관련 설치

```
	# metric server 설치
	kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml

	# helm, kafka 설치
	https://github.com/helm/helm/releases
	kubectl --namespace kube-system create sa tiller
	kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
	helm repo add incubator https://charts.helm.sh/incubator
	helm repo update
	kubectl create ns kafka
	helm install my-kafka --namespace kafka incubator/kafka
	
	# istio 설치
	https://github.com/istio/istio/releases/download/1.7.1/istio-1.7.1-linux-amd64.tar.gz
	https://github.com/istio/istio/releases/tag/1.7.1
	istioctl install --set profile=demo --set hub=gcr.io/istio-release
	kubectl apply -f samples/addons
	
	# kiali 설치 및 service type 변경
	kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/kiali.yaml
	kubectl edit svc kiali -n istio-system (ClusterIP -> LoadBalancer)
		aa61caf40a8fd45668ab8cdf40f761d9-207959624.ap-northeast-2.elb.amazonaws.com:20001
	kubectl edit svc tracing -n istio-system (ClusterIP -> LoadBalancer)
		a0b5c6ea41b8147c2be1b134e4dd6611-711153361.ap-northeast-2.elb.amazonaws.com
	kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.8/samples/addons/prometheus.yaml

```

-- 제휴상품 (Affiliateproduct)에 DestinationRule 를 생성하고 최소 connection pool 설정으로 circuit break 가 발생할 수 있도록 함
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: dr-affiliateproduct
  namespace: airbnb
spec:
  host: affiliateproduct
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
#    outlierDetection:
#      interval: 1s
#      consecutiveErrors: 1
#      baseEjectionTime: 10s
#      maxEjectionPercent: 100

# buildspec.yml 파일에 destination-rule.yml 추가
    commands:
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$_PROJECT_NAME:latest
      - echo connect kubectl
      - kubectl config set-cluster k8s --server="$KUBE_URL" --insecure-skip-tls-verify=true
      - kubectl config set-credentials admin --token="$KUBE_TOKEN"
      - kubectl config set-context default --cluster=k8s --user=admin
      - kubectl config use-context default
      - kubectl replace -f kubernetes/deployment.yml --force
      - kubectl replace -f kubernetes/service.yaml --force
      - kubectl replace  -f kubernetes/destination-rule.yml --force
```

* istio-injection 활성화 및 affiliateproduct pod container 확인

```
kubectl get ns -L istio-injection
kubectl label namespace airbnb istio-injection=enabled 
```
![CB_istio inject](https://user-images.githubusercontent.com/38099203/121407794-a8daf300-c99a-11eb-8f2c-2f651567e761.PNG)
![CB_istio product pod 2개](https://user-images.githubusercontent.com/38099203/121408604-8e554980-c99b-11eb-849b-69889d4274be.PNG)

* 부하테스터 siege 툴을 통한 서킷 브레이커 동작 확인:

siege 실행

```
kubectl run siege --image=apexacme/siege-nginx -n airbnb
kubectl exec -it siege -c siege -n airbnb -- /bin/bash
```


- 동시사용자 1로 부하 생성 시 모두 정상
```
siege -c1 -t10S -v --content-type "application/json" 'http://affiliateproduct:8080/affiliateproducts POST {"prdNm": "Acyivity ####", "qty": 1000, "desc": "스노클링"}'

HTTP/1.1 201     0.00 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.02 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.00 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.00 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts

Lifting the server siege...
Transactions:                   1051 hits
Availability:                 100.00 %
Elapsed time:                   9.63 secs
Data transferred:               0.31 MB
Response time:                  0.01 secs
Transaction rate:             109.14 trans/sec
Throughput:                     0.03 MB/sec
Concurrency:                    0.96
Successful transactions:        1051
Failed transactions:               0
Longest transaction:            0.04
```

- 동시사용자 2로 부하 생성 시 503 에러 1024개 발생
```
siege -c2 -t10S -v --content-type "application/json" 'http://affiliateproduct:8080/affiliateproducts POST {"prdNm": "Acyivity ####", "qty": 1000, "desc": "스노클링"}'

HTTP/1.1 201     0.02 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.00 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 503     0.01 secs:      81 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 503     0.00 secs:      81 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.02 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
siege aborted due to excessive socket failure; you
can change the failure threshold in $HOME/.siegerc

Transactions:                   6437 hits
Availability:                  86.28 %
Elapsed time:                  53.09 secs
Data transferred:               1.96 MB
Response time:                  0.02 secs
Transaction rate:             121.25 trans/sec
Throughput:                     0.04 MB/sec
Concurrency:                    1.91
Successful transactions:        6437
Failed transactions:            1024
Longest transaction:            0.69
Shortest transaction:           0.00
```

- kiali 화면에 서킷 브레이크 확인

![CB_kiali](https://user-images.githubusercontent.com/38099203/121415849-1b4fd100-c9a3-11eb-8a19-ee9ab3971126.PNG)


- 다시 최소 Connection pool로 부하 다시 정상 확인

```
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.02 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.01 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts

Lifting the server siege...
Transactions:                    941 hits
Availability:                 100.00 %
Elapsed time:                   9.89 secs
Data transferred:               0.27 MB
Response time:                  0.01 secs
Transaction rate:              95.15 trans/sec
Throughput:                     0.03 MB/sec
Concurrency:                    0.96
Successful transactions:         941
Failed transactions:               0
Longest transaction:            0.04
Shortest transaction:           0.00

```


### 팀 프로젝트

시나리오는 예약(reservation)--> 룸(room) 시의 연결을 RESTful Request/Response 로 연동하여 구현이 되어있고, 예약 요청이 과도할 경우 CB 를 통하여 장애격리.

- DestinationRule 를 생성하여 circuit break 가 발생할 수 있도록 설정
최소 connection pool 설정
```
# destination-rule.yml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: dr-room
  namespace: airbnb
spec:
  host: room
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
#    outlierDetection:
#      interval: 1s
#      consecutiveErrors: 1
#      baseEjectionTime: 10s
#      maxEjectionPercent: 100
```

* istio-injection 활성화 및 room pod container 확인

```
kubectl get ns -L istio-injection
kubectl label namespace airbnb istio-injection=enabled 
```

![Circuit Breaker(istio-enjection)](https://user-images.githubusercontent.com/38099203/119295450-d6812600-bc91-11eb-8aad-46eeac968a41.PNG)
![Circuit Breaker(pod)](https://user-images.githubusercontent.com/38099203/119295568-0cbea580-bc92-11eb-9d2b-8580f3576b47.PNG)


* 부하테스터 siege 툴을 통한 서킷 브레이커 동작 확인:

siege 실행

```
kubectl run siege --image=apexacme/siege-nginx -n airbnb
kubectl exec -it siege -c siege -n airbnb -- /bin/bash
```


- 동시사용자 1로 부하 생성 시 모두 정상
```
siege -c1 -t10S -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'

** SIEGE 4.0.4
** Preparing 1 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.49 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.05 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     254 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     256 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     256 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     256 bytes ==> POST http://room:8080/rooms
```

- 동시사용자 2로 부하 생성 시 503 에러 168개 발생
```
siege -c2 -t10S -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'

** SIEGE 4.0.4
** Preparing 2 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.10 secs:      81 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.04 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.05 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.22 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.08 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.07 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.01 secs:      81 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.01 secs:      81 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     258 bytes ==> POST http://room:8080/rooms
HTTP/1.1 503     0.00 secs:      81 bytes ==> POST http://room:8080/rooms

Lifting the server siege...
Transactions:                   1904 hits
Availability:                  91.89 %
Elapsed time:                   9.89 secs
Data transferred:               0.48 MB
Response time:                  0.01 secs
Transaction rate:             192.52 trans/sec
Throughput:                     0.05 MB/sec
Concurrency:                    1.98
Successful transactions:        1904
Failed transactions:             168
Longest transaction:            0.03
Shortest transaction:           0.00
```

- kiali 화면에 서킷 브레이크 확인

![Circuit Breaker(kiali)](https://user-images.githubusercontent.com/38099203/119298194-7f7e4f80-bc97-11eb-8447-678eece29e5c.PNG)


- 다시 최소 Connection pool로 부하 다시 정상 확인

```
** SIEGE 4.0.4
** Preparing 1 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms

:
:

Lifting the server siege...
Transactions:                   1139 hits
Availability:                 100.00 %
Elapsed time:                   9.19 secs
Data transferred:               0.28 MB
Response time:                  0.01 secs
Transaction rate:             123.94 trans/sec
Throughput:                     0.03 MB/sec
Concurrency:                    0.98
Successful transactions:        1139
Failed transactions:               0
Longest transaction:            0.04
Shortest transaction:           0.00

```

- 운영시스템은 죽지 않고 지속적으로 CB 에 의하여 적절히 회로가 열림과 닫힘이 벌어지면서 자원을 보호하고 있음을 보여줌.
  virtualhost 설정과 동적 Scale out (replica의 자동적 추가,HPA) 을 통하여 시스템을 확장 해주는 후속처리가 필요.


### 오토스케일 아웃

#### 개인 과제
- affiliateproduct 서비스의 deployment.yml 파일에 resources 설정을 추가하고 배포한다
![auto scale (1)](https://user-images.githubusercontent.com/38099203/121416903-4e469480-c9a4-11eb-92d4-9f8f95a0e677.PNG)

- affiliateproduct 서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 30프로를 넘어서면 replica 를 10개까지 늘려준다:
```
kubectl autoscale deployment affiliateproduct -n airbnb --cpu-percent=30 --min=1 --max=10
C:\my_project\airbnb_affiliateproduct>kubectl get hpa -n airbnb
NAME               REFERENCE                     TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
affiliateproduct   Deployment/affiliateproduct   <unknown>/30%   1         10        0          11s
```
- 부하를 동시사용자 100명, 1분 동안 걸어준다.
```
siege -c100 -t60S -r10 -v --content-type "application/json" 'http://affiliateproduct:8080/affiliateproducts POST {"prdNm": "Acyivity ####", "qty": 1000, "desc": "스노클링"}'
```
- 오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다
```
C:\my_project\airbnb_affiliateproduct>kubectl get deploy affiliateproduct -w -n airbnb
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
affiliateproduct   1/1     1            1           118s
affiliateproduct   1/2     1            1           2m14s
affiliateproduct   1/2     1            1           2m14s
affiliateproduct   1/2     1            1           2m14s
affiliateproduct   1/2     2            1           2m14s
affiliateproduct   2/2     2            2           2m46s
affiliateproduct   2/3     2            2           4m16s
affiliateproduct   2/3     2            2           4m16s
affiliateproduct   2/3     2            2           4m16s
affiliateproduct   2/3     3            2           4m16s
```


#### 팀 프로젝트
앞서 CB 는 시스템을 안정되게 운영할 수 있게 해줬지만 사용자의 요청을 100% 받아들여주지 못했기 때문에 이에 대한 보완책으로 자동화된 확장 기능을 적용하고자 한다. 

- room deployment.yml 파일에 resources 설정을 추가한다
![Autoscale (HPA)](https://user-images.githubusercontent.com/38099203/119283787-0a038680-bc79-11eb-8d9b-d8aed8847fef.PNG)

- room 서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 50프로를 넘어서면 replica 를 10개까지 늘려준다:
```
kubectl autoscale deployment room -n airbnb --cpu-percent=50 --min=1 --max=10
```
![Autoscale (HPA)(kubectl autoscale 명령어)](https://user-images.githubusercontent.com/38099203/119299474-ec92e480-bc99-11eb-9bc3-8c5246b02783.PNG)

- 부하를 동시사용자 100명, 1분 동안 걸어준다.
```
siege -c100 -t60S -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'
```
- 오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다
```
kubectl get deploy room -w -n airbnb 
```
- 어느정도 시간이 흐른 후 (약 30초) 스케일 아웃이 벌어지는 것을 확인할 수 있다:
![Autoscale (HPA)(모니터링)](https://user-images.githubusercontent.com/38099203/119299704-6a56f000-bc9a-11eb-9ba8-55e5978f3739.PNG)

- siege 의 로그를 보아도 전체적인 성공률이 높아진 것을 확인 할 수 있다. 
```
Lifting the server siege...
Transactions:                  15615 hits
Availability:                 100.00 %
Elapsed time:                  59.44 secs
Data transferred:               3.90 MB
Response time:                  0.32 secs
Transaction rate:             262.70 trans/sec
Throughput:                     0.07 MB/sec
Concurrency:                   85.04
Successful transactions:       15675
Failed transactions:               0
Longest transaction:            2.55
Shortest transaction:           0.01
```

## 무정지 재배포

#### 개인 과제

```
# 초기화 
kubectl delete destinationrules dr-affiliateproduct -n airbnb
kubectl label namespace airbnb istio-injection-
kubectl delete hpa affiliateproduct -n airbnb

# deployment.yml 파일에 readinessProbe, livenessProbe 주석처리 후 배포
#          resources:
#            requests:
#              memory: "256Mi"
#              cpu: "1000m"
#            limits:
#              memory: "512Mi"
#              cpu: "2500m"
#          args:
#            - /bin/sh
#            - -c
#            - touch /tmp/healthy; sleep 90; rm -rf /tmp/healthy; sleep 600
#          readinessProbe:
#            httpGet:
#              path: '/actuator/health'
#              port: 8080
#            exec:
#              command:
#              - cat
#              - /tmp/healthy
#            initialDelaySeconds: 25
#            timeoutSeconds: 2
#            periodSeconds: 5
#            failureThreshold: 5
#            successThreshold: 1
#          livenessProbe:
#            httpGet:
#              path: '/actuator/health'
#              port: 8080
#            exec:
#              command:
#              - cat
#              - /tmp/healthy
#            initialDelaySeconds: 60
#            timeoutSeconds: 2
#            periodSeconds: 5
#            failureThreshold: 5
#            successThreshold: 1
```

- seige 로 배포작업 직전에 워크로드를 모니터링 함.
```
siege -c100 -t60S -r10 -v --content-type "application/json" 'http://affiliateproduct:8080/affiliateproducts POST {"prdNm": "Acyivity ####", "qty": 1000, "desc": "스노클링"}'

** SIEGE 4.0.4
** Preparing 1 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.51 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.03 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.29 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.10 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     1.13 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.38 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.02 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.04 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.03 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.30 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.06 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.77 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.70 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.15 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.28 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.44 secs:     306 bytes ==> POST http://affiliateproduct:8080/affiliateproducts
HTTP/1.1 201     0.20 secs:     308 bytes ==> POST http://affiliateproduct:8080/affiliateproducts

```

- 새버전으로의 배포 시작
```
kubectl set image ......
```

- seige 의 화면으로 넘어가서 Availability 가 100% 미만으로 떨어졌는지 확인

```

Transactions:                   7732 hits
Availability:                  87.32 %
Elapsed time:                  17.12 secs
Data transferred:               1.93 MB
Response time:                  0.18 secs
Transaction rate:             451.64 trans/sec
Throughput:                     0.11 MB/sec
Concurrency:                   81.21
Successful transactions:        7732
Failed transactions:            1123
Longest transaction:            0.94
Shortest transaction:           0.00

```
- 배포기간중 Availability 가 평소 100%에서 87% 대로 떨어지는 것을 확인. 원인은 쿠버네티스가 성급하게 새로 올려진 서비스를 READY 상태로 인식하여 서비스 유입을 진행한 것이기 때문. 이를 막기위해 Readiness Probe 를 설정함

```
# deployment.yaml 의 readiness probe 의 설정:

      containers:
        - name: affiliateproduct
          image: 985702435631.dkr.ecr.ap-northeast-2.amazonaws.com/affiliateproduct:latest
          ports:
            - containerPort: 8080
#          resources:
#            requests:
#              memory: "256Mi"
#              cpu: "1000m"
#            limits:
#              memory: "512Mi"
#              cpu: "2500m"
#          args:
#            - /bin/sh
#            - -c
#            - touch /tmp/healthy; sleep 90; rm -rf /tmp/healthy; sleep 600
          readinessProbe:
            httpGet:
              path: '/actuator/health'
              port: 8080
#            exec:
#              command:
#              - cat
#              - /tmp/healthy
            initialDelaySeconds: 25
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 5
            successThreshold: 1

kubectl apply -f kubernetes/deployment.yml
```

- 동일한 시나리오로 재배포 한 후 Availability 확인:
```
Lifting the server siege...
Transactions:                  27657 hits
Availability:                 100.00 %
Elapsed time:                  59.41 secs
Data transferred:               6.91 MB
Response time:                  0.21 secs
Transaction rate:             465.53 trans/sec
Throughput:                     0.12 MB/sec
Concurrency:                   99.60
Successful transactions:       27657
Failed transactions:               0
Longest transaction:            1.20
Shortest transaction:           0.00

```

배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.


#### 팀 프로젝트

* 먼저 무정지 재배포가 100% 되는 것인지 확인하기 위해서 Autoscaler 이나 CB 설정을 제거함

```
kubectl delete destinationrules dr-room -n airbnb
kubectl label namespace airbnb istio-injection-
kubectl delete hpa room -n airbnb
```

- seige 로 배포작업 직전에 워크로드를 모니터링 함.
```
siege -c100 -t60S -r10 -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'

** SIEGE 4.0.4
** Preparing 1 concurrent users for battle.
The server is now under siege...
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.03 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.00 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.02 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms
HTTP/1.1 201     0.01 secs:     260 bytes ==> POST http://room:8080/rooms

```

- 새버전으로의 배포 시작
```
kubectl set image ...
```

- seige 의 화면으로 넘어가서 Availability 가 100% 미만으로 떨어졌는지 확인

```
siege -c100 -t60S -r10 -v --content-type "application/json" 'http://room:8080/rooms POST {"desc": "Beautiful House3"}'


Transactions:                   7732 hits
Availability:                  87.32 %
Elapsed time:                  17.12 secs
Data transferred:               1.93 MB
Response time:                  0.18 secs
Transaction rate:             451.64 trans/sec
Throughput:                     0.11 MB/sec
Concurrency:                   81.21
Successful transactions:        7732
Failed transactions:            1123
Longest transaction:            0.94
Shortest transaction:           0.00

```
- 배포기간중 Availability 가 평소 100%에서 87% 대로 떨어지는 것을 확인. 원인은 쿠버네티스가 성급하게 새로 올려진 서비스를 READY 상태로 인식하여 서비스 유입을 진행한 것이기 때문. 이를 막기위해 Readiness Probe 를 설정함

```
# deployment.yaml 의 readiness probe 의 설정:
```

![probe설정](https://user-images.githubusercontent.com/38099203/119301424-71333200-bc9d-11eb-9f75-f8c98fce70a3.PNG)

```
kubectl apply -f kubernetes/deployment.yml
```

- 동일한 시나리오로 재배포 한 후 Availability 확인:
```
Lifting the server siege...
Transactions:                  27657 hits
Availability:                 100.00 %
Elapsed time:                  59.41 secs
Data transferred:               6.91 MB
Response time:                  0.21 secs
Transaction rate:             465.53 trans/sec
Throughput:                     0.12 MB/sec
Concurrency:                   99.60
Successful transactions:       27657
Failed transactions:               0
Longest transaction:            1.20
Shortest transaction:           0.00

```

배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.


# Self-healing (Liveness Probe)

## 개인 과제
- affiliateproduct deployment.yml 파일 수정 
```
콘테이너 실행 후 /tmp/healthy 파일을 만들고 
90초 후 삭제
livenessProbe에 'cat /tmp/healthy'으로 검증하도록 함

          livenessProbe:
#            httpGet:
#              path: '/actuator/health'
#              port: 8080
            exec:
              command:
              - cat
              - /tmp/healthy
            initialDelaySeconds: 90
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 5
            successThreshold: 1

```
- kubectl describe pod affiliateproduct -n airbnb 실행으로 확인
```
컨테이너 실행 후 90초 동인은 정상이나 이후 /tmp/healthy 파일이 삭제되어 livenessProbe에서 실패를 리턴하게 됨
pod 정상 상태 일때 pod 진입하여 /tmp/healthy 파일 생성해주면 정상 상태 유지됨
```
![Liveness(1)](https://user-images.githubusercontent.com/38099203/121430113-a8e6ed00-c9b2-11eb-948e-11932f121d3b.PNG)
![Liveness(2)](https://user-images.githubusercontent.com/38099203/121430152-b56b4580-c9b2-11eb-99fa-573213dcec13.PNG)




## 팀 프로젝트
- room deployment.yml 파일 수정 
```
콘테이너 실행 후 /tmp/healthy 파일을 만들고 
90초 후 삭제
livenessProbe에 'cat /tmp/healthy'으로 검증하도록 함
```
![deployment yml tmp healthy](https://user-images.githubusercontent.com/38099203/119318677-8ff0f300-bcb4-11eb-950a-e3c15feed325.PNG)

- kubectl describe pod room -n airbnb 실행으로 확인
```
컨테이너 실행 후 90초 동인은 정상이나 이후 /tmp/healthy 파일이 삭제되어 livenessProbe에서 실패를 리턴하게 됨
pod 정상 상태 일때 pod 진입하여 /tmp/healthy 파일 생성해주면 정상 상태 유지됨
```


![get pod tmp healthy](https://user-images.githubusercontent.com/38099203/119318781-a9923a80-bcb4-11eb-9783-65051ec0d6e8.PNG)
![touch tmp healthy](https://user-images.githubusercontent.com/38099203/119319050-f118c680-bcb4-11eb-8bca-aa135c1e067e.PNG)





# Config Map/ Persistence Volume
## 개인 과제
- Persistence Volume

1: EFS 생성
```
EFS 생성 시 클러스터의 VPC를 선택해야함
```
![클러스터의 VPC를 선택해야함](https://user-images.githubusercontent.com/38099203/119364089-85048580-bce9-11eb-8001-1c20a93b8e36.PNG)

![EFS생성](https://user-images.githubusercontent.com/38099203/119343415-60041880-bcd1-11eb-9c25-1695c858f6aa.PNG)

2. EFS 계정 생성 및 ROLE 바인딩
```
kubectl apply -f efs-sa.yml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: efs-provisioner
  namespace: airbnb


kubectl get ServiceAccount efs-provisioner -n airbnb
NAME              SECRETS   AGE
efs-provisioner   1         9m1s  
  
  
  
kubectl apply -f efs-rbac.yaml

namespace를 반듯이 수정해야함

  
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: efs-provisioner-runner
  namespace: airbnb
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-efs-provisioner
  namespace: airbnb
subjects:
  - kind: ServiceAccount
    name: efs-provisioner
     # replace with namespace where provisioner is deployed
    namespace: airbnb
roleRef:
  kind: ClusterRole
  name: efs-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-efs-provisioner
  namespace: airbnb
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-efs-provisioner
  namespace: airbnb
subjects:
  - kind: ServiceAccount
    name: efs-provisioner
    # replace with namespace where provisioner is deployed
    namespace: airbnb
roleRef:
  kind: Role
  name: leader-locking-efs-provisioner
  apiGroup: rbac.authorization.k8s.io


```

3. EFS Provisioner 배포
```
kubectl apply -f efs-provisioner-deploy.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: efs-provisioner
  namespace: airbnb
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: efs-provisioner
  template:
    metadata:
      labels:
        app: efs-provisioner
    spec:
      serviceAccount: efs-provisioner
      containers:
        - name: efs-provisioner
          image: quay.io/external_storage/efs-provisioner:latest
          env:
            - name: FILE_SYSTEM_ID
              value: fs-562f9c36
            - name: AWS_REGION
              value: ap-northeast-2
            - name: PROVISIONER_NAME
              value: my-aws.com/aws-efs
          volumeMounts:
            - name: pv-volume
              mountPath: /persistentvolumes
      volumes:
        - name: pv-volume
          nfs:
            server: fs-562f9c36.efs.ap-northeast-2.amazonaws.com
            path: /


kubectl get Deployment efs-provisioner -n airbnb
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
efs-provisioner   1/1     1            1           11m

```

4. 설치한 Provisioner를 storageclass에 등록
```
kubectl apply -f efs-storageclass.yml


kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: aws-efs
  namespace: airbnb
provisioner: my-aws.com/aws-efs


kubectl get sc aws-efs -n airbnb
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
aws-efs         my-aws.com/aws-efs      Delete          Immediate              false                  4s
```

5. PVC(PersistentVolumeClaim) 생성
```
kubectl apply -f volume-pvc.yml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: aws-efs
  namespace: airbnb
  labels:
    app: test-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 6Ki
  storageClassName: aws-efs
  
  
kubectl get pvc aws-efs -n airbnb
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
aws-efs   Bound    pvc-43f6fe12-b9f3-400c-ba20-b357c1639f00   6Ki        RWX            aws-efs        4m44s
```

6. affiliateproduct pod 적용
```
테스트를 위해 replicas: 2로 함

spec:
  replicas: 2
  ......
      spec:
      serviceAccount: efs-provisioner
      containers:
        - name: affiliateproduct
          image: 985702435631.dkr.ecr.ap-northeast-2.amazonaws.com/affiliateproduct:latest
          ports:
            - containerPort: 8080
  ......
          readinessProbe:
            httpGet:
              path: '/actuator/health'
              port: 8080
#            exec:
#              command:
#              - cat
#              - /tmp/healthy
            initialDelaySeconds: 25
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 5
            successThreshold: 1
          livenessProbe:
            httpGet:
              path: '/actuator/health'
              port: 8080
#            exec:
#              command:
#              - cat
#              - /tmp/healthy
            initialDelaySeconds: 60
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 5
            successThreshold: 1
          env:
            - name: MAX_RESERVATION_PER_PERSION
              valueFrom:
                configMapKeyRef:
                  name: airbnb-config
                  key: max_reservation_per_person
            - name: UI_PROPERTIES_FILE_NAME
              valueFrom:
                configMapKeyRef:
                  name: airbnb-config
                  key: ui_properties_file_name
          volumeMounts:
          - mountPath: "/mnt/aws"
            name: volume
      volumes:
        - name: volume
          persistentVolumeClaim:
            claimName: aws-efs
```



7. A pod에서 마운트된 경로에 파일을 생성하고 B pod에서 파일을 확인함
```
NAME                                    READY   STATUS             RESTARTS   AGE
pod/affiliateproduct-7d66cf477-d6r85    1/1     Running            2          5m5s
pod/affiliateproduct-7d66cf477-zlg2t    1/1     Running            2          5m5s

A pod
kubectl exec -it pod/affiliateproduct-7d66cf477-d6r85 affiliateproduct -n airbnb -- /bin/sh

C:\my_project\airbnb_affiliateproduct>kubectl exec -it pod/affiliateproduct-7d66cf477-d6r85 affiliateproduct -n airbnb -- /bin/sh
/ # cd /mnt/aws
/mnt/aws # touch intensive_course_work
/mnt/aws # ls -al
total 8
drwxrws--x    2 root     2000          6144 May 24 15:44 .
drwxr-xr-x    1 root     root            17 Jun  9 21:21 ..
-rw-r--r--    1 root     2000             0 Jun  9 21:26 intensive_course_work
/mnt/aws #
```
```
B pod
kubectl exec -it pod/affiliateproduct-7d66cf477-zlg2t affiliateproduct -n airbnb -- /bin/sh


C:\my_project\airbnb_affiliateproduct\kubernetes>kubectl exec -it pod/affiliateproduct-7d66cf477-zlg2t affiliateproduct -n airbnb -- /bin/sh
/ # cd /mnt/aws
/mnt/aws # ls -al
total 8
drwxrws--x    2 root     2000          6144 May 24 15:44 .
drwxr-xr-x    1 root     root            17 Jun  9 21:21 ..
-rw-r--r--    1 root     2000             0 Jun  9 21:26 intensive_course_work

```





## 팀 프로젝트
- Persistence Volume

1: EFS 생성
```
EFS 생성 시 클러스터의 VPC를 선택해야함
```
![클러스터의 VPC를 선택해야함](https://user-images.githubusercontent.com/38099203/119364089-85048580-bce9-11eb-8001-1c20a93b8e36.PNG)

![EFS생성](https://user-images.githubusercontent.com/38099203/119343415-60041880-bcd1-11eb-9c25-1695c858f6aa.PNG)

2. EFS 계정 생성 및 ROLE 바인딩
```
kubectl apply -f efs-sa.yml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: efs-provisioner
  namespace: airbnb


kubectl get ServiceAccount efs-provisioner -n airbnb
NAME              SECRETS   AGE
efs-provisioner   1         9m1s  
  
  
  
kubectl apply -f efs-rbac.yaml

namespace를 반듯이 수정해야함

  
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: efs-provisioner-runner
  namespace: airbnb
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-efs-provisioner
  namespace: airbnb
subjects:
  - kind: ServiceAccount
    name: efs-provisioner
     # replace with namespace where provisioner is deployed
    namespace: airbnb
roleRef:
  kind: ClusterRole
  name: efs-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-efs-provisioner
  namespace: airbnb
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-efs-provisioner
  namespace: airbnb
subjects:
  - kind: ServiceAccount
    name: efs-provisioner
    # replace with namespace where provisioner is deployed
    namespace: airbnb
roleRef:
  kind: Role
  name: leader-locking-efs-provisioner
  apiGroup: rbac.authorization.k8s.io


```

3. EFS Provisioner 배포
```
kubectl apply -f efs-provisioner-deploy.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: efs-provisioner
  namespace: airbnb
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: efs-provisioner
  template:
    metadata:
      labels:
        app: efs-provisioner
    spec:
      serviceAccount: efs-provisioner
      containers:
        - name: efs-provisioner
          image: quay.io/external_storage/efs-provisioner:latest
          env:
            - name: FILE_SYSTEM_ID
              value: fs-562f9c36
            - name: AWS_REGION
              value: ap-northeast-2
            - name: PROVISIONER_NAME
              value: my-aws.com/aws-efs
          volumeMounts:
            - name: pv-volume
              mountPath: /persistentvolumes
      volumes:
        - name: pv-volume
          nfs:
            server: fs-562f9c36.efs.ap-northeast-2.amazonaws.com
            path: /


kubectl get Deployment efs-provisioner -n airbnb
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
efs-provisioner   1/1     1            1           11m

```

4. 설치한 Provisioner를 storageclass에 등록
```
kubectl apply -f efs-storageclass.yml


kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: aws-efs
  namespace: airbnb
provisioner: my-aws.com/aws-efs


kubectl get sc aws-efs -n airbnb
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
aws-efs         my-aws.com/aws-efs      Delete          Immediate              false                  4s
```

5. PVC(PersistentVolumeClaim) 생성
```
kubectl apply -f volume-pvc.yml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: aws-efs
  namespace: airbnb
  labels:
    app: test-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 6Ki
  storageClassName: aws-efs
  
  
kubectl get pvc aws-efs -n airbnb
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
aws-efs   Bound    pvc-43f6fe12-b9f3-400c-ba20-b357c1639f00   6Ki        RWX            aws-efs        4m44s
```

6. room pod 적용
```
kubectl apply -f deployment.yml
```
![pod with pvc](https://user-images.githubusercontent.com/38099203/119349966-bd9c6300-bcd9-11eb-9f6d-08e4a3ec82f0.PNG)


7. A pod에서 마운트된 경로에 파일을 생성하고 B pod에서 파일을 확인함
```
NAME                              READY   STATUS    RESTARTS   AGE
efs-provisioner-f4f7b5d64-lt7rz   1/1     Running   0          14m
room-5df66d6674-n6b7n             1/1     Running   0          109s
room-5df66d6674-pl25l             1/1     Running   0          109s
siege                             1/1     Running   0          2d1h


kubectl exec -it pod/room-5df66d6674-n6b7n room -n airbnb -- /bin/sh
/ # cd /mnt/aws
/mnt/aws # touch intensive_course_work
```
![a pod에서 파일생성](https://user-images.githubusercontent.com/38099203/119372712-9736f180-bcf2-11eb-8e57-1d6e3f4273a5.PNG)

```
kubectl exec -it pod/room-5df66d6674-pl25l room -n airbnb -- /bin/sh
/ # cd /mnt/aws
/mnt/aws # ls -al
total 8
drwxrws--x    2 root     2000          6144 May 24 15:44 .
drwxr-xr-x    1 root     root            17 May 24 15:42 ..
-rw-r--r--    1 root     2000             0 May 24 15:44 intensive_course_work
```
![b pod에서 파일생성 확인](https://user-images.githubusercontent.com/38099203/119373196-204e2880-bcf3-11eb-88f0-a1e91a89088a.PNG)


- Config Map

1: cofingmap.yml 파일 생성
```
kubectl apply -f cofingmap.yml


apiVersion: v1
kind: ConfigMap
metadata:
  name: airbnb-config
  namespace: airbnb
data:
  # 단일 key-value
  max_reservation_per_person: "10"
  ui_properties_file_name: "user-interface.properties"
```

2. deployment.yml에 적용하기

```
kubectl apply -f deployment.yml


.......
          env:
			# cofingmap에 있는 단일 key-value
            - name: MAX_RESERVATION_PER_PERSION
              valueFrom:
                configMapKeyRef:
                  name: airbnb-config
                  key: max_reservation_per_person
           - name: UI_PROPERTIES_FILE_NAME
              valueFrom:
                configMapKeyRef:
                  name: airbnb-config
                  key: ui_properties_file_name
          volumeMounts:
          - mountPath: "/mnt/aws"
            name: volume
      volumes:
        - name: volume
          persistentVolumeClaim:
            claimName: aws-efs
```

