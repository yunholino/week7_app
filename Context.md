# Context Pack: AI 실시간 팩트 검증 및 전역 모드 시스템 (week07)

## 1. 개요 및 작업 배경
* **작업 대상**: AI 실시간 팩트 오정보 방지 및 전역 팩트 모드 환경 구현
* **핵심 목표**: 사용자가 매번 "팩트 기반으로 답해줘"라고 입력하는 수동 프롬프트를 자동화하고, 실시간 조회 시 최저가/최적 경로 누락 없이 확신도와 근거를 명확히 제시하는 API/DB/UI 환경 구축
* **작업 연관성**: proposal.md의 MVP 스펙 구현을 위한 기술적 맥락 및 데이터 명세 모음

---

## 2. 연관 데이터 구조 및 DB 매핑
* **USERS (사용자)**
  * user_id: Primary Key, BigInt (사용자 고유 식별자)
  * email: String, Unique (사용자 이메일)
  * created_at: Timestamp (가입 일시)

* **USER_SETTINGS (사용자 환경 설정)**
  * setting_id: Primary Key, BigInt (설정 고유 식별자)
  * user_id: Foreign Key (USERS.user_id 참조)
  * is_fact_mode_enabled: Boolean, Default TRUE (팩트 우선 모드 전역 자동 적용 여부)
  * prioritize_cost_saving: Boolean, Default TRUE (최저가/최적 옵션 우선 탐색 여부)
  * custom_constraints: Text, Optional (사용자 정의 전역 제약 프롬프트)
  * updated_at: Timestamp (설정 변경 일시)

* **CHAT_SESSIONS (대화 세션)**
  * session_id: Primary Key, BigInt (세션 고유 식별자)
  * user_id: Foreign Key (USERS.user_id 참조)
  * session_title: String (세션 제목)
  * created_at: Timestamp (세션 생성 일시)

* **MESSAGES (메시지)**
  * message_id: Primary Key, BigInt (메시지 고유 식별자)
  * session_id: Foreign Key (CHAT_SESSIONS.session_id 참조)
  * sender_type: String (USER 또는 ASSISTANT)
  * content: Text (메시지 본문 내용)
  * created_at: Timestamp (메시지 발송 일시)

* **FACT_VERIFICATION_LOGS (팩트 검증 및 오프셋 로그)**
  * log_id: Primary Key, BigInt (로그 고유 식별자)
  * message_id: Foreign Key (MESSAGES.message_id 참조)
  * confidence_level: String (HIGH / MEDIUM / LOW, 신뢰 수준)
  * is_realtime_queried: Boolean (실시간 외부 API 호출 여부)
  * optimal_options_found: JSON (최저가, 최단 경로 등 비교 파악된 최적 대안 데이터)
  * verified_at: Timestamp (검증 완료 일시)

* **FACT_SOURCES (팩트 근거 데이터)**
  * source_id: Primary Key, BigInt (근거 고유 식별자)
  * log_id: Foreign Key (FACT_VERIFICATION_LOGS.log_id 참조)
  * source_name: String (참조한 외부 시스템 이름, 예: Google Maps API, Booking.com API)
  * source_url: String, Optional (참조 출처 링크)
  * fetched_at: Timestamp (실시간 데이터 조회 기준 시점)

---

## 3. 핵심 API 엔드포인트 명세
* **사용자 설정 조회/수정**
  * 엔드포인트: GET /api/v1/users/settings
  * 설명: 전역 팩트 모드 활성화 여부 및 최저가 우선 설정 상태를 조회함
  * 엔드포인트: PATCH /api/v1/users/settings
  * 설명: is_fact_mode_enabled, prioritize_cost_saving 값을 업데이트함

* **메시지 전송 및 팩트 응답 생성**
  * 엔드포인트: POST /api/v1/chat/messages
  * 요청 파라미터: session_id (String), content (String)
  * 동작 매커니즘: USER_SETTINGS를 먼저 조회하여 is_fact_mode_enabled가 TRUE일 경우 내부 시스템 프롬프트 레이어에 "실시간 팩트 교차 검증 및 최저가 최적화 조건"을 자동 주입한 후 AI 응답을 생성함
  * 응답 데이터: message_id, content, confidence_level, is_realtime_queried, optimal_options, sources

* **1-Click 팩트 재검증 요청**
  * 엔드포인트: POST /api/v1/chat/messages/{message_id}/re-verify
  * 설명: 답변이 미심쩍을 때 사용자가 수동 프롬프트를 치지 않고 버튼을 눌러 최신 데이터 기반으로 재검증 응답을 강제 호출함

---

## 4. 프론트엔드 UI 컴포넌트 구조
* **GlobalFactModeToggle**
  * 위치: 설정 페이지 및 대화창 상단 헤더
  * 기능: 클릭 시 PATCH /api/v1/users/settings를 호출하여 전역 팩트 모드 상태 변경

* **FactVerifiedBadge**
  * 위치: AI 메시지 하단
  * 기능: confidence_level (HIGH/MEDIUM/LOW) 상태를 색상 및 뱃지로 표시

* **OptimalOptionCard**
  * 위치: AI 메시지 내부 하단 영역
  * 기능: optimal_options_found JSON 데이터를 읽어 사용자가 놓치기 쉬운 최저가/최적 경로를 독립 카드로 하이라이트 표시

* **FactSourceModal**
  * 위치: FactVerifiedBadge 클릭 시 오버레이 모달
  * 기능: FACT_SOURCES 정보를 읽어와 어떤 데이터 출처에서 언제(fetched_at) 확인된 정보인지 투명하게 공개

* **QuickReverifyButton**
  * 위치: AI 메시지 우측 하단
  * 기능: 클릭 시 1-Click 팩트 재검증 API 호출

---

## 5. 데이터 처리 흐름 (Data Flow Sequence)
1. 사용자가 메인 대화창에서 질문을 입력하고 전송함
2. 서버는 DB의 USER_SETTINGS에서 is_fact_mode_enabled 상태를 확인
3. 팩트 모드가 활성화되어 있다면, 사용자의 추가 프롬프트 타이핑 없이도 시스템에서 "최신 실시간 데이터 조회 및 최저가/최적 경로 검증 필수" 지침을 프롬프트 패킷에 자동 부가
4. 실시간 정보가 요구되는 질의일 경우 외부 API 연동을 거쳐 최신 데이터를 수집하고 FACT_SOURCES 및 FACT_VERIFICATION_LOGS 생성
5. 클라이언트에 응답 본문과 함께 뱃지, 최적 옵션 카드, 출처 시점 정보를 함께 전달하여 UI에 렌더링
