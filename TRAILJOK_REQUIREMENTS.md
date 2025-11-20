# 트레일족(Trailjok) 프론트·백엔드 전체 요구사항

아래 내용은 트레일족 프로젝트를 생성하거나 확장할 때 바로 활용할 수 있는 마스터 프롬프트 형태로 정리한 것입니다. React + Vite + TypeScript + TailwindCSS 기반 프론트엔드와 FastAPI 기반 백엔드를 동시에 고려합니다.

## 프로젝트 개요
- **서비스**: 북한산 등산 실시간 피드 + GPT 기반 산 정보 제공 플랫폼.
- **프론트**: React + Vite + TypeScript + TailwindCSS.
- **백엔드**: FastAPI.

## 프론트엔드 요구사항 (React + Vite + Tailwind)
### 1) PC/모바일 반응형 UI 규칙
- **모바일**
  - 상단 `Header`: 로고, 산 선택, 다크모드.
  - 하단 `BottomNav`: 홈, 업로드, 내 정보.
  - 하단 고정 검색바: "무엇이 궁금하세요?...".
- **PC**
  - 모바일 `Header`는 숨김 (`lg:hidden`).
  - 대신 왼쪽 고정 사이드바(인스타그램 PC 스타일) 배치, 너비 `w-64`.
  - 메인 콘텐츠는 `lg:ml-64`로 배치.
  - 하단 검색바는 PC에서도 동일하게 유지.

### 2) PC 왼쪽 사이드바 구성
- 항목: 트레일족 로고, 홈 피드, 사진 업로드, 샘플 게시물(예시), 산 선택(MountainSelector), 다크모드 토글.
- 하단 메시지: "오늘도 안전한 산행 되세요 🌲".
- 스타일: 인스타그램 유사 아이콘 + 텍스트 메뉴. 클래스 예시 `hidden lg:flex fixed inset-y-0 left-0 w-64 bg-slate-950 border-r border-slate-800 backdrop-blur-xl`.

### 3) Header (모바일 전용)
- PC에서는 보이지 않도록 `<header className="lg:hidden">` 사용.
- 로고, 산 선택 드롭다운, 다크모드 토글 포함.

### 4) 메인 UI 구성
- **TodayCard (GPT 오늘의 북한산)**: 네온 그라데이션 테두리, 날짜, 날씨 요약, GPT 자동 생성 텍스트.
- **실시간 피드 카드(PostCard)**: 2~3열 masonry grid, 이미지, 태그(백운대, 족두리봉 등), 좋아요 수, 업로드 시간.
- **TagChips**: 전체, 백운대, 족두리봉, 둘레길, 비봉능선 (active 색상 강조).

### 5) 하단 검색바 (모바일/PC 공통)
- 고정 Sticky.
- 입력 시 `/search` API 호출.
- 검색 결과 오버레이(`SearchOverlay`) 표시.

### 6) 전체 레이아웃 구조
```jsx
<MountainContext.Provider>
  <Sidebar />   {/* PC만 */}
  <Header />    {/* 모바일만 */}
  <main className="lg:ml-64">  {/* PC에서 사이드바 공간 확보 */}
     ... pages ...
  </main>
  <BottomSearchBar />
  <BottomNav />  {/* 모바일만 */}
  <SearchOverlay />
</MountainContext.Provider>
```

## 백엔드 요구사항 (FastAPI)
### 1) API 라우트 구성
- `/auth`: Kakao/Google OAuth + JWT 발급
- `/mountain`: 산 리스트, GPT 오늘의 산 정보
- `/feed`: 실시간 피드 목록, 좋아요, 신고
- `/upload`: 이미지 업로드
- `/search`: GPT 검색
- `/recommend`: 개인 맞춤 추천
- `/user`: 내정보, 뱃지 시스템

### 2) 백엔드 파일 구조
```
backend/
 ├─ main.py
 ├─ routers/
 │    ├─ auth.py
 │    ├─ mountain.py
 │    ├─ feed.py
 │    ├─ upload.py
 │    ├─ search.py
 │    ├─ recommend.py
 │    └─ user.py
 ├─ services/
 │    ├─ auth_service.py      ← JWT 발급
 │    ├─ gpt_service.py       ← GPT 응답 템플릿
 └─ static/uploads/           ← 이미지 저장
```

### 3) FastAPI 기본 코드 스니펫
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from routers import auth, mountain, feed, upload, search, recommend, user

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(auth.router, prefix="/auth")
app.include_router(mountain.router, prefix="/mountain")
app.include_router(feed.router, prefix="/feed")
app.include_router(upload.router, prefix="/upload")
app.include_router(search.router, prefix="/search")
app.include_router(recommend.router, prefix="/recommend")
app.include_router(user.router, prefix="/user")

@app.get("/")
def home():
    return {"msg": "Trailjok Backend Running!"}
```

### 4) GPT 관련 API
- `/mountain/today-info`: "오늘의 산 상태 설명" GPT 자동 생성.
- `/search?q=`: "족두리봉 눈왔어?"와 같은 질문에 GPT 답변 제공.

### 5) 이미지 업로드 API
- `/upload`: `multipart/form-data`로 이미지 수신.
- `static/uploads`에 저장 후 저장 경로 JSON 반환.

## 요약 포인트
- 인스타그램 유사 UI, 모바일/PC 완전 분리 레이아웃.
- Sidebar + Header + BottomNav + BottomSearchBar + SearchOverlay 구성.
- GPT 카드 + 실시간 피드 그리드 UI.
- FastAPI 기반 Mock API로 확장성 확보(OAuth/GPT/DB 연동 확장 가능).

## Codex 실행 예시
- 프론트/백 동시 생성을 원할 때 이 파일 전체를 Codex에 붙여넣어 프롬프트로 활용하세요.
- 필요 시 DB 연동, 실제 OAuth/GPT 호출 버전 등 확장 템플릿도 쉽게 추가 가능합니다.
