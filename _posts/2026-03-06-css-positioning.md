---
title: "CSS Positioning"
date: 2026-03-06
---

이번 포스트에서는 CSS의 `position` 속성이 어떤 방식으로 요소의 위치를 결정하는지, 그리고 레이아웃에서 어떻게 활용되는지 살펴봤습니다.
<br><br>

<img src='/images/css-positioning/css-positioning.jpg' width="800">

## CSS 레이아웃과 포지셔닝

### Positioning Scheme

<!-- CSS에서는 요소가 화면에 배치되는 규칙을 positioning scheme을 통해 정의합니다.<br> -->

CSS 2.1 스펙에서는 다음과 같이 세 가지 위치 지정 방식을 정의하고 있습니다. 이는 요소가 문서의 흐름에 참여하는지, 그리고 다른 요소의 배치에 어떤 영향을 주는지에 따라 구분됩니다.

- Normal Flow
  - HTML 요소가 기본적으로 배치되는 방식이며, 대부분의 레이아웃은 이 흐름을 기반으로 형성됩니다
  - 인라인 요소는 가로 방향으로 배치되다가 공간이 부족해지면 다음 줄로 이어집니다
  - 블록 요소는 단락이나 목록 항목처럼 위에서 아래로 쌓이며 배치됩니다

- Floats
  - `float` 속성이 적용된 요소는 문서 흐름에서 부분적으로 벗어나 왼쪽 또는 오른쪽으로 이동합니다
  - 주변의 다른 콘텐츠는 해당 요소의 옆을 따라 흐르며 배치됩니다
  - 과거에는 레이아웃 구성에 자주 사용되었지만, 현재는 플렉스박스나 그리드가 이를 대체하고 있습니다

- Absolute Positioning
  - 요소가 문서 흐름에서 완전히 제거됩니다
  - 요소의 위치는 지정된 기준 컨테이너를 기준으로 계산되며 다른 요소의 배치에는 영향을 주지 않습니다
  - `position: absolute`와 `position: fixed`가 이 방식에 해당합니다

## 포지션 속성의 종류

CSS에서는 `position` 속성을 사용해 요소의 위치 계산 방식을 지정할 수 있습니다.<br>
기본적으로 CSS 레이아웃 알고리즘은 기본적으로 박스들이 서로 겹치지 않도록 크기와 위치를 계산합니다. 하지만 `position` 속성을 사용하면 이러한 기본 배치 규칙을 벗어나 요소를 이동시키거나 다른 요소 위에 겹치도록 배치할 수 있습니다.

### static

`static`은 `position`의 기본값입니다.<br>
요소는 문서의 흐름에 따라 배치되며 별도의 위치 지정 메커니즘을 사용하지 않습니다.
또한 `top`, `right`, `bottom`, `left`와 같은 위치 오프셋 속성은 적용되지 않습니다.

### relative

요소는 먼저 `static`과 동일한 방식으로 배치된 뒤, 자신의 원래 위치를 기준으로 이동합니다.<br>
이 이동은 시각적인 위치만 변경될 뿐, 다른 요소의 레이아웃에는 영향을 주지 않습니다.

### absolute

요소는 문서 흐름에서 완전히 제거됩니다.<br>
따라서 형제 요소나 부모 요소의 레이아웃에 영향을 주지 않으며 다른 요소와 겹칠 수도 있습니다.
대신 요소의 위치는 특정 containing block이라고 부르는, 기준 컨테이너를 기준으로 계산됩니다.

### fixed

`fixed`는 `absolute`와 유사하게 문서 흐름에서 제거되지만, 기준 컨테이너가 고정된 영역이라는 점에서 차이가 있습니다.

- 일반적인 화면 환경에서는 이 기준이 viewport입니다
- 인쇄 환경에서는 페이지 영역이 기준이 되어 각 페이지마다 동일한 위치에 반복적으로 표시됩니다

### sticky

`sticky`는 `relative`와 유사하게 배치되지만, 스크롤 위치에 따라 요소의 위치가 자동으로 조정됩니다.<br>
요소는 자신의 기준 컨테이너 범위를 벗어나지 않는 선에서 화면ㅇ에 고정된 것처럼 동작합니다.

## 포지션 요소의 기준과 오프셋

`position` 값이 `static`이 아닌 요소를 positioned element라고 합니다.<br>
이러한 요소는 위치 오프셋 속성을 사용해 기준 컨테이너를 기준으로 위치를 조정할 수 있습니다.

### Containing Block

오프셋 값은 항상 containing block을 기준으로 계산됩니다.

| position   | containing block                                  |
| ---------- | ------------------------------------------------- |
| `static`   | N/A                                               |
| `relative` | 요소의 원래 위치                                  |
| `absolute` | 가장 가까운 positioned ancestor (없다면 viewport) |
| `fixed`    | viewport                                          |
| `sticky`   | 가장 가까운 scroll container                      |

이 때문에 아래 예시와 같은 패턴이 자주 사용됩니다. 이 경우 `.child`의 위치는 `.container`를 기준으로 계산됩니다.

```css
.container {
  position: relative;
}

.child {
  position: absolute;
  top: 0;
  right: 0;
}
```

### top / right / bottom / left

`top`, `right`, `bottom`, `left` 속성은 요소의 위치를 containing block에 대한 오프셋 값으로 지정합니다.
아래 예시에서는 요소가 기준 컨테이너의 왼쪽 상단에서 `10px`, `20px` 떨어진 위치에 배치됩니다.

```css
.box {
  position: absolute;
  top: 10px;
  left: 20px;
}
```

## 요소가 겹칠 때

앞서 살펴본 것처럼 `position` 속성을 사용하면 요소의 위치가 이동하면서 다른 요소와 겹칠 수 있습니다.
이때 어떤 요소가 앞에 보일지는 CSS의 painting order에 의해 결정됩니다.

### Painting Order

painting order는 브라우저가 요소를 화면에 그리는 렌더링 순서를 의미합니다.<br>
이 순서는 DOM 구조와 `position`, `z-index`, 그리고 stacking context의 영향을 받습니다.<br>
같은 영역에 여러 요소가 존재할 경우 나중에 그려지는 요소가 앞에 표시되고, 일반적으로 요소는 다음과 같은 순서로 그려집니다.

- 요소의 background
- 요소의 border
- 요소의 자식 요소
- positioned element
- `z-index` 값이 지정된 요소

### z-index

`z-index` 속성은 요소의 stacking level을 지정합니다.<br>
이 값은 positioned element에만 적용되며, 값이 클수록 요소는 앞쪽 레이어에 배치됩니다. 아래 예시에서는 `.box-b`가 `.box-a`보다 앞에 표시됩니다.

```css
.box-a {
  position: absolute;
  z-index: 1;
}

.box-b {
  position: absolute;
  z-index: 10;
}
```

### Stacking Context

stacking context는 요소들이 하나의 독립적인 레이어 그룹으로 묶이는 구조입니다. 어떤 요소가 stacking context를 생성하면 그 내부의 요소들은 하나의 단위로 묶이며 외부 요소와 독립적으로 정렬됩니다.<br>
대표적으로 다음과 같은 경우에 새로운 stacking context가 생성됩니다.

- `position` + `z_index`
- `opacity < 1`
- `transform`
- `filter`
- `position: fixed`
- `position: sticky`

> [CSS Painting Order](https://css-tricks.com/css-painting-order)

## Takeaways

1. CSS 레이아웃은 기본적으로 문서의 흐름을 기반으로 동작합니다
2. `position` 속성을 사용하면 요소를 문서 흐름에서 이동시키거나 분리할 수 있습니다
3. 요소가 겹칠 때의 표시 순서는 `z-index`와 stacking context에 의해 결정됩니다
   <br><br>

_출처:<br>
[1] Wikipedia (["CSS"](https://en.wikipedia.org/wiki/CSS))<br>
[2] [CSS Positioned Layout Module Level 3](https://drafts.csswg.org/css-position/)_
