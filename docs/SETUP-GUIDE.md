# 검색·수익화 설정 가이드

대상 사이트: `https://osori-workbench.github.io`
기존 블로그(OsoriAndOmori.github.io)와 같은 구조(Chirpy `_config.yml` 키 + `_includes/metadata-hook.html`)를 그대로 쓴다.
이 repo에는 자리(키, 훅, `ads.txt`, sitemap, robots)가 이미 준비돼 있고, **본인 계정에서 발급한 값 3개만 채우면 된다.**

| 순서 | 할 일 | 채울 곳 |
|---|---|---|
| 1 | Search Console 소유권 확인 | `_config.yml` → `webmaster_verifications.google` |
| 2 | Google Analytics(GA4) | `_config.yml` → `analytics.google.id` |
| 3 | AdSense | `_config.yml` → `adsense.client` + `ads.txt` |

값을 넣고 `main`에 푸시하면 Actions가 자동 배포한다(약 2분).

## 0. 이미 되어 있는 것

- `https://osori-workbench.github.io/sitemap.xml` (jekyll-sitemap, 글 발행 때마다 자동 갱신)
- `https://osori-workbench.github.io/robots.txt` (sitemap 경로 포함)
- `https://osori-workbench.github.io/feed.xml`
- AdSense 스크립트 훅: `_includes/metadata-hook.html` (`adsense.client`가 비어 있으면 아무것도 싣지 않는다)
- `ads.txt` (repo 루트 → 사이트 루트로 배포됨)

## 1. Google Search Console

1. https://search.google.com/search-console 접속 → 속성 추가
2. **URL 접두어** 방식 선택 → `https://osori-workbench.github.io/` 입력 (끝의 `/` 포함)
3. 소유권 확인 방법 중 **HTML 태그**를 고른다. 나오는 메타 태그의 `content` 값만 복사한다.
   ```html
   <meta name="google-site-verification" content="여기_값만_복사">
   ```
4. `_config.yml` 에 입력:
   ```yaml
   webmaster_verifications:
     google: 여기_값만_복사
   ```
5. 커밋·푸시 → 배포 완료 후(약 2분) Search Console에서 **확인** 클릭
6. 왼쪽 **Sitemaps** → `sitemap.xml` 입력 → 제출
7. (선택) **URL 검사**에 새 글 주소를 넣고 "색인 생성 요청". 새 사이트는 첫 색인까지 며칠~몇 주 걸린다.

참고: 기존 블로그는 같은 방식으로 `webmaster_verifications.google`에 토큰을 넣어 쓰고 있다. 도메인이 다르므로 **토큰은 이 사이트용으로 새로 받아야 한다.**

## 2. Google Analytics (GA4)

1. https://analytics.google.com → 관리 → 속성 만들기 (또는 기존 속성 사용)
2. **데이터 스트림 → 웹** 추가, URL은 `https://osori-workbench.github.io`
3. 스트림 상세의 **측정 ID**(`G-XXXXXXXXXX`)를 복사
4. `_config.yml`:
   ```yaml
   analytics:
     google:
       id: G-XXXXXXXXXX
   ```
5. 푸시 후 GA 실시간 보고서에서 본인 방문이 잡히는지 확인

선택지:
- **새 스트림(권장)**: 같은 속성 안에 스트림을 추가하면 두 블로그를 한 계정에서 보면서 데이터는 분리된다.
- 기존 블로그의 측정 ID(`G-MHT4546M0V`)를 그대로 쓰면 설정은 빠르지만 두 사이트 트래픽이 섞인다.
- Search Console과 GA를 연결하면(GA 관리 → 제품 링크) 검색어 데이터를 GA에서 볼 수 있다.

## 3. Google AdSense

1. https://adsense.google.com → 사이트 추가 → `osori-workbench.github.io`
2. 발급되는 **게시자 ID**(`ca-pub-XXXXXXXXXXXXXXXX`)를 `_config.yml` 에 입력:
   ```yaml
   adsense:
     client: ca-pub-XXXXXXXXXXXXXXXX
   ```
3. `ads.txt` 의 `pub-` 뒤 숫자가 본인 게시자 ID와 같은지 확인 (`ca-` 접두어는 `ads.txt`에서는 뺀다)
   ```
   google.com, pub-XXXXXXXXXXXXXXXX, DIRECT, f08c47fec0942fa0
   ```
4. 푸시·배포 후 AdSense에서 **사이트 확인**(코드 스니펫 방식)과 **ads.txt** 상태를 확인
5. 승인 심사를 기다린다. 신규 사이트는 보통 며칠~몇 주 걸린다.

승인 전 체크리스트(자주 반려되는 이유):
- 글 수가 너무 적음. 최소 15~20편은 쌓은 뒤 신청하는 쪽이 안전하다.
- 개인정보처리방침/소개/연락처 페이지가 없음. `_tabs/about.md` 아래에 연락처를 쓰고, 방침 페이지를 하나 추가한다.
- 자동 생성 글 의심. 원문 링크 표기, 직접 쓴 비교·분석 비중, 차트 같은 독자 가치 요소가 중요하다.
- `github.io` 서브도메인은 승인이 까다롭다. 반려되면 커스텀 도메인 연결을 고려한다.

특정 글에서 광고를 빼려면 front matter에 `ads: false`.

## 4. 확인 방법

```bash
curl -s https://osori-workbench.github.io/ | grep -E "google-site-verification|googletagmanager|adsbygoogle"
curl -s https://osori-workbench.github.io/ads.txt
curl -s -o /dev/null -w "%{http_code}\n" https://osori-workbench.github.io/sitemap.xml
```

- 소유권 확인 메타 태그 / GA 스크립트 / AdSense 스크립트가 각각 한 번씩 보여야 한다.
- 로컬에서는 `JEKYLL_ENV=production`일 때만 GA 코드가 들어간다(배포본은 해당).

## 5. 개인정보/동의 배너

EU/영국 방문자에게 개인 맞춤 광고를 쓰려면 AdSense의 **개인정보 보호 및 메시지** 메뉴에서 동의 메시지(CMP)를 켜야 한다. 한국 대상 블로그라면 우선순위는 낮지만, 승인 후 한 번 확인해 둔다.
