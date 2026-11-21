# 포트폴리오 근거 모음

노현수 포트폴리오의 수치와 주장을 커밋 단위로 확인할 수 있게 정리했습니다.
- 저장소는 모두 `github.com/HumanRhoid` 아래 포크이고, 팀 저장소는 원본 이력을 그대로 담고 있습니다.
- 커밋 작성자 `SSANTA11`, `RHO HYUNSOO`, `HumanRhoid`는 모두 본인 계정입니다.
- 코드 대부분은 Claude, Codex와 함께 작성했습니다. 아래 "누가" 칸은 커밋 작성자 기준입니다.

## 1. 구순 인식 (hanium-lipreading, 팀 프로젝트)

| 주장 | 근거 | 확인 방법 | 누가 |
|---|---|---|---|
| 8번 화자 분리 검증 90.51%, 8명 평균 64.51%, 마지막 에폭 평균 62.10% | [results/cv192.json @ bbca684](https://github.com/HumanRhoid/hanium-lipreading/blob/bbca684/results/cv192.json) | 화자별 3시드의 `best`, `last` 평균. 아래 스크립트 | 본인 |
| 화자 8명, 영상 1,238개, 192×96, 60프레임, 3시드 | 같은 파일의 `val_size` 합, `config.crop`, `config.frames` | 같은 스크립트 | 본인 |
| 3D Conv, ResNet18, 양방향 GRU, 평균 풀링 분류 | [src/ml/models @ d170661](https://github.com/HumanRhoid/hanium-lipreading/tree/d170661/src/ml/models) | `backbone.py`, `temporal.py`, `classification_head.py` | 본인 |
| 입술 랜드마크, 입꼬리 닮음 변환 정렬, 발화 구간, 60프레임, CLAHE | [lip_crop.py @ 7030993](https://github.com/HumanRhoid/hanium-lipreading/blob/7030993/src/ml/preprocess/lip_crop.py), [normalize.py @ e316d85](https://github.com/HumanRhoid/hanium-lipreading/blob/e316d85/src/ml/preprocess/normalize.py) | 함수 `alignment_matrix`, `trim_to_speech`, `resample_frames`, `to_grayscale_clahe` | 본인 |
| ML 파트 전담 | [src/ml 커밋 이력](https://github.com/HumanRhoid/hanium-lipreading/commits/develop/src/ml) | 작성자가 모두 본인 | 본인 |
| 업로드 API, Redis Streams 작업 큐, Compose | [upload_api.py @ 8b2feab](https://github.com/HumanRhoid/hanium-lipreading/blob/8b2feab/src/backend/recognition/upload_api.py), [redis_job_queue.py @ 6df8a0c](https://github.com/HumanRhoid/hanium-lipreading/blob/6df8a0c/src/backend/recognition/adapters/redis_job_queue.py) | | 팀원 jimini4417 |
| 추론 Worker, 상태 조회 API | [inference_worker.py @ 38f5c2d](https://github.com/HumanRhoid/hanium-lipreading/blob/38f5c2d/src/backend/recognition/inference_worker.py), [job_status_api.py @ 6324dab](https://github.com/HumanRhoid/hanium-lipreading/blob/6324dab/src/backend/recognition/job_status_api.py) | Worker 커밋은 본인, jimini4417, shail1027 각 1개 | 팀 |

```python
import json, statistics as st, collections as c
d = json.load(open("results/cv192.json"))
for key in ("best", "last"):
    by = c.defaultdict(list)
    for r in d: by[r["speaker"]].append(r[key])
    m = {k: st.mean(v) for k, v in by.items()}
    print(key, round(m["s08"]*100, 2), round(st.mean(m.values())*100, 2))
# best 90.51 64.51 / last 89.18 62.10
```
`best`는 학습 중 검증 점수가 가장 높은 에폭의 값이라 낙관적입니다. 마지막 에폭 값을 함께 적었습니다.

## 2. 온디바이스 추론 (campus-nlp)

| 주장 | 근거 | 확인 방법 |
|---|---|---|
| S21 4스레드 72.13 tok/s, 6스레드 47.73 tok/s, 51.1% 차이 | [raw/s21-llama-bench-2026-10-15.txt](raw/s21-llama-bench-2026-10-15.txt) | llama-bench 원본 표의 `tg64` 행. 72.13 ÷ 47.73 = 1.511 |
| 측정 스크립트 | [android/s21_bench.sh @ f03f3d2](https://github.com/HumanRhoid/campus-nlp/blob/f03f3d2/android/s21_bench.sh) | |
| Gemma 3 270M Q4_K_M 모델 결과 | [커밋 3c52ee0](https://github.com/HumanRhoid/campus-nlp/commit/3c52ee0) | |
| 요청 1건 총 5.49초 | 같은 raw 파일의 `total time` | |

한계: 측정 출력은 커밋되지 않아 이 저장소에 옮겨 적었습니다. 측정은 AI가 스크립트를 실행했고, 본인은 기기 지정과 연결을 맡았습니다. 반복 수와 발열은 통제하지 않았습니다.

## 3. 앱 시작 병목 (campus-client)

| 주장 | 근거 |
|---|---|
| 모든 노드 쌍 거리표를 시작 시 계산하던 것을 출발점별 힙 다익스트라 지연 계산과 캐시로 변경 | [커밋 db8f8ad](https://github.com/HumanRhoid/campus-client/commit/db8f8ad), `graph.ts`, 단위 시험 `graph.test.ts` |
| buildGraph 35.1초, 측정 구간의 85.3% | 개발 빌드 시스템 트레이스 캡처. 저장소에 없음 |
| 수정 후 시작부터 화면 표시까지 2초대 | 본인 확인값. 측정 기록 없음 |

## 4. MCP (campus-tracker)

| 주장 | 근거 |
|---|---|
| MCP로 시험용 DB에 6개 묶음, 변경 31건 적용 | [커밋 d7d1b38](https://github.com/HumanRhoid/campus-tracker/commit/d7d1b38), [docs/audit/n01/results.json](https://github.com/HumanRhoid/campus-tracker/blob/d7d1b38/docs/audit/n01/results.json), 적용 스크립트 `n01.ts`는 `/mcp` 도구를 호출 |
| 층 노드 미결합 5건에서 0건 | 같은 결과 파일 `validation.before/after.byCode.LEVEL_NODES_NOT_JOINED`. 전체 경고는 176건에서 195건으로 늘었음 |
| 사람만 할 때와 MCP 도입 후 비교 | [docs/EDITOR_MCP_PLAN.md @ aaa4df3](https://github.com/HumanRhoid/campus-tracker/blob/aaa4df3/docs/EDITOR_MCP_PLAN.md) 0절, 2절 |
| 도구 약 20개 | [editor-mcp/tools @ cf67063](https://github.com/HumanRhoid/campus-tracker/tree/cf67063/backend/src/modules/editor-mcp/tools) |
| 역할: MCP 확장과 적용 검증 | [editor-mcp 커밋 이력](https://github.com/HumanRhoid/campus-tracker/commits/main/backend/src/modules/editor-mcp). 첫 코드는 팀원 hoshi2710 커밋 `677bfbc` |

## 5. 서버와 배포

| 주장 | 근거 |
|---|---|
| campus-tracker: Express, TypeScript, PostgreSQL, socket.io, JWT, MCP | [backend/package.json @ 90e26c0](https://github.com/HumanRhoid/campus-tracker/blob/90e26c0/backend/package.json) |
| 테스트 파일 49개 | [backend/test @ 1420d97](https://github.com/HumanRhoid/campus-tracker/tree/1420d97/backend/test) |
| campus-backend 단독 개발 | [커밋 이력](https://github.com/HumanRhoid/campus-backend/commits/develop), 작성자 모두 본인 |
| pytest 시험 함수 131개 | [tests @ e4d0832](https://github.com/HumanRhoid/campus-backend/tree/e4d0832/tests), `grep -c "def test_" tests/*.py` 합 |
| CI | [.github/workflows/ci.yml @ 51be88f](https://github.com/HumanRhoid/campus-backend/blob/51be88f/.github/workflows/ci.yml), push와 pull_request |
| 운영 배포 구성 | [커밋 59030af](https://github.com/HumanRhoid/campus-backend/commit/59030af) compose, [aa138ca](https://github.com/HumanRhoid/campus-backend/commit/aa138ca) Dockerfile과 릴리스 가져오기 |
| PR 단위 개발과 코드 리뷰 | [원본 저장소 PR 목록](https://github.com/SW-ECS-contest-Team-9/campus-backend/pulls?q=is%3Apr) |


## 6. 중고거래 웹 (jsp-secondhand-market, 팀 프로젝트)

원본은 팀원의 비공개 저장소라, 커밋 이력을 그대로 옮긴 공개 사본을 만들었습니다. 본인 계정은 `SSANTA11`, `RHOHYUNSOO`, 팀원은 jominho37.

| 주장 | 근거 |
|---|---|
| Docker Compose로 앱, PostgreSQL, Redis 실행 구성 | [커밋 331deee](https://github.com/HumanRhoid/jsp-secondhand-market/commit/331deee) |
| EC2 배포 중 DB 초기화 스크립트 충돌 수정 | [커밋 c038b5a](https://github.com/HumanRhoid/jsp-secondhand-market/commit/c038b5a). EC2 설정 파일은 저장소에 없음 |
| 카카오맵 위치, 결제, SNS 로그인 | [커밋 16c112c](https://github.com/HumanRhoid/jsp-secondhand-market/commit/16c112c), 결제 교체 [2eca3d1](https://github.com/HumanRhoid/jsp-secondhand-market/commit/2eca3d1) |
| Redis 연동 | [커밋 4fd550d](https://github.com/HumanRhoid/jsp-secondhand-market/commit/4fd550d), 랭킹 코드는 팀원도 수정 |
| 계층 구조, 예외 처리, 보안 | 팀원 jominho37 커밋 |

## 7. 저장소 밖 근거

| 항목 | 근거 |
|---|---|
| AWS Student Builder Groups Core Team Member | [배지](raw/aws-student-builder-group-core-team.png) |
| DI-lab 퇴원기록 선별과 NER 학습 | MIMIC 사용 규정상 비공개. 건수와 점수만 포트폴리오에 표시 (테스트 F1 0.377 → 0.592, 학습 문서 30건 → 60건) |
| Backend.AI 실습 | 실행 화면 캡처만 있음 |
| 다음 학습 계획 (JVM, 모니터링, 부하 시험) | 아직 시작 전 |
