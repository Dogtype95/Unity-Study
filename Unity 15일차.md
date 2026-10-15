

## Unity InputSystem


### Direct 방식

```cs
using UnityEngine;
using UnityEngine.InputSystem; // 필수 네임스페이스

public class DirectInputExample : MonoBehaviour
{
    void Update()
    {
        // 1. 키보드 직접 폴링 (Keyboard.current)
        if (Keyboard.current != null)
        {
            // 눌려 있는 중인지 확인 (구 Input.GetKey)
            if (Keyboard.current.wKey.isPressed)
            {
                transform.Translate(Vector3.forward * 5f * Time.deltaTime);
            }

            // 이번 프레임에 막 눌렸는지 확인 (구 Input.GetKeyDown)
            if (Keyboard.current.spaceKey.wasPressedThisFrame)
            {
                Debug.Log("점프!");
            }

            // 이번 프레임에 손을 뗐는지 확인 (구 Input.GetKeyUp)
            if (Keyboard.current.leftShiftKey.wasReleasedThisFrame)
            {
                Debug.Log("달리기 해제");
            }
        }

        // 2. 마우스 직접 폴링 (Mouse.current)
        if (Mouse.current != null)
        {
            // 좌클릭 감지
            if (Mouse.current.leftButton.wasPressedThisFrame)
            {
                Debug.Log("좌클릭 발사");
            }

            // 마우스 커서 화면 좌표 (Vector2)
            Vector2 mouseScreenPos = Mouse.current.position.ReadValue();

            // 마우스 델타(이동량) 값
            Vector2 mouseDelta = Mouse.current.delta.ReadValue();
        }

       
    }
}


```

### Input Action 바인딩(Embedded 방식)
![[Pasted image 20261015140355.png]]

유니티 New Input System에서 **임베디드(Embedded / Inline) 방식**은 별도의 `.inputactions` 에셋 파일을 프로젝트 창에 생성하지 않고, **스크립트의 인스펙터나 컴포넌트 내부에 입력 액션(InputAction)을 직접 포함시켜 정의하는 방식**


- input system의 바인딩 옵션은 3가지가 있다
```cs
private InputAction _action;

private void OnEnable()
{
	_action.started += Move; // 키 입력이 시작된다면 바로 감지
	_action.performed += Move; // 유효한 값에 도달하면 감지 (유효값은 DeadZone이라함)
					   // Hold 옵션도 있음 일정시간 Hold해야 감지(철권점프, 횡)
	_action.canceled += Move; // 키에서 때지면 감지
}

private void Move(InputAction.CallbackContext context)
{
	_rb.linearVelocityX = context.ReadValue<float>()* _speed;
}


```
![[Pasted image 20261015131907.png]]
- 인스펙터에서 옵션을 보면 액션 타입이 Value , Button 등이 있는데
- Value는 입력된 값을 가져오고 싶을떄, Button은 단순 키입력 감지용도이다.
![[Pasted image 20261015140643.png]]



### 임베디드 액션즈 방식의 간단한 이동 예제
```cs
using UnityEngine;
using UnityEngine.InputSystem;

public class SimplePlayerMove : MonoBehaviour
{
    // 인스펙터에 임베디드 형태로 노출되는 입력 액션
    [SerializeField] private InputAction moveAction;
    [SerializeField] private InputAction jumpAction;

    private void OnEnable()
    {
        // 사용 전 반드시 활성화
        moveAction.Enable();
        jumpAction.Enable();
    }

    private void OnDisable()
    {
        // 비활성화 처리
        moveAction.Disable();
        jumpAction.Disable();
    }

    private void Update()
    {
        // 1. 값 읽기 (Vector2 등)
        Vector2 input = moveAction.ReadValue<Vector2>();
        transform.Translate(new Vector3(input.x, 0, input.y) * 5f * Time.deltaTime);

        // 2. 버튼 트리거 확인
        if (jumpAction.WasPressedThisFrame())
        {
            Debug.Log("Jump!");
        }
    }
}
```

### InputAction 에셋

![[Pasted image 20261015141256.png]]