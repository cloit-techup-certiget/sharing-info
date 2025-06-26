# Apache Kafka 소개

## 목차

1. [Kafka 간략 소개](#kafka-간략-소개)  
2. [일반적인 애플리케이션과 Kafka 애플리케이션 비교](#일반적인-애플리케이션과-kafka-애플리케이션-비교)  
3. [Kafka의 장점/효과](#kafka의-장점효과)  
4. [Kafka의 주요 개념 설명](#kafka의-주요-개념-설명)  
5. [예시 애플리케이션](#예시-애플리케이션)  
6. [유사 사례 분석](#유사-사례-분석)  
7. [Kafka 사용시 주의사항](#kafka-사용시-주의사항)  
8. [참고 서적 및 영상](#참고-서적-및-영상)  

---

## Kafka 간략 소개

- LinkedIn이 대규모 데이터 처리 및 실시간 스트리밍을 위해 Java로 개발
- 확장 가능하고 내결함성을 보장하는 분산 스트리밍 플랫폼
- 데이터 파이프라인의 정중앙에서 메인 허브 역할
- 실시간 데이터 처리 시나리오에 사용됨 (예: 넷플릭스 영상 추천)

---

## 일반적인 애플리케이션과 Kafka 애플리케이션 비교

- **일반적인 웹 애플리케이션**: 백엔드에서 대부분의 로직과 DB 트랜잭션 수행
- **Kafka 애플리케이션**: 일부 로직만 처리하고, 대부분은 Real-time Processor가 처리  
  DB 대신 Kafka Broker를 통해 메시지를 주고받음

---

## Kafka의 장점/효과

- **부하 분산**: 백엔드 로직의 부담이 적음
- **DB 트랜잭션 감소**: 데이터 조회 및 Join이 없음
- **데이터 유실 방지**: Processor만 정지해도 Broker가 메시지를 보관함

---

## Kafka의 주요 개념 설명

- **message**: Kafka에서 주고받는 데이터 단위
- **offset**: 메시지 순서를 보장하는 ID
- **topic**: 메시지를 수신하는 주소 (partition으로 분산 처리됨)
- **partition**: 병렬 처리 단위
- **replication**: 데이터 백업 기능
  - 예: partition 3개, replication 3이면 총 9개 복제본
- **Broker**: 메시지를 저장하는 서버
- **Producer**: 메시지를 보내는 클라이언트
- **Consumer**: 메시지를 받는 클라이언트

---

## 예시 애플리케이션

### 호텔 매니저 시스템

- **목표**: 비용 절감, 청소 시간 최적화 등
- **예시 1**: 고객이 외출 시 난방 자동 차단 (`sensor_edge.py → consumer.py`)
- **예시 2**: 체크아웃 시 청소 자동 요청 (`checkout_edge.py → cron`)

**소스 코드:**  
https://github.com/K-hyuk06/sample_for_study_kafka_python

---

## 유사 사례 분석

- **외국 개발자 예시**  
  https://github.com/K-hyuk06/sample_for_study_kafka_python  
- **참고 글**  
  [IoT Architecture - Nishank Singla](https://www.linkedin.com/pulse/iot-architecture-nishank-singla)

---

## Kafka 사용시 주의사항

- 데이터 안전성을 위한 추가 설정 필요
- 컨테이너 환경 필수 (Docker, Kubernetes)
- 보안 설정은 따로 학습 필요

---

## 참고 서적 및 영상

### 강의 (유료)

- [Apache Kafka Crash Course for Java and Python Developers (Udemy)](https://www.udemy.com/course/apache-kafka-crash-course-for-java-and-python-developers/)
- [Apache Kafka 시리즈 – 초보자를 위한 아파치 Kafka 강의 (한글자막)](https://www.udemy.com/course/apache-kafka-korean/)

### 책

- [실전 카프카 개발부터 운영까지](http://www.yes24.com/Product/Goods/104410708)
- [아파치 카프카 애플리케이션 프로그래밍 with 자바](http://www.yes24.com/Product/Goods/99122569)

### 유튜브

- [Knowledge Amplifier - YouTube](https://www.youtube.com/@KnowledgeAmplifier1)

### 소개 기사

- [CIO Korea 소개 기사](https://www.ciokorea.com/news/227469)

---

> 작성일: 2023-04-03  
> 발표자: 레몬
