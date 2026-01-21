---
layout: default
title: Decisions over Patterns
nav_order: 3
has_children: true
permalink: /docs/decisions-over-patterns
---

# Decisions over Patterns

이 섹션은  
"**어떤 패턴을 써야 하는가**"보다  
"**왜 이 결정이 필요한가**"를 먼저 묻는 글들로 구성되어 있다.

패턴은 결과물이지,  
시작점이 아니다.

---

## 왜 Decisions over Patterns인가

패턴을 먼저 선택하고  
문제를 그 안에 끼워 맞추려 하면,  
불필요한 복잡도가 생긴다.

패턴은  
문제와 제약을 이해한 후에  
자연스럽게 도출되는 **결정의 결과**다.

이 섹션은  
"이 패턴을 써야 해"가 아니라,  
"이 상황에서 이런 이유로 이런 결정을 내렸다"를 기록한다.

---

## 이 섹션이 다루는 결정들

Decisions over Patterns의 질문은  
"어떤 패턴이 좋은가"가 아니라,  
항상 "이 선택이 무엇을 최적화하는가"에서 시작한다.

- 언제 제어 컴포넌트를 선택하고, 언제 비제어 컴포넌트를 선택하는가
- 상태를 어디에 둘 것인가 — 컴포넌트? Context? 외부 스토어?
- Context는 전역 상태인가, 의존성 주입인가
- Props를 내릴 것인가, Composition으로 합칠 것인가
- 커스텀 훅은 언제 만들고, 언제 만들지 않는가

이 질문들은  
"정답"이 아니라,  
**트레이드오프를 인식하고 선택하는 과정**을 기록한다.

---

## Mental Model과의 차이

Mental Model이 **프레임워크에 독립적인 사고 방식**이라면,  
Decisions over Patterns는 **React 생태계에서의 구체적인 선택**이다.

하지만 둘 다  
"왜"를 먼저 묻고,  
"어떻게"는 그 다음에 온다는 점에서 같다.
