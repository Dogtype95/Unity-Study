
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
- Unity Input System 
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

### 쿼터니언 Quaternion

- 



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
- transform.eulerAngles 의 값을 수정해서 각도를 수정할 수 도있음 근데 아마
	transform.Rotate()를 쓰는 게 맞는 것 같음
