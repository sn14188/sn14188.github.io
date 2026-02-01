---
title: "Deep Copy & Shallow Copy"
date: 2026-01-07
---

자바스크립트에서 객체는 참조로 다뤄지기 때문에 복사 방식에 따라 동작이 달라집니다. 이번 포스트에서는 얕은 복사와 깊은 복사의 개념과, 깊은 복사를 구현하는 여러 방법을 정리해봤습니다.
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

원시값은 참조가 아닌 값 자체이기 때문에, 대입 연산자(`=`)를 사용해 변수를 복사하면 항상 새로운 값이 생성됩니다.

```js
let x = 8;
let y = x;

y = 42;

console.log(x); // 8
```

이처럼 원시값의 복사는 원본과 복사본이 서로 영향을 주지 않으므로, 복사 방식에 따른 문제가 발생하지 않습니다.<br>
반면 객체는 참조를 통해 다뤄지기 때문에, 대입 연산자(`=`)를 사용하면 객체 자체가 복사되는 것이 아니라 동일한 객체를 가리키는 참조가 복사됩니다.

```js
const object_1 = { value: 8 };
const object_2 = object_1;

object_2.value = 42;

console.log(object_1.value); // 42
```

위 예제에서 `object_1`과 `object_2`는 같은 객체를 참조하고 있습니다. 따라서 한쪽에서 객체를 변경하면, 다른 쪽에서도 그 변경 내용을 그대로 확인할 수 있습니다.<br>
이러한 특성 때문에 객체를 복사할 때는, 복사 범위에 따라 얕은 복사와 깊은 복사를 구분해 다뤄야 합니다.

### 얕은 복사

얕은 복사는 객체를 복사할 때 최상위 프로퍼티만 새로운 객체로 복사하고, 내부에 중첩된 객체나 배열은 원본과 동일한 참조를 유지하는 방식입니다.

```js
const object_1 = {
  value: 8,
  nested: {
    value: 42,
  },
};

const object_2 = { ...object_1 };

object_2.value = 42;
object_2.nested.value = 8;

console.log(object_1.value); // 8
console.log(object_1.nested.value); // 8
```

위 코드에서 `{ ...object_1 }`은 `object_1`의 최상위 프로퍼티들을 펼쳐 새로운 객체를 생성합니다. 최상위 프로퍼티인 `value`는 원시값이므로 값 자체가 복사되어 서로 영향을 주지 않습니다.<br>
반면 `nested`는 객체이기 때문에 참조가 복사되어, 얕은 복사본에서 내부 객체를 변경하면 원본 객체에도 동일한 변경이 발생합니다.<br>
이런 참조 공유 문제를 해결하고, 원본 객체와 완전히 독립적인 복사본을 만들기 위해 깊은 복사가 필요합니다.

### 깊은 복사

깊은 복사는 객체를 복사할 때 내부에 중첩된 모든 프로퍼티를 새로운 값으로 복사하여, 원본 객체와 복사본이 어떠한 참조도 공유하지 않도록 하는 방식입니다.<br>
얕은 복사는 최상위 프로퍼티만 새로운 객체로 복사하지만, 객체의 모든 프로퍼티가 원시값으로만 구성된 경우에는 얕은 복사와 깊은 복사의 결과가 동일합니다.<br>
깊은 복사를 통해 원본이나 복사본 중 어느 하나를 변경하더라도 다른 객체에는 영향을 미치지 않도록 할 수 있습니다.

## 깊은 복사 구현하기

### `JSON.parse(JSON.stringify(object))`

구현이 가장 간단한 깊은 복사 방법입니다. 객체를 JSON 문자열로 직렬화한 뒤 다시 객체로 변환하는 과정에서 모든 참조가 끊어지기 때문입니다.<br>

```js
const object_1 = {
  value: 8,
  nested: {
    value: 42,
  },
};

const object_2 = JSON.parse(JSON.stringify(object_1));

object_2.value = 42;
object_2.nested.value = 8;

console.log(object_1.value); // 8
console.log(object_1.nested.value); // 42
```

하지만 JSON은 표현할 수 있는 데이터 타입이 제한적입니다.<br>
함수, `Symbol`, DOM 객체, 재귀 데이터 등을 포함하는 경우와 같이 직렬화할 수 없는 객체의 경우 `JSON.stringify()`는 실패하거나 일부 값이 손실되며, 이 방식으로는 올바른 깊은 복사를 수행할 수 없습니다.<br>
예를 들어 객체에 함수가 포함된 경우 정보가 손실됩니다.

```js
const object = {
  value: 8,
  fn: () => 42,
};

console.log(JSON.stringify(object)); // {"value":8}
```

### 커스텀 재귀 함수를 활용한 복사

객체를 순회하며 재귀적으로 복사하는 방법입니다. 원시값과 객체를 구분해 처리할 수 있고, 함수도 그대로 복사가 가능한데 구현이 복잡하다는 단점이 있습니다.

```js
function deepCopy(obj) {
  if (obj === null || typeof obj !== "object") {
    return obj;
  }

  const result = Array.isArray(obj) ? [] : {};

  for (const key in obj) {
    result[key] = deepCopy(obj[key]);
  }

  return result;
}

const object_1 = {
  value: 8,
  nested: {
    value: 42,
  },
};

const object_2 = deepCopy(object_1);

object_2.value = 42;
object_2.nested.value = 8;

console.log(object_1.value); // 8
console.log(object_1.nested.value); // 42
```

### Lodash와 같은 외부 라이브러리 활용

외부 라이브러리를 사용할 수 있는 환경이라면, 가장 안정적이고 범용적인 해결책이 될 수 있습니다. 다만 외부 의존성이 추가되며, 번들 크기가 증가할 수 있다는 점은 고려해야 합니다.

```js
import { cloneDeep } from "lodash";

const object_1 = {
  value: 8,
  nested: {
    value: 42,
  },
};

const object_2 = cloneDeep(object_1);

object_2.value = 42;
object_2.nested.value = 8;

console.log(object_1.value); // 8
console.log(object_1.nested.value); // 42
```

### `structuredClone(object)`

대부분의 일반적인 객체를 안전하게 깊은 복사할 수 있는 최신 표준 API이지만, `JSON.parse(JSON.stringify(object))`처럼 함수나 DOM 객체처럼 구조적으로 복제할 수 없는 값이 포함된 경우에는 사용할 수 없습니다.

```js
const object_1 = {
  value: 8,
  nested: {
    value: 42,
  },
};

const object_2 = structuredClone(object_1);

object_2.nested.value = 8;

console.log(object_1.nested.value); // 42
```

### 성능 비교

JSON 기반 방식, Lodash, `structuredClone`을 기준으로 실행 시간 관점에서 간단한 비교를 진행했습니다.<br>
테스트는 depth 3 & breadth 3의 중첩 객체를 만들어 진행했으며, 충분한 워밍업 이후 10개 라운드에 걸쳐 실행 시간을 측정했고 평균을 계산했습니다.

```js
import { performance } from "node:perf_hooks";
import lodash from "lodash";

const { cloneDeep } = lodash;

const OBJECT_DEPTH = 3;
const OBJECT_BREADTH = 3;
const WARMUP_ROUNDS = 2;
const ROUNDS = 10;
const ITERATIONS = 10000;

const createObject = (depth, breadth) => {
  if (depth === 0) return 42;

  const obj = {};
  for (let i = 0; i < breadth; i++) {
    obj[`key_${i}`] = createObject(depth - 1, breadth);
  }

  return obj;
};

const average = (arr) => arr.reduce((sum, v) => sum + v, 0) / arr.length;

const runBenchmark = (name, task) => {
  const durations = [];

  for (let r = 0; r < WARMUP_ROUNDS; r++) {
    for (let i = 0; i < ITERATIONS; i++) task();
  }

  for (let r = 0; r < ROUNDS; r++) {
    const start = performance.now();

    for (let i = 0; i < ITERATIONS; i++) task();

    const duration = performance.now() - start;
    durations.push(duration);
  }

  const avg = average(durations);

  console.log(name);
  console.log(`- average: ${avg.toFixed(2)}ms`);
  console.log(`- rounds : [${durations.map((d) => d.toFixed(2)).join(", ")}]`);
  console.log("");
};

const testObject = createObject(OBJECT_DEPTH, OBJECT_BREADTH);

runBenchmark("JSON.parse(JSON.stringify(object))", () => {
  JSON.parse(JSON.stringify(testObject));
});

runBenchmark("lodash.cloneDeep", () => {
  cloneDeep(testObject);
});

runBenchmark("structuredClone(object)", () => {
  structuredClone(testObject);
});
```

실행 결과 JSON 기반 방식과 Lodash는 비슷한 수준의 실행 시간을 보였고, 반면 `structuredClone`은 동일한 조건에서 상대적으로 더 긴 시간이 소요되었습니다.

```text
JSON.parse(JSON.stringify(object))
- average: 26.03ms
- rounds : [25.60, 25.57, 25.34, 25.52, 26.16, 26.12, 25.33, 26.96, 26.76, 26.89]

lodash.cloneDeep
- average: 29.76ms
- rounds : [29.18, 29.23, 29.05, 29.86, 30.59, 29.21, 29.48, 30.36, 30.44, 30.19]

structuredClone(object)
- average: 51.12ms
- rounds : [51.52, 51.44, 53.26, 51.90, 50.81, 49.85, 50.44, 49.77, 51.11, 51.06]
```

다만 실제 앱에서 다루는 객체는 훨씬 복잡하고 포함하는 데이터 타입도 다양합니다. 깊은 복사 방식마다 지원하는 데이터 타입에 제약이 있고, 외부 라이브러리에 대한 의존성이 필요한 경우도 있습니다.<br>
이러한 이유로 깊은 복사 방식은 시간 비교는 물론 다뤄야 할 데이터 타입과 필요한 안정성 수준을 기준으로 선택하는 것이 적절할 거라 생각합니다.

## Takeaways

1. 자바스크립트에서 복사 문제는 원시값이 아니라 객체의 참조 방식에서 발생합니다
2. 얕은 복사는 1 depth까지만 안전하며, 중첩된 참조를 포함하는 객체에서는 의도치 않은 변경을 유발할 수 있습니다
3. 깊은 복사에는 단일한 정답이 없으며, 데이터 타입과 실행 환경에 따라 적절한 방법을 선택해야 합니다
   <br><br>

_출처:<br>
[1] MDN Docs (
["JavaScript data types and data structures"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures),
["Shallow copy"](https://developer.mozilla.org/en-US/docs/Glossary/Shallow_copy),
["Deep copy"](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy)
)<br>_
