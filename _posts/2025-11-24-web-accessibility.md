---
title: "Web Accessibility"
date: 2025-11-24
---
웹 접근성은 모든 사용자가 더 편하고 더 쉽게 웹을 이용할 수 있도록 해 주는 중요한 개념입니다.
이번 포스트에서는 웹 접근성이 무엇인지, 어떤 이점이 있는지, 그리고 어떤 기준을 통해 구현할 수 있는지 살펴보겠습니다.
<br><br>

## Web Accessibility
웹 접근성이란 장애가 있든 없든 누구나 웹을 동등하게 이용할 수 있도록 보장하는 것을 의미합니다.
예를 들어, 팔을 자유롭게 쓰지 못하는 사람은 마우스 스틱을 사용해 타이핑할 수 있습니다.
청각이 불편한 사용자는 영상의 자막을 통해 정보를 얻을 수 있고,
시력이 좋지 않은 사용자는 Screen Reader로 화면의 내용을 음성으로 들을 수 있습니다.

### Benefits
- 자막은 시끄러운 환경이나 조용해야 하는 환경 등 모든 사용자에게 유용합니다
- 높은 색상 대비는 눈부심이 있거나 화면을 보기 어려운 상황에서 도움이 됩니다
- 고령 사용자 등 시각/청각 및 인지 기능이 저하된 사람들에게 특히 유익합니다
- 개선된 레이아웃과 디자인은 모든 사용자에게 더 나은 경험을 제공합니다

## 웹 표준은 누가 정하는가
웹 표준은 국제 기구와 다양한 이해관계자가 함께 논의하며 만들어집니다.

### W3C
W3C는 World Wide Web Consortium의 약자로, 웹 기술 표준을 개발하는 국제 비영리 기관입니다.

### WAI
WAI는 Web Accessibility Initiative의 약자로, W3C의 프로젝트입니다. 접근성을 이해하고 구현하는 데 도움이 되는 표준과 지원 자료를 개발합니다.

## Accessibility Standards
아래 표준들은 모두 WAI에서 만들어진 웹 접근성 관련 가이드라인입니다.

### WCAG
WCAG는 Web Content Accessibility Guidelines의 약자로, WAI에서 발행한 시리즈의 일부입니다. 웹사이트와 디지털 콘텐츠가 누구에게나 접근 가능하도록 하기 위한 기준을 제공합니다.<br>
**WCAG 1.0:**
- 14개의 가이드라인, 65개의 체크포인트로 구성됩니다
- 각 가이드라인은 웹 접근성의 기본 주제를 다루며, 하나 이상의 체크포인트와 관련이 있습니다
- 각 체크포인트는 우선순위 레벨이 있습니다:
  - Priority 1: 웹 개발자가 반드시 충족해야 하는 요구사항으로, 그렇지 않으면 하나 이상의 그룹이 웹 콘텐츠에 접근하는 것이 불가능해집니다 -> 충족하면 *A*
  - Priority 2: 웹 개발자가 충족해야 하는 요구사항으로, 그렇지 않으면 일부 그룹은 웹 콘텐츠 접근에 어려움이 있을 수 있습니다 -> 충족하면 *AA*
  - Priority 3: 웹 개발자가 일부 그룹의 웹 콘텐츠 접근을 용이하게 하기 위해 충족할 요구사항입니다 -> 충족하면 *AAA*

**WCAG 2.0:**
- 4가지 핵심 원칙에 따라 구성된 12개의 가이드라인으로 구성됩니다
- 핵심 원칙:
  - 인지 가능(Perceivable): 사람들이 콘텐츠를 보거나 들을 수 있어야 합니다
  - 작동 가능(Operable): 사람들이 인터페이스를 조작할 수 있어야 합니다
  - 이해 가능(Understandable): 콘텐츠와 사용 방법이 이해 가능해야 합니다
  - 견고함(Robust): 다양한 사용자 도구와 보조 기술에서 동작해야 합니다
- WCAG 1.0과 같은 3가지 수준의 적합성(*A*, *AA*, *AAA*)을 재정의해 사용합니다
- WCAG 2.1에 17개의 성공 기준이 추가됐습니다
- WCAG 2.2에 9개의 성공 기준이 추가됐습니다 (WCAG 2.0 기준으로 26개가 추가됐습니다)

### ATAG
ATAG는 Authoring Tool Accessibility Guidelines의 약자로, 콘텐츠 제작 도구가 접근성을 지원하도록 요구사항을 정의합니다. 코드 편집기, 기타 소프트웨어가 있습니다.

### UAAG
UAAG는 User Agent Accessibility Guidelines의 약자로, 웹 브라우저와 미디어 플레이어 등 사용자 에이전트의 접근성 요구사항을 정의합니다.

## How to Apply Web Accessibility
웹 접근성과 관련된 리소스는 매우 방대하기 때문에, 실제 구현할 때는 먼저 [Quick Reference 페이지](https://www.w3.org/WAI/WCAG22/quickref/)에서 필요한 성공 기준을 확인하는 것이 제일 효율적일 것 같습니다.<br>
그리고 각 기준에 대한 구체적인 코드나 구현 방법이 필요할 때에는 해당 항목에 연결된 [Techniques 페이지](https://www.w3.org/WAI/WCAG22/Techniques/)에서 예시를 참고하면 좋습니다.

### 이미지에 웹 접근성 구현해보기
먼저 Quick Reference 페이지에 들어가 필터 탭에서 `images` 필터를 적용해보았습니다.

<img src='/images/web-accessibility/quick-ref-page.png' width="800">

이미지는 WCAG 2.0의 4가지 핵심 원칙 중 "인지 가능(Perceivable)" 원칙과 관련이 있으며, 아래 가이드라인 및 성공 기준들에 해당합니다:
- Guideline 1.1 - Text Alternatives
  - 1.1.1 Non-text Content -- Level *A*
- Guideline 1.2 - Time-based Media
  - 1.2.8 Media Alternative (Prerecorded) -- Level *AAA*
- Guideline 1.4 - Distinguishable
  - 1.4.5 Images of Text -- Level *AA*

Quick Reference 페이지에서 `1.1.1 Non-text Content` 항목을 펼쳐보면, 해당 성공 기준을 충족하기 위해 참고할 수 있는 다양한 기술 목록이 나옵니다.<br>
그중 가장 위에 있는 "ARIA6: Using aria-label to provide labels for objects" 링크를 클릭해봤습니다.

<img src='/images/web-accessibility/techniques.png' width="800">
<img src='/images/web-accessibility/techniques-page.png' width="800">

연결된 Techniques 페이지에서 다음 내용들을 확인할 수 있습니다:
- 이 기술이 어떤 성공 기준을 충족하는지
- `aria-label`을 사용해 텍스트가 없는 객체에 대체 설명을 제공하는 방법
- 구현 HTML/ARIA 코드 예시

이를 통해 단순히 "이미지에 `alt`를 넣어야 한다"를 넘어서 특정 상황에서 어떤 방법을 선택해야 하는지 구체적으로 이해할 수 있게 됐습니다.
