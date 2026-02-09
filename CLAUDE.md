# CLAUDE.md

이 파일은 Claude Code(claude.ai/code)가 이 저장소의 코드를 다룰 때 참고하는 가이드입니다.

## 프로젝트 개요

**Why-Driven Development**라는 이름의 한국어 Jekyll 문서 사이트입니다. 프론트엔드 아키텍처 의사결정에 대한 개인 지식 저장소로, "어떻게"보다 "왜"에, 패턴보다 결정에 초점을 둡니다. 모든 콘텐츠는 한국어로 작성되며 TypeScript/React 코드 예제를 포함합니다.

배포 사이트: https://hthstudy.github.io/me/

## 빌드 및 개발

```bash
bundle install            # Ruby 의존성 설치
bundle exec jekyll serve  # 개발 서버 실행 (http://127.0.0.1:4000/me/)
bundle exec jekyll build  # _site/ 디렉토리로 빌드
```

Node.js/npm 없이, `just-the-docs` 원격 테마를 사용하는 순수 Jekyll/Ruby 프로젝트입니다.

## 콘텐츠 구조

`docs/` 아래 세 개의 단계적 섹션으로 구성됩니다:

1. **Mental Model** (`docs/mental-model/`) — 프레임워크에 구애받지 않는 해석 프레임워크. "왜 이런 일이 생기는가"
2. **Decisions over Patterns** (`docs/decisions-over-patterns/`) — React 특화 의사결정 과정. "이 상황에서 무엇을 선택할 것인가"
3. **Deep Dives** (`docs/deep-dives/`) — 심층 개념 탐구 (계획 중)

각 섹션은 `index.md`와 예정 문서 목록인 `_roadmap.md`를 포함합니다.

## 콘텐츠 작성 규칙

### Mental Model 문서 (`docs/mental-model/`)
- 답이 아닌 해석 프레임워크를 제공한다
- 제목 + 독자의 사고를 고정시키는 질문형 인용구로 시작한다
- "왜 이런 선택이 일어나는가"를 둘러싼 상황, 힘, 긴장, 트레이드오프를 서술한다
- **금지 사항:** 결론/처방을 먼저 제시하지 않는다, 맥락 없이 "답"으로 인용 가능한 문장을 만들지 않는다, "~해야 한다 / ~가 옳다" 패턴을 반복하지 않는다

### Decisions over Patterns 문서 (`docs/decisions-over-patterns/`)
- 패턴을 처방하지 않고 의사결정 과정을 보여준다
- 필수 섹션: **선택지들** ("언제?/왜?" + 코드가 포함된 선택지), **판단의 질문들** (결정을 위한 질문들), **결정 흐름 예시** (의사결정 흐름 예시), **정리하며** (비교 표가 포함된 요약)
- 규칙은 "절대적 원칙이 아닌, 반복적으로 유효한 패턴"으로 표현한다
- **금지 사항:** 하나의 선택지를 "정답"으로 단정하지 않는다, "항상 ~해야 한다"를 사용하지 않는다, 트레이드오프를 생략하지 않는다

### 공통
- 톤: 성찰적, 질문적, 비처방적
- 규칙처럼 보이는 문장은 관찰 가능한 현상이나 반복되는 패턴으로 다시 쓴다
- Markdown 줄바꿈 시 줄 끝에 공백 2칸(`  `)을 반드시 넣는다 (Jekyll이 `<br>`로 변환)

## Git 컨벤션

커밋 메시지는 한국어로 작성하며 접두사를 사용합니다: `update:`, `style:`, `chore:`, `fix:`

## 주요 설정 파일

- `_config.yml` — Jekyll 설정 (테마, 검색, 기본 URL `/me/`)
- `assets/css/style.scss` — 커스텀 타이포그래피 오버라이드
- `_includes/head_custom.html` — 커스텀 CSS 링크
- 라이선스: CC BY 4.0
