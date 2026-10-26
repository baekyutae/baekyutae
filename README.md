<div align="center">

## 배규태 · Backend / AI 서비스 개발자

<a href="https://github.com/f-lab-edu/Biblio"><img src="https://img.shields.io/badge/Biblio-영상_RAG-2F6FEB?style=flat-square&logo=github&logoColor=white" alt="Biblio"/></a>
<a href="https://github.com/SoftBank-Hackathon-2026-Team-Amethyst/one-tatchi-platform"><img src="https://img.shields.io/badge/oneTatchi-AI_배포_플랫폼-2F6FEB?style=flat-square&logo=github&logoColor=white" alt="oneTatchi platform"/></a>
<a href="https://github.com/baekyutae/portfolio"><img src="https://img.shields.io/badge/Bee_careful-꿀벌_질병_진단-2F6FEB?style=flat-square&logo=github&logoColor=white" alt="Bee careful"/></a>

</div>

---

### Introduction

**AI 서비스의 데이터 파이프라인 구축과 성능 개선에 강점이 있는 개발자입니다.**

- 영상의 음성과 장면 정보를 검색 데이터로 가공하고, 검색 결과를 근거로 LLM이 답변하는 RAG 서비스를 설계·구현해 GCP에 배포
- 영상 처리 중 발생한 메모리 부족과 처리 대기의 원인을 분석하고, 처리 구조를 개선해 동일 조건의 테스트에서 메모리 사용률 감소(62.54% → 14.37%)와 처리량 향상(17.4%)

---

### 대표 프로젝트

#### 🎬 [Biblio](https://github.com/f-lab-edu/Biblio) — 영상 검색 · RAG 서비스와 임베딩 모델 운영
`개인 프로젝트` `2026.01 ~` `PR 50개 머지`

영상을 올리면 음성 대본·장면 설명·화면 속 글자를 검색 데이터로 바꾸고, 질문에 근거 영상과 재생 시점을 붙여 답하는 백엔드입니다.

| 문제 | 해결 | 결과 |
|---|---|---|
| 외부 API를 기다리는 동안 영상 단위 작업이 워커 슬롯을 잡고 있어 다음 영상 처리가 밀림 | 처리 단계를 독립 작업 큐로 나누고 단계별 동시 처리 수를 따로 제어 | 시간당 처리량 **48.02 → 56.37편 (+17.4%)**, 평균 완료 시간 440.8 → 390.8초 |
| 원본 영상 전체를 메모리 기반 임시 파일시스템에 내려받아 워커가 메모리 부족으로 종료 | FFmpeg가 Signed URL의 HTTP Range로 필요한 구간만 읽게 바꾸고, 추출물은 업로드 직후 삭제 | 최대 메모리 사용률 **62.54% → 14.37%** |
| 외부 STT 타임스탬프가 영상 중간부터 약 163초씩 앞당겨져 테스트 영상 8건 중 7건에서 검색 데이터 생성 실패 | 정상 구간의 단어별 발화 시간으로 오차를 추정해 오류 구간만 보정 | 오류 영상 모두 정상 생성, 별도 STT 결과와 시작·종료 시각 차이 약 80ms |

- 키워드 검색(PostgreSQL FTS)과 벡터 검색(pgvector)을 RRF로 합쳐 LLM 답변의 근거로 사용
- 사용자 피드백을 검증해 학습 데이터로 만들고, 임베딩 모델을 바꿀 때 검색 인덱스를 함께 전환하거나 이전 조합으로 되돌리는 배포 흐름 구현
- Terraform으로 GCP(Cloud Run, Compute Engine, GCS)에 배포

<sub>Python · FastAPI · PostgreSQL · pgvector · pgmq · BGE-M3 · Vertex AI(Gemini) · Google STT · Terraform · k6 · testcontainers</sub>

#### 🚀 [oneTatchi platform](https://github.com/SoftBank-Hackathon-2026-Team-Amethyst/one-tatchi-platform) — AI 에이전트가 분석하고 검사를 통과한 변경만 배포하는 플랫폼
`2026 소프트뱅크 해커톤` `5인 팀` `담당 태스크 6개 · PR 14개 머지`

로컬 웹앱을 AI 에이전트가 분석해 배포 파일을 만들고, 정적분석·테스트·취약점 검사를 통과한 변경만 AWS·GCP·온프레미스에 Blue-Green으로 배포합니다.

| 담당 | 한 일 | 검증 |
|---|---|---|
| **AI 승격 판단** | 새 버전(green)에 smoke 요청을 보내 에러율·p95·파드 상태를 규칙으로 판정하고 Claude가 승격·중단을 결정. 규칙이 실패하면 AI 답과 상관없이 중단하고, API 오류·시간 초과도 중단으로 처리 | 가짜 클러스터·가짜 API로 27가지 경우 테스트 |
| **GCP 배포 모듈** | GKE(2개 영역), Artifact Registry, 사설 Cloud SQL, Secret Manager, WIF 인증을 Terraform 모듈로 추가하고 공통 배포 워크플로에 GCP 분기 연결 | AWS·온프레미스 렌더 결과 동일, Trivy HIGH/CRITICAL 0건, 실제 GCP 배포 |
| **배포 실패 AI 원인 진단** | 배포가 실패하면 실패 로그와 클러스터 상태를 모아 비밀값을 가린 뒤 Claude로 원인을 요약해 PR·Slack에 게시. 운영 환경 로그는 기본으로 외부 LLM에 보내지 않음 | 테스트 45개, demo-app 실제 실패로 Slack 전송 확인 |
| **yolo 배포 리포트** | 리뷰 없이 배포한 브랜치에서 Trivy 경고를 다시 찾아 그 브랜치가 새로 들여온 경고만 이슈로 등록 | 실제 스캔 21건 중 신규 1건만 이슈 대상으로 판정 |
| **비용 분석** | AWS·GCP 공개 가격 API로 월 비용·예산 경고·절감안을 계산. 조회하지 못한 비용을 0원으로 바꾸지 않음 | 테스트 191개, 실제 가격표 44개 조회 |
| **계층별 배포 위치** | 앱은 AWS, DB는 온프레미스에 두는 조합을 계약·검사·템플릿으로 지원 | 기존 렌더 결과 바이트 동일, 변이 시험 |

<sub>Python · Bash · Terraform · Helm · Argo Rollouts · GitHub Actions · AWS · GCP · Claude API · Trivy</sub>

#### 🐝 [Bee careful](https://github.com/baekyutae/portfolio) — 양봉장 영상으로 꿀벌 질병을 진단하는 서비스
`SW전문인재양성사업 최종 프로젝트` `팀장` `대상 수상`

요구사항 정의, ML 파이프라인과 백엔드 구현, 데이터 전처리를 맡았습니다.

- 비꿀벌 이미지에도 질병 판정이 나오는 문제를 보고 **YOLO로 꿀벌을 찾고 → 잘라낸 영역만 CNN으로 분류하는 2단계 구조** 설계
- 오분류 사례를 분석해 7개 클래스 중 진단 목적에 맞는 3개를 고르고 성충 영역 중심으로 데이터셋 재구성
- 테스트 이미지 192장에서 VGG16과 EfficientNet을 비교해 EfficientNet 선정: 정확도 **86%**(VGG16 81%), 질병 재현율 83%(78%), F1 0.82(0.75)

<sub>Python · Flask · YOLOv8 · EfficientNet · TensorFlow/Keras · OpenCV · MySQL</sub>

---

### 기술 스택

| 분야 | |
|---|---|
| Backend | ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white) ![Flask](https://img.shields.io/badge/Flask-000000?style=flat-square&logo=flask&logoColor=white) |
| Database · Search | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![pgvector](https://img.shields.io/badge/pgvector-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![pgmq](https://img.shields.io/badge/pgmq-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white) |
| AI · Data | ![Vertex AI](https://img.shields.io/badge/Vertex_AI-4285F4?style=flat-square&logo=googlecloud&logoColor=white) ![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat-square&logo=tensorflow&logoColor=white) ![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=flat-square&logo=opencv&logoColor=white) ![YOLO](https://img.shields.io/badge/YOLOv8-111F68?style=flat-square&logo=ultralytics&logoColor=white) |
| Infra | ![GCP](https://img.shields.io/badge/GCP-4285F4?style=flat-square&logo=googlecloud&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white) ![Terraform](https://img.shields.io/badge/Terraform-844FBA?style=flat-square&logo=terraform&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat-square&logo=helm&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) |
| Test | ![pytest](https://img.shields.io/badge/pytest-0A9EDC?style=flat-square&logo=pytest&logoColor=white) ![k6](https://img.shields.io/badge/k6-7D64FF?style=flat-square&logo=k6&logoColor=white) ![testcontainers](https://img.shields.io/badge/testcontainers-291A3F?style=flat-square&logo=docker&logoColor=white) |

---

### 수상 · 자격

| | |
|---|---|
| 🏆 | 2026 소프트뱅크 해커톤 예선전 우수상 (조 2위) |
| 🏆 | SW전문인재양성사업 대상 (단체상, 6개 조 중 1위) — Bee careful 팀장 |
| 🏅 | SW전문인재양성사업 Nexen 회장상 (개인상, 부트캠프 40명 중 성과 기반 수여) |
| 📜 | 정보처리기사 (2025) |
