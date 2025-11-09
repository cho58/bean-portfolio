# Bean (빈자리) – 실시간 카페 좌석 정보 & 큐레이션 서비스

## 1. Overview

**Bean(빈자리)**는 카페 이용자에게는 실시간 좌석 정보를 제공하고,  
카페 사장님에게는 좌석/회전율 관리와 데이터 기반 인사이트를 제공하는  
웹/모바일 기반 카페 큐레이션 & 좌석 관리 서비스입니다.

- 기간: 2024.06 ~ 2024.08
- 역할: 백엔드 개발 90%, 프론트엔드 10%, 아키텍처 설계, 배포
- 팀 구성: 3인 (디자인 1, FE 2, BE/AI 2) 


## 2. Problem Definition

- 카페에 가기 전, **“자리가 있을까?”를 알 수 있는 서비스 부재**
- 네이버/카카오 지도는 위치, 리뷰, 별점은 제공하지만
  - 실시간 좌석 수, 콘센트 유무, 공부/수다에 적합한지 등의 정보는 없음
- 카페 사장님 입장에서는
  - 좌석별 이용 패턴, 피크타임, 회전율 등 **데이터 기반 운영 지표 부족**


## 3. Solution & Key Features

### 사용자(게스트) 기능

- 현재 위치 기반 **카페 리스트 & 실시간 좌석 수 조회**
- 분위기/목적별 필터
  - `조용한/스터디용`, `수다용`, `콘센트 많음`, `단체 좌석 가능` 등
- 카페 상세 페이지
  - 플로어플랜 기반 실좌석 배치
  - 좌석별 상태 (비어 있음 / 사용 중 / 예약됨)

### 사장님(Owner) 기능

- 카페 등록/수정
- 플로어플랜 이미지 업로드
  - 업로드 → 좌석 자동 감지 → 테이블/의자 좌표 저장
  - 드래그&드롭으로 좌석 위치 미세조정
- 좌석 상태 관리 (수동/자동 이벤트 연동)
- 기본 통계
  - 시간대별 이용률, 요일별 피크타임, 좌석 유형별 선호도 등 *(구현 범위에 맞게 조정)*


## 4. Tech Stack

### Backend

- **Django / Django REST Framework**
- JWT Cookie 기반 인증 (access / refresh)

### Frontend / Mobile

- React / Next.js or React Native (Expo)
- Axios/TanStack Query for API 통신
- Kakao Map API로 카페 위치 표시

### ML / Detection

- Roboflow API (또는 Grounding DINO 등)로 플로어플랜 좌석 감지
- 감지 결과를 (x, y, width, height) 형태로 DB 저장
- 좌석 좌표를 프론트에서 SVG/Canvas로 렌더링

### Infra / DevOps

- Docker / Docker Compose
- AWS EC2 로 배포
- GitHub Actions로 CI (lint/test)

## 5. Database & Domain Model

주요 엔티티:

- **User**: User / Owner 권한 구분
- **Cafe**: 카페 기본 정보, 위치, 영업 시간, 태그(분위기 등)
- **FloorPlan**: 카페 한 층의 도면 이미지 및 메타 정보
- **Table / Chair**: 좌석 위치, 상태, 좌석 유형
- **SeatStatusLog**: 좌석 이용 히스토리 (시작/종료 시간)

<img width="1658" height="750" alt="image" src="https://github.com/user-attachments/assets/90b0eefc-56e6-482e-92e5-7ce5e858f3e5" />


## 6. Key Technical Challenges & How I Solved Them

### 6-1. JWT Cookie 기반 인증 & 프론트 연동

- 문제: 모바일/웹 환경에서 토큰 만료 시 401 연속 발생, 리프레시 흐름 꼬임
- 해결:
  - Django에서 `httponly`/`secure` 옵션을 가진 Refresh Token Cookie 사용
  - 프론트에서 Axios 인터셉터로 401 감지 → `/auth/refresh` 요청 → 재시도
  - 비동기 요청 여러 개가 동시에 401 나는 경우, **refresh 요청은 한 번만 보내도록 로직 직렬화**

### 6-2. 플로어플랜 이미지 기반 좌석 자동 감지

- 문제: 사장님이 직접 좌석 하나씩 등록하면 너무 번거롭고 실수 발생
- 해결:
  - Roboflow API를 통해 업로드된 플로어플랜 이미지에서
    - 테이블/좌석 bounding box 감지
  - 감지된 box를 `(x, y, width, height)`로 변환하여 DB에 저장
  - 프론트에서 해당 좌표를 기반으로 SVG/Canvas에 좌석 오버레이
  - 이후 사장님이 드래그로 좌표 미세조정 후 저장 가능하게 구현


## 7. Result & Learnings

- REST API 설계/문서화, JWT 기반 인증/권한 분리 경험
- 실제 서비스 관점에서
  - “유저에게 어떤 정보가 진짜 의미 있는가?”를 고민하게 됨
  - 사장님/게스트 **두 종류의 사용자 UX를 동시에 고려하는 연습**
- ML/Detection 결과를 서비스 레벨에서 **도메인 모델로 녹여내는 작업** 경험
- 추후에는
  - 추천 시스템 (선호 카페/시간대 기반 개인화 추천)
  - 좌석 혼잡도 예측 (이전 패턴 기반) 기능까지 확장할 계획


## 8. Links

- Backend Repository: [https://github.com/ajy121650/beanBack]
- Frontend Repository: [https://github.com/eileen914/Bean_Front_Clean]
