Real-Time Fact Assurer (RTFA) - 제안서 (Proposal)

1. 문제 (Problem)

실시간 팩트 오정보 및 경제적 손실: 여행 등 실시간 최적 정보가 중요한 상황에서 AI가 오정보를 당당하게 제공하여, 더 빠르고 저렴한 대안(최저가/최적 경로)을 놓치는 문제 발생.

수동 프롬프트 입력 피로감: 팩트 기반 답변을 얻기 위해 사용자가 질의할 때마다 "팩트 위주로 검증해줘" 등의 제약 명령어를 수동으로 입력해야 함.

답변 신뢰의 불확실성: 프롬프트를 추가 입력하더라도 AI 답변이 100% 정답이라는 보장이 없어, 사용자가 포털/지도 앱으로 직접 재검색해야 하는 피로가 지속됨.

2. 타겟 사용자 (Target User)

유럽 여행자 등 현장 실시간 최적화가 필수적인 사용자:

이동 경로, 예매 가격, 운영시간 등 정확한 팩트가 실질적 비용 및 시간 절약으로 이어지는 상황에 놓인 사용자.

매번 프롬프트를 덧붙이는 것이 번거롭고, AI 답변의 신뢰도를 한눈에 확인하고 싶은 사용자.

3. 핵심 가치 (Value Proposition)

경제적·시간적 손실 방지: 최적 대안(최저가/최단 경로)의 교차 검증을 통해 놓치는 혜택 최소화.

프롬프트 입력 제로화: 전역(Global) 팩트 모드 설정을 통해 질문만 던져도 자동으로 최신 팩트 제약 조건이 적용됨.

불확실성 해소 (시각적 신뢰도 제공): 답변과 함께 검증 상태(High/Medium/Low) 및 데이터 수집 시점을 명시하여 재검색 수고 감소.

4. MVP 범위 (Minimum Viable Product)

전역 팩트 모드 (Global Fact Mode): 사용자 설정에서 '팩트 검증 및 경제성 우선 모드'를 상시 On/Off 고정하는 기능.

실시간 팩트 검증 카드 (Fact Verification Card): 답변 생성 시 외부 데이터 조회 시점, 신뢰도 레벨, 최적 옵션(놓치기 쉬운 저렴한/빠른 경로)을 하이라이트로 표시.

1-Click 팩트 재검증 (Quick Re-verify): 프롬프트 재입력 없이 클릭 한 번으로 최신 데이터 교차 검증 요청.

5. 제약 사항 (Constraints)

API 레이턴시: 외부 실시간 데이터(교통, 가격 등) 교차 검증으로 인해 일반 AI 답변보다 생성 시간이 1~2초 지연될 수 있음.

데이터 출처의 한계: 제휴/공공 API 범위를 벗어나는 아주 제한적인 현지 정보는 신뢰도 레벨이 'Low'로 표기됨.

6. 화면 구조 및 UX 흐름 (Screens)

설정 화면 (Settings): 팩트 모드 전역 적용 토글, 최저가/최적 옵션 우선 토글 제공.

대화 화면 (Chat Screen):

메시지 입력창: 별도의 팩트 요구 프롬프트 없이 일반 질의 입력.

AI 답변 블록:

상단: [실시간 검증 완료 | 신뢰도 High] 뱃지

본문: 답변 내용 및 [추천 최적 옵션 카드 (더 저렴한/빠른 길)]

하단: 데이터 조회 시점 (예: 2026-10-28 14:30) 및 [1-Click 재검증] 버튼

7. 핵심 데이터 모델 (Data)

UserSetting: is_fact_mode_enabled (Boolean), prioritize_cost_saving (Boolean)

Message: message_id, session_id, sender_type, content

FactVerificationLog: log_id, message_id, confidence_level (HIGH/MEDIUM/LOW), optimal_options_found (JSON)

FactSource: source_id, log_id, source_name, fetched_at

8. 앱 인수 조건 (Acceptance Criteria, AC)

[ ] AC 1: 전역 팩트 모드가 ON 상태일 때, 사용자가 팩트 관련 프롬프트를 따로 적지 않아도 생성된 답변에 FactVerificationLog가 자동으로 결합되어야 한다.

[ ] AC 2: 실시간 최저가/최적 경로 등의 정보가 존재할 경우, AI 답변 내 별도의 하이라이트 카드로 구분되어 출력되어야 한다.

[ ] AC 3: 모든 팩트 기반 답변에는 데이터 수집 시점(fetched_at)과 신뢰도 레벨(confidence_level)이 시각적으로 노출되어야 한다.

[ ] AC 4: 사용자가 1-Click 재검증 버튼을 누르면 기존 질문에 대해 최신 실시간 데이터 조회가 재수행되고 결과 카드가 갱신되어야 한다.
