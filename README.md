# KAMP_LJP — 전자부품(배터리팩) 예지보전 AI 데이터셋 연습 프로젝트

2026년 KAMP(Korea AI Manufacturing Platform) AI 경진대회 참가를 위한 연습용 프로젝트입니다.
KAMP에서 제공하는 「전자부품(배터리팩) 예지보전 AI 데이터셋」과 분석실습 가이드북을 바탕으로
데이터 탐색(EDA) 및 예지보전 모델링을 학습합니다.

## 개요

- **대상 공정**: 전기버스/전기상용차용 배터리팩 조립 공정 중 배터리모듈 **레이저 용접 설비**
- **분석 목적**: 용접 설비의 공정 데이터(RealPower 등)를 분석하여 설비 이상을 사전에 예측하고 불량을 검출
- **적용 알고리즘**: 레시피(SetPower) 조건부 Z-score+규칙 기반 베이스라인, N-HiTS(시계열 예측,
  단변량/다변량), 두 모델의 예측을 결합한 앙상블
- **데이터 출처**: 중소벤처기업부, Korea AI Manufacturing Platform(KAMP), 전자부품(배터리팩) 예지보전 AI 데이터셋, 스마트제조혁신추진단(㈜인터엑스, 네스트필드㈜), 2022.12.23., www.kamp-ai.kr

## 폴더 구조

```
pre-kamp/
├── AASX 및 변환파일/          # AAS(Asset Administration Shell) 표준 관련 파일
├── data/
│   ├── raw_data/
│   │   ├── train/             # Training_Data.csv (학습용, 정상 데이터)
│   │   └── test/               # WeldingTest_0X_OK.csv / _NG.csv (테스트용)
│   ├── preprocessed/           # 전처리 결과 및 이상 구간 라벨(Label.csv) 저장 위치
│   └── Guidebook_*.pdf          # KAMP 분석실습 가이드북 원본
├── notebooks/
│   ├── 00_가이드북_요약.ipynb          # 가이드북 핵심 내용 정리
│   ├── 01_eda.ipynb                    # 데이터셋 탐색적 데이터 분석(EDA)
│   ├── 02_baseline_zscore.ipynb        # 베이스라인: 레시피(SetPower) 조건부 Z-score + 규칙
│   ├── 03_nhits_model.ipynb            # N-HiTS(단변량) 잔차 기반 이상탐지, 베이스라인과 비교
│   ├── 04_nhits_multivariate.ipynb     # N-HiTS(다변량, SetPower를 known covariate로 추가)
│   ├── 05_ensemble.ipynb               # 베이스라인 × N-HiTS 앙상블 (OR/AND 결합)
│   ├── 06_nhits_stable_training.ipynb  # N-HiTS 학습 안정화 (전체 데이터+LR 스케줄러+검증셋 확대)
│   └── 07_holdout_retrain.ipynb        # Train:Val=7:3 홀드아웃 재학습 + 베이스라인/N-HiTS/앙상블 최종 test 평가
├── docs/
│   ├── 도메인_배경지식.md              # 예지보전/용접 공정/모델 개념 배경지식 (계속 업데이트)
│   └── 도메인_배경지식.pdf             # 위 문서의 PDF 버전
├── src/                         # 재사용 함수 모듈 (전처리, 이상치 탐지 등)
├── outputs/                     # EDA/모델링 결과물(그림, 비교표, 체크포인트 등) 저장 위치
├── requirements.txt
├── .gitignore
└── README.md
```

## 데이터셋 요약

- 총 데이터 개수: 1,268,865개 (학습용 1,221,831개 / 테스트용 47,034개), 총 7.96MB
- 수집 방법: AAS(Asset Administration Shell) 표준 기반 제조데이터 수집/저장 체계, 약 3초 주기로 수집 (2022.07.01 ~ 2022.09.30)
- 주요 변수 (9개 중 분석에 사용하는 7개):

| 변수명 | 설명 |
|---|---|
| PageNo | 용접 작업 시퀀스 정보 (Count) |
| Speed | 용접속도설정 (mm/s) |
| Length | 용접길이설정 (mm) |
| RealPower | 용접 포인트 별 실제 용접 출력 (W) — **핵심 분석 대상 변수** |
| SetFrequency | 발광횟수설정 (Hz) — 단일값(상수)이라 실습에서 제거됨 |
| SetDuty | 최대용접출력설정 (%) — 단일값(상수)이라 실습에서 제거됨 |
| SetPower | 용접출력설정 (%) |
| GateOnTime | 용접시간 (s) |
| WorkingTime | 작업시간 (타임스탬프) |

- 테스트 파일 구성:
  - `WeldingTest_01_OK.csv`, `WeldingTest_02_OK.csv` — 정상 데이터
  - `WeldingTest_03_NG.csv` — 고립된 시점에서 이상값이 발생하는 비정상 데이터 (→ N-HiTS 적합)
  - `WeldingTest_04_NG.csv` — 특정 시점 이후 연속적으로 이상값이 발생하는 비정상 데이터 (→ 통계분포 Z-score 적합)

## 사용법

### 1. 가상환경 설정 (Python 내장 venv 기준)

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1      # Windows PowerShell
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name KAMP --display-name "KAMP"
jupyter notebook
```

`requirements.txt`에는 EDA/데이터처리 패키지와 N-HiTS 모델링에 필요한 `torch`,
`pytorch-lightning`, `pytorch-forecasting`이 함께 정리되어 있어 위 명령 한 번으로 모두
설치됩니다(버전 충돌을 피하기 위해 대부분 최소 버전(`>=`)으로 지정했습니다. 자세한 배경은
`requirements.txt` 상단 주석 참고).

### 2. 노트북 실행 순서

1. `notebooks/00_가이드북_요약.ipynb` — 가이드북 핵심 내용(공정 배경, 데이터 정의, 분석 프로세스, AI 모델)을 먼저 확인합니다.
2. `notebooks/01_eda.ipynb` — 실제 데이터를 로드하여 기술통계, 결측치/이상치 점검, 분포/상관관계 시각화, OK vs NG 데이터 비교를 수행합니다.
3. `notebooks/02_baseline_zscore.ipynb` — SetPower(레시피)별 정상 분포를 기준으로 한 Z-score +
   규칙 기반 베이스라인 모델을 만들고 성능을 확인합니다.
4. `notebooks/03_nhits_model.ipynb` — N-HiTS(단변량) 시계열 예측 모델을 학습하고, 예측 잔차를
   이용한 이상탐지를 베이스라인과 비교합니다. GPU가 있으면 자동으로 사용하고, 학습이 중간에
   끊겨도 체크포인트에서 자동으로 이어서 학습합니다.
5. `notebooks/04_nhits_multivariate.ipynb` — 03번을 확장해 SetPower를 N-HiTS가 미리 아는
   공변량(known covariate)으로 추가했을 때의 개선 효과를 확인합니다.
6. `notebooks/05_ensemble.ipynb` — 베이스라인과 N-HiTS(단변량)의 예측을 OR/AND로 결합한
   앙상블을 시도하고, 단독 모델 대비 성능을 비교합니다.
7. `notebooks/06_nhits_stable_training.ipynb` — 03번에서 관찰된 학습 손실 튐 현상을 진단하고
   (무작위 배치 서브샘플링 + 고정 학습률이 원인), 전체 데이터 사용 + 학습률 자동탐색/스케줄러 +
   검증셋 확대로 학습을 안정화합니다. 학습 자체는 훨씬 안정되지만 이상탐지 threshold를
   모델에 맞게 다시 보정해야 한다는 중요한 교훈도 함께 정리되어 있습니다.
8. `notebooks/07_holdout_retrain.ipynb` — 02~06번에서 test 파일(`WeldingTest_03/04_NG.csv`)을
   반복 열람하며 규칙/threshold/앙상블 방식을 정한 것을 바로잡기 위해, `Training_Data.csv`를
   시간순 7:3(train:validation)으로 분리하고 **test 데이터는 노트북 맨 마지막 평가 셀에서
   단 한 번만** 불러옵니다(코드 상 `TEST_DATA_UNLOCKED` 플래그로 강제). 남아있던 epoch 손실
   스파이크는 `SpikeGuardCallback`(손실이 튀는 즉시 학습률을 절반으로 감소)과
   EarlyStopping/ReduceLROnPlateau의 patience 간격 재조정으로 추가 완화했습니다. 베이스라인·
   N-HiTS·앙상블(OR 기본/AND 참고) 3개 모델을 학습해 마지막에 한 번에 비교 평가합니다.

`docs/도메인_배경지식.md`(및 `.pdf`)에는 위 노트북들을 이해하는 데 필요한 개념 설명(예지보전이란
무엇인지, N-HiTS 잔차 점수의 계산 방식 등)이 정리되어 있으며, 분석이 진행될 때마다 계속
업데이트됩니다.

## 참고 / 저작권

본 프로젝트는 KAMP 제조AI데이터셋을 학습 목적으로 활용합니다. 연구/공식적으로 인용 시
아래 출처를 표기해야 합니다.

> 중소벤처기업부, Korea AI Manufacturing Platform(KAMP), 전자부품(배터리팩) 예지보전 AI 데이터셋,
> 스마트제조혁신추진단(㈜인터엑스, 네스트필드㈜), 2022.12.23., www.kamp-ai.kr
