---
title: "[Unity 3D] ProBuilder로 간단한 3D 오브젝트 만들기"
date: 2026-06-22 22:09:13 +0900
categories: [Unity3d]
tags: [probuilder]     # TAG names should always be lowercase
---

> 유니티 ProBuilder의 Boolean Tool을 이용해 창문 있는 벽 만들기.
> 

<aside>
💡

먼저 유니티 6보다 낮은 버전은 ProBuilder 패키지를 직접 설치해주셔야 합니다.

(유니티 6에는 이미 Probuilder가 내장되어 있습니다.)

</aside>

 

# 오브젝트 ProBuilderize하기

Probuilder로 편집하고 싶은 오브젝트를 선택하고 **Tools > ProBuilder > Object > Pro Builderize**를 선택합니다. (또는 **Window > ProBuilder > …** 또는 상단 바의 **ProBuilder** 메뉴)

![image.png](/assets/img/20260622_2209/image.png)

**[Scene]** 상단에서 Object 모드와 ProBuilder 모드를 선택할 수 있습니다.

![image.png](/assets/img/20260622_2209/image%201.png)

 

# Boolean Tool 알아보기

**Boolean Tool**은 다음과 같은 기능을 제공합니다.

- Union - 두 개의 모델을 합치거나
- Intersection - 겹치는 부분만 남기거나
- Subtraction - 한 모델에서 다른 모델 부분을 빼거나

할 수 있습니다. 특히 **Subtraction** 기능은 벽에 창문을 뚫거나 문을 뚫기 위해 많이 쓰입니다.

먼저 벽(남겨야 할 부분) 오브젝트와 창문(뚫어야 할 부분) 오브젝트를 모두 **Probuilderize** 해줍니다.

![image.png](/assets/img/20260622_2209/image%202.png)

그 다음 **Tools > ProBuilder > Experimental > Boolean (CSG) Tool**을 선택해 창을 엽니다.

![image.png](/assets/img/20260622_2209/image%203.png)

왼쪽 칸에는 벽을, 오른쪽 칸에는 창문을 할당해 줍니다.

![image.png](/assets/img/20260622_2209/image%204.png)

Operation은 **Subtraction**를 선택하고 Apply를 누르면 새로운 오브젝트(창문 있는 벽)가 만들어집니다.

![image.png](/assets/img/20260622_2209/image%205.png)

 

# Collider 적용하는 법

위에서 Boolean Tool을 통해 새로 생성한 오브젝트는 Collider가 없는 상태입니다. 

이 오브젝트에 Collider를 적용하려면 **Mesh Collider** 컴포넌트를 추가해주세요.