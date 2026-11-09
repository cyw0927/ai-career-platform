# T26 Skill Gap 계산 초안

Redmine #1530. `app/services/skill_gap.py`는 외부 API·DB와 연결되지 않은 순수 계산 함수다. T04 FR-ANL-002의 검토 중 요구사항을 기준으로 `max(요구 - 보유, 0)`을 사용하며, 미입력 계산은 0으로 하되 원본 `None`과 미등록/미기재 구분을 결과에 남긴다. 필수·선택은 표기만 하며 가중치에 영향을 주지 않는다. 동점은 코드순으로 정한다. 모두 팀 승인 전 제안이다.

값은 1~5 정수, 코드 중복·빈 코드 등은 거부한다. 결과는 자기평가 기반이며 합격 확률이 아니다. 사용자 인증·권한, 실제 코드 정규화, 직무·사용자 데이터 조회, API·DB·이력 저장, 합의된 DTO 및 CI는 구현하지 않았다. T05/T06/T17/T25의 최신 계약이 필요하다. 예제만으로 전체 기능 완료를 주장하지 않는다.

백엔드 폴더에서 `python -m pytest tests/test_skill_gap.py -q`를 실행한다.

## T05 병합본과 대조 (2026-11-06)

T05 PR #17은 main `a004efb`에 병합됐다. 데이터 사전은 아직 DRAFT/PM·DB 리뷰 대기이며 DDL 미적용이다. `user_skills.proficiency`는 NULL을 허용하고 계산 시 0으로 취급한다. `skill_gap_items.observed_level`은 NULL이 아닌 0~5 계산 스냅샷이다. 현재 함수는 원본 수준을 `None`으로 보존하므로 DB 저장용 0과 직접 동일시하면 안 된다. 저장 변환과 미등록/미입력 표시의 DTO 연결은 미구현·확인 필요다. `model_versions.kind='rule'`이며 `approved_at`이 NULL이면 서비스 사용 금지다. T06 PR #29는 병합됐고 실제 API 구현은 별도다.

`to_snapshot_example`은 결과의 미입력 `None`을 계산 스냅샷의 0으로 바꾸는 가상 변환이며 원본은 변경하지 않는다. `skill_code`는 예제 식별자이고 실제 DB의 UUID·assessment_id 매핑, 저장 및 권한 검사는 포함하지 않는다.

## T06 병합 계약 대조 (2026-11-06)

T06 PR #29는 main `d1fae3a`에 병합됐다. SkillGapItem의 observed_level은 0~5이고 0은 미입력을 뜻한다. 응답에는 skill 객체(UUID·code·name)가 필수이며 현재 가상 스냅샷의 skill_code만으로는 DTO가 완성되지 않는다. SkillGap 응답은 assessment_id, job_role, model_version, is_stale, evaluated_at이 필요하고 이 함수는 해당 조회·조립을 제공하지 않는다. 응답 items 설명의 빈 배열 처리와 함수가 gap=0 항목도 반환하는 차이는 API 담당자와 협의한다. 0은 미입력 표시 근거가 될 수 있지만 미등록과 미기재의 차이를 별도 전달하는 필드는 없다.

## 2026-11-09 PM 결정 및 Golden Fixture

T25 PR #26의 PM 결정에 따라 Skill Gap을 1차 구현 대상으로 우선하며, 사용자 원본 숙련도 NULL과 계산 스냅샷의 0을 구분한다. `tests/fixtures/t26_skill_gap_golden.json`은 합성 입력과 예상 결과를 코드 밖에 고정해 정상·미입력·미등록·초과 숙련·요구 기술 없음 및 반복 계산을 확인한다. 점수 0인 항목 포함 여부 및 보유 기술이 전혀 없는 사용자에 대한 API 사전조건은 후속 T43/API 담당자와 확인한다.

T17 PR #117은 진행 중이고 정식 통합은 아직 확인되지 않았다. 이 계산 예시는 실제 T17·T43 조회/API/DB·인증 연동이 아니다. 공용 Staging DB는 수정하지 않았다.
