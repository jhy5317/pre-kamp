# KAMP_LJP — 전자부품(배터리팩) 예지보전 AI 데이터셋 연습 프로젝트

2026년 KAMP(Korea AI Manufacturing Platform) AI 경진대회 참가를 위한 연습용 프로젝트입니다.
KAMP에서 제공하는 「전자부품(배터리팩) 예지보전 AI 데이터셋」과 분석실습 가이드북을 바탕으로
데이터 탐색(EDA) 및 예지보전 모델링을 학습합니다.

## 개요

- **대상 공정**: 전기버스/전기상용차용 배터리팩 조립 공정 중 배터리모듈 **레이저 용접 설비**
- **분석 목적**: 용접 설비의 공정 데이터(RealPower 등)를 분석하여 설비 이상을 사전에 예측하고 불량을 검출
- **적용 알고리즘**: N-HiTS(시계열 예측), 통계분포 기반 Z-score 이상탐지
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
│   ├── 00_가이드북_요약.ipynb    # 가이드북 핵심 내용 정리
│   └── 01_eda.ipynb              # 데이터셋 탐색적 데이터 분석(EDA)
├── src/                         # 재사용 함수 모듈 (전처리, 이상치 탐지 등)
├── outputs/                     # EDA/모델링 결과물(그림, 리포트 등) 저장 위치
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

### 1. 가상환경 설정 (Anaconda 기준)

```bash
conda create -n KAMP python=3.8.8 jupyter
conda activate KAMP
pip install -r requirements.txt
python -m ipykernel install --user --name KAMP --display-name "KAMP"
jupyter notebook
```

`requirements.txt`의 EDA/데이터처리 패키지는 기본 설치되며, N-HiTS 모델링에 필요한
`torch`, `pytorch_lightning`, `pytorch_forecasting` 등은 해당 노트북 작성 시점에
주석 해제 후 별도 설치합니다.

### 2. 노트북 실행 순서

1. `notebooks/00_가이드북_요약.ipynb` — 가이드북 핵심 내용(공정 배경, 데이터 정의, 분석 프로세스, AI 모델)을 먼저 확인합니다.
2. `notebooks/01_eda.ipynb` — 실제 데이터를 로드하여 기술통계, 결측치/이상치 점검, 분포/상관관계 시각화, OK vs NG 데이터 비교를 수행합니다.

## 참고 / 저작권

본 프로젝트는 KAMP 제조AI데이터셋을 학습 목적으로 활용합니다. 연구/공식적으로 인용 시
아래 출처를 표기해야 합니다.

> 중소벤처기업부, Korea AI Manufacturing Platform(KAMP), 전자부품(배터리팩) 예지보전 AI 데이터셋,
> 스마트제조혁신추진단(㈜인터엑스, 네스트필드㈜), 2022.12.23., www.kamp-ai.kr
