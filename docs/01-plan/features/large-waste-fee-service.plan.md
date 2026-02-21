# Plan: 대형폐기물 수수료 조회 웹 서비스

> 작성일: 2026-02-20
> 최종 수정일: 2026-02-21
> 프로젝트명: Thowit (대형폐기물 배출 도우미)
> 참고 프로젝트: C:\Users\csj20\Desktop\throw_it

---

## 1. 프로젝트 개요

### 1.1 프로젝트 목적
전국 대형폐기물 수수료를 지역별로 조회하고, 온라인 배출 신청, 재활용 역경매, 오프라인 배출 안내 등 대형폐기물 처리와 관련된 통합 웹 서비스를 구축한다.

### 1.2 핵심 기능
| 기능 | 설명 | 인증 필요 |
|------|------|:---------:|
| 수수료 조회 | 시도/시군구별 대형폐기물 수수료 검색 | X |
| 온라인 배출 신청 | 4단계 배출 신청 프로세스 (정보입력→검토→결제→완료) | O |
| 마이페이지 | 신청 내역 조회, 상태 확인, 영수증 확인 | O |
| 재활용 역경매 | 재활용 가능 물품 등록/조회/상태 관리 | O |
| 오프라인 배출 안내 | 스티커 판매소, 주민센터, 처리시설, 운반대행 안내 | X |
| 사용자 인증 | 회원가입, 로그인, 세션 관리 | - |

### 1.3 데이터 규모
- 전국 17개 시도, 131개 시군구
- 대형폐기물 수수료 데이터: 22,819건
- 폐기물 처리시설 데이터: 9,113건 이상

---

## 2. 팀 구성 및 역할

| 역할 | 담당자 | 주요 업무 |
|------|--------|----------|
| Project Manager | 최세진 | 프로젝트 일정 관리, 코드 리뷰, 배포 관리 |
| Server | 최세진 | 서버 환경 구성, DB 서버 관리, 배포 환경 세팅 |
| 기획 | 강해원, 이재훈 | 서비스 기획, 와이어프레임, 사용자 흐름 정의 |
| Back-end | 최세진, 이재훈 | API 개발, DB 설계, 비즈니스 로직 구현 |
| Front-end | 최가을, 최은아 | UI 개발, API 연동, 상태 관리 |
| 디자인 | 강해원, 최은아 | UI/UX 디자인, 디자인 시스템, 반응형 레이아웃 |

---

## 3. 기술 스택

### 3.1 Backend
| 항목 | 기술 |
|------|------|
| 언어 | Java 17 |
| 프레임워크 | Spring Boot 3.4.x |
| ORM | Spring Data JPA + Hibernate |
| 데이터베이스 | MySQL 8+ |
| 빌드 도구 | Gradle (Kotlin DSL) |
| 라이브러리 | Lombok, Spring Validation |

### 3.2 Frontend
| 항목 | 기술 |
|------|------|
| 언어 | TypeScript 5.x |
| 프레임워크 | React 19.x |
| 빌드 도구 | Vite 7.x |
| CSS | Tailwind CSS 4.x |
| 라우팅 | React Router DOM 7.x |
| 상태관리 | Zustand 5.x + TanStack React Query 5.x |
| 폼 관리 | React Hook Form 7.x |
| 지도 | Kakao Maps SDK |
| 코드 품질 | ESLint + TypeScript-ESLint |

### 3.3 개발 환경
| 항목 | 기술 |
|------|------|
| Node.js | 18+ |
| npm | 9+ |
| 버전관리 | Git |
| 에디터 | VS Code |

---

## 4. 프로젝트 진행 계획 (단계별)

### Phase 1: 기획 (담당: 강해원, 이재훈)

#### 1-1. 서비스 기획서 작성 ✅ (2026-02-21 완료)
- [x] 프로젝트 목표 및 범위 정의
- [x] 타겟 사용자 분석 (일반 시민, 이사/청소업체, 재활용 관심 사용자)
- [x] 핵심 기능 명세서 작성 (F1~F6, 입력/출력/처리로직/예외처리 포함)
- [x] 사용자 흐름(User Flow) 정의 (5개 흐름)
  - 수수료 조회 흐름: 홈 → 시도 선택 → 시군구 선택 → 카테고리/검색 → 품목 선택 → 수수료 확인
  - 온라인 배출 신청 흐름: 홈 → 신청 안내 → 정보 입력 → 검토 → 결제 → 완료
  - 재활용 역경매 흐름: 홈 → 목록 → 등록/상세 → 상태 관리
  - 오프라인 안내 흐름: 홈 → 카테고리 선택 → 지역 선택 → 목록/지도 확인
  - 인증 흐름: 로그인/회원가입 → localStorage 저장 → 세션 유지
- **산출물**: `docs/01-plan/phase1-1-service-planning.md`

#### 1-2. 와이어프레임 작성
- [ ] 전체 17개 페이지 와이어프레임 설계
  1. 홈페이지 (메인 메뉴 5개 카드)
  2. 수수료 조회 페이지
  3. 오프라인 안내 메뉴 페이지
  4. 스티커 판매소 페이지 (지도 포함)
  5. 주민센터 페이지 (지도 포함)
  6. 운반 대행 페이지
  7. 온라인 배출 안내 페이지 (4단계 프로세스)
  8. 배출 신청 페이지 (폼)
  9. 신청 검토 페이지
  10. 결제 페이지
  11. 신청 완료 페이지
  12. 재활용 역경매 목록 페이지
  13. 재활용 물품 등록 페이지
  14. 로그인 페이지
  15. 회원가입 페이지
  16. 마이페이지 (신청 내역)
  17. 영수증 페이지

#### 1-3. 기능 정의서 작성
- [ ] 각 기능별 입력/출력/처리 로직 정의
- [ ] 예외 상황 정리 (미선택, 잘못된 입력, 인증 실패 등)
- [ ] 비인증 기능 vs 인증 필요 기능 구분

---

### Phase 2: 디자인 설계 (담당: 강해원, 최은아)

#### 2-1. 디자인 시스템 정의
- [ ] 컬러 팔레트 정의 (Primary, Secondary, 상태색)
- [ ] 타이포그래피 설정 (Font, Size, Weight)
- [ ] 간격 및 레이아웃 규칙 (Spacing, Grid)
- [ ] 모바일 퍼스트 디자인 기준 (최대 너비 428px)

#### 2-2. UI 컴포넌트 디자인
- [ ] 공통 컴포넌트 디자인
  - Button (Primary, Secondary, Outline, Disabled)
  - Card (기본, 선택형, 정보형)
  - Input (텍스트, 검색, 날짜)
  - Select (드롭다운)
  - Modal (알림, 확인)
  - Badge (상태 표시)
  - Header (뒤로가기 + 제목)
  - BottomNav (5개 메뉴)
  - ProgressBar (단계 진행)
- [ ] 폐기물 관련 컴포넌트 디자인
  - CategoryTree (카테고리 필터)
  - WasteSearchBar (품목 검색)
  - WasteItemCard (품목 정보 카드)
  - SizeSelector (크기/수량 선택)
  - FeeResultCard (수수료 결과)
- [ ] 지도 관련 컴포넌트 디자인
  - MapView (카카오맵 뷰)
  - LocationCard (위치 정보 카드)

#### 2-3. 페이지 디자인 (17개)
- [ ] 전체 페이지 고해상도 목업 제작
- [ ] 반응형 레이아웃 확인
- [ ] 인터랙션 및 전환 효과 정의
- [ ] 디자인 리뷰 및 피드백 반영

---

### Phase 3: DB 설계 (담당: 최세진, 이재훈)

#### 3-1. 데이터 모델링
- [ ] ERD(Entity Relationship Diagram) 작성
- [ ] 테이블 6개 설계:

| 테이블 | 설명 | 주요 컬럼 |
|--------|------|----------|
| `large_waste_fee` | 수수료 데이터 (22,819건) | 시도명, 시군구명, 대형폐기물명, 규격, 수수료 |
| `waste_facility` | 처리시설 데이터 (9,113건) | 시설명, 주소, 위도, 경도, 업종명, 전화번호 |
| `users` | 사용자 정보 | id, email, password, salt, nickname |
| `disposal_applications` | 배출 신청 | id, userId, sido, sigungu, status, totalFee |
| `disposal_items` | 배출 품목 (신청별) | id, application_id(FK), 품목명, 규격, 수량, 수수료 |
| `recycle_items` | 재활용 물품 | id, userId, title, photos(JSON), status |

#### 3-2. 인덱스 설계
- [ ] `large_waste_fee`: 시도명, 시군구명, 대형폐기물명
- [ ] `waste_facility`: 시설명, 업종명, 소재지도로명주소
- [ ] `recycle_items`: 시도명, 시군구명, status

#### 3-3. 데이터 준비 (일부 완료)
- [x] schema.sql 작성 (테이블 생성 DDL) ✅ → `backend/src/main/resources/sql/schema.sql`
- [ ] large_waste_fee_data.sql 준비 (공공데이터 22,819건) — 참고 프로젝트에서 복사 필요
- [ ] waste_facility_data.sql 준비 (공공데이터 9,113건) — 참고 프로젝트에서 복사 필요
- [x] 문자셋 설정: UTF-8 MB4 (utf8mb4_0900_ai_ci) ✅

---

### Phase 4: Backend 개발 (담당: 최세진, 이재훈)

#### 4-1. 프로젝트 초기 설정 ✅ (2026-02-20 완료)
- [x] Spring Boot 프로젝트 생성 → `ThowItApplication.java`
- [x] build.gradle.kts 의존성 설정 ✅
  - Spring Web, Spring Data JPA, MySQL Connector, Lombok, Validation
- [x] application.yml 설정 (포트 8080, DB 연결, JPA 설정) ✅
- [x] CORS 설정 (localhost:5173, 5174, 3000 허용) ✅ → `CorsConfig.java`
- [x] 글로벌 예외 처리기 구현 ✅ → `BusinessException.java`, `ErrorResponse.java`, `GlobalExceptionHandler.java`

#### 4-2. 도메인별 API 개발

**User 도메인 (최세진)**
- [ ] Entity: User (id, email, password, salt, nickname, createdAt)
- [ ] Repository: UserRepository
- [ ] Service: AuthService (회원가입, 로그인, 조회)
  - SHA-256 + Salt 패스워드 해싱
- [ ] Controller: AuthController
  - `POST /api/auth/signup` - 회원가입
  - `POST /api/auth/login` - 로그인
  - `GET /api/auth/me` - 내 정보 (X-User-Id 헤더)
- [ ] DTO: SignupRequest, LoginRequest, UserResponse

**Fee 도메인 (이재훈)**
- [ ] Entity: LargeWasteFee
- [ ] Repository: LargeWasteFeeRepository
- [ ] Service: LargeWasteFeeService
  - 시도/시군구 목록 조회
  - 카테고리 목록 조회
  - 품목 검색 (카테고리 + 키워드)
  - 수수료 조회 (null 규격 → 소형/중형/대형/특대형 자동 매핑)
- [ ] Controller: LargeWasteFeeController
  - `GET /api/regions/sido` - 시도 목록
  - `GET /api/regions/sigungu` - 시군구 목록
  - `GET /api/waste/categories` - 카테고리 목록
  - `GET /api/waste/items` - 품목 검색
  - `GET /api/fees` - 수수료 조회

**Disposal 도메인 (최세진)**
- [ ] Entity: DisposalApplication, DisposalItem, DisposalStatus(ENUM), PaymentMethod(ENUM)
- [ ] Repository: DisposalApplicationRepository
- [ ] Service: DisposalService
  - 신청 생성 (신청번호 자동 생성: GN-YYYYMMDD-00000)
  - 내 신청 목록, 상세 조회
  - 취소, 결제 처리
- [ ] Controller: DisposalController
  - `POST /api/disposals` - 신청 생성
  - `GET /api/disposals/my` - 내 신청 목록
  - `GET /api/disposals/{id}` - 신청 상세
  - `PATCH /api/disposals/{id}/cancel` - 취소
  - `POST /api/disposals/{id}/payment` - 결제

**Recycle 도메인 (이재훈)**
- [ ] Entity: RecycleItem, RecycleStatus(ENUM)
- [ ] Repository: RecycleItemRepository
- [ ] Service: RecycleService (CRUD, 사진 JSON 처리)
- [ ] Controller: RecycleController
  - `GET /api/recycle/items` - 목록 (시군구 필터)
  - `GET /api/recycle/items/my` - 내 물품
  - `POST /api/recycle/items` - 등록
  - `PATCH /api/recycle/items/{id}/status` - 상태 변경
  - `DELETE /api/recycle/items/{id}` - 삭제

**Offline 도메인 (이재훈)**
- [ ] Entity: WasteFacility
- [ ] Repository: WasteFacilityRepository
- [ ] Service: OfflineService
- [ ] Controller: OfflineController
  - `GET /api/offline/sticker-shops` - 스티커 판매소
  - `GET /api/offline/centers` - 주민센터
  - `GET /api/offline/transport` - 운반 대행
  - `GET /api/offline/waste-facilities` - 폐기물 처리시설

---

### Phase 5: Frontend 개발 (담당: 최가을, 최은아)

#### 5-1. 프로젝트 초기 설정 ✅ (2026-02-20 완료)
- [x] Vite + React + TypeScript 프로젝트 생성 ✅ → `package.json`
- [x] 의존성 정의 ✅ (npm install 필요)
  - react-router-dom, zustand, @tanstack/react-query
  - react-hook-form, tailwindcss, eslint
- [x] 경로 별칭 설정 (`@` → `./src`) ✅ → `vite.config.ts`, `tsconfig.app.json`
- [x] Tailwind CSS 설정 ✅ → `index.css` (디자인 시스템 테마 포함)
- [x] ESLint 설정 ✅ → `eslint.config.js`

#### 5-2. 공통 컴포넌트 개발 (최은아)
- [ ] Layout 컴포넌트: Header, BottomNav, MobileContainer, ProgressBar
- [ ] UI 컴포넌트: Button, Card, Input, Modal, DatePicker, Select, Badge, SearchBar
- [ ] 지도 컴포넌트: MapView, MapPlaceholder, LocationCard
- [ ] Kakao Maps 어댑터 패턴 구현 (Real + Mock)

#### 5-3. API 서비스 레이어 구현 (최가을)
- [ ] apiClient.ts (Base URL 설정, Fetch 래퍼, 에러 처리)
- [ ] authService.ts (회원가입, 로그인, 내 정보)
- [ ] regionService.ts (시도, 시군구 목록)
- [ ] wasteService.ts (품목 검색)
- [ ] feeService.ts (수수료 조회)
- [ ] disposalService.ts (배출 신청 CRUD)
- [ ] offlineService.ts (오프라인 시설 조회)
- [ ] recycleService.ts (재활용 물품 CRUD)

#### 5-4. 상태 관리 설정 (최가을)
- [ ] AuthContext (React Context) - 인증 상태, localStorage 연동
- [ ] useDisposalStore (Zustand) - 배출 신청 폼 상태
- [ ] useRegionStore (Zustand) - 지역 선택 캐시

#### 5-5. TypeScript 타입 정의 (최가을)
- [ ] auth.ts, fee.ts, disposal.ts, recycle.ts, offline.ts, waste.ts, region.ts

#### 5-6. 페이지 개발 (최가을 + 최은아 분담)

**최가을 담당 페이지:**
- [ ] HomePage (메인 메뉴 5개 카드)
- [ ] FeeCheckPage (수수료 조회 - 핵심 기능)
- [ ] OnlinePage (배출 안내)
- [ ] ApplyPage (배출 신청 폼)
- [ ] ReviewPage (신청 검토)
- [ ] PaymentPage (결제 UI)
- [ ] CompletePage (완료 확인)
- [ ] MyPage (신청 내역)
- [ ] ReceiptPage (영수증)

**최은아 담당 페이지:**
- [ ] LoginPage (로그인)
- [ ] SignupPage (회원가입)
- [ ] OfflinePage (오프라인 메뉴)
- [ ] StickerShopsPage (스티커 판매소 + 지도)
- [ ] CentersPage (주민센터 + 지도)
- [ ] TransportPage (운반 대행)
- [ ] RecyclePage (재활용 목록/관리)
- [ ] RegisterPage (재활용 등록)

#### 5-7. Feature 모듈 개발 (공통)
- [ ] auth: AuthContext, useAuth 훅
- [ ] fee: useFeeCheck 훅
- [ ] disposal: DisposalForm, ReviewSummary, PaymentForm, useDisposalForm
- [ ] mypage: ApplicationList, ApplicationCard, ReceiptView, StatusBadge, useMyApplications
- [ ] recycle: RecycleRegisterForm, RecycleItemCard, PhotoUploader, useRecycle

#### 5-8. 라우팅 설정 (최가을)
- [ ] React Router DOM 17개 라우트 설정
- [ ] App.tsx 레이아웃 (Header + Outlet + BottomNav)

---

### Phase 6: Frontend-Backend 연동 (담당: 전원)

#### 6-1. API 연동 테스트
- [ ] 인증 API 연동 (회원가입/로그인)
- [ ] 수수료 조회 API 연동
- [ ] 배출 신청 API 연동
- [ ] 재활용 API 연동
- [ ] 오프라인 시설 API 연동

#### 6-2. 통합 테스트
- [ ] 전체 사용자 흐름 시나리오 테스트
- [ ] 예외 상황 테스트 (인증 실패, 잘못된 입력, 서버 에러)
- [ ] 크로스 브라우저 테스트

---

### Phase 7: 테스트 및 QA (담당: 전원)

- [ ] 기능 테스트 (모든 페이지, 모든 API)
- [ ] UI/UX 검수 (디자인 일치 여부)
- [ ] 반응형 레이아웃 테스트 (모바일 기기)
- [ ] 성능 테스트 (22,819건 수수료 검색 속도)
- [ ] 보안 점검 (SQL Injection, XSS, 패스워드 해싱)
- [ ] 버그 수정 및 최종 확인

---

### Phase 8: 배포 (담당: 최세진)

- [ ] Frontend 빌드 및 배포 환경 구성
- [ ] Backend 빌드 및 서버 배포
- [ ] MySQL DB 서버 구성 및 데이터 이관
- [ ] 환경 변수 설정 (.env, application-local.yml)
- [ ] 도메인 연결 및 HTTPS 설정
- [ ] 최종 운영 테스트

---

## 5. 라우팅 구조 (전체 17개 페이지)

```
/ (App 레이아웃 - Header + Outlet + BottomNav)
├── /                    → HomePage (메인)
├── /fee-check           → FeeCheckPage (수수료 조회)
├── /offline             → OfflinePage (오프라인 메뉴)
├── /offline/sticker-shops → StickerShopsPage (스티커 판매소)
├── /offline/centers     → CentersPage (주민센터)
├── /offline/transport   → TransportPage (운반 대행)
├── /online              → OnlinePage (배출 안내)
├── /online/apply        → ApplyPage (배출 신청)
├── /online/review       → ReviewPage (검토)
├── /online/payment      → PaymentPage (결제)
├── /online/complete     → CompletePage (완료)
├── /recycle             → RecyclePage (재활용)
├── /recycle/register    → RegisterPage (물품 등록)
├── /login               → LoginPage (로그인)
├── /signup              → SignupPage (회원가입)
├── /mypage              → MyPage (마이페이지)
└── /mypage/receipt/:id  → ReceiptPage (영수증)
```

---

## 6. API 엔드포인트 (전체 19개)

| Method | Endpoint | 기능 | 인증 |
|--------|----------|------|:----:|
| POST | `/api/auth/signup` | 회원가입 | X |
| POST | `/api/auth/login` | 로그인 | X |
| GET | `/api/auth/me` | 내 정보 조회 | O |
| GET | `/api/regions/sido` | 시도 목록 | X |
| GET | `/api/regions/sigungu` | 시군구 목록 | X |
| GET | `/api/waste/categories` | 폐기물 카테고리 | X |
| GET | `/api/waste/items` | 폐기물 품목 검색 | X |
| GET | `/api/fees` | 수수료 조회 | X |
| POST | `/api/disposals` | 배출 신청 생성 | O |
| GET | `/api/disposals/my` | 내 신청 목록 | O |
| GET | `/api/disposals/{id}` | 신청 상세 | O |
| PATCH | `/api/disposals/{id}/cancel` | 신청 취소 | O |
| POST | `/api/disposals/{id}/payment` | 결제 처리 | O |
| GET | `/api/recycle/items` | 재활용 목록 | X |
| GET | `/api/recycle/items/my` | 내 재활용 물품 | O |
| POST | `/api/recycle/items` | 재활용 등록 | O |
| PATCH | `/api/recycle/items/{id}/status` | 상태 변경 | O |
| DELETE | `/api/recycle/items/{id}` | 물품 삭제 | O |
| GET | `/api/offline/*` | 오프라인 시설 (4종) | X |

---

## 7. 담당자별 작업 요약

### 최세진 (PM / Server / Backend)
1. 프로젝트 일정 관리 및 코드 리뷰
2. 서버 환경 구성 (MySQL, Spring Boot)
3. User 도메인 API 개발
4. Disposal 도메인 API 개발
5. CORS, 글로벌 예외 처리 설정
6. 배포 환경 구성

### 강해원 (기획 / 디자인)
1. 서비스 기획서 작성
2. 사용자 흐름(User Flow) 정의
3. 와이어프레임 작성
4. 디자인 시스템 정의
5. UI 컴포넌트 디자인
6. 전체 페이지 목업 제작

### 이재훈 (기획 / Backend)
1. 기능 정의서 작성 (강해원과 공동)
2. DB 설계 (최세진과 공동)
3. Fee 도메인 API 개발
4. Recycle 도메인 API 개발
5. Offline 도메인 API 개발

### 최가을 (Frontend)
1. Frontend 프로젝트 초기 설정
2. API 서비스 레이어 구현 (7개 서비스)
3. 상태 관리 설정 (Context, Zustand, React Query)
4. TypeScript 타입 정의
5. 핵심 페이지 개발 (9개: 홈, 수수료, 배출 신청 플로우, 마이페이지)
6. 라우팅 설정

### 최은아 (Frontend / 디자인)
1. 공통 컴포넌트 개발 (Layout, UI, Map)
2. 디자인 시스템 Tailwind CSS 구현 (강해원과 협업)
3. 페이지 개발 (8개: 인증, 오프라인, 재활용)
4. Kakao Maps 어댑터 구현

---

## 8. 작업 순서 (진행 로드맵)

```
[Step 1] 기획 단계                          ← 🔄 진행 중
  ├── ✅ 서비스 기획서 작성 (2026-02-21 완료)
  ├── ✅ 사용자 흐름 정의 (2026-02-21 완료)
  └── ⬜ 와이어프레임 작성 (강해원, 이재훈)    ← 다음 작업
         │
[Step 2] 설계 단계 (기획 완료 후)             ← ⏳ 대기
  ├── ⬜ 디자인 시스템 정의 (강해원, 최은아)
  ├── ⬜ UI/페이지 디자인 (강해원, 최은아)
  └── ⬜ DB 설계 및 ERD 작성 (최세진, 이재훈)
         │
[Step 3] 환경 구축                          ← ✅ 완료
  ├── ✅ Backend 프로젝트 생성 및 설정 (최세진) — 2026-02-20
  ├── ✅ Frontend 프로젝트 생성 및 설정 (최가을) — 2026-02-20
  ├── ✅ schema.sql 작성 완료 — 2026-02-20
  ├── ⬜ MySQL DB 생성 및 스키마 적용 (최세진)
  └── ⬜ 공공데이터 SQL Import (최세진)
         │
[Step 4] 핵심 개발 (병렬 진행)                ← ⏳ 대기
  ├── ⬜ Backend API 개발 (최세진: User+Disposal / 이재훈: Fee+Recycle+Offline)
  ├── ⬜ Frontend 공통 컴포넌트 (최은아)
  ├── ⬜ Frontend API 서비스 레이어 (최가을)
  └── ⬜ Frontend 상태 관리 + 타입 정의 (최가을)
         │
[Step 5] 페이지 개발 (Backend API + 공통 컴포넌트 완료 후)
  ├── ⬜ 최가을: 홈, 수수료 조회, 배출 신청 플로우 (9페이지)
  └── ⬜ 최은아: 인증, 오프라인 안내, 재활용 (8페이지)
         │
[Step 6] 연동 및 통합 (전원)
  ├── ⬜ Frontend-Backend API 연동
  ├── ⬜ 전체 플로우 테스트
  └── ⬜ 버그 수정
         │
[Step 7] QA 및 테스트 (전원)
  ├── ⬜ 기능 테스트
  ├── ⬜ UI/UX 검수
  ├── ⬜ 성능 테스트
  └── ⬜ 보안 점검
         │
[Step 8] 배포 (최세진)
  ├── ⬜ Frontend 배포
  ├── ⬜ Backend 배포
  ├── ⬜ DB 이관
  └── ⬜ 최종 운영 테스트
```

---

## 9. 개발 서버 환경

| 항목 | 설정 |
|------|------|
| Backend 서버 | `http://localhost:8080` |
| Frontend 서버 | `http://localhost:5173` |
| DB 서버 | `localhost:3306/waste_db` |
| CORS 허용 | localhost:5173, 5174, 3000 |

---

## 10. 참고 사항

- **참고 프로젝트**: `C:\Users\csj20\Desktop\throw_it` 의 구조, 기술 스택, 코드 패턴을 기준으로 개발
- **모바일 퍼스트**: 최대 너비 428px 기반 반응형 디자인
- **인증 방식**: X-User-Id 헤더 기반 (참고 프로젝트 동일)
- **패스워드 보안**: SHA-256 + 16byte Salt + Base64 인코딩
- **공공데이터 활용**: 대형폐기물 수수료 + 폐기물 처리시설 데이터
- **지도 API**: Kakao Maps SDK (어댑터 패턴으로 Mock 전환 가능)

---

## 11. 관련 문서

| 문서 | 경로 | 설명 |
|------|------|------|
| 요구사항 | `rule.md` | 원본 요구사항 |
| 서비스 기획서 | `docs/01-plan/phase1-1-service-planning.md` | Phase 1-1 산출물 |
| 실행 가이드 | `docs/02-design/features/large-waste-fee-service.design.md` | 코드 패턴, 단계별 가이드 |
| 진행 결과 기록 | `docs/05-progress/thowit.progress.md` | Phase별 작업 결과 기록 |
| 진행 상황 요약 | `PROGRESS.md` | AI 세션별 진행 기록 |
| 참고 프로젝트 | `C:\Users\csj20\Desktop\throw_it` | 완성된 예시 코드 |
