# 잡소리 쓰는 곳

개발자가 아무 말이나 쓰는 블로그. 기술을 "이게 없던 시절엔 어떻게 했나"부터 거슬러 올라가 주니어에게 설명하듯 푼다.
글은 봇이 매일 쓴다.

https://osori-workbench.github.io

- 테마: [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) (gem 방식, chirpy-starter 구조) + 베이지·브라운 팔레트
- 호스팅: GitHub Pages (GitHub Actions 배포)
- 글 생성: [dev-blog-bot](https://github.com/osori-workbench/dev-blog-bot) (비공개)

## 글이 올라오는 방식

```
HN / GeekNews / GitHub / Dev.to / 기업 테크블로그 / 릴리스 노트
        -> 후보 수집, 점수화, 중복 제거
        -> Hermes가 글 작성 (기술의 역사 흐름 + 샘플 코드 + Mermaid/SVG/표)
        -> 썸네일 생성
        -> _posts 에 커밋/푸시 -> Actions가 배포
```

## 구조

```
_config.yml                  사이트 설정 (제목, 분석, 광고 키)
_tabs/                       정보, 카테고리, 태그, 아카이브
_posts/                      글 (봇이 자동 커밋)
_includes/
  metadata-hook.html         head 훅: AdSense, mermaid 스타일 로드
  mermaid-tweaks.html        Mermaid 둥근 룩앤필 + 베이지 팔레트
assets/css/jekyll-theme-chirpy.scss   색상 테마 (라이트/다크 팔레트)
assets/img/                  아바타, 파비콘, 글별 썸네일
ads.txt                      AdSense
docs/SETUP-GUIDE.md          Search Console / GA4 / AdSense 설정 가이드
```

## 글 규칙 (front matter)

```yaml
title: "제목"
date: 2026-10-21 09:00:00 +0900   # 미래 시각이면 빌드되지 않는다
categories: [Dev, Web]
tags: [spring, servlet]
image:
  path: /assets/img/posts/<파일>.png
mermaid: true                      # Mermaid 블록을 쓰는 글만
ads: false                         # 이 글에서 광고 제외 (선택)
```

## 로컬 실행

Ruby 3.x 필요 (macOS 기본 Ruby 2.6으로는 안 된다).

```bash
bundle install
bundle exec jekyll serve --drafts
```

## 배포

`main` 푸시 시 `.github/workflows/pages-deploy.yml` 이 빌드·배포한다. 약 2분.
Pages 소스는 반드시 **GitHub Actions** 여야 한다.

## 설정 메모

- `baseurl` 은 항상 `""`. 전체 URL을 넣으면 CSS/JS 경로가 깨진다.
- PWA는 꺼 둠 (`pwa.enabled: false`, `pwa.cache.enabled: false`). 켜면 "새 콘텐츠" 팝업이 뜨고 오래된 캐시가 남는다.
- 색상은 `data-bs-theme` 속성 기준 선택자로 덮어쓴다 (`data-mode` 아님).
- 검색·광고·분석 설정은 [docs/SETUP-GUIDE.md](docs/SETUP-GUIDE.md).

## 라이선스

테마는 MIT (Chirpy, Cotes Chung). 글과 이미지는 별도 표기가 없으면 작성자에게 있다.
