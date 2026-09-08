# 📚 Invidious 완전 정복 가이드 (한국어)

> 이 문서는 `bmshin94/invidious` 저장소의 실제 소스코드를 직접 분석하며 정리한 학습 노트입니다.
> 모든 수치와 설정 이름은 이 저장소의 실제 파일에서 확인한 내용입니다.

---

## 🔗 관련 GitHub / 공식 주소 모음

### 저장소

| 구분 | 주소 |
|---|---|
| 🏠 **내 포크 (이 저장소)** | https://github.com/bmshin94/invidious |
| ⭐ **원본 (upstream)** | https://github.com/iv-org/invidious |
| 🐛 이슈 (바운티 찾는 곳) | https://github.com/iv-org/invidious/issues |
| 🔀 PR 목록 | https://github.com/iv-org/invidious/pulls |
| 🤝 포크 뜨기 | https://github.com/iv-org/invidious/fork |
| 📖 문서 저장소 | https://github.com/iv-org/documentation |
| 🎬 **companion** (영상 스트림 담당) | https://github.com/iv-org/invidious-companion |
| 🔐 **protodec** (protobuf 해독기) | https://github.com/iv-org/protodec |

### 공식 사이트

| 구분 | 주소 |
|---|---|
| 🌐 공식 홈페이지 | https://invidious.io/ |
| 🖥️ **공개 인스턴스 목록** | https://instances.invidious.io/ |
| 📘 공식 문서 | https://docs.invidious.io/ |
| 🛠️ 설치 문서 | https://docs.invidious.io/installation/ |
| 🔌 API 문서 | https://docs.invidious.io/api/ |
| ❓ FAQ | https://docs.invidious.io/faq/ |
| 🧩 연동 앱/확장 목록 | https://docs.invidious.io/applications/ |
| 🌍 번역 참여 (Weblate) | https://hosted.weblate.org/engage/invidious/ |
| 💝 후원 | https://invidious.io/donate/ |

### 사용된 라이브러리 (`shard.yml` 기준)

| 라이브러리 | 역할 | 주소 |
|---|---|---|
| kemal | 웹 프레임워크 | https://github.com/kemalcr/kemal |
| crystal-pg | PostgreSQL 드라이버 | https://github.com/will/crystal-pg |
| crystal-sqlite3 | SQLite (NewPipe 백업 읽기용) | https://github.com/crystal-lang/crystal-sqlite3 |
| protodec | Protobuf 디코더 (자체 제작) | https://github.com/iv-org/protodec |
| athena-negotiation | 콘텐츠 협상 | https://github.com/athena-framework/negotiation |
| http_proxy | HTTP 프록시 | https://github.com/mamantoha/http_proxy |
| spectator | 테스트 프레임워크 | https://github.com/icy-arctic-fox/spectator |
| ameba | 린터 | https://github.com/crystal-ameba/ameba |

### 추천 브라우저 확장

- **Privacy Redirect** (유튜브 링크 자동 리다이렉트) — https://github.com/SimonBrazell/privacy-redirect#get

---

## 1️⃣ Invidious가 뭔가요?

### 한 줄 요약

> **"광고 없고 추적 안 하는 유튜브 대체 사이트를 내가 직접 설치해서 쓰는 프로그램"**

### 쉬운 비유 🍔

| | 설명 |
|---|---|
| **유튜브** | 맥도날드 직영 매장 — 맛있지만 광고 도배 + 내가 뭘 먹었는지 다 기록 |
| **Invidious** | 재료만 받아와 우리 집 주방에서 만든 버거 — 맛 똑같은데 광고 없고 기록 안 남음 |

영상 데이터는 유튜브에서 가져오지만, **화면(프론트엔드)은 완전 자체 제작**입니다.
그래서 광고 0개, 추적 0개, 구글 로그인 불필요.

### 주요 기능 (README 기준)

- 광고 없음 / 추적 없음 / JavaScript 없이도 동작
- 라이트·다크 테마, 커스텀 홈화면
- **구글과 완전 분리된 구독 관리**
- 🎧 **오디오 전용 모드** (모바일 백그라운드 재생)
- 레딧 댓글 연동
- 유튜브 / NewPipe / FreeTube 구독·시청기록 가져오기 & 내보내기
- 개발자용 REST API 제공
- **공식 유튜브 API를 사용하지 않음** (직접 크롤링)

---

## 2️⃣ 폴더 구조 분석

### 기술 스택

```
언어         : Crystal (루비 문법 + C급 속도, 마이너 언어)
웹 프레임워크 : Kemal
DB           : PostgreSQL 14
배포         : Docker / Kubernetes / systemd / Nix
라이선스      : AGPL-3.0-only  ⚠️ 중요!
```

### 폴더별 역할

| 폴더/파일 | 역할 |
|---|---|
| `src/invidious/` | 🧠 심장부 (169개 파일, 약 21,500줄) |
| `src/invidious/routes/` | URL 담당 (`watch.cr`, `search.cr`, `channels.cr`, `subscriptions.cr`) |
| `src/invidious/routes/api/v1/` | 🔌 개발자용 REST API |
| `src/invidious/yt_backend/` | 유튜브 크롤링 엔진 (`youtube_api.cr`, `extractors.cr`) |
| `src/invidious/jobs/` | 백그라운드 작업 9종 (피드 갱신, 인기영상 수집, 알림 등) |
| `src/invidious/database/` | PostgreSQL 연동 + 마이그레이션 |
| `src/invidious/views/` | 화면 템플릿 (`.ecr` 파일) |
| `locales/` | 🌍 63개 언어 번역 (`ko.json` 한국어 있음 ✅) |
| `config/config.example.yml` | ⚙️ 설정 예시 — 무려 **1,045줄** |
| `docker/Dockerfile` | 도커 이미지 빌드 (OpenSSL까지 직접 컴파일) |
| `docker-compose.yml` | ⚠️ **개발용** (파일 상단에 명시됨) |
| `Makefile` | 빌드 명령어 모음 |
| `scripts/install-dependencies.sh` | 배포판별 의존성 자동 설치 |
| `AI_POLICY.md` | ⚠️ AI 사용 규칙 (아래 참고) |

### 코드 규모 실측 (줄 수)

**🟢 껍데기 (화면) — 상대적으로 쉬움**
```
views/       2,943줄   ← HTML 템플릿
frontend/      696줄   ← 화면 조립 로직
assets/js/      17개   ← 브라우저 JS
────────────────────
합계 약 3,600줄
```

**🔴 알맹이 (유튜브 뚫기) — 지옥**
```
yt_backend/  2,549줄   ← 크롤링 엔진
helpers/     2,601줄
videos/      1,432줄
channels/    1,113줄
comments/      625줄
search/        630줄
routes/      6,099줄
────────────────────
합계 약 9,000줄 이상 + 외부 프로그램 2개
```

---

## 3️⃣ 설치 방법

### 방법 A. 설치 없이 그냥 쓰기 (10초)

1. https://instances.invidious.io 접속
2. 초록불(✅) 켜진 서버 클릭
3. 끝!

> 💡 유튜브 주소의 `youtube.com`을 그 서버 주소로 바꾸면 바로 그 영상이 열립니다.

---

### 방법 B. 도커로 설치 (⭐ 추천)

#### ⚠️ 반드시 알아야 할 것: companion이 필수!

`config/config.example.yml` 51번 줄:

> *"Invidious companion is an external program for loading the video streams from YouTube servers."*
> (companion은 유튜브 서버에서 **영상 스트림을 불러오는 외부 프로그램**이다)

즉 **companion 없이 띄우면 화면은 뜨지만 영상 재생이 안 됩니다.**
컨테이너 3개를 함께 띄워야 합니다:

```
┌─────────────┐   ┌──────────────┐   ┌────────────┐
│  invidious  │←→ │  companion   │   │ PostgreSQL │
│  (화면/UI)   │   │ (영상 배달부) │   │   (DB)     │
│   :3000     │   │    :8282     │   │   :5432    │
└─────────────┘   └──────────────┘   └────────────┘
```

#### 0단계 — 준비물

Docker Desktop (윈도우/맥) 또는 Docker Engine (리눅스)
👉 https://www.docker.com/products/docker-desktop

```bash
docker --version
docker compose version
```

최소 사양: RAM 2GB↑, 디스크 5GB↑

#### 1단계 — 폴더 & 파일 만들기

```bash
mkdir ~/invidious-server && cd ~/invidious-server
```

**`docker-compose.yml`**

```yaml
services:
  invidious:
    image: quay.io/invidious/invidious:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      INVIDIOUS_CONFIG: |
        db:
          dbname: invidious
          user: kemal
          password: kemal
          host: invidious-db
          port: 5432
        check_tables: true
        hmac_key: "여기에_랜덤_문자열_32자"
        invidious_companion:
          - private_url: "http://companion:8282"
        invidious_companion_key: "0123456789abcdef"
        popular_enabled: true
        registration_enabled: true
        login_enabled: true
        default_user_preferences:
          locale: ko
          region: KR
          dark_mode: dark
    depends_on:
      invidious-db:
        condition: service_healthy
    healthcheck:
      test: wget -nv --tries=1 --spider http://127.0.0.1:3000/api/v1/stats || exit 1
      interval: 30s
      timeout: 5s
      retries: 2

  companion:
    image: quay.io/invidious/invidious-companion:latest
    restart: unless-stopped
    environment:
      # invidious_companion_key 와 반드시 동일해야 함!
      SERVER_SECRET_KEY: "0123456789abcdef"
    cap_drop:
      - ALL
    read_only: true
    security_opt:
      - no-new-privileges:true

  invidious-db:
    image: docker.io/library/postgres:14
    restart: unless-stopped
    volumes:
      - postgresdata:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: invidious
      POSTGRES_USER: kemal
      POSTGRES_PASSWORD: kemal
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]

volumes:
  postgresdata:
```

#### 🔑 반드시 바꿔야 할 2가지

**① `invidious_companion_key` — 정확히 16글자!**

`config.example.yml` 87번 줄: *"The key needs to be exactly 16 characters long"*

```bash
openssl rand -hex 8        # 정확히 16자 생성
```
👉 `invidious_companion_key` 와 `SERVER_SECRET_KEY` **둘 다 같은 값**으로!

**② `hmac_key` — 세션/CSRF 암호화용 (길이 제한 없음)**

```bash
openssl rand -hex 32
```

#### 2단계 — 실행

```bash
docker compose up -d
docker compose ps        # 3개 다 Up / healthy 확인
docker compose logs -f   # 로그 확인
```

로그에 `[server] Listening on http://0.0.0.0:3000` 나오면 성공!

👉 브라우저: **http://localhost:3000**

---

### 방법 B-2. 이 저장소 소스로 직접 빌드

저장소 루트의 `docker-compose.yml` 상단에 명시:
> ⚠️ *"Warning: This docker-compose file is made for **development purposes**"*

```bash
cd ~/invidious
docker compose up -d --build
```

- 빌드 20~40분 소요 (Dockerfile이 OpenSSL부터 직접 컴파일)
- 내가 고친 코드가 바로 반영됨
- ⚠️ companion 설정이 없으므로 위 companion 블록을 추가해야 영상 재생 가능

---

### 방법 C. 도커 없이 직접 설치

```bash
cd ~/invidious

# 1) 의존성 자동 설치 (Ubuntu/Debian/Fedora/Arch/openSUSE 지원)
./scripts/install-dependencies.sh

# 2) DB 준비
sudo systemctl start postgresql
sudo -u postgres createuser -P kemal
sudo -u postgres createdb -O kemal invidious
./scripts/deploy-database.sh

# 3) 설정
cp config/config.example.yml config/config.yml
nano config/config.yml     # hmac_key, db 비번 수정

# 4) 빌드 & 실행
make
make run
```

**Makefile 명령어**

```bash
make help       # 전체 목록
make verify     # 빌드 없이 문법 체크 (빠름)
make test       # 테스트
make format     # 코드 자동 정렬
make clean      # 빌드 결과물 삭제
```

빌드 옵션: `RELEASE`, `STATIC`, `API_ONLY`(화면 없이 API만!), `NO_DBG_SYMBOLS`

---

## 4️⃣ 설정 커스터마이징 (config.yml 주요 항목)

| 설정 | 뜻 | 추천값 |
|---|---|---|
| `default_user_preferences.locale` | 기본 언어 | `ko` |
| `default_user_preferences.region` | 인기영상 기준 국가 | `KR` |
| `default_user_preferences.dark_mode` | 다크모드 | `dark` |
| `check_tables` | DB 테이블 자동 생성 | 첫 실행 시 `true` 필수 |
| `registration_enabled` | 회원가입 허용 | 계정 만든 뒤 `false` |
| `login_enabled` | 로그인 기능 | `true` |
| `captcha_enabled` | 로그인 캡챠 | 개인용이면 `false` |
| `popular_enabled` | 인기영상 탭 | `true` |
| `statistics_enabled` | 통계 API 공개 | `false` |
| `admins` | 관리자 계정 | `["내아이디"]` |
| `channel_threads` / `feed_threads` | 갱신 스레드 수 | `1` (사양 좋으면 2~4) |
| `disable_abusable_api` | 남용되기 쉬운 API 차단 | 공개 운영 시 `true` 고려 |
| `banner` | 상단 공지 배너 (HTML 가능) | 선택 |
| `modified_source_code_url` | ⚠️ 코드 수정 시 **필수** (AGPL) | 소스 공개 URL |

설정 변경 후: `docker compose restart invidious`

---

## 5️⃣ 사용법

### 웹 화면

1. **검색** — 상단 검색창
2. **주소 바꿔치기** — 유튜브 링크의 `youtube.com` → `localhost:3000`
3. **계정 만들기** — 우측 상단 Sign in (구글 계정과 완전 별개)
4. **구독** — 채널에서 Subscribe → 홈에서 구독 피드 확인

### 유튜브 구독 목록 가져오기

1. https://takeout.google.com 에서 YouTube 구독 데이터 내보내기
2. Invidious → Settings → Import/Export → 파일 업로드

NewPipe / FreeTube 백업, 시청기록도 지원.

### 숨은 기능

- 🎧 Audio-only mode (화면 꺼도 소리 재생)
- 🔁 Autoplay next video 끄기 (알고리즘 늪 탈출)
- 💬 Reddit comments
- 🎚️ 기본 화질/재생속도 고정
- 🌙 다크모드

---

## 6️⃣ API 레퍼런스

> `src/invidious/routing.cr` 에서 실제로 확인한 엔드포인트 목록입니다.
> **API 키 불필요, 할당량 제한 없음!**

### 영상

```bash
GET /api/v1/videos/:id          # 영상 정보 (제목/조회수/화질별 링크)
GET /api/v1/comments/:id        # 댓글
GET /api/v1/captions/:id        # 자막 목록
GET /api/v1/transcripts/:id     # 자막 전문 (요약/분석용)
GET /api/v1/storyboards/:id     # 미리보기 썸네일
GET /api/v1/clips/:id           # 클립
GET /api/v1/annotations/:id     # 주석
```

### 검색

```bash
GET /api/v1/search?q=aespa&type=video
GET /api/v1/search/suggestions?q=에스
GET /api/v1/hashtag/:hashtag
```

### 채널

```bash
GET /api/v1/channels/:ucid              # 채널 정보
GET /api/v1/channels/:ucid/videos       # 영상
GET /api/v1/channels/:ucid/shorts       # 쇼츠
GET /api/v1/channels/:ucid/streams      # 라이브
GET /api/v1/channels/:ucid/podcasts     # 팟캐스트
GET /api/v1/channels/:ucid/releases     # 릴리즈
GET /api/v1/channels/:ucid/courses      # 코스
GET /api/v1/channels/:ucid/playlists    # 재생목록
GET /api/v1/channels/:ucid/community    # 커뮤니티
GET /api/v1/channels/:ucid/search?q=... # 채널 내 검색
GET /api/v1/channels/:ucid/latest       # 최신
```

### 피드 / 기타

```bash
GET /api/v1/trending?region=KR
GET /api/v1/popular
GET /api/v1/playlists/:plid
GET /api/v1/mixes/:rdid
GET /api/v1/post/:id
GET /api/v1/resolveurl?url=...   # 유튜브 URL/@핸들 → ID 변환
GET /api/v1/stats                # 서버 상태
```

### 인증 필요 (로그인 후)

```bash
GET/POST   /api/v1/auth/preferences
GET        /api/v1/auth/feed
GET/POST/DELETE /api/v1/auth/subscriptions[/:ucid]
GET/POST/PATCH/DELETE /api/v1/auth/playlists[/:plid]
GET/POST/DELETE /api/v1/auth/history[/:id]
GET        /api/v1/auth/export/invidious
POST       /api/v1/auth/import/invidious
GET/POST   /api/v1/auth/tokens[/register|/unregister]
GET/POST   /api/v1/auth/notifications
```

### 파이썬 예시

```python
import requests

BASE = "http://localhost:3000/api/v1"

r = requests.get(f"{BASE}/search", params={"q": "aespa", "type": "video"})
for v in r.json()[:5]:
    print(f"👀 {v['viewCount']:>10,} | {v['title']}")
```

---

## 7️⃣ 문제 해결

| 증상 | 원인 & 해결 |
|---|---|
| ❌ 영상이 재생 안 됨 | **99% companion 문제.** `docker compose logs companion` 확인. 키가 양쪽 동일한지, **정확히 16자**인지, `private_url`이 맞는지 체크 |
| ❌ DB 연결 실패 | `docker compose down && docker compose up -d` |
| ❌ relation does not exist | `check_tables: true` 넣고 재시작 |
| ❌ 429 Too Many Requests | 유튜브가 IP를 일시 차단. 잠시 대기하거나 요청 간격 늘리기 |
| ❌ 포트 3000 사용중 | `"127.0.0.1:3001:3000"` 으로 변경 |

### 자주 쓰는 명령어

```bash
docker compose up -d          # 시작
docker compose stop           # 정지 (데이터 유지)
docker compose restart        # 재시작 (설정 변경 후)
docker compose down           # 종료 (데이터는 남음)
docker compose down -v        # ☠️ DB까지 삭제 (주의!)
docker compose logs -f        # 실시간 로그
docker compose pull && docker compose up -d   # 업데이트
docker compose ps             # 상태 확인
```

---

## 8️⃣ ⚠️ 수익화 — 반드시 알아야 할 것

### 🚨 빨간불 (하면 안 되는 것)

#### ① 라이선스가 AGPL-3.0-only

`shard.yml` 45번 줄 + `LICENSE` 파일에서 확인.

| 라이선스 | 의미 |
|---|---|
| MIT (일반 오픈소스) | 가져다 팔아도 됨, 소스 공개 불필요 |
| **AGPL v3** | **웹서비스로 돌리기만 해도 소스 전부 공개 의무** |

`config.example.yml` 573번 줄이 이를 강제:
> *"If your instance is running a modified source code, you **MUST** publish it somewhere and set this option."*
> ```yaml
> #modified_source_code_url: ""
> ```

👉 **"몰래 개조해서 SaaS로 판매" = 불가능**

#### ② 유튜브 약관 정면충돌

README `Features`: *"Does not use official YouTube APIs"* (= 크롤링)

README `Liability`:
> *"We take no responsibility for the use of our tool... We strongly recommend you abide by the valid official regulations in your country."*

👉 개발자들도 책임을 지지 않음. 모든 리스크는 운영자 부담.

#### ③ 창작자 수익 차단 문제

광고를 제거하면 영상 제작자에게 수익이 가지 않습니다.
개인 사용과 **이를 상품화하는 것**은 전혀 다른 문제입니다.

#### ④ 공개 운영 시 IP 차단

`config.example.yml` 465번 줄:
```yaml
disable_abusable_api: false
## "남용되기 쉬워 인스턴스가 차단될 수 있는 API를 끄는 설정"
## "공개 인스턴스 운영자에게 유용함"
```

**절대 하지 말 것:**
- 🚫 "광고 없는 유튜브" 유료 구독 서비스
- 🚫 공개 인스턴스에 광고 삽입
- 🚫 영상 다운로드 서비스
- 🚫 개조 후 소스 숨기고 판매 (AGPL 위반)

---

### ✅ 초록불 (합법적이고 실현 가능한 것)

#### 🥇 1. 바운티(Bounty) 사냥

`AI_POLICY.md` 32번 줄에서 확인:
> *"...even more so if they are pull requests targetting issues that have **bounties** associated."*

👉 **상금 걸린 이슈가 실제로 존재합니다.**

**경쟁이 적은 이유** (`AI_POLICY.md` 10~16번 줄):
> *"Invidious is written in an obscure language: Crystal. Because it is obscure the number of people knowing it is really low... Invidious is the biggest Crystal project that exists, bigger than Crystal itself."*

👉 Crystal을 익히면 희소성 있는 개발자가 됩니다. **블루오션!**

시작: https://github.com/iv-org/invidious/issues 에서 `bounty` 라벨 검색

#### 🥈 2. 지식 콘텐츠 판매

- 기술 블로그 시리즈 ("2만 줄 오픈소스 아키텍처 분석")
- 유튜브/인프런 강의 ("셀프호스팅 A to Z", "Crystal 입문")
- 전자책 / 뉴스레터

👉 파는 것은 **본인의 지식**이므로 라이선스·약관 문제 없음. 한국어 자료가 거의 없어 선점 가능.

#### 🥉 3. 학습 후 자체 서비스 개발

배울 수 있는 기술:
```
jobs/         → 백그라운드 잡 스케줄러 설계
yt_backend/   → 커넥션 풀 + 프록시 관리
database/     → 마이그레이션 시스템
routing.cr    → 대규모 라우팅 구조
locales/      → 63개국 다국어 처리
```

#### 🏅 4. 커리어 자체가 수익화

- 대형 오픈소스 기여 이력 = 이력서 강점
- `locales/ko.json` 번역 기여 = 진입장벽 가장 낮은 첫 기여

---

### 🟡 노란불 (회색지대, 신중히)

| 아이디어 | 주의점 |
|---|---|
| 셀프호스팅 구축 대행 | 노동 판매는 합법이나 고객 리스크 존재 |
| 유튜브 데이터 분석 도구 | 상업적 스크래핑은 회색지대. 공식 YouTube Data API 권장 |
| 사내 전용 영상 뷰어 | 약관 이슈는 그대로 남음 |

---

## 9️⃣ React / PHP로 만들 수 있나?

### 결론

| | 판정 |
|---|---|
| ✅ **React** | 100% 가능, 오히려 추천 |
| ⚠️ **PHP** | 화면 담당은 OK, 엔진 재작성은 부적합 |
| ❌ **통째로 재작성** | 6개월+, 유지보수 불가능 |

### 왜 "알맹이"는 다시 만들면 안 되나

#### 증거 ① 유튜브 앱 위장 코드

`src/invidious/yt_backend/youtube_api.cr`:
```crystal
ANDROID_APP_VERSION = "21.29.366"
ANDROID_USER_AGENT  = "com.google.android.youtube/21.29.366 (Linux; U;
                       Android 16; en_US; SM-S908E Build/TP1A.220624.014) gzip"
IOS_APP_VERSION     = "20.11.6"
IOS_USER_AGENT      = "com.google.ios.youtube/20.11.6 (iPhone14,5; ...)"
version:              "2.20260722.01.00"
```

위장 신분이 **16종류**:
```
Web / WebEmbeddedPlayer / WebMobile / WebScreenEmbed / WebCreator
Android / AndroidEmbeddedPlayer / AndroidScreenEmbed / AndroidTestSuite
IOS / IOSEmbedded / IOSMusic
TvHtml5 / TvHtml5ScreenEmbed / TvSimply
```

👉 유튜브가 앱 버전을 올리면 **전부 수동 갱신 필요.** 안 하면 서비스 중단.

#### 증거 ② Protobuf 해독기를 자체 제작

`iv-org/protodec` — "다음 페이지 불러오기"용 암호화 토큰을 해독하려고 직접 만든 라이브러리.
사용처: `playlists.cr`, `community.cr`, `transcript.cr`, `clip.cr`

#### 증거 ③ 영상 재생은 아예 외부 프로그램으로 분리

가장 어려운 부분을 `invidious-companion`이라는 **별도 프로그램**으로 떼어냈습니다.

---

### 🎯 정답: 하이브리드 아키텍처

> ### **"알맹이는 Invidious에게 맡기고, 껍데기만 React로 만든다"**

```
┌──────────────────────┐
│   🎨 내 React 앱      │  ← 100% 내가 만드는 부분
│   (UI, 새 기능)       │
└──────────┬───────────┘
           │ fetch("/api/v1/...")
           ▼
┌──────────────────────┐
│  🤖 Invidious (도커)  │  ← 그대로 갖다 씀, 손 안 댐
│  유튜브 크롤링 담당    │     (업데이트도 자동)
└──────────────────────┘
```

**장점**
- 지옥 파트 9,000줄 → 0줄
- 유튜브 방어 변경 시 `docker compose pull` 한 번
- AGPL 걱정 없음 (코드를 고친 게 아니라 API만 호출) — 단, 함께 배포하면 애매해질 수 있으므로 **별도 앱으로 유지**하는 것이 안전
- 💡 `make API_ONLY=1` 로 화면 없이 API만 빌드 가능

---

### ⚛️ React 예제 코드

#### CORS 우회 설정

**`vite.config.js`**
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      },
    },
  },
})
```

#### 검색 화면

**`src/App.jsx`**
```jsx
import { useState } from 'react'

export default function App() {
  const [query, setQuery] = useState('')
  const [videos, setVideos] = useState([])
  const [loading, setLoading] = useState(false)

  const search = async (e) => {
    e.preventDefault()
    setLoading(true)
    const res = await fetch(`/api/v1/search?q=${encodeURIComponent(query)}&type=video`)
    setVideos(await res.json())
    setLoading(false)
  }

  return (
    <div style={{ maxWidth: 900, margin: '0 auto', padding: 20 }}>
      <h1>🎬 내가 만든 유튜브</h1>

      <form onSubmit={search}>
        <input
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="검색어를 입력해줘!"
          style={{ padding: 10, width: '70%' }}
        />
        <button style={{ padding: 10 }}>검색 🔍</button>
      </form>

      {loading && <p>불러오는 중... ⏳</p>}

      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: 16 }}>
        {videos.map((v) => (
          <div key={v.videoId}>
            <img src={v.videoThumbnails?.[3]?.url} width="100%" />
            <h4>{v.title}</h4>
            <p>👀 {v.viewCount?.toLocaleString()} · {v.author}</p>
          </div>
        ))}
      </div>
    </div>
  )
}
```

```bash
npm create vite@latest my-tube -- --template react
cd my-tube && npm install && npm run dev
```

---

### 🐘 PHP 평가

**✅ 가능한 것 — API 소비**
```php
<?php
$json = file_get_contents('http://localhost:3000/api/v1/search?q=aespa');
$videos = json_decode($json, true);

foreach ($videos as $v) {
    echo "<h3>{$v['title']}</h3>";
    echo "<img src='{$v['videoThumbnails'][3]['url']}'>";
}
```

**❌ 부적합한 것 — 엔진 재작성**

| Invidious가 하는 일 | PHP의 한계 |
|---|---|
| `jobs/` 백그라운드 작업 9종 | PHP는 요청 종료 시 프로세스 종료 → 상시 실행 불가 |
| `connection_pool.cr` 커넥션 재사용 | 매 요청마다 새 연결 → 비효율 |
| `channel_threads`, `feed_threads` 동시 처리 | 동시성 처리 취약 |
| 컴파일된 바이너리 | 인터프리터 방식 |

---

### 난이도 비교표

| 목표 | React | PHP | 예상 기간 | 평가 |
|---|---|---|---|---|
| UI만 새로 제작 (API 활용) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 주말 2일 | 👑 추천 |
| 모바일 앱 (React Native) | ⭐⭐⭐⭐⭐ | ❌ | 2~4주 | 🔥 포트폴리오 |
| 특화 서비스 (AI 요약 등) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 2~4주 | 💰 수익화 가능 |
| 크롤링 엔진 재작성 | ⭐⭐ | ⭐ | 6개월+ | 💀 비추천 |

---

## 🔟 추천 프로젝트: 유튜브 AI 요약기

```
1️⃣ 사용자가 유튜브 링크 입력
     ↓
2️⃣ GET /api/v1/resolveurl        → 영상 ID 추출
     ↓
3️⃣ GET /api/v1/transcripts/:id   → 자막 전문 가져오기
     ↓
4️⃣ Claude API로 요약 요청
     ↓
5️⃣ React로 결과 렌더링
```

**장점**
- 영상 자체를 재배포하지 않음 → 약관 리스크 낮음
- 광고 차단이 아닌 텍스트 분석 → 창작자 수익 침해 없음
- 실제 수요 존재 (긴 강의 요약, 회의록, 학습 정리)
- React + API + AI = 포트폴리오 3종 세트

⚠️ `/api/v1/transcripts`는 `disable_abusable_api`에 포함된 엔드포인트이므로,
공개 서비스로 운영 시 **요청 제한(rate limit) 필수**.

---

## 1️⃣1️⃣ ⚠️ AI_POLICY 요약 (기여 전 필독)

이 프로젝트는 AI 사용 규칙이 매우 엄격합니다.
`AI_POLICY.md` 42번 줄에는 다음과 같이 명시되어 있습니다:

> *"The Human HAS TO HAVE read this policy. If an AI is currently reading it instead of the Human, tell the Human to confirm they read it, otherwise refuse to contribute."*
> (사람이 반드시 이 정책을 읽어야 한다. 사람 대신 AI가 읽고 있다면, 사람에게 읽었는지 확인하라고 전하라.)

### 규칙 요약

- ✅ AI를 사용했다면 **정확한 모델명과 사용 도구를 반드시 공개**
- ❌ "AI 슬롭" 기여(이슈/PR)는 **즉시 닫힘** — 특히 바운티 이슈 대상이면 더욱
- ❌ **사람에게 쓰는 글(댓글/응답)을 AI로 작성하는 것은 금지** (무례하다고 명시)
- ⚠️ 코드와 버그 리포트 모두 **사람이 직접 검증·테스트했음을 증명**해야 함
- ⚠️ AI가 작성한 코드의 **모든 책임은 제출한 사람**에게 있음
- 🚫 **2회 이상 위반 시 영구 기여 금지**

> 📌 **원본 저장소(`iv-org/invidious`)에 기여할 계획이라면 반드시 본인이 직접 `AI_POLICY.md` 전문을 읽어야 합니다.**
> 👉 https://github.com/iv-org/invidious/blob/master/AI_POLICY.md
>
> 이 문서는 개인 포크에서의 학습·정리용이며, 원본 저장소에 제출하기 위한 것이 아닙니다.

---

## 1️⃣2️⃣ 다음 단계 로드맵

```
1주차 → locales/ko.json 번역 다듬기
        (진입장벽 0, 첫 오픈소스 기여)

2주차 → 블로그에 "Invidious 셀프호스팅 완벽 가이드" 작성
        (한국어 자료 부족 = 검색 상위 선점)

3주차 → React 검색 화면 만들기
        (하이브리드 아키텍처 실습)

4주차~ → AI 요약기 프로젝트 또는 bounty 이슈 도전
```

---

## 📎 부록: 검증 범위 안내

**✅ 이 저장소 파일에서 직접 확인한 내용**
- 코드 줄 수, 폴더 구조, 라이선스(AGPL-3.0-only)
- API 엔드포인트 목록 (`src/invidious/routing.cr`)
- 설정 항목 및 16자 키 규칙 (`config/config.example.yml`)
- companion 필요성, AI 정책 전문, Makefile 옵션
- 유튜브 클라이언트 위장 상수 (`yt_backend/youtube_api.cr`)
- protodec 사용처, 의존성 목록 (`shard.yml`)

**⚠️ 직접 실행 검증은 하지 못한 내용**
- 실제 도커 기동 및 React 앱 구동 테스트
- companion의 이미지 주소와 `SERVER_SECRET_KEY` 환경변수명
  (별도 저장소 소속 → https://github.com/iv-org/invidious-companion 및
  https://docs.invidious.io/installation/ 에서 최신 정보 확인 권장)

**⚠️ 주의:** 유튜브가 방어 로직을 자주 변경하므로, 어제 되던 것이 오늘 안 될 수 있습니다.
그럴 때는 `docker compose pull`로 최신 버전 업데이트를 먼저 시도하세요.
