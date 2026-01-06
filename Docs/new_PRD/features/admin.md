# 관리자 페이지 (Admin Dashboard)

## 요약 ⚡

- 별도 웹 대시보드 형태의 관리자 전용 페이지 (React/Next.js)
- 사용자 관리, 서비스 통계, API 모니터링 기능 제공
- `ADMIN` 역할 사용자만 접근 가능 (JWT role 검증)
- [P1] 사용자 관리 + 기본 통계 + API 모니터링
- [P2] 멤버십 관리 + 알림 발송 / [P3] 고급 분석 + 포트폴리오 모니터링

---

## Phase 1 (현재)

### 기능 목록

- [ ] 관리자 로그인 (Google OAuth + ADMIN 역할 검증)
- [ ] 사용자 목록 조회 (검색, 필터링, 페이지네이션)
- [ ] 사용자 상세 조회
- [ ] 사용자 역할 변경 (USER ↔ ADMIN)
- [ ] 사용자 상태 관리 (활성화/비활성화)
- [ ] 대시보드 통계 (전체 사용자, 신규 가입자, 활성 사용자)
- [ ] API 호출 모니터링 (한투 API 호출 현황)
- [ ] 에러 로그 조회

---

### 1. 관리자 인증

#### 로그인 플로우

```
1. 관리자 웹에서 Google OAuth 로그인
2. 서버에서 users 테이블 role 확인
3. role = 'ADMIN'인 경우만 JWT 발급
4. role = 'USER'인 경우 접근 거부 (403 Forbidden)
```

| 항목 | 규칙 |
|------|------|
| **인증 방식** | 기존 앱과 동일한 Google OAuth 2.0 |
| **권한 검증** | JWT 토큰 내 role = 'ADMIN' 확인 |
| **토큰 정책** | Access Token 1시간, Refresh Token 14일 (앱과 동일) |
| **접근 거부** | "관리자 권한이 없습니다" 메시지 표시 + 일반 사용자용 앱 안내 |
| **세션 유지** | 브라우저 localStorage에 토큰 저장 (Secure) |

#### 보안 요구사항

| 항목 | 규칙 |
|------|------|
| **HTTPS** | 프로덕션 환경 필수 |
| **CORS** | 관리자 도메인만 허용 |
| **Rate Limiting** | 로그인 시도 5회/분 per IP |
| **로그 기록** | 모든 관리자 액션 admin_logs 테이블에 기록 |

---

### 2. 사용자 관리

#### 2.1 사용자 목록 조회

| 표시 정보 | 설명 |
|----------|------|
| **이메일** | 사용자 이메일 주소 |
| **닉네임** | 사용자 닉네임 |
| **역할** | USER / ADMIN 배지 표시 |
| **멤버십** | NOT / PRO 배지 표시 |
| **상태** | 활성 / 비활성 표시 |
| **가입일** | YYYY-MM-DD 형식 |
| **마지막 접속** | YYYY-MM-DD HH:mm 또는 "N일 전" |
| **포트폴리오 수** | 보유 포트폴리오 개수 |

| 기능 | 규칙 |
|------|------|
| **검색** | 이메일, 닉네임으로 검색 |
| **필터링** | 역할(전체/USER/ADMIN), 멤버십(전체/NOT/PRO), 상태(전체/활성/비활성) |
| **정렬** | 가입일, 마지막 접속일, 닉네임 (오름/내림차순) |
| **페이지네이션** | 페이지당 20/50/100건 선택 가능 |
| **CSV 내보내기** | 현재 필터 조건 기준 CSV 다운로드 |

#### 2.2 사용자 상세 조회

| 섹션 | 표시 정보 |
|------|----------|
| **기본 정보** | 프로필 사진, 이메일, 닉네임, 역할, 멤버십, 상태 |
| **계정 정보** | 가입일, 마지막 접속일, 로그인 횟수 |
| **활동 현황** | 포트폴리오 수, 총 종목 수, 연동 계좌 수 |
| **포트폴리오 목록** | 포트폴리오명, 종목 수, 생성일 (상세 조회 불가, 개인정보 보호) |

#### 2.3 사용자 역할 변경

```
플로우:
1. 사용자 상세 페이지에서 '역할 변경' 버튼 클릭
2. 변경할 역할 선택 (USER / ADMIN)
3. 확인 다이얼로그: "정말 [닉네임]님의 역할을 [역할]로 변경하시겠습니까?"
4. 확인 시 서버 업데이트 → admin_logs 기록
5. 성공 토스트: "역할이 변경되었습니다"
```

| 항목 | 규칙 |
|------|------|
| **변경 가능 역할** | USER ↔ ADMIN |
| **자기 자신** | 본인 역할 변경 불가 (보안) |
| **로그 기록** | 변경 전/후 역할, 변경자, 변경 시각 기록 |
| **알림** | 역할 변경 시 해당 사용자에게 이메일 알림 (선택) |

#### 2.4 사용자 상태 관리

| 항목 | 규칙 |
|------|------|
| **상태 종류** | 활성 (is_active = true) / 비활성 (is_active = false) |
| **비활성화 효과** | 앱 로그인 차단, JWT 발급 거부 |
| **비활성화 시** | 확인 다이얼로그 + 사유 입력 (필수) |
| **활성화 시** | 확인 다이얼로그 (사유 입력 선택) |
| **로그 기록** | 상태 변경 사유, 변경자, 변경 시각 기록 |

---

### 3. 대시보드/통계

#### 3.1 메인 대시보드

| 카드 | 내용 |
|------|------|
| **전체 사용자** | 총 가입자 수, 전일 대비 증감 (+n명) |
| **오늘 신규 가입** | 오늘 가입한 사용자 수 |
| **활성 사용자 (DAU)** | 오늘 접속한 사용자 수 |
| **전체 포트폴리오** | 총 포트폴리오 수 |
| **전체 종목 수** | 등록된 총 종목 수 (중복 포함) |
| **연동 계좌 수** | 활성 연동 계좌 수 |

#### 3.2 차트/그래프

| 차트 | 내용 |
|------|------|
| **가입자 추이** | 최근 30일 일별 신규 가입자 수 (라인 차트) |
| **활성 사용자 추이** | 최근 30일 DAU (라인 차트) |
| **역할 분포** | USER vs ADMIN 비율 (도넛 차트) |
| **멤버십 분포** | NOT vs PRO 비율 (도넛 차트) |

#### 3.3 통계 API

```
GET /api/admin/stats/overview
```

**Response**:
```json
{
  "totalUsers": 1234,
  "todayNewUsers": 15,
  "dailyActiveUsers": 456,
  "totalPortfolios": 2345,
  "totalStocks": 12345,
  "connectedAccounts": 789,
  "usersByRole": {
    "USER": 1230,
    "ADMIN": 4
  },
  "usersByMembership": {
    "NOT": 1200,
    "PRO": 34
  },
  "lastUpdated": "2026-01-06T18:00:00Z"
}
```

---

### 4. API 모니터링

#### 4.1 한투 API 호출 현황

| 표시 정보 | 설명 |
|----------|------|
| **오늘 호출 횟수** | 당일 총 API 호출 수 |
| **일일 호출 제한** | 한투 API 일일 호출 제한 (예: 10,000회) |
| **남은 호출 횟수** | 제한 - 현재 호출 수 |
| **사용률** | 호출 수 / 제한 × 100 (%) |
| **최근 7일 추이** | 일별 호출 횟수 차트 |

| 알림 기준 | 동작 |
|----------|------|
| **80% 도달** | 경고 배너 표시 (노란색) |
| **95% 도달** | 위험 배너 표시 (빨간색) + Slack 알림 |
| **100% 도달** | 차단 상태 표시 + 긴급 Slack 알림 |

#### 4.2 에러 로그 조회

| 표시 정보 | 설명 |
|----------|------|
| **발생 시각** | YYYY-MM-DD HH:mm:ss |
| **에러 레벨** | ERROR / WARN / INFO |
| **에러 코드** | HTTP 상태 코드 또는 커스텀 코드 |
| **메시지** | 에러 메시지 요약 |
| **요청 경로** | API 엔드포인트 |
| **사용자** | 에러 발생 사용자 (있는 경우) |

| 기능 | 규칙 |
|------|------|
| **기간 필터** | 오늘, 최근 7일, 최근 30일, 커스텀 기간 |
| **레벨 필터** | 전체 / ERROR / WARN / INFO |
| **검색** | 에러 메시지 키워드 검색 |
| **상세 보기** | 클릭 시 스택 트레이스, 요청 상세 정보 표시 |
| **보관 기간** | 30일 (30일 경과 시 자동 삭제) |

---

## Phase 2+ (확장 고려사항)

### [P2] 멤버십 관리

- 멤버십 현황 대시보드 (NOT/PRO 분포, 전환율)
- 수동 멤버십 업그레이드/다운그레이드
- 멤버십 변경 이력 조회
- 멤버십 프로모션 코드 발급

**P1 설계 시 고려사항**:
- [P2-고려] admin_logs 테이블에 membership_change 액션 타입 예약
- [P2-고려] membership_history 테이블 구조 미리 설계

### [P2] 알림 관리

- 전체 공지 푸시 알림 발송
- 타겟 알림 발송 (특정 조건 사용자 대상)
- 알림 템플릿 관리
- 알림 발송 이력 및 통계 (발송 수, 열람률)

**P1 설계 시 고려사항**:
- [P2-고려] admin_notifications 테이블 구조 미리 설계
- [P2-고려] FCM 토큰 관리 API 준비

### [P2] 포트폴리오 모니터링

- 전체 포트폴리오 현황 통계
- 인기 종목 TOP 10 (가장 많이 등록된 종목)
- 리밸런싱 실행 통계

### [P3] 공유 포트폴리오 관리

- 공개 포트폴리오 목록 조회
- 부적절한 포트폴리오 신고 처리
- 관리자 추천 포트폴리오 설정

### [P3] 고급 분석

- 사용자 행동 분석 (리밸런싱 실행률, 기능 사용률)
- 코호트 분석 (리텐션, 이탈률)
- 비즈니스 지표 대시보드 (전환율, LTV)

---

## API 엔드포인트

### 인증

```
POST /api/admin/auth/login     # 관리자 로그인
POST /api/admin/auth/logout    # 관리자 로그아웃
POST /api/admin/auth/refresh   # 토큰 갱신
```

### 사용자 관리

```
GET    /api/admin/users                # 사용자 목록 조회
GET    /api/admin/users/{id}           # 사용자 상세 조회
PATCH  /api/admin/users/{id}/role      # 역할 변경
PATCH  /api/admin/users/{id}/status    # 상태 변경 (활성화/비활성화)
GET    /api/admin/users/export         # CSV 내보내기
```

### 통계

```
GET    /api/admin/stats/overview       # 전체 통계
GET    /api/admin/stats/users          # 사용자 통계 (차트 데이터)
GET    /api/admin/stats/portfolios     # 포트폴리오 통계
```

### 모니터링

```
GET    /api/admin/monitoring/api       # API 호출 현황
GET    /api/admin/monitoring/errors    # 에러 로그 조회
GET    /api/admin/monitoring/errors/{id}  # 에러 상세 조회
```

### 관리자 로그

```
GET    /api/admin/logs                 # 관리자 활동 로그 조회
```

---

## DB 스키마 변경

### users 테이블 수정

```sql
-- 추가 컬럼
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT true;
ALTER TABLE users ADD COLUMN last_login_at TIMESTAMP NULL;
ALTER TABLE users ADD COLUMN login_count INT DEFAULT 0;

-- 인덱스
CREATE INDEX idx_users_is_active ON users(is_active);
CREATE INDEX idx_users_last_login ON users(last_login_at);
```

### admin_logs 테이블 (신규)

```sql
CREATE TABLE admin_logs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  admin_user_id INT NOT NULL,
  action VARCHAR(100) NOT NULL,
  target_type VARCHAR(50),
  target_id INT,
  before_value JSON,
  after_value JSON,
  reason TEXT,
  ip_address VARCHAR(45),
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (admin_user_id) REFERENCES users(id),
  INDEX idx_admin_logs_action (action),
  INDEX idx_admin_logs_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### api_call_logs 테이블 (신규)

```sql
CREATE TABLE api_call_logs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  api_type VARCHAR(50) NOT NULL,
  endpoint VARCHAR(255),
  status_code INT,
  response_time_ms INT,
  user_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_api_logs_type_date (api_type, created_at),
  INDEX idx_api_logs_user (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### error_logs 테이블 (신규)

```sql
CREATE TABLE error_logs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  level ENUM('ERROR', 'WARN', 'INFO') NOT NULL,
  error_code VARCHAR(50),
  message TEXT NOT NULL,
  stack_trace TEXT,
  request_path VARCHAR(255),
  request_method VARCHAR(10),
  request_body JSON,
  user_id INT,
  ip_address VARCHAR(45),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_error_logs_level (level),
  INDEX idx_error_logs_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 기술 스택

| 레이어 | 기술 |
|--------|------|
| **Frontend** | Next.js 14 (App Router) + TypeScript |
| **UI 라이브러리** | shadcn/ui + Tailwind CSS |
| **차트** | Chart.js 또는 Recharts |
| **상태 관리** | React Query (TanStack Query) |
| **인증** | NextAuth.js + Google OAuth |
| **Backend** | 기존 Spring Boot API 확장 (Admin 전용 엔드포인트) |

---

## 화면 구성

```
📁 관리자 대시보드
│
├── 📄 로그인 페이지
│
├── 📁 대시보드 (/)
│   ├── 통계 카드
│   ├── 가입자 추이 차트
│   └── 활성 사용자 차트
│
├── 📁 사용자 관리 (/users)
│   ├── 사용자 목록
│   ├── 검색/필터
│   └── 사용자 상세 모달
│
├── 📁 모니터링 (/monitoring)
│   ├── API 호출 현황
│   └── 에러 로그
│
└── 📁 설정 (/settings)
    ├── 내 정보
    └── 관리자 로그
```

---

## 성능 목표

| 지표 | 목표 |
|------|------|
| 대시보드 로딩 | < 2초 |
| 사용자 목록 조회 | < 1초 |
| 통계 API 응답 | < 500ms |
| 에러 로그 조회 | < 1초 |

---

## 관련 문서

- **인증**: `features/auth.md` - Google OAuth, JWT 정책
- **DB 스키마**: `reference/db-schema.md` - users 테이블
- **보안**: `reference/security.md` - 관리자 보안 체크리스트
- **인프라**: `reference/infra.md` - 관리자 웹 배포

---

> 📅 **작성일**: 2026-01-06  
> 📝 **Phase**: Phase 1 (MVP)  
> 🎯 **중요도**: 높음 (운영 필수)
