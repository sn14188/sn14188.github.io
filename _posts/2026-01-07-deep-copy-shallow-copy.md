---
title: "Deep Copy and Shallow Copy"
date: 2026-01-07
---
<br><br>

## 자바스크립트 타입과 자료 구조

### 원시값
객체를 제외한 모든 타입은 참조가 아닌 값 자체로 취급되며 불변성을 가집니다.
이러한 타입의 값을 원시값이라고 하는데, `Null`, `Undefined`, `Boolean`, `Number`, `BigInt`, `String`, `Symbol` 타입이 있습니다.

- `Null`: `null`이라는 하나의 값만 가질 수 있습니다
- `Undefined`: `undefined`라는 하나의 값만 가질 수 있습니다

개념적으로 `undefined`는 값이 없음을 의미하고, `null`은 객체가 없음을 의미합니다.
관용적으로 `undefined`는 값이 아직 할당되지 않았음을 나타내고, `null`은 의도적으로 객체 참조가 없음을 표현하기 위해 사용됩니다.<br>
`Symbol`은 객체 프로퍼티 키의 충돌을 방지하기 위해 도입된 타입으로, 생성될 때마다 고유한 값을 가지며 주로 객체의 내부 식별자나 은닉된 속성을 표현하는 데 사용됩니다.

### 객체
컴퓨터 과학에서 객체란 식별자를 통해 참조할 수 있는 값을 의미하며, 자바스크립트에서 객체는 변경이 가능한 값입니다.<br>
객체는 키-값 쌍으로 이루어진 프로퍼티의 컬렉션이며, 프로퍼티 키는 `String` 또는 `Symbol` 타입을, 프로퍼티 값은 객체를 포함한 모든 타입을 가질 수 있습니다. 이를 통해 다양한 형태의 복잡한 자료 구조를 표현할 수 있습니다.

## 자바스크립트 복사

### 값 복사와 참조 복사
원시값은 참조가 아닌 값 자체이기 때문에, 변수를 복사하면 항상 새로운 값이 생성됩니다.
```js
let x = 8;
let y = x;

y = 42;
console.log(x); // 8
```
이처럼 원시값의 복사는 원본과 복사본이 서로 영향을 주지 않으므로, 복사 방식에 따른 문제가 발생하지 않습니다.<br>
반면 객체는 참조를 통해 다뤄지기 때문에, 객체를 복사하면 동일한 객체를 가리키는 참조가 복사됩니다.
```js
const object_1 = { value: 8 };
const object_2 = object_1;

object_2.value = 42;
console.log(object_1.value); // 42
```
위 예제에서 object_1과 object_2는 같은 객체를 참조하고 있습니다. 따라서 한쪽에서 객체를 변경하면, 다른 쪽에서도 그 변경 내용을 그대로 확인할 수 있습니다.<br>
이러한 특성 때문에 객체를 복사할 때는, 복사 범위에 따라 얕은 복사와 깊은 복사를 구분해 다뤄야 합니다.

### 얕은 복사
얕은 복사는 객체를 복사할 때 최상위 프로퍼티만 새로운 객체로 복사하고, 내부에 중첩된 객체나 배열은 원본과 동일한 참조를 유지하는 방식입니다.
```js
const object_1 = {
  value: 8,
  nested: {
    value: 42
  }
};

const object_2 = { ...object_1 };

object_2.value = 42;
object_2.nested.value = 8;

console.log(object_1.value);        // 8
console.log(object_1.nested.value); // 8
```
최상위 프로퍼티인 `value`는 원시값이므로 값 자체가 복사되어 서로 영향을 주지 않습니다.<br>
반면 `nested`는 객체이기 때문에 참조가 복사되어, 얕은 복사본에서 내부 객체를 변경하면 원본 객체에도 동일한 변경이 발생합니다.

## Takeaways
1. takeaway 1
2. takeaway 2
<br><br>

*출처:<br>
[1] MDN Docs (
["JavaScript data types and data structures"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures),
["Shallow copy"](https://developer.mozilla.org/en-US/docs/Glossary/Shallow_copy),
["Deep copy"](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy)
)<br>*
