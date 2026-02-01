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
시력이 좋지 않은 사용자는 스크린 리더로 화면의 내용을 음성으로 들을 수 있습니다.

- Screen Reader: 텍스트와 이미지 콘텐츠를 음성이나 점자로 출력하는 기술의 한 형태입니다

### Web Accessibility의 이점

- 자막은 시끄러운 환경이나 조용해야 하는 환경 등 모든 사용자에게 유용합니다
- 높은 색상 대비는 눈부심이 있거나 화면을 보기 어려운 상황에서 도움이 됩니다
- 고령 사용자 등 시각/청각 및 인지 기능이 저하된 사람들에게 특히 유익합니다
- 개선된 레이아웃과 디자인은 모든 사용자에게 더 나은 경험을 제공합니다

## 웹 표준은 누가 정하는가

웹 표준은 국제 기구와 다양한 이해관계자가 함께 논의하며 만들어집니다.

- W3C: World Wide Web Consortium의 약자로, 웹 기술 표준을 개발하는 국제 비영리 기관입니다
- WAI:
  - Web Accessibility Initiative의 약자로, W3C의 프로젝트입니다
  - 접근성을 이해하고 구현하는 데 도움이 되는 표준과 지원 자료를 개발합니다

## Accessibility Standards

아래 표준들은 모두 WAI에서 만들어진 웹 접근성 관련 가이드라인입니다.

### WCAG

WCAG는 Web Content Accessibility Guidelines의 약자로, WAI에서 발행한 시리즈의 일부입니다. 웹사이트와 디지털 콘텐츠가 누구에게나 접근 가능하도록 하기 위한 기준을 제공합니다.<br>

- WCAG 1.0
  - 14개의 가이드라인, 65개의 체크포인트로 구성됩니다
  - 각 가이드라인은 웹 접근성의 기본 주제를 다루며, 하나 이상의 체크포인트와 관련이 있습니다
  - 각 체크포인트는 우선순위 레벨이 있습니다:
    - Priority 1: 웹 개발자가 반드시 충족해야 하는 요구사항으로, 그렇지 않으면 하나 이상의 그룹이 웹 콘텐츠에 접근하는 것이 불가능해집니다 -> 충족하면 _A_
    - Priority 2: 웹 개발자가 충족해야 하는 요구사항으로, 그렇지 않으면 일부 그룹은 웹 콘텐츠 접근에 어려움이 있을 수 있습니다 -> 충족하면 _AA_
    - Priority 3: 웹 개발자가 일부 그룹의 웹 콘텐츠 접근을 용이하게 하기 위해 충족할 요구사항입니다 -> 충족하면 _AAA_
- WCAG 2.0
  - 4가지 핵심 원칙에 따라 구성된 12개의 가이드라인으로 구성됩니다
  - 핵심 원칙:
    - 인지 가능(Perceivable): 사람들이 콘텐츠를 보거나 들을 수 있어야 합니다
    - 작동 가능(Operable): 사람들이 인터페이스를 조작할 수 있어야 합니다
    - 이해 가능(Understandable): 콘텐츠와 사용 방법이 이해 가능해야 합니다
    - 견고함(Robust): 다양한 사용자 도구와 보조 기술에서 동작해야 합니다
  - WCAG 1.0과 같은 3가지 수준의 적합성(_A_, _AA_, _AAA_)을 재정의해 사용합니다
  - WCAG 2.1에 17개의 성공 기준이 추가됐습니다
  - WCAG 2.2에 9개의 성공 기준이 추가됐습니다 (WCAG 2.0 기준으로 26개가 추가됐습니다)

### ATAG

ATAG는 Authoring Tool Accessibility Guidelines의 약자로, 콘텐츠 제작 도구가 접근성을 지원하도록 요구사항을 정의합니다. 코드 편집기, 기타 소프트웨어가 있습니다.

### UAAG

UAAG는 User Agent Accessibility Guidelines의 약자로, 웹 브라우저와 미디어 플레이어 등 사용자 에이전트의 접근성 요구사항을 정의합니다.

## 웹 접근성 적용하기

웹 접근성과 관련된 리소스는 매우 방대하기 때문에, 실제 구현할 때는 먼저 [Quick Reference 페이지](https://www.w3.org/WAI/WCAG22/quickref/)에서 필요한 성공 기준을 확인하는 것이 제일 효율적일 것 같습니다.<br>
그리고 각 기준에 대한 구체적인 코드나 구현 방법이 필요할 때에는 해당 항목에 연결된 [Techniques 페이지](https://www.w3.org/WAI/WCAG22/Techniques/)에서 예시를 참고하면 좋습니다.

### 접근성 트리

브라우저는 HTML 요소를 DOM으로 만들고, 그중 접근성 정보만 따로 모아 DOM 트리를 기반으로 한 접근성 트리를 생성합니다.
스크린 리더 같은 기기는 이 접근성 트리를 기반으로 콘텐츠를 이해할 수 있습니다.<br>
접근성 트리 객체에는 다음 4가지 속성들이 있습니다:

- 이름(name): 요소가 사용자에게 어떻게 불려야 하는지
- 설명(description): 이름 외 추가 설명
- 역할(role): 이미지/버튼/리스트 등
- 상태(state): 체크됨/확장됨/비활성화됨 등의 상태

```html
<img src="photo.png" alt="도서관에서 책을 읽는 사람" />
```

이 요소는 브라우저 내부에서 다음과 같은 접근성 트리 속성들로 변환됩니다:

- 이름(name): "도서관에서 책을 읽는 사람" -> `alt` 속성이 접근성 트리의 이름으로 사용됩니다
- 설명(description): 없음
- 역할(role): "img"
- 상태(state): 기본값

스크린 리더는 접근성 트리를 통해 이 요소를 "이미지, 도서관에서 책을 읽는 사람"으로 이해합니다.

```html
<img src="photo.png" alt="" />
```

스크린 리더는 이 이미지를 건너뛸 수 있습니다.

### 이미지에 웹 접근성 구현해보기

먼저 Quick Reference 페이지에 들어가 필터 탭에서 `images` 필터를 적용해보았습니다.

<img src='/images/web-accessibility/quick-ref.png' width="800">

이미지는 WCAG 2.0의 4가지 핵심 원칙 중 "인지 가능(Perceivable)" 원칙과 관련이 있으며, 아래 가이드라인 및 성공 기준들에 해당합니다:

- Guideline 1.1 - Text Alternatives
  - 1.1.1 Non-text Content -- Level _A_
- Guideline 1.2 - Time-based Media
  - 1.2.8 Media Alternative (Prerecorded) -- Level _AAA_
- Guideline 1.4 - Distinguishable
  - 1.4.5 Images of Text -- Level _AA_

[Quick Reference 페이지](https://www.w3.org/WAI/WCAG22/quickref/?currentsidebar=%23col_customize&tags=images)에서 `1.1.1 Non-text Content` 항목을 펼쳐보면, 해당 성공 기준을 충족하기 위해 참고할 수 있는 다양한 기술 목록이 나옵니다.<br>
그중 가장 위에 있는 **ARIA6: Using aria-label to provide labels for objects** 링크를 클릭해봤습니다.

<img src='/images/web-accessibility/techniques.png' width="800">
<img src='/images/web-accessibility/techniques-aria6.png' width="800">

[페이지](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA6)에서 다음 내용들을 확인할 수 있습니다:

- 이 기술이 어떤 성공 기준에 대응하는지
- 이 기술이 (`aria-label` 속성) 어떤 목적으로 사용되는지, 어떤 상황에서 써야 하는지
- HTML/ARIA 구현 예시

그런데 우리는 보통 이미지에 `alt`를 사용하지 않나요?<br><br>
기술 목록을 조금 더 살펴보니, 이미지에 텍스트 대체를 제공하는 방법으로 **H37: Using alt attributes on img elements**가 포함되어 있었습니다.<br>

<img src='/images/web-accessibility/techniques-h37.png' width="800">

[페이지](https://www.w3.org/WAI/WCAG22/Techniques/html/H37)의 내용을 통해 이미지 접근성의 우선 적용 원칙은 `alt` 속성을 사용하는 것임을 다시 확인할 수 있었고,
ARIA6는 `alt`만으로 충분하지 않은 상황에서 보조 기술을 위한 이름(name)을 제공하기 위한 보조적 접근 방식이라는 점도 구체적으로 이해할 수 있었습니다.

## Takeaways

1. 웹 접근성은 장애 유무와 관계없이 모두가 웹을 동등하게 이용할 수 있도록 하는 핵심 개념입니다
2. 웹 접근성 표준은 W3C/WAI에서 개발하며, WCAG가 가장 널리 쓰입니다
   <br><br>

_출처:<br>
[1] [W3C Web Accessibility Initiative](https://www.w3.org/WAI/)<br>
[2] Wikipedia (["Web accessibility"](https://en.wikipedia.org/wiki/Web_accessibility), ["Web Content Accessibility Guidelines"](https://en.wikipedia.org/wiki/Web_Content_Accessibility_Guidelines), ["Screen reader"](https://en.wikipedia.org/wiki/Screen_reader))<br>
[3] MDN Docs (["Accessibility tree"](https://developer.mozilla.org/en-US/docs/Glossary/Accessibility_tree))<br>_
