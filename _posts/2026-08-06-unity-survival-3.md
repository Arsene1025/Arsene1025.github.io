---
title: "유니티 생존 게임 리팩토링 - 2"
date: 2026-08-06 09:00:00 +0900
categories: [PortfolioProjects, UnitySurvival]
tags: [Csharp, Unity]
---
# Survival Refactoring 변경 내역
- 기준일: 2026-08-06
- Unity: 6000.3.14f1

## 1. UI 상태 관리자

기존 `Queue<UI_Base>` 방식은 실제로 닫는 UI와 큐에서 제거되는 UI가 달라질 수 있었음. 이를 `UIStateManager`의 `HashSet<UI_Base>` 관리 방식으로 변경.

- 같은 UI의 중복 등록 방지
- 닫는 순서와 관계없이 정확한 UI 제거
- `UI_Base.OnDisable`을 통한 예외 경로 정리
- `Canvas_Handler.HasOpenUI`로 플레이어 입력 차단 상태 제공
- `OpenUI`, `CloseUI`, `CloseAllUI`를 중앙 관리자 기준으로 처리

주요 변경 파일:

- `Assets/00.Scripts/UI/UIStateManager.cs`
- `Assets/00.Scripts/UI/UI_Part/UI_Base.cs`
- `Assets/00.Scripts/UI/Canvas_Handler.cs`

## 2. 이벤트와 Unity 생명주기

초기화는 `Awake`, 전역 이벤트 연결은 `OnEnable`, 해제는 `OnDisable`로 통일.

- `Character`, `Player_Movement`, `Worker` 초기화 순서 통일
- `Interaction_Hit.Start`가 `M_Object.Start`를 가리던 문제 제거
- Player, 상호작용 탐지기, Canvas, Inventory 이벤트 구독과 해제를 대칭화
- 해제가 어려운 익명 람다 이벤트를 명명 메서드로 교체
- 싱글턴 중복 인스턴스 방어 및 파괴 시 정적 참조 초기화
- `Particle_Handler` 캐시를 `Awake`에서 준비

## 3. Inventory와 Recipe 통합

건물과 Worker 제작 데이터가 `IRecipe`를 구현하도록 통합.

```text
IRecipe
 ├─ RecipeKey
 ├─ Duration
 └─ Costs
```

`ItemDrop_Manager`의 책임은 다음 서비스로 분리.

- `InventoryService`: 아이템 추가, 보유량, 무게, 비용 검사 및 원자적 차감
- `RecipeService`: Recipe 제작 가능 여부 및 비용 소비
- `LootService`: 드롭 테이블 확률과 수량 계산

기능상 중요한 변경:

- 건물 또는 Worker 생성을 확정하면 재료가 실제로 차감.
- 같은 ItemID가 비용 목록에 여러 번 존재해도 합산한 뒤 검사.
- 모든 재료가 충분한 경우에만 한 번에 차감.
- 인벤토리 아이템이 제거되면 빈 슬롯도 즉시 정리.
- 아이템 종류가 기존 50슬롯을 넘으면 슬롯을 동적으로 추가.

주요 변경 파일:

- `Assets/00.Scripts/Crafting/IRecipe.cs`
- `Assets/00.Scripts/Manager/InventoryService.cs`
- `Assets/00.Scripts/Manager/LootService.cs`
- `Assets/00.Scripts/UI/UI_Part/UI_Inventory.cs`
- `Assets/00.Scripts/UI/UI_Part/UI_Building.cs`
- `Assets/00.Scripts/UI/UI_Part/UI_Portal.cs`

## 4. InteractionDetector

Player와 Worker에 중복되어 있던 다음 기능을 `InteractionDetector`로 통합.

- 주변 Collider 탐색
- 가장 가까운 대상 선택
- 활성 거리 검사
- LayerMask 필터
- 자식 Collider에서 실제 `M_Object` 탐색

`Physics.OverlapSphereNonAlloc`과 재사용 버퍼를 사용해 반복 배열 생성을 줄임. Collider의 계층 깊이가 달라도 문제 없도록 `GetComponentInParent<M_Object>`사용.

Worker는 0.2초 간격으로 채집 가능한 대상만 검색.

주요 파일:

- `Assets/00.Scripts/InteractionDetector.cs`
- `Assets/00.Scripts/Player/Player_FindObject.cs`
- `Assets/00.Scripts/Worker/Worker.cs`

## 5. Character 장비와 Worker 상태

### EquipmentController

장비 활성화, 전체 비활성화, Player 장비 구성을 Worker로 복사하는 책임을 `EquipmentController`로 이동.

Worker 생성 시 다음 값이 Player와 일치하도록 설정.

- 장비 배열 순서
- 장비가 연결되는 손 본
- 로컬 위치와 회전
- 로컬 크기
- 초기 활성 상태

Worker 모델에 대응 장비가 없으면 Player 장비를 대응 본 아래에 복제.

### Character 캡슐화

- 공개 `m_Object`를 비공개 `interactionTarget`으로 변경
- 공개 `colliders`를 비공개 `attackTargets`로 변경
- 외부 접근은 `SetInteractionTarget`, `SetAttackTargets` 사용
- `[FormerlySerializedAs]`로 기존 씬 및 프리팹 값 보존
- `ParitcleTransform` 오타를 `particleTransform`으로 수정하고 이전 직렬화 이름 보존

### Worker 상태

상태별 동작을 다음 메서드로 분리.

- `EnterIdle`
- `EnterArrived`
- `SearchForTarget`
- `WaitForDestination`
- `StopNavigation`
- `StopTargetSearch`

추가 변경:

- Animator 문자열 해시 캐시
- 중복 이동 코루틴 방지
- `StopAllCoroutines` 제거
- NavMeshAgent와 잘못된 경로 방어
- 목적지 설정 실패 시 Idle 복귀
- 목표 파괴 시 새 목표 검색

## 6. 미사용 코드와 부가 오류

삭제한 파일:

- `Assets/00.Scripts/Manager/Player_Handler.cs`
- `Assets/00.Scripts/Utils/UI_Debug.cs`
- `Assets/00.Scripts/Manager/ItemDrop_Manager.cs`

제거한 항목:

- 사용되지 않는 Rain 이벤트
- `M_Object.GetInteraction`
- `BuildingObject.Working`
- `Utils.SetLayer`
- 런타임 코드의 `NUnit.Framework`와 기타 불필요한 namespace

함께 수정한 기능 오류:

- `ObjectManager`가 마지막 Object 데이터를 선택하지 못하던 Random 범위
- 생성 위치 검색 무한 반복 방지를 위한 최대 100회 제한
- Loot 최대 수량이 나오지 않던 정수 Random 범위
- 아이템 분산 방향을 X/Y에서 지면 X/Z로 수정
- `arcHeight`를 아이템 포물선 이동에 실제 적용
- CharacterController에 Player 중력 적용
- HP와 Stamina를 0~최대값 범위로 제한
- 여러 Collider가 겹치는 건물 배치 영역을 `HashSet<Collider>`로 집계
- `Comfirm`을 `Confirm`으로 수정
- 자원 흔들림 방향을 실제 공격자 기준으로 계산
- `GreenTree.prefab`이 `Tree_Blue.asset`을 참조하던 오류 수정
- Editor 로드 시 프리팹 자동 덮어쓰기를 경고 기반 수동 복구로 변경

## 7. F Key UI가 사라지지 않던 오류

### 원인

추측한 오류의 원인

1. `UI_AnimationHandler`가 Animator를 `Start`에서 초기화.
2. F Key UI가 생성된 동일 프레임에 상호작용 시작 또는 숨김 요청이 오면 `Start` 이전에 Out 트리거를 호출.
3. 기존 Dictionary는 Out 애니메이션을 요청한 즉시 UI 추적 항목을 제거.

Animator가 아직 null이면 Out 애니메이션과 `Destroy_Object` 이벤트가 실행되지 않음. Dictionary에서도 이미 제거되었기 때문에 남은 GameObject를 이후 다시 정리할 수 없었음.

추가로 F Key 프리팹의 TMP와 Image가 Raycast Target이어서, 마우스가 프롬프트에 겹치면 프롬프트 자신을 일반 UI로 판단하고 상호작용 대상을 순간적으로 해제함. 이 과정에서 숨김과 재생성이 빠르게 반복.

### 해결

- Animator 초기화를 `Start`에서 `Awake`로 이동
- 항상 하나만 필요한 프롬프트 Dictionary를 제거
- `activePrompt`와 `promptTarget`으로 실제 화면 인스턴스를 끝까지 추적
- 생성, 위치 갱신, 애니메이션 숨김, 즉시 제거 경로 분리
- 다음 모든 경로가 `HideInteractionPrompt`를 사용하도록 통합
  - 상호작용 시작
  - 상호작용 종료
  - 대상 거리 이탈
  - 몬스터 감지
  - 일반 UI 위로 포인터 이동
  - Player 탐지 컴포넌트 비활성화
- Out 애니메이션 이벤트가 실패해도 0.5초 뒤 강제 제거하는 fallback 추가
- 컴포넌트 비활성화 시 프롬프트 즉시 제거
- `'F'Key.prefab`의 TMP와 Image 세 개에서 Raycast Target 비활성화

주요 변경 파일:

- `Assets/00.Scripts/Player/Player_FindObject.cs`
- `Assets/00.Scripts/UI/UI_AnimationHandler.cs`
- `Assets/02.Prefabs/'F'Key.prefab`
