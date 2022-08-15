---
layout: page
title: 유용한 vscode 옵션
category : 개발환경
---

## group editor당 navigation하기

vscode 기본설정에서는 `go back`이나 `go foward`를 할때 editor간에 넘어가는 설정이 기본옵션이다. 이걸 `workbench.editor.navigationScope`옵션의 설정을 `editorGroup`으로 변경해주면 네비게이션이 그룹별로 나뉘게 되기에 편하게 사용할 수 있다.