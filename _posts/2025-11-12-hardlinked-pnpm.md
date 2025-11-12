---
title: "Hardlinked pnpm"
date: 2025-11-12
---
pnpm이 하드/심볼릭 링크를 사용한다고 알려져 있고, 아마 `pnpm add axios`처럼 단순한 커맨드로 자동으로 하드 링크가 적용된다고 생각하는 분들도 있을 것 같습니다.<br>
하지만 지난 포스트에서 살펴봤듯이, 기본값인 `--package-import-method=auto`가 항상 하드 링크를 선택하는 것은 아닙니다.
파일 시스템 환경에 따라 `clone`이나 `copy` 방식으로 진행될 수 있고, 제 맥북에서는 실제로 `clone` 방식으로 동작하는 것을 확인했었습니다.<br>
그래서 이번에는 `--package-import-method=hardlink` 옵션을 직접 지정해 하드 링크 기반의 pnpm을 확인해봤습니다.
<br><br>
