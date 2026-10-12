
### 초당 속도 적용
```text
 - 게임에서는 프레임 단위로 이동을 구현하지만 현실에서는 프레임이란 단위가 없다
 - 그래서 게임에서도 초단위로 이동하는 로직을 구현해주는게 좋다.(왜냐하면 모든 유저가 프레임이 고정되서 보장된다면 문제가 없지만 모든 하드웨어는 스펙이 다르고 게임도 요구하는 사양이 달라 프레임을 보장하는 건 불가함)
``` 

```cs
public float _speed // 인스펙터 초기화
public Vector3 _dir // 인스펙터 초기화

private void Update()
{
	// 기존 프레임 단위로 이동
	transform.position += _dir.normalized * _speed;
	
	// 초당 속도
	// deltaTime(이전프레임과 다음 프레임 사이를 누적시킨 값을 누적시킴) 
	transform.position += _dir.normalized * _speed * Time.deltaTime;
}
```

- 캐릭터는 게임 시작 시에 항상 방향은 (0,0,0) 으로 둔다 안그러면 이동시 방향이 잘못 설정된 채로 이동할 수 있음


### Unity 입력처리

- Unity Manager (과거)
```cs
if(Input.getKey)
{
	~~~~~~
}
```
- Unity Input System 예시 (IsPressed로 해도댐)
```cs
public class DHavior : MonoBehaviour
{
    private Vector3 _dir;
    public float _speed;
    public bool isMove;

    // Update is called once per frame
    void Update()
    {
            if(Keyboard.current.wKey.wasPressedThisFrame)
            {
                isMove = true;
                _dir += Vector3.forward;
            }

            if (Keyboard.current.wKey.wasReleasedThisFrame)
            {
                _dir -= Vector3.forward;
            }

            if (Keyboard.current.sKey.wasPressedThisFrame)
            {
                isMove = true;
                _dir += Vector3.back;
            }

            if (Keyboard.current.sKey.wasReleasedThisFrame)
            {
                _dir -= Vector3.back;
            }

            if (Keyboard.current.dKey.wasPressedThisFrame)
            {
                isMove = true;
                _dir += Vector3.right;
            }

            if (Keyboard.current.dKey.wasReleasedThisFrame)
            {
                _dir -= Vector3.right;
            }

            if (Keyboard.current.aKey.wasPressedThisFrame)
            {
                isMove = true;
                _dir += Vector3.left;
            }

            if (Keyboard.current.aKey.wasReleasedThisFrame)
            {
                _dir -= Vector3.left;
            }

            if (isMove)
            {
                _speed = 1;
                // 기존위치에 방향 * 속도 * 초당속도 적용
                transform.position = transform.position + _dir.normalized * _speed * Time.deltaTime;

                transform.Rotate(0, _dir.y, 0, Space.Self);
            }
            else
            {
                _speed = 0;
            }

    }

}
```


### Vector B to A

- B벡터에서 A벡터로 이동하는 방향을 알고 싶다면 B-A 해주면 된다
- 그리고 정규화 .Nomalized 해주면 방향 구하기 끝

![[Pasted image 20261008133247.png]]

***도착지 <-(요거는 빼기임) 출발지***

이렇게 해서 방향을 구해줄 수가 있다. 거기에 speed 곱해서 알아서 하자


### Vector.Distance (Vector3 a, Vector3 b)
- 두 벡터간의 길이 구해줌

### 유니티 지터링(Jittering) 현상

![[Pasted image 20261008134753.png]]
- 어떤 추적체가 플레이어 추적 기능 구현 시 추적체가 플레이어에 도달 했을때
	완전히 도달하지 못해서 덜덜 떨리는 현상 일정 거리에 들어오면 if문안에서
	 스냅처리 해주는게 좋다(그냥 추적체 위치를 목적지에 똑같이 대입해버리기)

### 로컬 좌표계, 월드 좌표계

### 2. 로컬 좌표계 vs 월드(글로벌) 좌표계

씬 뷰 상단 도구 모음(Tool Settings)에서 **Global / Local** 토글로 핸들 방향을 변경할 수 있습니다.

| **구분**          | **설명**                     | **회전 시 기즈모 동작**                               | **코드 접근**                                              |
| --------------- | -------------------------- | --------------------------------------------- | ------------------------------------------------------ |
| **Global (월드)** | 씬 전체의 고정된 절대 기준축           | 오브젝트가 회전해도 화살표 방향은 변하지 않고 항상 세상 기준(동서남북)을 가리킴 | `transform.position`<br>`transform.rotation`           |
| **Local (로컬)**  | 해당 오브젝트(또는 부모)가 바라보는 상대적 축 | 오브젝트가 회전하면 화살표(앞/위/옆)도 함께 회전함                 | `transform.localPosition`<br>`transform.localRotation` |


- 로컬 좌표계는 자기 자신 오브젝트를 기준으로 하고 월드 좌표계는 씬을 기준으로 함
- 부모 자식 오브젝트에서 자식은 부모를 기준으로 transform값이 잡힌다 부모가 없는 오브젝트는 씬 자체가 부모라 생각하면 모든 transform은 자기 부모의 기준으로 값이 잡힌다 생각 할 수 있겠다.


- 씬 화면에 있는 월드 좌표계 (Orientation)

![[Pasted image 20261008144140.png]]

### 3. 피벗(Pivot) vs 센터(Center)

에디터 상단에서 기즈모(트랜스폼 핸들)의 위치 기준을 전환할 수 있습니다.

- **Pivot:** 3D 모델링 툴에서 제작자가 **원점(0, 0, 0)으로 지정한 기준점**에 기즈모가 표시됩니다.
    
    - 회전이나 크기 조절의 중심축이 됩니다.
        
    - 예: 문의 피벗이 경첩 쪽에 있어야 회전 시 문이 자연스럽게 열립니다.
        
    - 캐릭터의 피벗은 보통 **발바닥 정중앙**에 두어 바닥에 딱 맞게 배치하도록 만듭니다.
        
- **Center:** 현재 선택된 메시(또는 다중 선택된 오브젝트들)의 **외곽 박스(Bounding Box) 기하학적 중심**에 기즈모가 배치됩니다.
    
    - 모델의 실제 원점과 무관하게 외형의 한가운데를 잡고 옮기고 싶을 때 유용합니다.
        

### 4. 피벗 위치가 잘못되었을 때 해결 방법

3D 모델의 피벗이 엉뚱한 곳에 있어서 회전할 때 비정상적으로 도는 경우, 유니티 내에서 다음과 같이 해결할 수 있습니다.

1. **빈 부모 오브젝트 트릭 (가장 흔한 방식):**
    
    - 빈 게임오브젝트(Parent)를 생성하고 원하는 피벗 위치(예: 문의 경첩 위치)에 둡니다.
        
    - 실제 모델(Child)을 그 자식으로 넣은 뒤, 부모 오브젝트의 위치를 기준으로 모델의 상대 위치를 맞춥니다.
        
    - 이후 부모 오브젝트를 회전시키면 원하는 축을 중심으로 회전합니다.
        
2. **Unity ProBuilder 패키지 활용:**
    
    - 유니티 내장 패키지인 `ProBuilder`를 설치하면 `Set Pivot` 기능을 통해 메시의 버텍스를 기준으로 피벗 위치를 직접 영구 수정할 수 있습니다.
        
3. **모델링 원본 수정:**
    
    - 3D DCC 툴(Blender, Maya 등)에서 모델 원점을 재설정하고 다시 Export하는 것이 가장 깔끔합니다.
### Surface 스냅핑
- ctrl + shift 누른채로 오브젝트를 옮기며 되고 표면에 딱 붙일때 편리함

### 언리얼 좌표축 차이와 모델링 호환 문제

- 언리얼은 유니티와 비슷하게 왼손 좌표계를 쓰지만 검지부분이 z축이라는 차이점 떄문에 모델러가 언리얼을 기준으로 모델링을 만들었다면 유니티에서는 축이 바뀌어져서 나올 수 있음

언리얼 엔진(Unreal Engine)과 유니티(Unity)는 좌표계 축(Up/Forward)과 기본 스케일 단위(cm vs m)가 다르기 때문에, 언리얼용으로 제작된 메시를 유니티로 가져오면 크기가 100배로 커지거나, 모델이 90도 누워 있거나, 회전값에 `-90` 같은 오프셋이 생깁니다.

### 엔진 간 핵심 차이점

|**구분**|**언리얼 엔진 (Unreal Engine)**|**유니티 (Unity)**|**변환 필요 사항**|
|---|---|---|---|
|**기준 축**|**Z-Up**, X-Forward (왼손 좌표계)|**Y-Up**, Z-Forward (왼손 좌표계)|Z축과 Y축의 전환 (회전축 변경)|
|**기본 단위**|**1 Unit = 1 cm**|**1 Unit = 1 m**|스케일 0.01배 축소 (100cm → 1m)|

### 방법 1. 유니티 Model Import Settings에서 설정 (DCC 툴 없이 해결)

유니티 프로젝트 창에서 FBX 파일을 선택한 뒤 **Inspector 창의 [Model] 탭**에서 설정합니다.

1. **스케일 보정 (`Scale Factor`):**
    
    - 언리얼의 cm 단위를 유니티의 m 단위로 맞추기 위해 `Scale Factor`를 `0.01`로 설정합니다.
        
    - 또는 `Convert Units` 옵션을 체크하고 `File Scale`이 0.01인지 확인합니다.
        
2. **축 회전 보정 (`Bake Axis Conversion`):**
    
    - `Bake Axis Conversion`을 체크합니다.
        
    - 언리얼의 Z-Up 좌표계를 유니티의 Y-Up 좌표계로 베이크하여, 하이라키에 배치했을 때 트랜스폼 회전값에 `-90`도 같은 지저분한 오프셋이 남지 않고 `(0, 0, 0)`으로 깔끔하게 정렬됩니다.
        
3. **하단 [Apply] 클릭:**
    
    - 설정을 적용하면 모델이 똑바로 서고 올바른 크기로 씬에 배치됩니다.

```text
fbx-model 탭에서 Bake Axis Conversion을 체크하시면 모델링 툴에서 기준으로 한 축을 감지해서 자동으로 유니티에 맞게 변환해줍니다.
```

### 오일러 각 변수와 인스펙터 표기

- 유니티의 xyz yaw pitch roll 오일러각 이라고도 함
- transform.eulerAngles 의 값을 수정해서 각도를 수정할 수 도있음 하지만 오일러각은 인간이 디버깅하고 이해하기 쉽게 표기하는 값으로 실제 동작은 쿼터니언을 통해서 오브젝트를 회전 시키기 때문에 오일러각을 직접 증감시키거나해서 각도를 조절하면 짐벌락 현상이 발생할 수 있음


### 1. 오일러 각 변수와 인스펙터 표기

- **내부 데이터와 인스펙터의 차이**:
    
    - 유니티 내부에서 오브젝트의 실제 회전 데이터(`transform.rotation`)는 쿼터니언(Quaternion, $x, y, z, w$)으로 저장됩니다.
        
    - 쿼터니언은 사람이 직관적으로 이해하기 어렵기 때문에, 인스펙터 창에서는 이를 오일러 각(Euler Angles, $X, Y, Z$ 도 단위)으로 변환하여 보여줍니다.
        
- **불일치 현상**:
    
    - 코드에서 `transform.eulerAngles = new Vector3(0, 180, 0);`을 입력해도, 내부 변환 과정을 거치며 인스펙터에는 `(180, 0, 180)` 같은 다른 등가 각도로 표기될 수 있습니다.
        
    - 회전 축의 결합 방식에 따라 같은 3D 회전을 표현하는 오일러 각 조합이 무수히 많기 때문에 발생하는 정상적인 현상입니다.
        

### 2. 회전 값에 대한 올바른 처리

- **`eulerAngles` 직접 조작의 한계**:
    
    - `transform.eulerAngles += new Vector3(0, 10, 0);`처럼 매 프레임 오일러 각을 직접 더하는 방식은 각도가 0도와 360도를 넘나들 때 튀는 현상이나 보간 오류를 유발하기 쉽습니다.
        
- **권장 처리 방식**:
    
    - **회전 누적/연산**: 쿼터니언 곱셈(`*`)을 사용합니다.
        
        C#
        
        ```
        // y축 기준으로 30도 회전 누적
        transform.rotation *= Quaternion.Euler(0, 30f, 0);
        ```
        
    - **부드러운 회전 보간**: 오일러 각 대신 `Quaternion.Slerp`를 사용합니다.
        
        C#
        
        ```
        transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, Time.deltaTime * speed);
        ```
        
    - **간단한 상대 회전**: `transform.Rotate(Vector3.up * speed * Time.deltaTime)`를 사용하는 것이 안전합니다.


### 오일러 각과 짐벌락(Gimbal Lock) 문제

- **짐벌락의 발생 원인**:
    
    - 오일러 각 회전은 한 번에 3개 축을 동시에 돌리는 것이 아니라, 특정 순서(유니티 기준: $Z \rightarrow X \rightarrow Y$)로 축을 순차적으로 회전시킵니다.
        
    - 이 과정에서 두 번째 회전축($X$축)이 $\pm 90^\circ$ 회전하여 첫 번째 축($Z$축)과 세 번째 축($Y$축)이 동일 평면상에 겹치게 되면, **3차원 회전 중 1개 자유도를 잃어버리는 현상**이 발생합니다.
        
- **영향**:
    
    - 짐벌락에 빠지면 특정 방향으로의 회전 제어가 불가능해지거나, 회전 애니메이션 보간 시 원치 않는 축으로 360도 꼬여버리는 오작동이 일어납니다.
        
- **해결책**:
    
    - **쿼터니언(사원수) 사용**: 쿼터니언은 축 순서에 의존하지 않고 4차원 복소수 공간에서 회전을 단일 축-각도 방식으로 표현하므로 짐벌락이 원천적으로 발생하지 않습니다.
        

- 유니티의 축의 의존성 

![[Pasted image 20261012111734.png]]

- Y X Z 순으로 부모 자식 관계라 생각하자 Y를 움직이면 X, Z 모두 영향
	Z만 움직이면 영향이 없다 자식 혼자 움직인게 되니까

- 축을 돌리는 순서는 아니다 축의 부모관계일 뿐

![[Pasted image 20261012112110.png]]

- x를 돌리다 보면z축이 Y축과 똑같아 져버리는 순간이 옴 이때는 축이 그냥 2개밖에안되는 순간이 온거임 Z, Y 축이 구분이 없어져버림

**이때 발생하는게 짐벌락이다 **
**축이 겹쳐지는 현상으로 인해 하나의 각도를 표현하는게 2개 이상이 된다(축이 의존해서 영향을 받으니까 축의 양의 방향 음의 방향이 뒤집어지고) **


### 오일러 vs 쿼터니언

| **구분**                    | **오일러 각 (Euler Angles)**       | **쿼터니언 (Quaternion)**                |
| ------------------------- | ------------------------------ | ------------------------------------ |
| **구성 요소**                 | 3차원 벡터 $(X, Y, Z)$             | 4차원 복소수 $(x, y, z, w)$               |
| **회전 방식**                 | 각 축($X, Y, Z$)을 정해진 순서대로 순차 회전 | 임의의 회전축 벡터와 해당 축 기준 회전각으로 표현         |
| **직관성**                   | 직관적 (각도 단위로 사람이 이해하기 쉬움)       | 비직관적 (수학적 4원수 체계로 직관적 해석 어려움)        |
| **짐벌락 위험**                | **발생함** (특정 각도에서 회전 자유도 1개 상실) | **발생하지 않음**                          |
| **회전 보간 (Interpolation)** | 각도 간 보간 시 축 꼬임 및 부자연스러운 궤적 발생  | `Slerp`(구면 선형 보간)를 통해 최단 경로로 부드럽게 보간 |
| **메모리 / 연산량**             | float 3개 (적은 메모리)              | float 4개 (정규화 등 추가 연산 필요)            |


★ **정리 하자면 오일러각과 쿼터니언 등 각도의 값을 720,540 등으로 넣어도 360도 한바퀴에 대한 정보 밖에 없기 때문에 각도의 값이 중요하다면 각도의 값을 최종적으로 대입하기 이전에 값을 따로 캐싱해서 처리한 뒤 대입하는게 중요하다**


### 그래서 쿼터니언을 쓰자
- transform.Roatate(Vector3.right); // 쿼터니언 기반으로 돔
- transform.Roatate(Vector3.right, Space); // 쿼터니언 기반으로 돔
- 쿼터니언을 써도 일괄적으로 증감시키는 방식은 축의 영향을 주긴함

### Quaternion.Euler 함수

`Quaternion.Euler`는 유니티에서 **사람이 이해하기 쉬운 오일러 각도(Degree 단위의 $X, Y, Z$)를 엔진 내부 연산용 쿼터니언(Quaternion) 구조체로 변환**해 주는 함수.

쿼터니언을 직접 인스턴스화하거나 복잡한 수식을 계산하지 않고도 직관적인 각도로 회전 값을 생성할 때 가장 많이 사용

### 1. 문법 및 기본 사용법


C#

```
// 1. float 값 3개를 직접 전달
Quaternion rotationA = Quaternion.Euler(xAngle, yAngle, zAngle);

// 2. Vector3 형태로 전달
Quaternion rotationB = Quaternion.Euler(new Vector3(xAngle, yAngle, zAngle));
```

**기본 예시:**

C#

```
void Start()
{
    // Y축 기준으로 90도 회전한 쿼터니언 생성 후 오브젝트에 적용
    transform.rotation = Quaternion.Euler(0f, 90f, 0f);
}
```

### 2. 내부 회전 축 적용 순서

`Quaternion.Euler(x, y, z)`는 축을 동시에 돌리지 않고, 유니티의 기본 오일러 회전 순서인 **Z축 $\rightarrow$ X축 $\rightarrow$ Y축 순서**로 축을 연속 회전하여 최종 쿼터니언을 합성합니다.

1. **Z축(Roll)** 기준 회전 적용
    
2. **X축(Pitch)** 기준 회전 적용
    
3. **Y축(Yaw)** 기준 회전 적용
    

> **참고:** 이 축 적용 순서 때문에 X축 각도가 $\pm 90^\circ$에 도달할 때 오일러 특유의 짐벌락(Gimbal Lock) 위험이 발생할 수 있습니다.

### 3. 실무 활용 패턴

#### 1) 기존 회전에 추가 회전 합성 (쿼터니언 곱셈)

회전을 누적할 때는 덧셈(`+`)이 아닌 쿼터니언 곱셈(`*`)을 사용해야 합니다.

C#

```
// 현재 회전 상태에서 추가로 Y축 기준 45도 회전
transform.rotation = transform.rotation * Quaternion.Euler(0f, 45f, 0f);
```

#### 2) 부드러운 회전 보간 (`Slerp`)

목표 각도를 `Quaternion.Euler`로 생성한 뒤 보간합니다.

C#

```
[SerializeField] private float targetYAngle = 180f;
[SerializeField] private float rotateSpeed = 5f;

void Update()
{
    Quaternion targetRotation = Quaternion.Euler(0f, targetYAngle, 0f);
    transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, Time.deltaTime * rotateSpeed);
}
```

#### 3) FPS 카메라 상하/좌우 각도 제한 (Clamp)

카메라 회전 시 `transform.eulerAngles`에 값을 직접 더하지 않고, 오일러 누적 변수를 `Mathf.Clamp`로 제한한 뒤 매 프레임 `Quaternion.Euler`로 재할당하는 방식이 표준적입니다.

C#

```
private float pitch = 0f; // 상하 회전각

void Update()
{
    float mouseY = Input.GetAxis("Mouse Y");
    pitch -= mouseY * 2f;
    pitch = Mathf.Clamp(pitch, -80f, 80f); // 고개 꺾임 한계 설정

    // 계산된 각도로 카메라 로컬 회전 갱신
    transform.localRotation = Quaternion.Euler(pitch, 0f, 0f);
}
```

### 4. 주의해야 할 점

- **단위는 라디안(Radian)이 아닌 도(Degree)**: 라디안 변환 없이 각도 수치(예: $0 \sim 360$)를 그대로 넘겨야 합니다.
    
- **`transform.eulerAngles`와의 차이**:
    
    - `transform.eulerAngles` 프로퍼티 내부에서도 사실상 `Quaternion.Euler`와 역변환 연산이 일어납니다.
        
    - 하지만 `transform.eulerAngles.x += 10f`처럼 구조체 필드를 직접 수정하는 것은 허용되지 않으며, `transform.rotation = Quaternion.Euler(...)` 형태로 명시적으로 할당하는 것이 의도를 명확히 하고 잠재적 버그를 줄입니다.


![[Pasted image 20261012141418.png]]