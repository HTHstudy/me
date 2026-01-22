---
layout: home
title: Home
nav_order: 1
---

# Why-Driven Development

왜(Why)에 집착하는 개발자입니다.  
저는 구현보다 판단을, 패턴보다 결정을 먼저 생각합니다.  
이 문서는 개발 과정에서 내려진 선택들을 통해  
사고의 기준을 남기기 위한 기록입니다.  

---

## 이것은 무엇인가?

이것은 **핸드북이 아니며**, **모범 사례 모음도 아닙니다**.

이 문서는 올바른 방법을 설명하지 않는다.  
개발 과정에서 내려진 결정과  
그 결정이 왜 필요했는지를 기록한다.

"올바른 방법"을 제시하기보다는,  
**의사결정과 트레이드오프**에 집중합니다.

- 문제를 어떻게 바라보는지,
- 대안들 사이에서 어떻게 선택하는지,
- 추상화가 언제 도움이 되고 해가 되는지,
- 시스템이 성장할 때 복잡도를 어떻게 통제하는지.

---

## 구성

이 저장소는 두 개의 주요 섹션으로 구성됩니다.

### [Mental Model](docs/mental-model) — 코드보다 먼저

코드를 작성하기 _전에_ 프론트엔드 시스템을 어떻게 개념화하는지에 대한 사고 모델입니다.

프레임워크에 독립적인 질문들:
- [컴포넌트 복잡도](docs/mental-model/component-complexity) — 무엇이 컴포넌트를 읽기 어렵게 만드는가
- [관심사 분리](docs/mental-model/separation-of-concerns) — 경계를 어떻게 나눌 것인가
- [추상화](docs/mental-model/abstraction) — 무엇을 하나로 묶을 것인가
- [재사용과 유연성](docs/mental-model/reusability-flexibility) — 왜 재사용하려다 복잡해지는가
- [비용](costs) — 무엇을 지불하고 있는가

### [Decisions over Patterns](docs/decisions-over-patterns) — 패턴이 아닌 결정

패턴은 결과물이지, 시작점이 아닙니다.  
React 생태계에서 마주하는 구체적인 선택과 그 이유를 기록합니다.

_(이 섹션은 작성 중입니다)_

---

## 읽는 방법

**처음 방문한다면:**  
[Mental Model](docs/mental-model)부터 시작하세요.  
순서대로 읽으면 사고 모델이 점진적으로 형성됩니다.

**특정 주제에 관심이 있다면:**  
원하는 문서로 바로 가도 됩니다.  
각 문서는 독립적으로 읽을 수 있습니다.

**React 특화 결정을 찾는다면:**  
[Decisions over Patterns](docs/decisions-over-patterns)를 확인하세요.

---

## 왜 이것이 존재하는가

프론트엔드의 복잡도는 도구 자체에서 거의 발생하지 않습니다.  
보통 **불명확한 경계, 시기상조한 추상화, 검토되지 않은 결정**에서 발생합니다.

이 저장소는 그러한 결정들을 명시적으로 만들기 위해 존재합니다.

---

## 라이센스

Creative Commons Attribution 4.0 International License (CC BY 4.0)
