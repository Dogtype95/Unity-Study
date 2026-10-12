
### Align With View 단축키


Unity의 **Align With View**는 선택한 오브젝트(주로 메인 카메라)의 위치와 회전을 **현재 Scene 뷰에서 보고 있는 시점과 완벽하게 일치**시키는 기능


- **단축키**: `Ctrl + Shift + F` (macOS: `Cmd + Shift + F`)
    
- **메뉴 바**: 상단 메뉴에서 `GameObject` → `Align With View`

| **기능**                     | **단축키**            | **설명**                                  |
| -------------------------- | ------------------ | --------------------------------------- |
| **Align With View**        | `Ctrl + Shift + F` | 선택한 오브젝트를 **Scene 뷰 시점으로 이동**시킴         |
| **Align View to Selected** | `Ctrl + Alt + F`   | **Scene 뷰 카메라**를 선택한 오브젝트의 위치/시점으로 이동시킴 |
| **Frame Selected**         | `F`                | 선택한 오브젝트가 화면 중앙에 꽉 차도록 Scene 뷰를 포커스함    |
![[Pasted image 20261012103348.png]]



### Transform.LookAt(Transform target);

- 타겟을 바라보도록 해주는 함수
- 두번째 매개변수로 Vector3 worldUp를 넣는 오버라이드 버전을 쓰면 뒤집어져서 바라보게 만들 수도 있다

![[Pasted image 20261012111434.png]]

### Quaternion.LookRotation()


`Quaternion.LookRotation()`은 지정한 바라볼 방향 벡터(forward)와 머리 위 방향 벡터(upwards)를 기반으로 오브젝트의 회전 쿼터니언을 생성하는 유니티의 핵심 회전 함수입니다.

`transform.LookAt()`과 달리 대상 오브젝트의 회전을 즉시 바꾸지 않고 **회전 값(`Quaternion`)만 반환**하므로, 부드러운 회전 보간이나 특정 축 회전 제한을 구현할 때 필수로 사용됩니다.

### 1. 문법 및 매개변수

C#

```
public static Quaternion LookRotation(Vector3 forward, Vector3 upwards = Vector3.up);
```

- **`forward` (필수)**: 바라보고자 하는 앞방향 벡터 ($+Z$ 축 기준).
    
- **`upwards` (선택, 기본값: `Vector3.up`)**: 오브젝트의 위쪽 방향 벡터 ($+Y$ 축 기준). 생략 시 월드 좌표계의 위쪽 `(0, 1, 0)`이 사용됩니다.
    

### 2. `transform.LookAt`과의 차이점

|**구분**|**transform.LookAt(target)**|**Quaternion.LookRotation(direction)**|
|---|---|---|
|**반환값**|없음 (`void`)|**회전 값 반환 (`Quaternion`)**|
|**회전 적용 방식**|호출 즉시 타깃을 향해 즉각 회전|연산된 쿼터니언을 사용자가 원하는 방식(`Slerp`, `RotateTowards` 등)으로 적용|
|**인자 형태**|목표 위치(`Transform` 또는 `Vector3` 월드 좌표)|**방향 벡터** (`목표 위치 - 현재 위치`)|
|**용도**|단순 고정 시선, 즉시 조준|**부드러운 회전 보간, 특정 축 회전 고정, 유도 탄환 등**|

### 3. 실무 대표 활용 패턴

#### 1) 타깃을 향해 부드럽게 회전하기 (`Slerp` 결합)

대상을 향해 즉시 휙 돌지 않고 자연스럽게 몸을 돌리도록 구현합니다.

C#

```
[SerializeField] private Transform target;
[SerializeField] private float rotateSpeed = 5f;

void Update()
{
    if (target == null) return;

    // 타깃을 향하는 방향 벡터 계산
    Vector3 direction = target.position - transform.position;

    // 방향 벡터의 크기가 0이 아닐 때만 회전 수행 (안전 검사)
    if (direction.sqrMagnitude > 0.001f)
    {
        Quaternion targetRotation = Quaternion.LookRotation(direction);
        transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, Time.deltaTime * rotateSpeed);
    }
}
```

#### 2) 캐릭터의 수평 회전(Yaw)만 유지하기

적을 바라볼 때 캐릭터 몸체가 위아래로 꺾이거나 기우는 것을 막고, 지면 평면(Y축) 기준으로만 회전하도록 처리합니다.

C#

```
Vector3 direction = target.position - transform.position;
direction.y = 0f; // 수직 성분을 0으로 만들어 수평 회전만 남김

if (direction != Vector3.zero)
{
    transform.rotation = Quaternion.LookRotation(direction);
}
```

#### 3) 이동 방향을 바라보며 걷는 캐릭터

WASD 입력 벡터를 기반으로 이동하는 방향을 앞면으로 설정할 때 사용합니다.

C#

```
float h = Input.GetAxisRaw("Horizontal");
float v = Input.GetAxisRaw("Vertical");
Vector3 moveDir = new Vector3(h, 0f, v).normalized;

if (moveDir != Vector3.zero)
{
    Quaternion targetRot = Quaternion.LookRotation(moveDir);
    transform.rotation = Quaternion.RotateTowards(transform.rotation, targetRot, 720f * Time.deltaTime);
}
```

### 4. 주의해야 할 예외 상황

- **`Vector3.zero` 전달 시 경고 발생**:
    
    - 타깃과 내 위치가 완전히 겹치거나 입력이 없어 방향 벡터가 `(0, 0, 0)`이 되면, 유니티 에디터에 `"Look rotation viewing vector is zero"` 경고가 출력됩니다.
        
    - 따라서 호출 전 `if (direction != Vector3.zero)` 또는 `if (direction.sqrMagnitude > 0.001f)`로 0 벡터 검사를 수행해야 합니다.
        
- **`forward`와 `upwards`가 평행할 때의 특이점 (Gimbal Lock 유사 문제)**:
    
    - 캐릭터가 수직 바로 위(`Vector3.up`)나 수직 바로 아래(`Vector3.down`)를 바라보게 되면 기본 `upwards` 벡터와 평행해져 기준 축을 결정할 수 없어 모델이 급격히 튀거나 떨리는 현상이 일어납니다.
        
    - 수직 조준이 필요한 FPS/비행기 시뮬레이션의 경우 `upwards` 기준을 카메라의 로컬 위쪽 벡터(`transform.up`)로 명시하거나 별도의 Pitch/Yaw 오일러 분리 방식을 사용합니다.