# 정보편식 진단기

유튜브 시청기록(Google Takeout)을 브라우저 안에서 분석해 콘텐츠 유형별 비중, 채널 집중도(HHI·CR3), 그리고 예상한 시청 습관과 실제 기록을 비교하는 웹서비스입니다.

## GitHub에 올리는 방법

1. GitHub에서 새 저장소를 만듭니다.
2. 이 ZIP을 압축 해제합니다.
3. 압축을 푼 폴더 안의 `infodiet` 폴더 **내용 전체**를 저장소 최상위에 업로드합니다. (최상위에 `index.html`, `data/`, `js/`, `css/`, `.github/`가 보여야 합니다.)
4. GitHub 저장소의 **Settings → Pages → Build and deployment → Source**를 `GitHub Actions`로 설정합니다.
5. `main` 또는 `master` 브랜치에 커밋하면 Actions가 자동으로 테스트·빌드·배포합니다.
6. 저장소의 **Actions** 탭에서 `GitHub Pages 배포`가 성공하면 사이트 주소가 표시됩니다.

이 저장소는 ZIP 업로드 후 압축을 푸는 구형 배포 방식이 아니라, 소스 코드와 데이터 파일을 저장소에 직접 올리는 일반적인 GitHub Pages 방식입니다.

## 로컬 실행

- `index.html`을 더블클릭해 실행할 수 있습니다. 차트 표시에는 인터넷 연결이 필요합니다.
- 또는 로컬 서버를 사용합니다. 예: `python -m http.server 8000` 후 `http://localhost:8000` 접속

## 개발 및 테스트

Node.js 22 이상을 권장합니다.

```bash
npm install
npm test
npm run build
```

- `npm test`: 분석·파싱 로직 자동 테스트
- `npm run build`: `js/app.js`를 `js/app.bundle.js`로 묶고, `data/channel-types.json`에서 `data/channel-types.js`를 생성합니다.
- GitHub Actions는 push 때 자동으로 위 테스트와 빌드를 실행합니다.

## 폴더 구조

```text
index.html
css/style.css
js/
  app.js                 화면 및 라우팅
  app.bundle.js          브라우저 실행용 번들
  analyzer.js            유형 분류·비중·HHI·CR3 계산
  parser.js              Takeout JSON/HTML 파서
  zip.js                 Takeout ZIP 처리
  config.js              8개 유형 및 분석 설정
  charts.js              차트 렌더링
  storage.js             브라우저 LocalStorage 처리
  sample.js              예시 데이터
  explore.js             유형별 탐색 링크
data/
  channel-types.json     최종 채널 DB 원본
  channel-types.js       file:// 실행을 위한 DB 브라우저 fallback
asset/fonts/             Pretendard 글꼴
tests/                   Node 자동 테스트와 샘플 파일
tools/                   DB JS 생성 도구
docs/                    팀 참고용 카테고리 요약 및 임시 매핑표 (사이트에는 배포하지 않음)
.github/workflows/       GitHub Pages 자동 배포
```

## 채널 DB 및 분류 기준

- DB에는 채널 11,468개가 있습니다.
- `channelId`가 있는 채널은 10,468개이며, 기존 수동 DB 1,000개는 채널명으로 매칭합니다.
- `type`은 앱에서 사용하는 분류 코드입니다: `vlog`, `entertain`, `game`, `beauty`, `knowledge`, `music`, `sports`, `news`, `unclassified`.
- `category8`은 8개 대분류이며, `category26`은 기존 세부 분류입니다.
- `category26`이 `기타`인 채널 4,994개는 억지로 8개 유형에 넣지 않고 `type: unclassified`, `category8: 미분류`로 처리했습니다.
- `unclassified`는 8개 콘텐츠 유형과 별개의 미분류 상태입니다.

### 임시 매핑 안내

기존 26개 세부 분류를 8개 대분류로 합친 기준은 팀에서 최종 확정한 정책이 아니라 작업자가 설정한 **임시 매핑**입니다. 계획서·발표 자료·분석 결과에는 팀 검토 전까지 반드시 임시 기준이라고 표시해야 합니다.

예시:
- 먹방·반려동물·여행·요리 → 브이로그·일상형
- 테크·재테크·부동산 → 지식·교육형
- 키즈·코미디·드라마/영화 → 엔터테인먼트형
- 기타 → 미분류

팀이 매핑 기준을 확정하면 `data/channel-types.json`의 `type`, `category8`, `category8Code`를 함께 검토하고 수정합니다.

## 개인정보 및 업로드 데이터

- 사이트용 DB에는 개인별 `takeoutWatchCount*`, `takeoutObserved*` 항목을 포함하지 않습니다.
- 원본 Google Takeout 시청기록 및 `watch-history-*.csv`는 저장소와 사이트 배포 파일에 포함하지 않습니다.
- 사용자가 업로드한 Takeout 파일은 브라우저 안에서 처리됩니다. 원본 시청기록을 서버로 전송하지 않습니다.
- 브라우저 LocalStorage에는 사용자가 저장한 예상 유형과 분석 요약만 보관합니다. 원본 파일·영상 제목·채널명은 저장하지 않습니다.

## 계산 기준

- 단위: 시청기록 건수 (Takeout에 실제 시청 시간이 제공되지 않는 경우 재생 시간으로 환산하지 않음)
- 광고, 중복 기록, 검색 기록, 날짜를 읽을 수 없는 항목은 분석 기준에 따라 제외
- 기간: 파일 안의 마지막 시청 시각을 기준으로 최근 1·3·6·12개월, 전체 또는 직접 지정
- 유형 매칭: 채널 ID → 정규화한 채널명/별칭 → ` - Topic` 패턴 순으로 확인하며, 매칭되지 않으면 미분류
- HHI: 채널별 시청 비중(%)의 제곱합, 0~10,000
- CR3: 상위 3개 채널의 시청 비중 합
- HHI 단계 경계값은 팀 데이터로 검증해야 하는 임시값입니다.

## 테스트

```bash
node --test tests/analyzer.test.js
```
