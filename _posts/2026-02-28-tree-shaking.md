---
title: "Tree Shaking"
date: 2026-02-28
---

이전 포스트에서는 훅의 등장 배경과 클래스 기반 구조의 한계를 다루면서 트리 셰이킹을 간단히 언급했습니다.
이번에는 그 개념을 확장해 트리 셰이킹의 개념과 작동 원리를 살펴보았습니다.
<br><br>

<img src='/images/tree-shaking/tree-shaking.jpg' width="800">

트리 셰이킹은 사용되지 않는 `export`를 빌드 단계에서 제거하는 최적화 기법입니다.<br>
애플리케이션의 모듈 의존 관계를 정적으로 분석해, 실제로 참조되는 코드만 번들에 포함합니다.

## 왜 필요한가

### 애플리케이션의 모듈 의존성

현대 프런트엔드 애플리케이션은 다양한 외부 라이브러리와 모듈을 조합해 구성됩니다.
UI 컴포넌트 라이브러리나 상태 관리 도구처럼, 이미 잘 만들어진 외부 패키지를 조합해 애플리케이션을 구성하는 것이 일반적입니다.<br>
예를 들어 [Material UI](https://mui.com/)를 사용하는 경우 아래와 같이 패키지를 가져올 수 있습니다.

```js
import * from "@mui/material";
```

그런데 UI 라이브러리는 수많은 컴포넌트와 스타일 시스템, 테마 로직 등을 포함합니다.
일부 컴포넌트만 사용하더라도 관련 구현 코드와 내부 의존 모듈이 함께 포함되면서 번들 크기가 예상보다 커질 수 있습니다.<br>
그래서 보통은 아래와 같이 필요한 것만 명시적으로 `import`합니다.

```js
import {
  AppBar,
  Toolbar,
  Typography,
  Button,
  Box,
  InputBase,
  FormControl,
} from "@mui/material";
import {
  styled,
  alpha,
  createTheme,
  ThemeProvider,
} from "@mui/material/styles";
import SearchIcon from "@mui/icons-material/Search";
import StoreIcon from "@mui/icons-material/Store";
```

그러나 이렇게 필요한 항목만 명시하더라도, 실제로 참조된 컴포넌트의 구현과 그 내부 의존성은 번들에 포함됩니다.

### 네트워크 비용과 성능 문제

브라우저는 자바스크립트 파일을 다운로드한 뒤 파싱, 컴파일 후 실행합니다.<br>
번들 크기가 커질수록 네트워크 전송 시간은 물론 파싱, 컴파일 비용과 메인 스레드 점유 시간도 증가합니다.<br>
이처럼 사용되지 않는 코드도 브라우저 입장에서는 처리해야 할 비용입니다. 따라서 불필요한 코드를 정리하는 건 초기 로딩 성능과 사용자 경험을 개선하기 위한 중요한 전략이라고 할 수 있습니다.

## 작동 원리

트리 셰이킹은 전통적인 컴파일러 최적화 기법인 Dead Code Elimination에서 확장된 개념입니다.

### Dead Code Elimination

DCE는 프로그램 실행 결과에 영향을 주지 않는 코드를 제거하는 기법입니다.<br>
제어 흐름을 분석해 도달할 수 없는 코드나 사용되지 않는 변수를 제거하는데, 대표적인 제거 대상은 도달할 수 없는 코드, 호출되지 않는 함수, 사용되지 않는 변수, 계산되지만 결과가 사용되지 않는 표현식이 있습니다.

```c
int foo(void) {
  int a = 24;
  int b = 25;
  int c;

  c = a * 4;
  return c;

  b = 24;
  return 0;
}
```

- `return c;` 이후의 코드는 흐름상 도달할 수 없습니다
- 그래서 `b = 24;`와 `return 0;`은 실행 경로에 포함되지 않는 코드입니다
- 이 코드는 프로그램의 최종 결과에 영향을 주지 않기 때문에 컴파일 단계에서 제거됩니다

### 트리 셰이킹

트리 셰이킹은 이 개념을 모듈 시스템 수준으로 확장한 최적화입니다.

- 정적 분석: 코드를 실제로 실행하지 않고 문법 구조와 참조 관계를 분석해 의존성, 타입, 사용 여부 등을 판단하는 방식을 말합니다
- ECMAScript 모듈: ESM은 `import`, `export`가 정적으로 선언되어 있어 빌드 시점에 사용 여부를 판단할 수 있습니다

여기서 중요한 점은 제거 단위가 파일 전체가 아니라 `export` 단위라는 것입니다. 하나의 모듈 안에서도 사용되지 않은 `export`만 선택적으로 제거될 수 있습니다.

```js
// math.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

```js
// main.js
import { add } from "./math.js";

console.log(add(1, 2));
```

위 경우는 `subtract`는 번들에서 제거되고, `math.js` 파일은 `add`가 사용되므로 유지됩니다.<br>
<br>
트리 셰이킹은 다음과 같은 단계로 이루어집니다.

- 애플리케이션의 진입점에서 분석을 시작합니다
- 모든 `import`를 추적하여 모듈 의존성 그래프를 생성합니다
- 사용된 `export`를 식별합니다
- 번들링 및 압축 과정에서 해당 코드를 최종적으로 제거합니다

DCE가 제어 흐름 기반 제거라면, 트리 셰이킹은 모듈 의존성 그래프 기반 제거라고 할 수 있습니다.

```js
import {
  AppBar,
  Toolbar,
  Typography,
  Button,
  Box,
  InputBase,
  FormControl,
} from "@mui/material";
```

위 코드에서 실제로 `AppBar`, `Toolbar`, `Typography`만 사용하고 `Button`, `Box`, `InputBase`, `FormControl`을 사용하지 않는 경우를 가정해보겠습니다.<br>
트리 셰이킹이 제대로 동작한다면 사용된 `export`인 `AppBar`, `Toolbar`, `Typography`만 최종 번들에 포함되고, 나머지는 포함되지 않습니다.

## 작동 주체

트리 셰이킹은 번들러가 수행하는 최적화입니다. 대표적인 번들러 및 빌드 도구는 다음과 같습니다.

- Webpack:
  - 범용 번들러입니다
  - 프로덕션 모드일 때 `usedExports` 분석과 `minification` 과정을 통해 트리 셰이킹을 수행합니다
- Rollup:
  - 설계 자체가 ESM 중심이라 정적 분석에 최적화되어 있습니다
  - `export` 단위 제거가 자연스러워 라이브러리 번들링에 많이 사용됩니다
- Vite:
  - 개발과 빌드 파이프라인을 분리한 빌드 도구입니다
  - 개발 모드에서는 `esbuild` 기반 변환과 의존성 최적화를 수행하며, 최종 번들 수준의 트리 셰이킹 결과는 프로덕션 빌드에서 확정됩니다
  - 프로덕션 빌드에서는 내부적으로 Rollup을 사용해 트리 셰이킹을 적용합니다
- `esbuild`:
  - 빠른 빌드를 목표로 합니다
  - ESM 기반 정적 분석을 통해 트리 셰이킹을 지원하며, 단독 번들러로도 사용되고 다른 도구의 내부 엔진으로도 활용됩니다

## 작동 시점

트리 셰이킹은 런타임이 아닌 빌드 타임에 수행됩니다. 일반적으로 프로덕션 빌드 단계에서 정적 분석과 코드 압축 과정과 함께 적용됩니다.<br>
일반적인 개발 서버에서는 빠른 빌드 속도와 디버깅 편의를 우선하기 때문에 최적화가 비활성화되는 경우가 많습니다. 그래서 트리 셰이킹이 완전히 적용되지 않아 최적화가 완전히 적용되지 않을 수 있습니다.

## Takeaways

1. 트리 셰이킹은 번들러가 빌드 타임에 사용되지 않는 코드를 제거하는 최적화 기법입니다
2. 트리 셰이킹은 ESM의 정적 구조 덕분에 가능하며 모듈 의존성 그래프를 기반으로 동작합니다
3. 트리 셰이킹 제거 단위는 모듈 내 `export` 단위이며, 실제로 참조된 코드만 번들에 포함됩니다
   <br><br>

_출처:<br>
[1] Wikipedia (["Tree shaking"](https://en.wikipedia.org/wiki/Tree_shaking), ["Dead-code elimination"](https://en.wikipedia.org/wiki/Dead-code_elimination))<br>_
