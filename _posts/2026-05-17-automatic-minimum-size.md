---
title: "Automatic Minimum Size"
date: 2026-05-17
---

이번 포스트에서는 CSS Grid에서 `1fr` 컬럼이 균등 분배되지 않는 현상과, 그 원인이 되는 grid/flex 아이템의 Automatic Minimum Size 동작에 대해 살펴봤습니다.
<br><br>

<img src='/images/automatic-minimum-size/automatic-minimum-size.jpg' width="800">

## 이슈

우리 디자인 시스템에는 접기/펼치기 컨테이너와, 그 안에 들어가는 라벨-값 쌍 아이템이 별도 컴포넌트로 정의되어 있었습니다. 각각의 아이템 스타일은 있었지만, 이 아이템들이 표 형태로 나열될 것은 처음부터 고려되지 않았습니다.<br>
그러다 디자인 시스템 사용처에서 아이템을 여러 2열로 배치해야 하는 요구사항이 생겼고, 디자인 시스템에 그리드 컴포넌트가 없어서 사용처에서 직접 그리드를 짜야 했습니다.

```css
.fields {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
```

```jsx
<Collapsible title="">
  <div className="fields">
    {fields.map((field) => (
      <FieldItem key={field.id} label={field.label} value={field.value} />
    ))}
  </div>
</Collapsible>
```

`1fr 1fr`이면 컨테이너 가로 폭을 정확히 반씩 나눠 가질 거라고 기대했지만, 실제로는 그렇지 않았습니다.

- 한 컬럼에 긴 텍스트나 인풋 컴포넌트가 들어가면 그 컬럼이 더 넓어졌습니다
- 더 심한 경우 컬럼이 컨테이너를 빠져나가 가로 스크롤이 생겼습니다
- 간단한 케이스를 테스트하는 스토리에서는 의도대로 잘 동작하다가, 실제 데이터를 적용할 때 발견된 패턴이었습니다

남은 공간을 똑같이 분배하려는 의도로 `1fr`을 사용했는데, 왜 콘텐츠의 길이가 분배 결과에 영향이 있는지 혼란스러웠습니다.

## `min-width: auto`

문제의 핵심은 grid(또는 flex) 아이템의 `min-width` 기본값에 있습니다.<br>
블록 요소의 일반적인 `min-width` 기본값은 `0`이지만, grid 아이템과 flex 아이템은 기본값이 `auto`입니다.
이 `auto`가 실제로 어떻게 계산되는지가 `1fr` 분배가 깨지는 직접적인 원인이라고 할 수 있습니다.

### 계산 방식

CSS Grid Layout Module의 [Automatic Minimum Size of Grid Items](https://www.w3.org/TR/css-grid-1/#min-size-auto) 섹션에 정의돼 있습니다.<br>
이 내용을 요약하면 다음과 같습니다.

- grid 아이템의 `min-width: auto`는 해당 아이템의 min-content size로 해석됩니다
- min-content size는 "이 박스가 콘텐츠를 자르지 않고 표시할 수 있는 가장 작은 너비"입니다
- 텍스트의 경우 가장 긴 unbreakable 단어 하나의 너비가 됩니다
- 이미지나 `<input>`처럼 고유 너비를 가진 요소는 그 너비가 됩니다

즉 grid 아이템은 기본적으로 "내 콘텐츠를 자르지 않을 만큼은 무조건 차지하겠다"는 의지를 갖게 됩니다.

### `1fr`과의 충돌

`1fr`은 남은 공간을 분배하는 단위입니다. 그런데 grid 아이템의 minimum size가 보장되어 있다면, “남은 공간” 자체가 줄어들거나 음수가 됩니다.<br>
긴 단어가 들어간 컬럼이 자기 min-content를 확보하고 나면 다른 컬럼이 받을 수 있는 `1fr`이 더 작아지고, 결과적으로 두 컬럼의 폭이 균등하지 않게 됩니다. 컨테이너 폭을 모두 합해도 콘텐츠의 min-content보다 작으면 컬럼이 컨테이너를 빠져나가 스크롤이 생기는 케이스도 같은 원인입니다.<br>

## 해결 방법

### `min-width: 0` 명시

가장 직접적인 방법은 자식 아이템에 `min-width: 0`을 명시해 암묵적 minimum size 동작을 해제하는 것입니다.

```css
.fields > * {
  min-width: 0;
}
```

이렇게 하면 자식 아이템이 자기 콘텐츠보다 좁아져도 괜찮다고 선언하게 되어 `1fr` 분배가 의도대로 동작합니다.<br>
콘텐츠가 잘려야 하는 케이스라면 `overflow: hidden` 또는 `text-overflow: ellipsis`를 함께 사용합니다.

### `minmax(0, 1fr)`

`grid-template-columns` 자체를 수정하는 방법도 있습니다.

```css
.fields {
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
}
```

`minmax(0, 1fr)`은 컬럼의 최소 크기를 명시적으로 `0`으로 지정해 자동 minimum size를 우회합니다. 자식 컴포넌트의 스타일을 손대지 않아도 된다는 게 장점입니다.

### flex의 경우

flex에서도 같은 방식이 통합니다.

```css
.row > * {
  min-width: 0;
}
```

특히 자식이 또 다른 flex 컨테이너인 중첩 구조에서 자주 발생하므로, 텍스트 truncation이 필요한 경계마다 명시적으로 `min-width: 0`을 끊어주는 패턴이 안전합니다.

## 왜 이렇게 디자인됐는가

`min-width: auto`가 grid/flex 아이템의 기본값이 된 것은 의도된 디자인입니다.<br>

대부분의 경우 사용자는 콘텐츠가 잘리지 않고 보이기를 원합니다. 만약 기본값이 `0`이라면 컨테이너가 약간만 좁아져도 텍스트나 이미지가 잘려보이게 될 것입니다.<br>
그래서 spec은 "명시적으로 줄여도 된다고 선언하지 않는 한 콘텐츠를 보여준다"는 쪽을 기본으로 잡은 것 같습니다.<br>
문제는 이 보호 동작이 `1fr` 같은 분배 단위와 만나면 직관에 반하는 결과를 만든다는 것이고, 그래서 grid/flex 레이아웃에서는 `min-width: 0`을 명시하는 일이 일종의 관용구처럼 굳어졌습니다.

## Takeaways

1. grid/flex 아이템의 `min-width` 기본값은 `auto`이고, 콘텐츠의 min-content size로 해석됩니다
2. `1fr` 또는 `flex: 1`이 의도대로 분배되지 않는다면 대부분 암묵적 minimum size가 원인입니다
3. 자식에 `min-width: 0`을 주거나 컨테이너 쪽에서 `minmax(0, 1fr)`을 써서 해결할 수 있습니다
4. 테스트 시 긴 텍스트/이미지를 포함한 실제 케이스로 검증하는 것이 좋습니다
   <br><br>

_출처:<br>
[1] W3C CSS Grid Layout Module Level 1 ([“Automatic Minimum Size of Grid Items”](https://www.w3.org/TR/css-grid-1/#min-size-auto))<br>
[2] W3C CSS Flexible Box Layout Module Level 1 ([“Automatic Minimum Size of Flex Items”](https://www.w3.org/TR/css-flexbox-1/#min-size-auto))<br>_
