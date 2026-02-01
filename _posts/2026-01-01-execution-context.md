---
title: "Execution Context"
date: 2026-01-01
---

자바스크립트 실행 컨텍스트는 코드가 실행되는 환경을 이해하기 위한 중요한 개념입니다.<br>
이번 포스트에서는 실행 컨텍스트가 무엇인지, 어떤 종류가 있는지, 그리고 자바스크립트 엔진이 코드를 어떤 단계와 순서로 실행하는지 살펴보겠습니다.
<br><br>

## 실행 컨텍스트

코드를 해석하고 실행하는 주체는 자바스크립트 엔진입니다.<br>
자바스크립트 엔진은 코드를 만나면 바로 실행하지 않고, 먼저 코드를 실행하기 위한 환경인 실행 컨텍스트를 만든 뒤 그 컨텍스트 안에서 코드를 실행합니다.<br>
다시 말해서, 실행 컨텍스트는 자바스크립트 엔진이 코드를 실행하기 위해 만드는 실행 단위이자 환경이라고 정의할 수 있습니다.

## 실행 컨텍스트의 종류

자바스크립트에는 세 가지 유형의 실행 컨텍스트가 있습니다.<br>

### Global Execution Context

전역 실행 컨텍스트는 자바스크립트 파일이 실행되면 가장 먼저 생성되며, 프로그램 전체에서 단 하나만 존재합니다.<br>
어떤 함수에도 포함되지 않는 코드는 전역 실행 컨텍스트에 포함됩니다.

### Functional Execution Context

함수가 호출될 때마다, 해당 함수를 위한 새로운 실행 컨텍스트가 생성됩니다.<br>
따라서 함수 실행 컨텍스트는 함수 호출 횟수만큼 여러 개가 존재할 수 있습니다.

### Eval Execution Context

`eval()` 함수로 실행된 코드도 자체 실행 컨텍스트를 가집니다. 일반적으로 보안과 성능 문제로 실무에서 사용하지 않는다고 합니다.

## 실행 컨텍스트의 실행 흐름

실행 컨텍스트는 스택 구조로 관리되며, 이를 실행 스택 또는 콜 스택이라고 부릅니다.

```js
function foo() {
  var x = 10;
  bar();
}

function bar() {
  var y = 10;
}

foo();
bar();
```

항상 실행 스택의 맨 위에 있는 실행 컨텍스트만 실행되며, 위 코드가 실행될 때의 흐름은 다음과 같습니다.

1. 전역 실행 컨텍스트가 생성되어 `stack`에 `push`됩니다
2. `foo()` 호출 -> `foo` 실행 컨텍스트가 생성되어 `stack`에 `push`됩니다
3. `bar()` 호출 -> `bar` 실행 컨텍스트가 생성되어 `stack`에 `push`됩니다
4. `bar` 실행 컨텍스트 종료 -> `stack`에서 `pop`됩니다
5. `foo` 실행 컨텍스트 종료 -> `stack`에서` pop`됩니다
6. `bar()` 호출 -> 새로운 `bar` 실행 컨텍스트가 생성되어 `stack`에 `push`됩니다
7. `bar` 실행 컨텍스트 종료 후 전역 실행 컨텍스트도 종료됩니다

## 실행 컨텍스트의 생성 과정

실행 컨텍스트는 생성 단계와 실행 단계의 두 단계에 걸쳐 처리됩니다.

### 실행 컨텍스트의 구조

ECMAScript는 자바스크립트의 공식 언어 사양으로 자바스크립트가 어떻게 동작해야 하는지를 정의한 표준입니다.
ECMAScript 스펙 기준으로 실행 컨텍스트는 다음과 같은 구조를 가지며, 이는 실제 메모리 구조가 아닌 동작을 설명하기 위한 추상적인 모델입니다.

```text
Execution Context
 ├─ LexicalEnvironment
 │   ├─ Environment Record
 │   └─ Outer Lexical Environment Reference
 ├─ VariableEnvironment
 └─ ThisBinding
```

- `LexicalEnvironment`: 스코프와 식별자 바인딩을 관리합니다
- `VariableEnvironment`: `var` 선언을 관리합니다
- `ThisBinding`: `this` 값을 관리합니다

### 생성 단계

생성 단계에서는 코드를 실행하기 위해 필요한 구조와 정보가 준비됩니다.<br>
이 단계에서는 실행 컨텍스트에 `LexicalEnvironment`와 `VariableEnvironment`에 대한 참조가 설정되며, 이에 대응하는 환경 레코드가 생성되고, this 값이 결정됩니다.

### Lexical Environment

렉시컬 환경은 식별자와 실제 변수 또는 함수를 코드의 렉시컬 구조를 기준으로 연결해주는 개념입니다.

```js
function foo() {
  let x = 1;

  function bar() {
    console.log(x);
  }
}
```

이 구조에서 중요한 것은, `bar` 함수가 어디에서 호출되는지가 아니라 어디에 정의되어 있는지입니다. 스코프가 코드가 작성된 위치를 기준으로 결정되기 때문입니다.

### Environment Record & Outer Lexical Environment Reference

렉시컬 환경은 내부적으로 환경 레코드를 가집니다. 환경 레코드는 현재 스코프에 선언된 식별자들의 매핑 테이블입니다.<br>
위 예시 코드에서 `foo` 함수의 환경 레코드는 다음과 같습니다.

```text
foo Environment Record
 ├─ x   -> 1
 └─ bar -> function bar
```

`foo`의 렉시컬 환경은 전역 렉시컬 환경을 외부 렉시컬 환경 참조로 가집니다.<br>

### Temporal Dead Zone (TDZ)

`let`과 `const`로 선언된 변수는 실행 컨텍스트 생성 단계에서 렉시컬 환경의 환경 레코드에 등록되지만, 초기화되기 전까지는 `uninitialized` 상태로 유지됩니다.<br>
이처럼 식별자는 존재하지만 값이 할당되지 않아 접근할 수 없는 구간을 Temporal Dead Zone이라고 부릅니다.

```js
console.log(x); // ReferenceError
let x = 10;
```

```text
Global Lexical Environment
 ├─ Environment Record
 │   └─ x -> <uninitialized>
 └─ Outer Lexical Environment Reference -> null
```

한편 `bar` 함수 내부에서 `x`를 참조할 때의 탐색 순서는 다음과 같습니다.

1. `bar`의 환경 레코드에서 `x`를 찾습니다
2. 없으면 외부 렉시컬 환경 참조를 따라 이동합니다
3. `foo`의 환경 레코드에서 `x` 발견합니다
4. 해당 값을 사용합니다

### Variable Environment

변수 환경은 구조적으로 렉시컬 환경와 동일하지만, `var` 선언을 관리하기 위해 분리된 환경입니다.

```js
function foo() {
  console.log(x);
  var x = 10;
}
```

- 생성 단계에서 `x`는 변수 환경에 등록됩니다
- 초기값은 `undefined`입니다

### this 바인딩

`this`는 실행 컨텍스트의 ThisBinding 슬롯에 저장되며, 이 값은 실행 컨텍스트가 생성될 때 결정됩니다.<br>
`this` 값은 변수처럼 선언되는 것이 아니라 샐행 컨텍스트가 생성되는 시점에 결정되며 함수가 어떻게 호출되었는지에 따라 달라집니다.

```js
function foo() {
  console.log(this);
}

foo();
```

위 코드에서 `foo`는 일반 함수로 호출됐기 때문에 기본적으로 전역 객체(`window`), strict mode에서는 `undefined`가 `this`로 바인딩됩니다.

```js
const obj = {
  value: 10,
  foo() {
    console.log(this.value);
  },
};

obj.foo();
```

이 경우 `foo`는 객체의 메서드로 호출됐기 때문에 `this`는 해당 객체인 `obj`를 가리킵니다.

### 실행 단계

실행 단계에서는 코드가 위에서 아래로 실제로 실행됩니다. 구체적으로 변수에 값을 할당하고, 함수를 호출하며, 표현식을 평가합니다.<br>
위 예시 코드에서 `x`는 `undefined`로 출력되는데, 생성 단계에서 이미 `x`가 환경 레코드에 등록됐기 때문입니다. 이 현상을 호이스팅이라고 합니다.<br>
실행 단계 동안에도 함수가 호출되면 실행 컨텍스트에 `push`되고, 함수가 종료되면 `pop`되며 변화합니다. 이 구조로 인해 자바스크립트는 단일 스레드를 가집니다.

## Takeaways

1. 실행 컨텍스트는 자바스크립트 엔진이 코드를 실행하기 위해 만드는 실행 단위이자 환경입니다
2. 실행 컨텍스트는 생성 단계와 실행 단계로 처리되며, 생성 단계에서 스코프 구조와 식별자 바인딩이 결정됩니다
3. 렉시컬 환경과 환경 레코드는 스코프의 핵심이며, 외부 렉시컬 환경 참조를 통해 스코프 체인이 형성됩니다
4. 변수 환경은 `var` 선언을 관리하기 위해 존재하며, 이로 인해 호이스팅과 `undefined` 초기화가 발생합니다
5. 실행 컨텍스트는 스택 구조로 관리되며 항상 맨 위에 있는 컨텍스트만 실행됩니다
   <br><br>

_출처:<br>
[1] [Understanding Execution Context and Execution Stack in Javascript](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)<br>_
