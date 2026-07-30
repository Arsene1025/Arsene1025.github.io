---
title: "유니티 생존 게임 리팩토링 - 1"
date: 2026-07-01 09:00:00 +0900
categories: [PortfolioProjects, UnitySurvival]
tags: [Csharp, Unity]
---
# 2026/07/30 

## 리팩토링 진행 이유
- 기존 프로젝트는 포트폴리오로 제출하기에 조금 아쉬움이 남는 프로젝트라고 생각.
- 리팩토링을 통해 시작과 끝이 있는 프로젝트로 완성하는 것이 목표.


## 리팰토링 예정 목록

![작업 결과 화면](/assets/img/unitysurvive/SurvivalStruct.png)

- 시작화면 만들기, 엔딩 만들기

- 인벤토리 개선 : 구조 변경, 무게 제한 활성화, 아이템 드래그 앤 드랍 추가.

- 아이템 개선 : 정보 관리와 이름 한글로 통일.

- Monster가 플레이어 추적, 공격 하는 것을 Update가 아니라 행동트리로 구현하기. 애니메이션 추가. 다양한 바리에이션 추가

- 플레이어, 몬스터, 일꾼의 구조를 변경 -> 기능이 Player_FindObject, Player_Movement 같은 스크립트로 분할되어 있어 코드를 파악하기 어려움.

- Manager클래스 가 너무 분할되어 있어서 코드 검토 후 통합.