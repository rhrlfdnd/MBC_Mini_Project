# Mood Code 🎨

### YOLO-World 객체탐지 기반 인테리어 상품 탐색 서비스

> **원하는 분위기를 선택하고, 공간 이미지 속 상품을 직접 탐색하는 쇼핑 서비스**  
> 사용자가 무드별 인테리어 공간을 둘러보고 이미지 위 핫스팟을 통해 상품 상세페이지로 이동할 수 있도록 구현한 **4인 팀 프로젝트**입니다.

[🌐 배포 서비스](https://mood-code-latest.onrender.com/) · [📘 상세 포트폴리오](https://app.notion.com/p/40e6d9f6c5f5819283ded2a2763c34db) · [👥 팀 저장소](https://github.com/mood-code-team/flask-team-project)

---

## 1. 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 프로젝트명 | Mood Code |
| 개발 기간 | 2026년 8월 |
| 개발 형태 | 4인 팀 프로젝트 |
| 서비스 유형 | 무드 기반 인테리어 공간·상품 탐색 웹 서비스 |
| 개인 담당 | 프론트엔드 UI·CSS 공동 개선, YOLO-World 탐지 결과 기반 상품 핫스팟 연동, 통합 검수 |
| 배포 | [Render 서비스](https://mood-code-latest.onrender.com/) |
| Docker | `docker pull gygs1090/mood-code:latest` |

### 기획 배경

일반적인 쇼핑몰이 상품명과 카테고리 검색에서 출발한다면, Mood Code는 **어떤 분위기의 공간을 원하는가**에서 출발합니다. 사용자는 **BLOOM · CLEAR · CALM · CHIC** 네 가지 무드를 선택하고 거실·다이닝·침실·발코니 공간을 탐색할 수 있습니다. 총 **4개 무드 × 4개 공간 = 16개 공간 시나리오**를 구성했습니다.

## 2. 주요 기능

다음은 **팀 전체가 구현한 서비스 기능**입니다.

- **무드·공간 갤러리:** 원하는 분위기와 공간을 선택해 인테리어 이미지 탐색
- **이미지 속 상품 핫스팟:** 공간 이미지 위 `+` 표시를 선택해 연결 상품 확인
- **상품 탐색:** 카테고리·검색·상품 상세 정보 확인
- **쇼핑 기능:** 회원, 장바구니, 주문, 위시리스트, 쿠폰·포인트, 리뷰·문의 등
- **관리자 기능:** 상품·주문·공지·문의·회원 관리

**사용자 흐름**

```text
무드 선택 → 공간 갤러리 → 공간 상세 이미지
                              ↓
                     상품 위치 핫스팟 선택
                              ↓
                     상품 상세 → 장바구니·주문
```

## 3. 기술 스택

| 구분 | 기술 |
| --- | --- |
| Backend | Python, Flask, Flask Blueprint, SQLAlchemy |
| Frontend | HTML, CSS, JavaScript |
| AI / Data | YOLO-World, Pandas, CSV |
| Database | SQLite / MySQL 관련 프로젝트 구성 |
| Collaboration / Deployment | Git, GitHub, Docker, Render |

### 서비스 구조

```text
Browser (HTML / CSS / JavaScript)
              ↓
      Flask Blueprint (Route)
              ↓
       Service / Business Logic
              ↓
      SQLAlchemy Model / Database
```

공간 이미지 핫스팟은 객체탐지 결과와 상품 매핑 데이터를 활용해 Flask 화면에 표시합니다.

## 4. 나의 담당 역할

> **핵심 기여:** YOLO-World로 탐지한 객체 위치를 실제 상품 정보와 연결하고, 사용자가 공간 이미지에서 상품을 선택할 수 있도록 핫스팟 기능을 연동했습니다.

- **프론트엔드 공동 작업:** CSS, 화면 레이아웃, 반응형 UI 및 사용자 인터랙션 보완
- **공간 화면 개선:** 공간 리스트·상세 화면의 이미지·상품 표시 수정 및 동작 검수
- **객체탐지 연동:** YOLO-World Bounding Box를 이미지 내 상대 좌표로 변환하고 CSV·상품 코드와 연결
- **팀 협업:** Git 병합 충돌 대응, 기능 통합 과정의 UI 오류 수정·검수

회원·주문·상품 DB·관리자 기능 등은 **팀 전체 서비스 범위**이며 개인 단독 구현으로 표현하지 않습니다.

## 5. 핵심 구현 — 객체탐지 결과를 상품 핫스팟으로 연결

### 구현 배경

이미지에서 가구를 탐지하는 것만으로는 사용자가 상품을 구매할 수 없습니다. **탐지한 위치와 실제 상품 데이터를 연결하고, 화면 크기가 달라도 핫스팟이 올바른 위치에 표시되도록** 처리해야 했습니다.

### 처리 과정

1. 공간 이미지와 연결할 상품 코드를 준비합니다.
2. YOLO-World로 가구·소품을 탐지하고 Bounding Box 좌표를 추출합니다.
3. Bounding Box 중심점을 이미지 크기 대비 백분율 좌표로 변환합니다.
4. 공간(Scene)·상품 코드·핫스팟 좌표를 CSV 데이터로 관리합니다.
5. Flask에서 상품 정보를 조회해 공간 상세 화면에 연결합니다.
6. 사용자가 이미지 위 `+` 핫스팟을 선택하면 해당 상품의 정보를 확인할 수 있도록 구현합니다.

```text
공간 이미지
   ↓
YOLO-World 객체탐지
   ↓
Bounding Box 중심점 계산
   ↓
이미지 크기 대비 좌표 정규화 (%)
   ↓
CSV: 공간 / 상품 코드 / 핫스팟 위치
   ↓
Flask 상품 데이터 연동
   ↓
공간 상세 화면의 '+' 핫스팟
```

### 좌표 변환 원리

아래 코드는 **핵심 계산 원리를 설명하는 예시**이며, 저장소의 특정 함수 원문을 그대로 인용한 것은 아닙니다.

```python
x_percent = ((x1 + x2) / 2) / image_width * 100
y_percent = ((y1 + y2) / 2) / image_height * 100
```

절대 픽셀 좌표 대신 이미지 크기에 대한 비율을 사용해, 이미지가 다른 크기로 표시되더라도 상대적인 상품 위치를 표현하도록 설계했습니다.

## 6. 문제 해결 — 작은 객체 탐지와 위치 오차

**문제**  
소파나 테이블처럼 큰 가구에 비해 작은 소품은 탐지 누락이나 위치 오차가 발생했습니다. 탐지 결과를 그대로 사용하면 상품 핫스팟이 잘못된 위치에 나타나거나 연결할 상품이 누락될 수 있었습니다.

**해결**  
탐지 결과를 직접 확인하고, 필요한 경우 위치 좌표를 수동 보정했습니다. 이후 상품 코드·CSV 매핑과 실제 공간 상세 화면을 함께 검수해 사용자가 선택하는 위치와 상품 정보가 맞도록 보완했습니다.

**결과와 배운 점**  
객체탐지 결과를 실제 웹 서비스의 상품 선택 인터랙션으로 연결했습니다. 이 과정에서 **모델의 출력 정확도와 서비스에서 사용할 수 있는 데이터 품질은 별도로 검증해야 한다**는 점을 배웠습니다.

> 수동 검수·좌표 보정은 서비스 연결 품질을 보완한 작업입니다. 별도의 정량 비교 실험이 없어 **YOLO-World 모델 자체의 탐지 정확도가 향상됐다고 주장하지 않습니다.**

## 7. 로컬 실행

아래는 기본적인 실행 절차입니다. 실제 환경에 따라 `.env` 설정과 데이터베이스 준비가 필요할 수 있습니다.

```bash
git clone https://github.com/rhrlfdnd/MBC_Mini_Project.git
cd MBC_Mini_Project
python -m venv .venv
```

가상환경 활성화:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

의존성 설치 및 실행:

```bash
pip install -r requirements.txt
python hspace_server.py
```

브라우저에서 `http://127.0.0.1:5000/`에 접속합니다.

**주의:** 저장소 버전과 로컬 데이터·환경 설정에 따라 초기화 작업이 추가로 필요할 수 있습니다. `.env.example`이 있는 경우 이를 참고해 개인 환경변수를 설정하고, **실제 비밀번호·API 키·DB 접속 정보는 커밋하지 마세요.**

## 8. 프로젝트 구조

```text
MBC_Mini_Project/
├── app.py               # Flask 애플리케이션
├── hspace_server.py     # 서버 실행
├── routes/              # 라우팅
├── services/            # 비즈니스 로직
├── models/              # 데이터 모델
├── templates/           # HTML 템플릿
├── static/              # CSS, JavaScript, 이미지
├── data/csv/            # 상품·공간 관련 CSV
├── data-analysis/       # 데이터 수집·정제 관련 코드
├── scripts/             # 데이터 준비 및 유틸리티
└── docs/                # 프로젝트 문서
```

## 9. 관련 링크

- **배포 서비스:** https://mood-code-latest.onrender.com/
- **상세 Notion 포트폴리오:** https://app.notion.com/p/40e6d9f6c5f5819283ded2a2763c34db
- **팀 원본 저장소:** https://github.com/mood-code-team/flask-team-project
- **개인 프로젝트 저장소:** https://github.com/rhrlfdnd/MBC_Mini_Project
- **Docker Hub:** https://hub.docker.com/r/gygs1090/mood-code

---

**프로젝트 성격:** 4인 팀 프로젝트 · 개인 핵심 담당: 프론트엔드 UI 공동 개선 및 YOLO-World 객체탐지 결과 기반 상품 핫스팟 연동.
