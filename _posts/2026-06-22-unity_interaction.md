---
title: "[Unity 3D] 1인칭 플레이어 상호작용 구현하기 (문 열기)"
date: 2026-06-22 22:06:11 +0900
categories: [Unity 3d]
tags: [c#, raycast]     # TAG names should always be lowercase
---

# [Unity 3D] 1인칭 플레이어 상호작용 구현하기 (문 열기)

> 플레이어가 바라보는 방향에 상호작용 가능한 물체가 있으면 마우스 클릭(또는 E키)을 했을 때 상호작용을 하는 기능.
> 

![game_test.png](/assets/img/20260622_2206/game_test.png)

![image.png](/assets/img/20260622_2206/image.png)

```
**핵심 로직**

화면의 정중앙 방향으로 Ray를 쏘기

맞은 물체가 있다면 currentTarget에 할당하고 

입력이 들어왔을 때 currentTarget에 뭔가가 있다면 

그 물체의 상호작용 이벤트를 발생
```

# Input System에 Action 추가하기

플레이어의 입력을 받는 Input System을 **[Project]** 에서 찾아 **Edit Asset**을 클릭해 창을 열어주세요.

![action_map_editor.png](/assets/img/20260622_2206/action_map_editor.png)

새로운 Button 타입 Action과 바인딩을 추가해 주세요.

자세한 방법은 아래 게시물에서 확인하실 수 있습니다.

# 오브젝트 상호작용 스크립트 작성하기

```csharp
public class Interactable : MonoBehaviour
{
    public UnityEvent OnInteract;

		// OnInteract에 등록된 함수가 있다면 실행.
    public void OnRayInteract() => OnInteract?.Invoke();
}
```

오브젝트에 해당 스크립트를 할당해주고 Inspector 창에서 OnInteract에 실행될 함수(ex. 문 열기)를 할당해 주세요.

![image.png](/assets/img/20260622_2206/image%201.png)

 

# 플레이어 상호작용 스크립트 작성하기

```csharp
public float dist;   // 최대 상호작용 가능 거리
InputAction interact;   // 상호작용 입력 
Interactable current_target;   // 현재 보고 있는 물체

void Start()
{
    interact = InputSystem.actions.FindAction("Interact");
    interact.performed += ctx => Interact();
    current_target = null;
}

// 플레이어가 상호작용 키를 눌렀을 때 실행
void Interact()
{
		// 현재 보고 있는 상호작용 가능한 타겟이 있다면
    if(current_target != null)
    {
		    // 상호작용 이벤트 실행
        current_target.OnRayInteract();

    }
}
```

**Start**에서는 입력이 들어왔을 때 인식할 수 있도록 변수들을 초기화시켜줍니다.

**FindAction** 부분에 Input System에서 추가해줬던 action 이름을 넣어주세요.

**performed** 이벤트에 **Interact**라는 함수를 구독합니다. action에 할당된 입력(ex. 마우스 클릭, E키 누름)이 들어왔을 때 performed 이벤트가 트리거됩니다.

current_target은 현재 상호작용 가능한 물체이니 처음엔 null로 초기화해주세요.

**Interact** 함수는 실제 상호작용 이벤트를 발생시키는 함수입니다.

**current_target**의 **Interactable** 클래스 또는 인터페이스에 **OnInteract**라는 이벤트가 선언되어 있어야 합니다.

```csharp
void Update()
{
		// 플레이어가 바라보는 방향으로 Ray 발사
    Ray ray = Camera.main.ScreenPointToRay(new Vector2(Screen.width / 2.0f, Screen.height / 2.0f));
    RaycastHit hit;

		// Ray에 맞은 물체가 있다면
    if (Physics.Raycast(ray, out hit, dist))
    {
        Debug.DrawLine(ray.origin, hit.point);
				
        var target = hit.collider.GetComponent<Interactable>();

				// 그 물체가 상호작용 가능한 물체라면
        if (target != null)
        {
		        // 그 물체를 currentTarget에 할당
            if (current_target != target)
            {
                current_target = target;
            }

        }
        
        // 상호작용이 불가능한 물체라면
        else
        {
		        // currentTarget에 null 할당
            if (current_target != null)
            {
                current_target = null;
            }
        }
    }
    // 맞은 물체가 없다면
    else
    {
		    // currentTarget에 null 할당
        if (current_target != null)
        {
            current_target = null;
        }
    }
}
```

**Update**에서는 실시간으로 플레이어가 바라보는 방향에 Ray를 쏴서 **current_target**을 설정해주는 일을 합니다.

화면 정중앙으로 Ray를 쏘는 코드는 다음과 같습니다.

```csharp
Ray ray = Camera.main.ScreenPointToRay(new Vector2(Screen.width / 2.0f, Screen.height / 2.0f));
```

화면의 왼쪽 아래 부분이 (0, 0)이고 오른쪽 위 부분이 (Screen.width-1, Screen.height-1) 입니다.

**Physics.Raycast()** 는 Ray에 맞은 물체가 있다면 true를 반환하고 hit에 맞은 물체를 할당합니다.

물체에는 **Collider** 컴포넌트가 있어야 합니다.

**target** 에 Ray에 맞은 물체의 Interactable 컴포넌트를 할당해줍니다.

# 테스트 해보기

실제로 실행했을 때 상호작용이 안 된다면 확인해보세요.

상호작용이 가능한 물체가 **[Game]** 화면 중앙에 놓인 상태에서 **[Scene]** 을 봤을 때 **Debug.DrawLine**에 의해 선이 생기는가?

![image.png](/assets/img/20260622_2206/image%202.png)

![image.png](/assets/img/20260622_2206/scene_test.png)

안 생긴다 → 물체의 **Collider** 컴포넌트 확인.

생긴다 → 물체에 **Interactable** 스크립트가 할당되어 있는지 확인

그리고 interact 입력을 받았을 때 **Debug.Log** 를 출력하게 해보세요.

만약 입력을 받았을 때 Log를 출력하지만 작동을 안 한다면 **OnInteract** 이벤트에 뭔가가 할당되어 있는지도 확인해보시면 좋습니다.

# 번외: 문을 열고 닫기

1. 문 오브젝트에 애니메이션 4개를 만들어주세요.
    
    Closed - 닫힌 상태
    
    Opening - 열리는 중
    
    Opened - 열린 상태
    
    Closing - 닫히는 중
    
2. 애니메이터를 편집해주세요
    
    ![image.png](/assets/img/20260622_2206/animator.png)
    
    파라미터: Open (Integer 타입)
    
    트랜지션
    
    Closed → Opening : Open / Equals / 1
    
    Opened → Closing: Open / Equals / 0
    
3. 애니메이터를 제어하는 스크립트를 만들어 주세요.

```csharp
public class Controll_Animator : MonoBehaviour
{
    Animator animator;
		
    public string parameter;  // 파라미터 명. 인스펙터에서 Open이라고 입력해주세요.

    void Start()
    {
        animator = GetComponent<Animator>();
    }

    public void AnimSetInteger()
    {
		    // 0은 닫힘. 1은 열림.
        int value = (animator.GetInteger(parameter)==0)?1:0;
        animator.SetInteger(parameter, value);
    }
}
```

 4. Interactable의 OnInteract 이벤트에 AnimSetInteger를 할당해 주세요.

![image.png](/assets/img/20260622_2206/image%203.png)