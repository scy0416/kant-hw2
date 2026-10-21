# AI 사용 기록

AI는 개념 질문과 직접 작성한 코드의 디버깅에만 사용할 수 있습니다. 사용하지 않았다면 아래에 `사용하지 않음`이라고 적습니다. 사용량을 늘릴 필요는 없습니다.

| 허용되는 도움 | 예시 |
|---|---|
| 개념 이해 | HTTP 404와 422의 차이, await의 의미를 다른 예시로 설명해 달라고 질문 |
| 자신의 코드 디버깅 | 직접 작성한 짧은 함수와 오류를 보여 주고 원인과 확인 방법을 질문 |

과제 전체 풀이, TODO의 완성 코드, 참고 답안을 요청하지 않습니다. 디버깅 답변도 이해한 원인과 수정 내용을 자신의 말로 기록하고 다시 실행합니다. API 키, `.env`와 인증 헤더를 질문에 포함하지 않습니다.

## 사용 여부

- 사용 여부: 사용 (필수 1 자료 수집, 필수 2 문서 정제, 필수 3 문서 API, 필수 4 질문 API. 선택 심화는 진행하지 않음)
- 사용한 도구: Claude Code (Claude Sonnet 5.5, claude-sonnet-5-5)

AI는 과제 분석, 테스트 방법 설명, 제가 작성한 코드의 오류 원인 지적과 검토에만 사용했습니다. `collect_releases`, `normalize_releases`, 필수 3의 모델 필드와 `find_document`, `register_document`, `response_models`, 필수 4의 `build_prompt`, `call_model_once`, `query_with_retry`, `admit_query`, `request_logging`의 코드와 `작성 메모`는 AI가 써 주지 않았고 제가 직접 작성했습니다. 이 표는 대화 내용을 바탕으로 AI(Claude Code)가 정리했고, 저는 사실과 맞는지 확인했습니다. 날짜는 대화 시점 기준이며 자정을 넘긴 작업은 `10-20~21`로 적었습니다.

## 도움 기록

도움을 받은 건마다 표의 행을 추가합니다. 대화 전문보다 질문과 직접 확인한 결과를 구체적으로 적습니다.

| 단계/함수 | 직접 시도한 내용과 문제 | 질문 요약 | 받은 설명 | 내가 수정하거나 확인한 내용 | 재실행 결과 |
|---|---|---|---|---|---|
| 2026-10-20, 과제 파악 | 아직 코드를 작성하기 전 | 이제 데이터 수집 & API 서버 구축를 진행할거야 | AI가 `ASSIGNMENT.md`, `README.md`, `app.py`, `data/DATA_CARD.md`, `AI_USAGE.md`를 읽고 필수 4문제와 선택 1문제의 구조, `작성 메모` 4곳, fixture와 live의 구분(결과 폴더가 `artifacts/fixture/`와 `artifacts/live/`로 나뉘고 `run collect`는 외부에 접속하지 않음)을 설명함. 필수 1의 빈칸 4곳이 각각 무엇을 판단하는지(쿼리 파라미터, HTTP 상태 검사, `response.links`, 저장할 값)와 진행 순서를 안내함. 코드는 제공하지 않음. | 수정 없음 | 실행 없음 |
| 2026-10-20, 필수 1 `collect_releases` | 빈칸 4곳을 작성했지만 어떻게 테스트할지 몰랐음. `check collect`가 9개 모두 같은 `TypeError`로 실패 | 일단 맞는거 같은대로 해봤거든? 근데 어떻게 테스트를 해봐야할지를 모르겠어 | `check`, `run`, `run --live`가 각각 하는 일과 통과 기준을 설명함. `check`와 `run`이 에러 내용을 가려서, 제공된 가짜 서버(`support.collection_transport()`)로 함수를 직접 호출하면 진짜 오류 줄을 볼 수 있다고 알려 줌. 세 곳을 지적함. (1) 저장 반복에서 `value["items"]`를 써서 리스트에 문자열 인덱스를 쓴 `TypeError`(저장할 값은 반복 변수 `value` 자체), (2) `params`에 `per_page`가 5로 고정되어 있고 GitHub API에 없는 `max_pages`가 섞여 있음, (3) `response.faise_for_status`의 오타와 괄호 누락(호출해야 하는 줄이고 결과를 `report["error"]`에 저장하면 안 됨). 이 대화에서 AI가 원인 줄과 수정 방향을 구체적으로 알려 줌. | `collect_releases`: 저장할 값, `params`, 상태 검사 호출 세 줄을 직접 수정함. | `check collect` 0개 → 9개 통과. `run collect`: `success=true`, 2페이지, `raw_count=10` |
| 2026-10-20, 필수 1 `collect_releases` | 세 줄을 고친 뒤 구현이 끝났는지 알고 싶었음 | 이제 collect_releases가 전부 구현되었는지 확인해줘 | AI가 `check collect`와 `run collect`를 직접 실행해 9/9 통과와 `raw_pages.json`의 구조(두 페이지, 각 5행, 원본 ID 보존)를 확인함. 수집 단계에서는 중복 ID를 그대로 두고 정제 단계에서 처리해야 한다고 설명함. `작성 메모`가 비어 있고 `run collect --live`를 아직 안 했다고 알림. | 수정 없음. | `check collect` 9/9 통과 |
| 2026-10-20~21, 필수 2 `normalize_releases` | 빈칸 7곳을 작성한 뒤 `check documents`가 실패함 | 필수2 진행해봤는데 문제없는지 확인해줘 | AI가 `check documents`(22개 중 18개 통과)와 `run documents`(채택 2, 격리 6, 제외 2. 기대는 3, 5, 2)를 실행해 세 문제를 지적함. (1) 제목 대체: `name`이 공백뿐이어도 문자열은 참으로 평가되어 `tag_name`으로 대체되지 않음(ID 102가 `missing_title`로 격리), (2) 문서의 `collected_at`에 수집시각이 아니라 게시시각을 넣음, (3) `item["draft"]` 등 `[]` 접근이 키 누락 시 `KeyError`. `DATA_CARD.md`의 규칙(누락, null, 공백뿐이면 `tag_name` 사용)과 연결해 설명함. 수정 코드는 제공하지 않음. | `normalize_releases`: 7곳의 빈칸을 처음 작성한 뒤, 지적받은 세 곳을 직접 수정함. | `check documents` 18개 → 20개 통과. `run documents` 채택 3, 격리 5, 제외 2 |
| 2026-10-21, 필수 2 `normalize_releases` | 위를 고친 뒤 제목 쪽에서 새 실패가 남음 | 이제 문서 제목쪽에서 문제가 발생하는데 그 이유를 모르겠어 | AI가 `AttributeError`의 원인을 설명함. `name`이 없거나 `null`이면 `None`이 `invalid_name` 가드를 통과해 내려오는데 `None`에는 `.strip()`이 없음. 문자열일 때만 `strip`을 호출하도록 `None`과 문자열을 모두 안전하게 처리해야 한다고 방향만 안내함. | 163번째 줄(제목 대체)을 직접 수정함. | `check documents` 20개 중 2개 실패 → 22개 모두 통과 |
| 2026-10-21, 필수 2 `normalize_releases` | 수정 후 확인을 원함 | 올바르게 바꿔봤는데 맞는지 확인해줘 | AI가 `check documents` 22/22 통과, `run documents`의 채택 ID(`release-uv-101`, `102`, `109`), 문서 `collected_at`이 수집시각으로 들어간 것, ID 102의 제목이 `fixture-102`로 대체된 것을 확인함. `작성 메모`가 비어 있다고 알리고 `run documents --live`를 안내함. | 수정 없음. | `check documents` 22/22. `run documents` 채택 3, 격리 5, 제외 2 |
| 2026-10-21, `작성 메모` (필수 1, 2) | 필수 1, 2의 `작성 메모` 두 곳을 작성함 | 메모 전부 작성했는데 확인해봐 | 두 메모의 방향은 맞지만 자기 코드에 연결하는 부분이 부족하다고 설명함. 메모 1은 "구조화"가 `response.json()`, `response.links`로, "문제 추적"이 `raise_for_status()`와 `report["error"]`로 실현된다는 점을 연결하라고 제안함. 메모 2는 출처와 두 시각을 각각 기록한다고만 말하고 "구분하는 이유"가 없다고 지적함(게시시각은 `records`에, 수집시각은 문서에 들어가며 처음에 둘을 섞었을 때 `check`가 실패한 경험). 메모 4곳 중 2곳만 해당 단계에서 작성하는 것이라고 안내함. | 메모 보강은 이후 직접 진행. | `check collect` 9/9, `check documents` 22/22. `run collect --live` 실행됨(`success=true`, 2페이지, `limited=true`) |
| 2026-10-21, 필수 3 문서 API | 모델 필드 9개와 `find_document`, `register_document`, `response_models`를 작성함. 메모의 요구가 무엇인지 이해하지 못함 | 필수 3번 다 했어. 확인해봐주고, "GET/POST의 입력 위치와 없는 ID/잘못된 입력의 차이를 1~2줄로 적으세요."이게 뭘 작성하라는건지 이해 못하겠으니까 뭘 적으라는건지 좀 알려줘 | AI가 `check document-api`를 실행해 `support.LIMITS["source_id"]`의 `KeyError`(키는 `source_url`)가 모듈 전체를 로드하지 못하게 하고 있다고 지적함. 나머지 모델과 함수는 정적으로 읽어 요구와 맞다고 확인함. 메모가 요구하는 것을 설명함. (1) 입력 위치는 GET이 URL 경로의 문자열, POST가 요청 본문 JSON이라는 점. (2) 없는 ID는 요청 형식은 맞고 대상이 없어서 404(`find_document`가 저장소 조회로 결정), 잘못된 입력은 요청 자체가 규칙 위반이라 422(pydantic이 함수 호출 전에 거부), 중복 등록은 409라는 점. 메모 문장은 제공하지 않음. | `Document.source_url`의 `LIMITS` 키를 직접 수정함. 메모 두 줄을 직접 작성함. | `check document-api` 시작 불가(`KeyError`) → 7개 중 7개 통과 |
| 2026-10-21, 필수 3 문서 API | 키를 고친 뒤 다시 확인을 원함 | 다시 확인해봐줘 | AI가 `check document-api` 7/7 통과를 확인함. 서버를 직접 호출해 과제 표의 응답(health 200 `document_count: 9`, 등록 201, 단건 조회 200, 없는 ID 404, 같은 ID 또는 URL 409, `manual/a`와 누락, 자료형, 빈 값, 추가 필드, 없는 `category` 422, `title` 300자 201과 301자 422, `/query` 501)을 확인함. 메모 둘째 줄에서 GET의 잘못된 입력도 404라고 쓴 부분이 부정확하며 잘못된 입력은 422라고 지적함. | 메모의 둘째 줄 보강은 이후 진행. 아직 반영하지 않음. | `check document-api` 7/7 통과 |
| 2026-10-21, 기록 정리 | 필수 4와 선택 심화는 다음에 진행하기로 함 | 일단 나머지는 다음에 하게 지금까지의 대화들을 AI_USAGE.md를 채워넣어줘 | 이 표를 AI가 대화 내용을 바탕으로 정리함. 정리하면서 `artifacts/live/`에 `run documents --live` 결과가 생성된 것을 확인함(입력 10, 채택 10, 격리 0, 제외 0). | `AI_USAGE.md` 작성 | 실행 없음 |
| 2026-10-21, 필수 4 질문 API | 필수 4의 `support.todo`를 모두 작성한 뒤 검사 결과를 확인하고 싶었음 | 필수4의 todo를 전부 작성한거 같은데 확인해줘 | AI가 `check query`(6개 중 1개 실패), `check retry`(17개 중 3개 실패), `check observe`(6개 중 3개 실패)를 실행해 원인을 지적함. (1) `call_model_once`에서 모델 응답 `text`가 JSON 문자열인데 `json.loads` 없이 `model_validate`에 넘겨 `ValidationError`가 남. (2) 선택하지 않은 출처를 검사하는 조건이 `not`으로 거꾸로 되어 정상 응답이 거부됨. 이 두 곳 때문에 성공해야 할 시나리오가 502가 되어 `check retry`의 성공 시나리오 3개와 `check observe`의 동시 요청 항목이 실패하는 것으로 설명함. `admit_query`는 설명과 맞다고 확인함. `build_prompt`의 `ensure_ascii=False` 누락, `작성 메모`의 "await로 기다리는 이유" 문장이 반대로 읽히는 점도 짚음. 수정 코드는 제공하지 않음. | `call_model_once`: 응답 JSON을 읽는 부분과 출처 검사 조건을 직접 수정함. | `check query` 6/6, `check retry` 17/17, `check observe` 6/6 통과 (fixture). 실제 모델 호출(live)은 아직 확인하지 않음. |
