---
title: "IME Composition"
date: 2026-05-10
---

이번 포스트에서는 한글 입력 환경에서 키보드 이벤트가 의도치 않게 두 번 발생하는 이슈와 해결하는 방법에 대해 살펴봤습니다.
<br><br>

## 이슈

웹 앱을 개발하던 중에 키보드 이벤트를 처리하던 중 다음과 같은 이상한 동작들을 경험했습니다.

- ESC 키를 눌렀을 때 종료 확인창이 2개 뜨는 현상
- 텍스트를 입력하고 Enter 키를 쳐서 태그를 생성하는 기능을 구현했는데, 마지막 글자가 별도 태그로 하나 더 생성되는 현상 (예를 들어 `기면스` 태그를 만들고 싶었는데 `스` 태그가 하나 더 생성됨)
- 그 외 키보드를 사용하는 경우 특정 액션이 중복 실행되는 현상

(한 번에 결론에 도달한 것은 아니었지만) 이 문제의 특징은 명확했습니다. 영문 입력에서는 정상 동작하지만 한글 입력 중일 때만 발생한다는 것이었습니다.

## Input Method Editor

IME는 키보드로 직접 입력하기 어려운 문자를 입력할 수 있도록 도와주는 OS 레벨의 컴포넌트입니다.

- 라틴 키보드 환경에서 한국어, 일본어, 중국어처럼 자모나 획의 조합으로 문자를 구성하는 언어를 입력할 수 있게 해줍니다
- IME라는 용어는 Windows에서 먼저 사용됐으며, 현재는 macOS, Linux, Android 등 대부분의 OS에서 동일한 개념으로 통용됩니다

이런 언어들은 알파벳처럼 키 하나가 문자 하나 구조가 아니라 여러 입력을 조합해서 하나의 문자를 만든다는 특징이 있습니다.
예를 들어 한글 "면"을 입력하려면 "ㅁ", "ㅕ", "ㄴ" 세 번의 입력이 조합되어야 합니다. 그래서 입력 과정에는 두 가지 상태가 존재합니다.

- 조합 중: 아직 글자가 확정되지 않은 상태
- 조합 완료: 최종 문자가 확정된 상태

이 상태를 관리하는 것이 바로 IME의 역할입니다.

### Composition 이벤트

브라우저는 OS에 내장된 IME와 통신하면서 조합 상태를 세 가지 이벤트로 노출합니다.<br>
`keydown`/`keyup`이 "어떤 키가 눌렸는지"를 알려준다면, composition 이벤트는 "IME가 지금 어떤 상태인지"를 알려줍니다.
둘 다 `UIEvent`를 상속하는 독립적인 이벤트로, 한쪽이 다른 쪽에 속하는 관계는 아닙니다.

- `compositionstart`:
  - 조합 세션이 시작될 때 발생합니다
  - 한글 모드에서 첫 자모를 타이핑하는 순간 발생합니다
  - 한/영 키로 입력 모드를 전환하는 것으로는 발생하지 않습니다
- `compositionupdate`:
  - 조합 중 값이 바뀔 때마다 발생합니다
  - `event.data`에는 그 시점의 조합 중인 문자열이 담겨있습니다
- `compositionend`:
  - 조합이 확정되는 상황에서 발생합니다
  - Space 키, Enter 키, 또는 현재 조합과 이어질 수 없는 자모가 입력될 때 발생합니다
  - 예를 들어 "기" 다음에 "ㅁ"이 오면 "김"이 될 수 있으므로 세션을 유지하지만, 더 이상 조합이 이어질 수 없는 시점에 발생합니다

### 이벤트 뷰어

W3C가 직접 제공하는 [Keyboard Event Viewer](https://w3c.github.io/uievents/tools/key-event-viewer.html)에서 실제 이벤트 발생 순서를 직접 확인할 수 있습니다.

<img src='/images/ime-composition/keyboard-event-viewer-chrome.png' width="800">

Safari의 경우에 Chrome과 속성이 조금 다릅니다.

<img src='/images/ime-composition/keyboard-event-viewer-safari.png' width="800">

### 왜 이벤트가 두 번 발생하는가

문제의 핵심은 IME 환경에서는 하나의 키 입력이 "조합 확정"과 "키 입력" 두 흐름으로 처리될 수 있다는 점입니다. 실제로는 Enter를 한 번 눌렀는데, 같은 키 입력이 조합 확정과 실제 Enter 입력을 동시에 트리거하기 때문에 핸들러가 두 번 실행됩니다.
composition 이벤트를 구독하지 않으면 `keydown`이 조합 중에 발생한 것인지 아닌지 구분할 방법이 없어서, 한글 입력 중 모든 `keydown`을 그냥 처리해버리게 됩니다.

## 해결 방법

### `event.isComposing` 체크

`KeyboardEvent`에는 `isComposing`이라는 boolean 속성이 있습니다. `compositionstart` 이후 `compositionend` 이전에 발생한 키보드 이벤트라면 (현재 이벤트가 IME 조합 세션 내부에서 발생했다면) `true`를 반환합니다.

```js
input.addEventListener("keydown", (e) => {
  if (e.isComposing) return; // IME 조합 중이면 무시
  if (e.key === "Enter") createTag();
});
```

가장 간단하고 일반적으로 많이 사용하는 방법입니다.

### `inputType === "insertReplacementText"` 활용

Safari에서는 한글 IME 조합 확정 시점에 `compositionend`가 먼저 발생하면서 `keydown` 시점에 이미 `isComposing === false` 상태인 것 같습니다.
대신 Safari는 `beforeinput`/`input` 이벤트에서 `insertReplacementText` 값을 제공합니다.

```js
input.addEventListener("beforeinput", (e) => {
  if (e.inputType === "insertReplacementText") {
    // Safari IME 조합 확정 처리
  }
});
```

때문에 `insertReplacementText`가 IME 조합 확정을 감지할 수 있는 실질적인 신호로 동작합니다.

### `keyCode` 229 체크

오래된 브라우저나 Android 가상 키보드 대응이 필요한 경우, IME가 처리한 키 이벤트는 `keyCode`가 `229`로 설정됩니다.

```js
input.addEventListener("keydown", (e) => {
  if (e.isComposing || e.keyCode === 229) return;
  if (e.key === "Enter") createTag();
});
```

`keyCode`는 deprecated된 속성이므로 신규 코드에서는 `isComposing`을 우선 사용하고, 레거시 환경 대응이 필요할 때만 추가 조건으로 활용하는 것이 좋습니다.

## Takeaways

1. IME 환경에서는 하나의 키 입력이 "조합 확정"과 "키 입력" 두 흐름으로 처리될 수 있습니다
2. `event.isComposing`은 가장 기본적인 해결 방법이지만 브라우저마다 이벤트 순서가 다를 수 있습니다
3. Safari에서는 `insertReplacementText`가 IME 조합 확정을 판단하는 중요한 신호가 될 수 있습니다
4. 멀티 브라우저 환경이라면 반드시 실제 기기에서 테스트해봐야 합니다
   <br><br>

_출처:<br>
[1] Wikipedia (["Input method"](https://en.wikipedia.org/wiki/Input_method))<br>
[2] W3C UI Events (["Composition Events"](https://www.w3.org/TR/uievents/#events-compositionevents))<br>_
