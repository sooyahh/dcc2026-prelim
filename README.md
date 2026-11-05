# 데이터+AI 크리에이터 캠프 2026 대학부 예선 — eigensinn

**Emergency-call recognition from 119 call audio and transcripts** (Data+AI Creator Camp 2026, university track preliminary, 5-person team). I led the team and built everything except the Mission 2 models: the Mission 1 and Mission 3 models, the data audit and EDA, the experiment-tracking system, and the collaboration CI.

| Mission | Task | Final model | Official validation |
|---|---|---|---|
| M1 | Caller gender (audio) | 52-dim acoustic features + WavLM x-vector, logistic regression | Accuracy **98.10%** |
| M2 | Speaker role: caller vs. 119 dispatcher (one utterance's audio) | CNN on log-mel × 3 seeds + fine-tuned WavLM-large × 2 seeds, fixed-weight blend | Accuracy **92.90%** |
| M3 | Multi-label symptoms (transcript) | RoBERTa-large × 3 seeds + TF-IDF logistic regression, fixed-weight blend | Macro F1 **0.6534** |

---

데이터+AI 크리에이터 캠프 2026 대학부 예선 **「전화대화 음성 및 전사 데이터를 이용한 응급상황 인식」** 팀 저장소의 공개용 사본입니다.

## 개인 기여 (김장수, sooyahh)

팀장으로 전체 일정과 제출을 총괄했고, Mission 2 모델을 제외한 작업을 맡았습니다.

- **Mission 1 모델:** 52차원 음향 특징 baseline(95.11%)에서 WavLM x-vector 결합(97.47%), 다중 창 평균(98.13%)을 거쳐 Training 전체 재학습 모델(98.10%)을 제출했습니다. ECAPA 점수 결합은 미리 정한 기준에 못 미쳐 채택하지 않았습니다.
- **Mission 3 모델:** TF-IDF baseline(0.5689)에서 KcBERT(0.6246), RoBERTa-large(0.6433)를 거쳐 3 seed와 TF-IDF의 고정 비율 앙상블(0.6534)을 제출했습니다. 규정에 맞춰 threshold 튜닝과 빈 예측 보정은 쓰지 않았습니다.
- **데이터 감사·EDA:** WAV↔JSON 매칭, 중복·무결성 감사([`reports/data_audit/`](reports/data_audit/2026-08-23/audit_report.md))와 통합 EDA([`reports/eda/eda_report.md`](reports/eda/eda_report.md))를 만들었습니다.
- **실험 관리 체계:** 모든 실험을 ID 단위로 기록하는 [`docs/experiment_log.md`](docs/experiment_log.md)와 채택·기각 근거를 남기는 [`docs/decisions.md`](docs/decisions.md)를 운영했습니다. 공식 Validation은 모델 선택이 아니라 채점에만 쓰고, 미션당 사용 횟수를 제한했습니다.
- **협업 CI:** PR마다 정책 검사와 테스트를 돌려 병합 가능 여부를 라벨로 표시하는 merge gate(`.github/workflows/`, `scripts/pr_policy.py`, `scripts/pr_verdict.py`)와 팀 Git 규칙([`WORKFLOW.md`](WORKFLOW.md))을 만들었습니다.

Mission 2 모델은 팀원(김광희, 지연우)이 담당했습니다.

## Mission과 결과

| Mission | 문제 | 입력 | 평가지표 |
|---|---|---|---|
| **M1** | 신고자 성별 분류 | 음성 | Accuracy |
| **M2** | 신고자 / 119 상황실 구분 | 발화 구간 음성 | Accuracy |
| **M3** | 증상 다중 레이블 분류 | 전사 텍스트 | Macro F1 |

**Mission 2는 speaker separation(음원 분리)이 아니라 speaker-role classification(발화자 역할 분류)입니다.** 이미 분할된 발화 구간 하나를 입력받아 해당 발화가 **신고자** 또는 **119 상황실** 중 어느 역할에 해당하는지 분류합니다. 별도 음원으로 목소리를 분리하거나 여러 화자를 시간순으로 찾아내는 source separation / diarization 과제는 아닙니다.

| Mission | 제출 모델 | 공식 Validation | 제출 폴더 |
|---|---|---|---|
| **M1** | 52D 음향 특징 + WavLM x-vector(다중 창 평균) 결합 로지스틱 회귀. Training 29,200통화 전체로 재학습 | Accuracy **98.10%** | [`submission/mission1/`](submission/mission1/) |
| **M2** | log-mel `SpeakerCNNLarge` 3 seed 앙상블(단독 91.49%)과 `microsoft/wavlm-large` 미세조정 2 seed를 0.4 : 0.3 : 0.3으로 혼합, 판정 0.5 고정. 비율은 Training에서 떼어 둔 발화 20,000개로 선택. 발화 하나의 `startAt`~`endAt` 구간만 사용 | Accuracy **92.90%** (103,994 / 111,947 발화) | [`submission/mission2/`](submission/mission2/) |
| **M3** | `klue/roberta-large` 3 seed + TF-IDF 로지스틱 회귀의 고정 가중치 앙상블(0.55 : 0.45), threshold 0.5 고정·후처리 없음 | Macro F1 **0.6534** | [`submission/mission3/`](submission/mission3/) |

- 각 제출 폴더의 `README.md`에 추론 명령, 학습 재현 방법, 제출 전 검증이 있습니다.
- 공식 Validation은 모델 선택에 쓰지 않았습니다. 앙상블 비율과 threshold는 모두 Training 안에서 떼어 둔 데이터로 정했습니다.
- 수치의 근거는 [`docs/experiment_log.md`](docs/experiment_log.md)의 `M1-REFIT-OFFVAL-001`, `M2-BLEND-001`, `M3-FINAL-002` 행입니다.

## 팀 역할

| 팀원 | 담당 |
|---|---|
| **김장수** | Mission 전체 총괄 |
| **김광희** | Mission 2 |
| **지연우** | Mission 2 |
| **이수빈** | Mission 3 + 전체 Validation·오류 분석 취합 |
| **조지은** | Mission 3 |

## 저장소 구조

```text
dcc2026-prelim/
├── .github/               # PR 템플릿, 정책 검사·테스트 CI, 병합 판정 설정
├── submission/            # 제출 폴더(mission1·2·3): 추론 코드, 학습 재현 노트북, 폴더별 README
├── notebooks/             # Colab/Jupyter 실험 노트북
├── src/dcc_data/          # 데이터 로딩·검증·Mission별 공통 처리 모듈
├── src/dcc_inference/     # Mission별 추론 모듈
├── scripts/               # 데이터 감사, 학습, 평가, 추론 실행 스크립트
├── tests/                 # 공통 모듈과 파이프라인 테스트
├── reports/               # 데이터 감사, EDA, Mission별 결과와 오류 분석
├── ppt/                   # 결과 발표 자료
├── docs/
│   ├── experiment_log.md  # Mission별 실험 결과 인덱스
│   ├── mission2_experiment_log.md  # Mission 2 상세 실험 기록
│   └── decisions.md       # 채택·기각·보류 결정과 근거
├── requirements.txt
└── WORKFLOW.md            # Git/GitHub 협업 규칙
```

## 실행

```bash
python -m pip install -r requirements.txt
python -m pytest
```

대회 데이터와 모델 가중치는 이 저장소에 없습니다. 원본은 `data/raw_original/`(Training·Validation의 원천 WAV와 라벨 JSON)에 두거나 `--data-root` 또는 `DCC_DATA_ROOT`로 지정합니다. 제출용 추론은 각 `submission/missionN/inference.py`가 진입점이며, 실행 명령은 폴더별 README를 따릅니다. 각 스크립트의 인자는 `--help`로 확인합니다.

```text
원본 데이터 준비 → WAV·JSON manifest 생성 → Mission별 dataset/feature 생성 → 학습·평가 → 실험 로그와 보고서 기록
```

## 공개용 사본 안내

- 이 저장소는 팀 저장소(비공개)의 대회 종료 시점 `main`(2026-10-14)을 포트폴리오용으로 옮긴 것입니다. 팀원 5명의 동의를 받아 공개합니다.
- 커밋 이력은 옮기지 않았습니다. 원 저장소 기준 커밋 수: sooyahh 199 · Ajou-Joa 143 · subin050203-prog 53 · wldusdn03 43 · 38grace 15.
- 대회 데이터 재배포 금지에 따라 음성·전사 원본, 샘플 단위 정답·예측 파일, 통화 ID가 출력된 노트북 결과는 넣지 않았습니다. 그래서 일부 문서가 가리키는 파일(예: `reports/mission1/m1_val_error_rows.parquet`, `reports/mission3/ensemble/probs/`)은 이 사본에 없습니다.
- 모델 가중치(M1 약 386MB, M3 약 4.05GB, M2 앙상블)는 용량 때문에 넣지 않았습니다.
- 문서 안의 팀 저장소 링크와 PR 번호는 비공개 저장소를 가리킵니다.
