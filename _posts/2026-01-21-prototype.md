---
title: "Prototype"
date: 2026-01-21
---

자바스크립트의 상속은 프로토타입 체인을 통해 이루어집니다. 이번 포스트에서 프로토타입의 관련 개념과 동작 원리, 그리고 프로토타입 체인이 실제로 어떻게 탐색되는지 살펴봤습니다.
<br><br>

## 프로토타입

자바스크립트는 프로토타입 기반 언어입니다.<br>
자바스크립트에서 프로토타입이란 객체들이 프로퍼티와 메서드를 공유하고, 상속 관계를 형성하기 위해 사용하는 객체를 말합니다.<br>

### 클래스 vs. 프로토타입

클래스 기반 언어에서는 클래스를 먼저 정의하고, 해당 클래스를 기반으로 인스턴스를 생성합니다. 상속 역시 클래스 간의 관계를 통해 이루어집니다.<br>
반면 프로토타입 기반 언어에서는 객체가 다른 객체를 직접 참조하며 상속 관계를 형성합니다. 자바스크립트에서는 생성자 함수의 프로토타입 객체를 통해 인스턴스들이 공통된 프로퍼티와 메서드를 공유합니다.<br>
자바스크립트는 ES6에서 `class` 문법을 도입했지만, 이는 새로운 상속 모델을 추가한 것은 아닙니다. 내부 동작은 여전히 기존의 프로토타입 기반 상속과 프로토타입 체인을 따릅니다.

### 왜 프로토타입 상속이 필요한가

아래와 같은 코드를 가정해보겠습니다.

```js
function Person(name) {
  this.name = name;

  this.sayHello = function () {
    console.log("Hello!");
  };

  this.sayBye = function () {
    console.log("Bye!");
  };
}

const yj = new Person("YJ");
const jy = new Person("JY");

console.log(yj.sayHello === jy.sayHello); // false
```

이렇게 구현하면 `Person` 생성자 함수로 만들어진 인스턴스는 `name` 속성과 `sayHello`, `sayBye` 메서드를 각각 개별적으로 가지게 됩니다.<br>
`name`처럼 인스턴스마다 다른 값을 가져야 하는 속성은 문제가 되지 않습니다.
하지만 모든 인스턴스에서 동일한 동작을 수행하는 메서드들이 인스턴스마다 새로 생성된다면 메모리에 중복으로 저장되는 문제가 발생합니다.<br>
<br>
프로토타입 객체는 인스턴스들을 상위에서 관리하며 공통 메서드를 정의한 뒤 인스턴스들이 이를 공유해서 사용할 수 있게 해줍니다.<br>
아래와 같이 생성자 함수의 프로토타입에 `sayHello`와 `sayBye`를 정의하면, 해당 생성자로 만들어진 모든 인스턴스가 이 메서드를 공유할 수 있습니다.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log("Hello!");
};

Person.prototype.sayBye = function () {
  console.log("Bye!");
};

const yj = new Person("YJ");
const jy = new Person("JY");

console.log(yj.sayHello === jy.sayHello); // true
```

이제 메서드들은 한 번만 생성되고, 모든 인스턴스가 프로토타입 체인을 통해 동일한 메서드를 참조합니다.

## 메커니즘

자바스크립트는 `prototype`, `[[Prototype]]`, `__proto__`라는 서로 다른 메커니즘을 통해 객체 간의 연결을 관리합니다.

### `prototype`

인스턴스의 부모로 사용될 객체를 가리키는 프로퍼티입니다. `prototype` 객체는 생성자 함수가 정의되는 시점에 자바스크립트 엔진에 의해 자동으로 생성됩니다.<br>
따라서 아래 코드에서 `prototype`은 존재합니다.

```js
function Person(name) {
  this.name = name;
}

console.log(Person.prototype); // {}
```

### `[[Prototype]]`

객체가 자신의 부모 객체를 가리키는 내부 링크입니다. 자바스크립트 명세에만 등장하는 내부 슬롯이며, 포인터와 같은 역할을 합니다.<br>
프로토타입 체인은 자식에서 부모 방향으로만 탐색되어야 하기 때문에 직접 조작할 수 없도록 숨겨져 있습니다.

### `__proto__`

`[[Prototype]]`을 간접적으로 확인하기 위한 접근자입니다. 구체적으로 `[[Prototype]]` 자체에 접근하는 것이 아니라, 그 링크가 가리키는 객체를 반환하는 API입니다. 이를 통해 자신의 프로토타입에 접근할 수 있습니다.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log("Hello!");
};

Person.prototype.sayBye = function () {
  console.log("Bye!");
};

const aiden = new Person("Aiden");

console.log(aiden.__proto__ === Person.prototype); // true
```

## 프로토타입 체인

객체에서 특정 프로퍼티나 메서드를 찾을 때, 자신의 객체 -> 부모 객체 -> 그 부모의 부모 객체로 연결된 프로토타입을 따라 순차적으로 탐색하는 구조를 말합니다.<br>
자바스크립트의 객체는 단독으로 존재하지 않고, 항상 다른 객체를 가리키는 `[[Prototype]]` 링크를 가집니다. 이 링크가 연속적으로 이어져 있는 구조이기 때문에
이를 프로토타입 체인이라고 부릅니다.<br>
아래 코드를 바탕으로 프로토타입 체인이 어떻게 동작하는지 살펴보겠습니다.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log("Hello!");
};

Person.prototype.sayBye = function () {
  console.log("Bye!");
};

const aiden = new Person("Aiden");
```

### 구조

여기서 `new Person("Aiden")`이 실행되면, `aiden` 객체는 다음과 같은 구조를 갖게 됩니다.

```txt
aiden
└─ [[Prototype]] -> Person.prototype
                    ├─ sayHello()
                    ├─ sayBye()
                    └─ [[Prototype]] -> Object.prototype
                                        └─ [[Prototype]] -> null
```

- `aiden`은 `name` 프로퍼티를 직접 소유합니다
- `sayHello`, `sayBye`는 `Person.prototype`에 정의되어 있습니다
- `Person.prototype`의 상위 프로토타입은 `Object.prototype`입니다

### 탐색 가능

`aiden.sayHello();`을 호출하면 자바스크립트 엔진은 다음 순서로 탐색합니다.

1. `aiden` 객체에 `sayHello`가 있는지 확인합니다
2. 없으면 `aiden.[[Prototype]]`이 가리키는 `Person.prototype`으로 이동합니다
3. `Person.prototype`에서 `sayHello`를 발견하고 실행합니다

### 탐색 불가

반면 `aiden.sayYes();`와 같이 존재하지 않는 메서드를 호출하면, 자바스크립트 엔진은 프로토타입 체인을 따라 끝까지 탐색합니다.

1. `aiden` 객체에 `sayYes`가 있는지 확인합니다
2. `Person.prototype`에서 `sayYes`를 찾습니다
3. `Object.prototype`에서도 찾지 못합니다
4. `Object.prototype`의 상위 프로토타입인 `null`에 도달하며 탐색이 종료됩니다

이 경우 `aiden.sayYes`의 값은 `undefined`가 되며, 이를 함수로 호출하려고 하면 `TypeError`가 발생합니다.

### 예시

```js
function Person(name) {
  this.name = name;
}

const aiden = new Person("Aiden");

console.log(aiden.__proto__.__proto__ === Object.prototype); // ?
```

```js
const object = { value: 42 };

console.log(object.__proto__ === Object.prototype); // ?
```

```js
const string = "Aiden";

console.log(string.__proto__ === Object.prototype); // ?
```

### 오토 박싱

원시값에 대해 프로퍼티에 접근하려고 할 때, 자바스크립트 엔진이 임시로 객체를 생성해주는 메커니즘입니다. 이를 통해 원시값이 객체가 아님에도 불구하고 메서드를 사용할 수 있습니다.

```js
console.log("Aiden".toUpperCase()); // AIDEN
console.log((123).toFixed(2)); // 123.00
console.log(true.toString()); // true
```

메서드 접근이 필요할 때 잠깐 객체처럼 만들어주기 때문에 아래와 같은 체인이 성립합니다.

```js
console.log("Aiden".__proto__ === String.prototype); // true
console.log((123).__proto__ === Number.prototype); // true
console.log(true.__proto__ === Boolean.prototype); // true
```

따라서 예시 코드의 실행 결과는 다음과 같습니다.

```js
const string = "Aiden";

console.log(string.__proto__ === Object.prototype); // false
console.log(string.__proto__ === String.prototype); // true
console.log(string.__proto__.__proto__ === Object.prototype); // true
```

## 왜 우리는 클래스를 사용하는가

프로토타입 기반 언어인 자바스크립트에서 클래스를 사용하는 이유는, 상속 구조를 더 안전하고 명확하게 표현할 수 있기 때문입니다.<br>
프로토타입 구현은 코드만 보고 상속 구조를 한눈에 파악하기 어렵고, 의도치 않게 프로토타입 체인을 변경해 전체 객체에 영향을 줄 수 있습니다.<br>
`class` 문법은 이러한 구조적 실수를 방지해 줍니다.

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    console.log("Hello!");
  }

  sayBye() {
    console.log("Bye!");
  }
}
```

위와 같이 작성하면 상속 구조가 문법적으로 명확하게 드러나고, 메서드들은 자동으로 `Person.prototype`에 추가되어 인스턴스마다 새로 생성되지 않습니다.<br>
<br>
라이브러리나 프레임워크를 구현하는 경우처럼 특수한 상황을 제외하면, 일반적인 애플리케이션 코드에서 프로토타입을 직접 조작할 일은 거의 없습니다.<br>
하지만 `class`를 사용하면서도 내부에서 어떤 일이 일어나는지는 알아야 할 것입니다!

## Takeaways

1. 프로토타입은 객체들이 프로퍼티와 메서드를 공유하기 위해 참조하는 상위 객체입니다
2. 자바스크립트에서 상속은 프로토타입을 통한 참조로 이루어집니다 (클래스 기반 언어에서 메서드는 클래스 정의에 속합니다)
3. 프로토타입 체인은 객체가 프로퍼티를 찾기 위해 `[[Prototype]]` 링크를 따라 상위 객체로 탐색해 나가는 구조입니다
   <br><br>

_출처:<br>
[1] MDN Docs (["Primitive"](https://developer.mozilla.org/en-US/docs/Glossary/Primitive))<br>_
