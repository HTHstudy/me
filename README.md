# Why-Driven Development

프론트엔드 개발에서의 의사결정과 트레이드오프를 기록하는 문서 저장소입니다.

## 📖 문서 보기

**온라인 문서 사이트:** [https://hthstudy.github.io/me/](https://hthstudy.github.io/me/)

문서 소개와 읽는 방법은 [문서 사이트](https://hthstudy.github.io/me/)에서 확인하세요.

## 🚀 로컬에서 실행하기

로컬에서 Jekyll 서버를 실행하여 문서를 미리 볼 수 있습니다.

### 필요 조건

- Ruby 3.0 이상
- Bundler

### 설치 및 실행

```bash
# 저장소 클론
git clone https://github.com/hthstudy/me.git
cd me

# 의존성 설치
bundle install

# Jekyll 서버 실행
bundle exec jekyll serve
```

서버가 시작되면 브라우저에서 `http://127.0.0.1:4000/me/`로 접속하세요.

### 참고

- `_site/` 폴더는 Jekyll 빌드 결과물로 자동 생성됩니다 (Git에 포함되지 않음)
- CSS 변경사항은 자동으로 재생성됩니다
- Ruby 버전이 3.0 미만인 경우 업그레이드가 필요합니다

## 📁 프로젝트 구조

```
.
├── _config.yml          # Jekyll 설정
├── _sass/               # 커스텀 스타일
├── docs/                # 문서 소스 파일
├── Gemfile              # Ruby 의존성
└── index.md             # 문서 사이트 홈페이지
```

## 라이센스

이 저작물은 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 라이센스를 따릅니다 - 출처를 표시하는 한 자유롭게 사용하고 수정할 수 있습니다.
