---
title: "Hard-linked pnpm"
date: 2025-11-12
---

pnpm이 하드/심볼릭 링크를 사용한다고 알려져 있고, 아마 `pnpm add axios`처럼 커맨드가 자동으로 하드 링크가 적용된다고 생각하는 분들도 있을 것 같습니다.<br>
하지만 지난 포스트에서 살펴봤듯이, 기본값인 `--package-import-method=auto`가 항상 하드 링크를 선택하는 것은 아닙니다.
파일 시스템 환경에 따라 `clone`이나 `copy` 방식으로 진행될 수 있고, 제 맥북에서는 실제로 `clone` 방식으로 동작하는 것을 확인했었습니다.<br>
그래서 이번에는 `--package-import-method=hardlink` 옵션을 직접 지정해 하드 링크 기반의 pnpm을 확인해봤습니다.
<br><br>

## 하드 링크 옵션으로 axios 설치

테스트 프로젝트에서 `--package-import-method=hardlink` 옵션을 명시적으로 지정해 `axios`를 설치했습니다.<br>
그리고 `ls -i` 커맨드로 아이노드 번호를 확인했습니다.

```bash
(base) aidenseo@Aidens-MacBook-Pro javascript-practice % pnpm add axios --package-import-method=hardlink
Packages: +23
+++++++++++++++++++++++
Progress: resolved 23, reused 23, downloaded 0, added 23, done

dependencies:
+ axios 1.13.2

Done in 661ms using pnpm v10.17.1
(base) aidenseo@Aidens-MacBook-Pro javascript-practice % ls -i node_modules/.pnpm/axios@1.13.2/node_modules/axios/lib/axios.js

96360157 node_modules/.pnpm/axios@1.13.2/node_modules/axios/lib/axios.js
(base) aidenseo@Aidens-MacBook-Pro javascript-practice %
```

아이노드 번호는 `96360157`입니다.<br>
하드 링크가 제대로 적용되었다면, 전역 스토어에도 같은 아이노드 번호를 가진 파일이 존재해야 합니다.<br>

## 전역 스토어에서 동일 아이노드 탐색

pnpm의 스토어 구조는 해시 기반이라, 단순히 패키지 이름으로 경로를 찾기 어렵습니다.<br>
그래서 아이노드 번호를 이용해 직접 검색해봤습니다.

```bash
(base) aidenseo@Aidens-MacBook-Pro ~ % cd Library/pnpm/store/v10/files
(base) aidenseo@Aidens-MacBook-Pro files % find . -inum 96360157
./af/a9b07c114c88290e5c1bc2dc87ca64130cb25c66495967cec3e0a22166d2dcce16ac8c44c092cb5cc5111d72c1bf7417d6060bf7fd51f455769b11cf51ea7d
(base) aidenseo@Aidens-MacBook-Pro files %
```

검색 결과, 전역 스토어의 해시 경로(`./af/...`) 아래에 동일한 아이노드를 가진 파일이 있었습니다.<br>

## 비교

이제 `ls -i`로 해당 스토어 파일을 직접 비교해보겠습니다.

```bash
(base) aidenseo@Aidens-MacBook-Pro files % ls -i ./af/a9b07c114c88290e5c1bc2dc87ca64130cb25c66495967cec3e0a22166d2dcce16ac8c44c092cb5cc5111d72c1bf7417d6060bf7fd51f455769b11cf51ea7d

96360157 ./af/a9b07c114c88290e5c1bc2dc87ca64130cb25c66495967cec3e0a22166d2dcce16ac8c44c092cb5cc5111d72c1bf7417d6060bf7fd51f455769b11cf51ea7d
(base) aidenseo@Aidens-MacBook-Pro files %
```

링크 수가 `3`으로 같고, 결정적으로 파일의 정체성에 해당하는 아이노드 번호가 `96360157`로 같습니다.<br>

## 결론

테스트 프로젝트에서 `--package-import-method=hardlink` 옵션으로 설치한 패키지의 파일이 전역 스토어의 파일과 동일한 아이노드 번호를 공유하는 것을 확인했습니다.<br>
즉, 두 파일은 물리적으로 같은 데이터를 가리키며, pnpm이 전역 스토어의 파일을 복사 없이 하드 링크로 연결해 효율적으로 재사용하고 있음을 직접 검증할 수 있었습니다.
