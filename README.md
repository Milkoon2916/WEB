# 영어 학습자료 제작소 (합본)

`passage-analyzer`와 `comprehension` 두 리포지토리를 **하나의 FastAPI 서버, 하나의 배포**로 합친 버전입니다.

## 왜 이렇게 합쳤나

- `comprehension`은 `index.html` 하나뿐인 완전 정적 사이트였고, 원래 있던 `server.js`는
  정적 파일을 그대로 서빙하는 역할만 했습니다. → **통째로 제거하고 FastAPI의 정적 파일
  서빙 기능으로 대체**했습니다.
- `passage-analyzer`는 PDF 렌더링(`/render`)을 위해 실제 Python 서버(FastAPI + WeasyPrint)가
  필요합니다. → 이 서버가 **두 사이트를 모두 서빙하는 단일 진입점**이 됩니다.

## 구조

```
app/
  main.py         FastAPI 앱. 아래 세 경로를 서빙:
                    /                 → landing/index.html (허브 랜딩 페이지)
                    /passage-analyzer/ → 구문분석기 프론트엔드 (static/)
                    /comprehension/    → OX 워크북 메이커 (comprehension/)
                    /prompt-config, /render, /download/{job_id}  → 기존 API 그대로
  prompt.py, render.py, schemas.py, thesaurus.py   기존 passage-analyzer 로직 그대로
static/            구문분석기 프론트엔드 (기존 passage-analyzer/static)
comprehension/     OX 워크북 메이커 정적 파일 (기존 comprehension/index.html)
landing/           새로 만든 허브 랜딩 페이지
Dockerfile         기존 passage-analyzer Dockerfile 그대로 사용 (WeasyPrint 의존성 포함)
requirements.txt   기존 passage-analyzer 그대로
```

## 로컬 실행

```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

uvicorn app.main:app --reload --port 8000
```

브라우저에서 `http://localhost:8000` 접속 → 랜딩 페이지에서 두 도구 중 선택.

## 배포 (Render 기준, 예시)

1. 이 폴더 전체를 새 GitHub 리포지토리에 올립니다 (혹은 기존 `passage-analyzer` 리포지토리의
   내용을 이 구조로 교체합니다).
2. Render 대시보드 → **New → Web Service** → 해당 리포지토리 연결
3. Runtime이 **Docker**로 자동 감지되는지 확인 (Dockerfile이 이미 WeasyPrint 시스템
   의존성과 한글 폰트까지 설치합니다).
4. 별도 환경변수 필요 없음 (두 도구 모두 사용자가 자신의 API 키를 브라우저에서 직접 입력하는
   BYOK 구조라서, 서버에는 어떤 비밀 키도 두지 않습니다).
5. 배포 완료되면 `https://xxx.onrender.com` 하나의 주소에서 두 도구가 모두 동작합니다.

## comprehension의 기존 Dockerfile / package.json / server.js

이 합본에서는 사용하지 않습니다. 정적 파일(`index.html`)만 가져와서 FastAPI가 직접
서빙하도록 했습니다. 원본 `comprehension` 리포지토리를 계속 별도로 유지·배포하고
싶다면 그건 그것대로 남겨두셔도 무방합니다 (이 합본과 무관하게 독립적으로 동작).

## 확인 완료

- `GET /` → 랜딩 페이지 200 OK
- `GET /passage-analyzer/` → 구문분석기 프론트엔드 200 OK
- `GET /comprehension/` → OX 워크북 메이커 200 OK
- `GET /prompt-config` → 기존 API 정상 동작 200 OK
