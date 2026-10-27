---
title: "유니티 생존 게임"
date: 2026-07-01 09:00:00 +0900
categories: [PortfolioProjects, UnitySurvival]
tags: [Csharp, Unity]
---



# Unity 서바이벌 게임

생성일: 2025년 9월 14일 오후 5:51

![image.png](/assets/img/unitysurvive/image.png)

# 게임 설명

나무와 돌을 캐서 자원을 얻어 건물을 생성, 동료들을 소환하면서 몬스터들의 위협을 피해 생존하는 생존게임 입니다.

# 게임 개발
- 인원 : 1인 개발 - 클라이언트(본인)
- 목적 : 더 넓은 공간과 많은 오브젝트가 존재하는 게임 제작 경험, 역량 축적.
- 제작기간 : 2025/08/07 ~ 2025/09/15
- 기술 : Unity, C#


### 1. 오브젝트 최적화

- 게임 내에 존재하는 오브젝트들은 미리 배치한 것이 아니라 게임이 시작 될 때 무작위 위치에 생성되게 함. 때문에 Occlusion Culling을 사용 할 수 없었고 Culling Group을 사용하여 플레이어와 가까운 오브젝트만 활성화 시켜 최적화 진행
- 코드
    
    ![image.png](/assets/img/unitysurvive/image-1.png)
    
    ![image.png](/assets/img/unitysurvive/image-2.png)
    
- 게임의 완성 단계에서 시작 시 생성하는 오브젝트의 숫자가 너무 많아지면서 노트북에서 게임 시작시에 에디터가 멈추거나 버벅거리는 현상 발생.
- 한 번에 모든 오브젝트를 생성하는게 아니라 프레임 단위로 나눠서 이런 현상을 최대한 완화
- 코드
    
    ![image.png](/assets/img/unitysurvive/image-3.png)
    

### 2. CompassBar

- 맵이 넓어서 플레이어가 배치한 건물과 일꾼의 위치를 알기 어려움.
- 미니맵을 구현하는 방식도 있지만 서바이벌 게임에 미니맵보단 나침반 역할을 하는 UI가 더 잘 어울린다고 생각.
- 플레이어가 바라보는 방향의 각도를 기준으로 방위와 배치한 건물, 일꾼의 위치와 거리를 표시하는 UI구현.
- 코드
    
    ![image.png](/assets/img/unitysurvive/image-4.png)
    

### 3.인벤토리 관리

- 게임에서 사용하는 아이템의 종류가 한정적이므로 Dictionary를 통해 플레이어가 가지고 있는 아이템의 종류와 개수 관리
- 아이템의 고유 ID를 Key로 사용하고 Item Data와 Count를 가지고 있는 클래스 객체를 Value로 사용해 소유한 아이템 개수를 쉽게 알 수 있게 관리.
- 코드
    
    ITEM클래스
    
    ![image.png](/assets/img/unitysurvive/image-5.png)
    
    Static Dictionary를 통한 관리
    
    ![image.png](/assets/img/unitysurvive/image-6.png)
    

- 게임 파일 : https://github.com/Alshen1025/Survival.git
- 게임 영상 :

[https://youtu.be/lB0bO1nqwaM](https://youtu.be/lB0bO1nqwaM)
