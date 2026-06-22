---
title: "[Unity 3D] InputAction 사용하기"
date: 2026-06-22 22:08:25 +0900
categories: [Unity3d]
tags: [c#, inputaction]     # TAG names should always be lowercase
---

> 유니티의 Input System을 통해 입력 받기
> 

# Input Action Asset 생성하기

**[Project]**에서 **Create>InputActions**

![image.png](/assets/img/20260622_2208/image.png)

# Input Action Asset 편집하기

새로 만든 Input Action Asset을 **[Project]**에서 더블 클릭하거나 **[Inspector]**에서 **Edit Asset**을 클릭하면 편집 창이 열립니다.

**Action Maps** 옆의 **+** 버튼을 눌러 새로운 액션 맵을 생성하고

**Actions** 탭에서 **+** 버튼을 눌러 새로운 액션을 생성할 수 있습니다.

새로운 액션을 만든 후 옆에 있는 **+** 버튼을 눌러 **Add Binding**을 선택해 주세요.

Binding은 다양한 플랫폼 입력을 지원합니다. (키보드, 마우스, 게임패드, 조이스틱 등)

원하는 바인딩을 추가해줍니다.

**Listen** 버튼을 누르고 원하는 키를 입력해주면 쉽게 추가할 수 있습니다.

![image.png](/assets/img/20260622_2208/image%201.png)

**Action은 크게 세 가지 타입이 있습니다.**

1. Value 타입

ReadValue<>() 를 통해 실시간 입력 값을 받습니다. 주로 이동 로직에서 많이 쓰이죠.

1. Button 타입

단순히 버튼을 누른 상태인지 아닌지가 중요하다면 Button 타입이 적합합니다.

1. Pass through 타입

실시간 입력 값을 필터링 없이 그대로 가져옵니다. 아무리 작은 값이라도 반응을 해야 한다거나 값을 빠르게 자주 읽는 것이 가능하죠. 마우스 커서 이동에 따라 카메라를 회전하는 로직에 적합합니다.

# 스크립트에서 InputAction 참조하기

스크립트에서 InputAction을 참조하는 방식은 두 가지를 소개합니다.

1. **public** 접근 지정자를 사용하여 **[Inspector]** 에서 직접 Action 과 바인딩을 추가하기.

이때 **InputAction**은 Action을 하나 지정할 수 있고 **InputActionMap**은 여러 Action을 한번에 지정할 수 있습니다.

주의할 점은 이 방식은 Enable을 실행하지 않으면 작동하지 않습니다.

```csharp
public InputAction action1;
public InputAction action2;
public InputActionMap actionmap;

private void OnEnable()
{
		action1.Enable();
		action2.Enable();
		actionmap.Enable();
}
```

![image.png](/assets/img/20260622_2208/image%202.png)

1. **FindAction**을 통해 Player Input에 할당된 Input Action Asset을 참조하기

이 방식은 Unity 최신 버전(Input System 1.7 이상)에서 가능합니다.

InputAction 변수를 public으로 선언하지 않아도 됩니다. 오브젝트의 **Player Input** 컴포넌트에 할당된 Input Action Asset에서 등록된 Action을 찾아냅니다. 

![image.png](/assets/img/20260622_2208/image%203.png)

주의할 점은 FindAction을 사용할 때는 Action 이름이 Input Action Asset에 등록된 Action과 일치해야 합니다.

```csharp
InputAction lookAction;
InputAction moveAction;
InputAction runAction;
InputAction hideAction;
InputAction jumpAction;

private void Start()
{
		  lookAction = InputSystem.actions.FindAction("Look");
		  moveAction = InputSystem.actions.FindAction("Move");
		  runAction = InputSystem.actions.FindAction("Run");
		  hideAction = InputSystem.actions.FindAction("Hide");
		  jumpAction = InputSystem.actions.FindAction("Jump");
 }
```

저는 두 번째 방법을 주로 사용합니다.

# 스크립트에서 InputAction 활용하기

InputAction 변수에 Action을 할당해 주었다면 이제 입력이 들어왔을 때 반응을 해야겠죠.

```csharp
void Update()
{
    ...

    if (lookAction != null) LookInput = lookAction.ReadValue<Vector2>();
    if (moveAction != null) MoveInput = moveAction.ReadValue<Vector2>();
}   
```

Value 타입 액션에서 **ReadValue**를 통해 값을 읽고 있습니다.

```csharp
void Move()
{
    bool isRunning = (runAction != null && runAction.IsPressed());
    bool isHiding = (hideAction != null && hideAction.IsPressed());
    bool isJumping = (jumpAction != null && jumpAction.triggered);
    
    ...
}
```

**IsPressed**는 현재 프레임에서 버튼을 눌렀는지 아닌지를 반환합니다.

**triggered**는 처음 버튼을 눌렀을 때에만 true를 반환합니다.

하지만 이렇게 매번 값을 읽는 것이 아닌 필요할 때에만 호출하는 방법이 있습니다.

```csharp
InputAction interact;

void Start()
{
    interact = InputSystem.actions.FindAction("Interact");
    // interact 버튼이 눌렸을 때 Interact 함수 실행.
    interact.performed += ctx => Interact();
    ...
}
```

Input System의 **Callback**은 입력이 들어오면 알아서 함수를 실행시켜줍니다.

| started | Action이 시작됐을 때 Callback. 버튼을 누르기 시작했을 때 1번 호출 |
| --- | --- |
| performed | Action이 수행됐을 때 Callback. 버튼 입력이 완료되었을 때(started 직후) 호출. 이후 계속 누르고 있는 상태라면 주기적으로 호출. |
| canceled | Action이 끝났을 때 Callback. 버튼을 떼었을 때 1번 호출. |

Button 타입에서는 started와 performed가 사실 상 동시에 호출됩니다. 처음 버튼이 눌렸을 때 딱 한 번 실행됩니다.

`+=` 를 통해 callback에 원하는 함수들을 등록할 수 있습니다.

`ctx`를 통해 읽은 값을 함수에 인자로 전달할 수 있습니다.

```csharp
// 읽은 값을 Move 함수에 전달.
moveAction.performed += ctx => Move(ctx.ReadValue<Vector2>());
moveAction.canceled += ctx => Move(Vector2.zero);
```