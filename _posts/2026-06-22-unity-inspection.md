---
title: "[Unity 3D] 1인칭 플레이어 조사 기능 구현하기"
date: 2026-06-22 22:06:10 +0900
categories: [Unity3d]
tags: [c#, lerp]     # TAG names should always be lowercase
---

> 책이나 편지 같은 단서를 수집할 때 플레이어 눈 앞으로 Lerp해서 단서를 조사할 수 있는 기능.              

 E키 → Lerp → 조사 → E키 → 조사 종료
> 

![screenshot.png](/assets/img/20260622_2210/screenshot.png)

![screenshot1.png](/assets/img/20260622_2210/screenshot1.png)

 

```
**핵심 로직**

이미 조사 중인지 확인

조사 상태 아님

		플레이어 눈 앞으로 Lerp
		
		플레이어 이동 및 화면 회전 잠금
		
		Lerp 끝난 후 UI 판넬 열기

조사 상태

		오브젝트 비활성화
		
		플레이어 이동 및 화면 잠금 해제
		
		UI 판넬 닫기
```

# 오브젝트 스크립트 작성하기

조사 가능한 오브젝트(책)에 다음 스크립트를 부착합니다.

```csharp
public class Object_Inspecatable : MonoBehaviour
{
		[Header("Player Character")]
		public Player_Move player;   // 플레이어 이동 스크립트
		
    [Header("Inspect Settings")]
    public float targetTime = 0.5f;   // Lerp는 0.5초 동안 이루어짐.
    public float inspectDistance = 1f;  // 목표 위치. 카메라로부터 1m 떨어진 곳.

    [Header("UI Settings")]
    public string disc;   // UI에 표시할 설명.
    public GameObject panel;   // UI 판넬 오브젝트

    private bool isInspecting = false;   // 이미 조사 중인지

    private Camera mainCamera;

    private void Start()
    {
        mainCamera = Camera.main;
    }

		// 조사 버튼 눌렀을 때 실행
    public void OnInspect()
    {
		    // 해당 오브젝트를 조사 중이지 않다면
        if (!isInspecting)
        {
		        // 코루틴 실행
            StartCoroutine(MoveToInspectPosition());
        }
		    // 조사 중이라면
        else
        { 
		        // 조사 화면 종료
            gameObject.SetActive(false);
            isInspecting = false;
            panel.SetActive(false);
            player.SetMoveLock(false);
            // player 인벤토리에 추가해야 한다면 인벤토리 함수 호출.
        }
    }

    ...
}
```

OnInspect는 실제로 플레이어가 조사 키를 눌렀을 때 실행되는 함수입니다.

![image.png](/assets/img/20260622_2210/image.png)

저는 상호작용 시에 실행되도록 하기 위해 OnInteract 이벤트에 할당해 주었습니다.

(이전 게시물 참고)

이미 조사 상태인데 플레이어가 조사 키를 한번 더 눌렀다면 조사 화면을 종료합니다.

만약 현재 조사 상태가 아니라면 StartCoroutine을 통해 아래 Coroutine을 실행합니다.

while 함수를 돌며 Lerp를 실행해 {경과 시간 / 목표 시간} 비율 만큼 오브젝트가 이동하도록 하였습니다.

```csharp
IEnumerator MoveToInspectPosition()
{
    isInspecting = true;

    // 플레이어 이동 잠금
    player.SetMoveLock(true);

		// 시작 위치
		Vector3 startPosition = transform.position;
    Quaternion startRotation = transform.rotation;
    
    // 목표 위치 (카메라 앞 1m)
    Vector3 targetPosition =
        mainCamera.transform.position +
        mainCamera.transform.forward * inspectDistance;
		// 목표 회전 (플레이어를 바라보도록)
    Quaternion targetRotation =
        Quaternion.LookRotation(mainCamera.transform.forward);

		// 경과 시간
    float elapsedTime = 0f;

		while (elapsedTime<targetTime)
		{
		    float t = elapsedTime / targetTime;
		
		    transform.position = Vector3.Lerp(startPosition, targetPosition, t);
		    transform.rotation = Quaternion.Lerp(startPosition, targetRotation, t*10);
		        
				elapsedTime += Time.deltaTime;
		    yield return null;
		}
		
		transform.position = targetPosition;
		transform.rotation = targetRotation;
		// UI 판넬 열기
    panel.GetComponentInChildren<TextMeshProUGUI>().text = disc;
    panel.SetActive(true);
}
```

플레이어 이동 로직 Player_Move에 플레이어의 이동을 잠그는 SetMoveLock() 함수도 추가해 주어야 합니다.

# 플레이어 스크립트 편집: 이동 및 회전 잠그기

조사 중에는 플레이어가 움직이거나 마우스로 카메라를 회전할 수 없습니다. 기존에 있던 플레이어 이동 관련 스크립트에 아래 코드를 추가해주면 됩니다.

```csharp
bool moveLocked = false;
...

private void Update()
{
		if (moveLocked) return;
		
		Look();   // 카메라 회전
		
		Move();   // 플레이어 이동
		...
}

public void SetMoveLock(bool locked)
{
    moveLocked = locked;
}
```

SetMoveLock 함수는  moveLocked라는 bool 변수를 설정해주는 함수입니다.

Update 초반에 moveLocked가 true이면 리턴하도록 

```csharp
if (moveLocked) return;
```

를 추가해 이후 Look 함수와 Move 함수가 실행되지 않도록 했습니다.