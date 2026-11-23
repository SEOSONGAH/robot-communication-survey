# 소셜 로봇 의사소통 평가 설문 (v5): 배포 방법

## 폴더 구성
- `index.html`: 설문 페이지
- `videos/`: 영상 10개 (사용자 음성을 지우고 사용자 대사를 자막으로 넣은 버전, 1280×720)
- `Code.gs`: 구글 시트에 응답을 저장하는 Apps Script

## 1. 응답 받을 구글 시트 연결
1. 새 구글 시트를 만들고 [확장 프로그램] → [Apps Script]를 엽니다.
2. `Code.gs` 내용을 붙여 넣고 저장합니다.
3. [배포] → [새 배포] → 유형 **웹 앱**, 실행 **나**, 액세스 권한 **모든 사용자**로 배포합니다.
4. 나온 URL(`https://script.google.com/macros/s/.../exec`)을 `index.html`의 `SUBMIT_URL = ""` 따옴표 안에 넣습니다.

응답 1건이 들어오면 시트에 탭 두 개가 자동으로 만들어집니다.
- `participants`: 참가자 1명 = 1줄 (SPSS 등 넓은 형식)
- `ratings`: 영상 1개 = 1줄, 참가자당 10줄 (R 등 반복측정용 긴 형식)

CSV/Excel 내보내기: 구글 시트에서 [파일] → [다운로드] → CSV 또는 Microsoft Excel(.xlsx)

`SUBMIT_URL`이 비어 있으면 미리보기 모드로 동작합니다. 이때는 응답을 전송하지 않고, 마지막 화면에 두 탭에 들어갈 데이터를 CSV로 보여 줍니다.

## 2. 웹에 올리기
`index.html`과 `videos/` 폴더를 함께 GitHub Pages나 Netlify 같은 정적 호스팅에 올립니다.
- Claude 아티팩트 미리보기에서는 외부 전송이 막혀 있어 구글 시트로 제출되지 않습니다. 점검용으로만 쓰세요.

## 3. 조건 제시 순서
지금은 **균형화하지 않고** 모든 참가자에게 같은 순서로 보여 줍니다: 상황 01(A) 영상 5개 → 상황 02(B) 영상 5개, 각 상황 안에서 ALL → noVOICE → noFACE → noLIGHTING → noGESTURE.
- 순서를 바꾸려면 `index.html`의 `FIXED_ORDER` 배열만 고치면 됩니다.
- 나중에 균형화하려면 `BALANCE = true`로 바꾸면 5×5 균형 라틴 방격(Williams, 순서 10개)이 참가자 ID 숫자로 배정됩니다(`?pid=P001` → 1번 순서). 시나리오 순서는 `SCENARIO_ORDER`("AB", "BA", "counterbalance").
- 구글 시트에 제출되는 순서대로 `p_no`(P1, P2, P3 …)가 자동으로 매겨집니다(두 탭 모두 맨 앞 열).
- `participant_id`는 별개입니다. 링크에 `?pid=…`를 붙이면 그 값이, 없으면 무작위 ID가 들어갑니다.
- 데이터의 `condition_order`에 실제 제시 순서가 기록됩니다. 참가자 화면에는 조건명이나 번호가 나오지 않습니다.

연구자 점검용: `?pid=TEST1&dev=1`처럼 `dev`를 붙이면 영상 건너뛰기 버튼이 나타납니다. 참가자에게는 `dev` 없는 링크를 보내세요.

## 4. 영상과 구간
파일명과 실제 조건이 일치하는지 ALL 영상과 프레임 단위로 비교해 확인했습니다. 각 영상은 ALL과 비교했을 때 아래 부분만 달랐습니다.
- noVOICE: 화면 차이 없음, 로봇 음성만 없음(무음)
- noFACE: 눈(얼굴 화면)
- noLIGHTING: 얼굴 화면 둘레의 조명
- noGESTURE: 팔

장면 평가는 같은 영상 파일의 구간만 재생합니다(별도 클립 없음). 구간 기준은 영상 속 0.1초 검은 화면 전환 지점입니다.
- A: 대화 시작 0–13.63초, 지갑 위치 안내 13.73–26.9초, 대화 마무리 27.0초–끝
- B: 대화 시작 0–12.47초, 사용자의 이야기에 대한 반응 12.57–26.9초, 대화 마무리 27.0초–끝

## 5. 진행 방식
- 전체 영상은 끝까지 봐야 장면 평가가 열립니다.
- 장면 영상은 자동 재생되고, 브라우저가 막으면 재생 버튼으로 시작합니다. 장면이 끝나기 전에도 응답할 수 있습니다(전체 영상을 이미 봤기 때문).
- 모든 필수 문항에 응답해야 다음으로 넘어갑니다.
- 새로고침하거나 창을 닫았다가 같은 브라우저로 다시 열면 이어서 진행됩니다(응답은 브라우저에 임시 저장, 제출 후 삭제).

## 6. 데이터 열
**ratings 탭 (영상 1개 = 1줄)**: `participant_id, scenario, condition, presentation_order, scenario_position, GF1, GF2, GF3, C1, C2, C3, S1, S2, S3, SAT1, SAT2, SAT3`

**participants 탭 (참가자 1명 = 1줄)**
- `participant_id, survey_ver, order_seq, order_seq_source, scenario_order, condition_order, started_at, submitted_at`
- `age, gender, robot_interest, robot_attitude`
- 영상별(`A_ALL` … `B_noGESTURE`, 제시 순서와 무관하게 고정된 열 순서): `_order`(제시 순서), `_GF1`~`_SAT3`, `_plays`, `_seg1_plays`~`_seg3_plays`(처음부터 재생한 횟수), `_sec_watch`, `_sec_overall`(화면별 소요 시간, 초)
- `sec_consent, sec_pre, sec_guide, sec_intro_A, sec_intro_B, sec_post`, `final_comment`
