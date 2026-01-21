---
title: "Prototype"
date: 2026-01-21
---
<br><br>

## 프로토타입
자바스크립트는 프로토타입 기반 언어입니다.<br>
자바스크립트에서 프로토타입이란 객체들이 속성과 메서드를 공유하고, 상속 관계를 형성하기 위해 사용하는 객체를 말합니다.<br>

### 클래스 vs. 프로토타입
클래스 기반 언어에서는 클래스를 먼저 정의하고, 해당 클래스를 기반으로 인스턴스를 생성합니다. 상속 역시 클래스 간의 관계를 통해 이루어집니다.<br>
반면 프로토타입 기반 언어에서는 객체가 다른 객체를 직접 참조하며 상속 관계를 형성합니다. 자바스크립트에서는 생성자 함수의 `prototype` 객체를 통해 인스턴스들이 공통된 속성과 메서드를 공유합니다.<br>
자바스크립트는 ES6에서 `class` 문법을 도입했지만, 이는 새로운 상속 모델을 추가한 것이 아닙니다.
내부 동작은 여전히 기존의 프로토타입 기반 상속과 프로토타입 체인을 따릅니다.

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

생성자 함수의 `prototype`에 `sayHello`와 `sayBye`를 정의하면, 해당 생성자로 만들어진 모든 인스턴스가 이 메서드를 공유할 수 있습니다.
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
