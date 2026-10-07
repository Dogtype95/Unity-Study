

### Coroutine
```Text
Corutine 함수는 유니티에서 사용하는 비동기 함수고 유니티에서 작성하는 C#스크립트단은 싱글스레드로 돌기 때문에 멀티쓰레드 환경의 비동기와는 다르다.
```

```cs
// 함수명 컨벤션은 Co를 접두사로 붙여주고나 Routine을 접미사로 붙이는 것
// IEnumerator를 반환형으로 작성해야함
// 유니티 엔진에서 StartCoroutine 함수에서 반복해주기 떄문에 이뉴머러블이 아닌
// 이뉴머 레이터로 돌리는 것
private IEnumerator CoMakeFriedPotato()
{

	yield return null;
	print("1. 기름 끓이기");
	yield return null;
	print("2. 감자 넣기");
	yield return null;
	print("3. 감자 뺴기");
}

private void Awake()
{
	StartCoroutine(CoMakeFriedPotato());
}

```
- Delayed Call Manager : StartCoroutine 함수를 등록하는 곳 

- yiled return의 반환형은 참조형밖에 못오기 때문에 값형을 둬서 박싱으로 인한 성능 문제를 걱정하지 않아도 된다.

## 코루틴 종료 및 중단 기준

코루틴은 다음 상황 중 하나를 만족하면 즉시 종료되거나 중단됩니다.

1. **메서드 완료:** 코루틴 내부 로직이 끝까지 실행되어 더 이상 `yield`할 항목이 없을 때.
    
2. **명시적 탈출 (`yield break`):** 실행 도중 조건을 만족하여 `yield break;`를 만났을 때.
    
3. **코루틴 명시적 정지:**
    
    - `StopCoroutine(coroutineReference)`: 특정 실행 인스턴스 정지.
        
    - `StopAllCoroutines()`: 해당 `MonoBehaviour` 컴포넌트에서 실행 중인 모든 코루틴 강제 정지.
        
4. **게임오브젝트 비활성화 (`SetActive(false)`):**
    
    - 컴포넌트(`enabled = false`)만 끄면 코루틴은 **계속 실행**됩니다.
        
    - 반면, **GameObject 자체가 비활성화되거나 파괴(`Destroy`)되면 실행 중이던 코루틴은 즉시 강제 종료**되며, 오브젝트를 다시 활성화해도 자동으로 재개되지 않습니다.
        
5. **씬 전환:** 씬이 언로드되면 씬 종속 오브젝트들이 파괴되면서 내부 코루틴도 종료됩니다 (`DontDestroyOnLoad` 오브젝트 제외).


### TIme Class

프레임률에 독립적인 물리 연산과 게임 로직을 구현할 때 필수적인 클래스입니다.

- `Time.deltaTime`: 이전 프레임 완료부터 현재 프레임까지 걸린 시간(초 단위). 60fps면 약 0.016초, 30fps면 약 0.033초입니다.
    
    - 이동 공식에 필수: `transform.position += direction * speed * Time.deltaTime;`
        
- `Time.time`: 게임 시작(씬 로드) 후 누적된 경과 시간.
    
- `Time.timeScale`: 게임 내 시간 흐름 배율 (기본값 `1.0f`).
    
    - `0`으로 설정하면 일시정지, `0.5f`는 슬로우 모션, `2.0f`는 2배속.
        
    - `Time.deltaTime`은 이 값의 영향을 받지만, `Time.unscaledDeltaTime`은 영향을 받지 않아 일시정지 중 UI 애니메이션 처리에 쓰입니다.
        
- `Time.fixedDeltaTime`: `FixedUpdate`가 실행되는 고정 간격 (기본값 `0.02f` = 초당 50회).

### start 함수

- 객체의 스크립트가 켜질 때 실행되는 함수로 OnEnable 이후에 바로 실행된다
	OnEnable가 스크립트가 준비 될때 실행된다면 Start는 스크립트 Start라고 이해해도 무방하다. (Awake는 개체 기준 Start는 컴포넌트 인스턴스 기준으로 실행)
	또는 Update Start라고 이해해도 무방
- Awake처럼 최초 한번만 실행 
- 활성화 될때 마다 돌려야 하는 로직이라면 OnEnable을 사용해야 함
- 모든 객체의 OnEnable 함수가 실행됨을 보장할 수 있기때문에 다른 객체를 참조해야하는 로직이 있다면 Start함수를 쓰는게 안전하다.

#### Awake와 Start를 구분한 이유
- 씬로드 중에 한개체의 Awake OnEnable이 끝났다고 모든 개체의 Awake와 OnEnable가 끝났다고 보장할 수 없기 떄문에 Start를 사용하면 모든 객체의 Awake OnEnable 함수가 끝났다고 보장할 수 있어 실행순서를 보장한다.

### Start 이전 Instantiate 실행
- 해당 오브젝트는 한 번에 Awake-Start까지 호출됨을 해당 프레임에 보장할 수 있다
### Start 이후 Instantiate 실행
- 해당 오브젝트는 Awake까지만 도달, 다음 프레임부터 오브젝트가 Start에 합류됨

### Math.Lerp(float x, float y, float t) (보간)
- x y 두지점의 중간점을 t값에 따라 얻어온다 (t값은 0~1 0이면 0% 1이면 100%)
	0.5라면 x,y 지점의 50% 지점을 얻어오게 된다
- Vector3 버전도 있음\


### Update 함수

### 1. 기본 특징 및 호출 주기

- **가변 호출 주기:** 물리 연산처럼 일정한 시간 간격으로 실행되는 것이 아니라, 기기의 연산 성능 및 렌더링 부하에 따라 호출 횟수가 달라집니다 (FPS에 종속적).
    
    - 60 FPS 환경: 1초에 약 60회 호출
        
    - 144 FPS 환경: 1초에 약 144회 호출
        
- **주요 용도:**
    
    - 사용자 입력 처리 (`Input.GetKeyDown`, 마우스 클릭 등)
        
    - 프레임 기반 타이머 및 경과 시간 누적
        
    - 비물리 기반의 이동, 회전, 상태 전환 등 매 순간 즉각 반응해야 하는 게임 로직
        

### 2. Time.deltaTime 활용 (필수)

기기마다 프레임률이 다르므로, 프레임에 비례하여 오브젝트를 이동시키면 고사양 기기에서 캐릭터가 더 빠르게 움직이는 치명적인 문제가 발생합니다. 이를 방지하기 위해 이전 프레임에서 현재 프레임까지 걸린 시간인 `Time.deltaTime`을 곱해 초당 이동량으로 보정해야 합니다.

C#

```
void Update()
{
    // 잘못된 방식: 60fps에선 초당 60 유닛, 144fps에선 초당 144 유닛 이동
    // transform.Translate(Vector3.forward * 1.0f);

    // 올바른 방식: 프레임률과 상관없이 1초에 5 유닛씩 등속 이동
    float speed = 5.0f;
    transform.Translate(Vector3.forward * speed * Time.deltaTime);
}
```

### 3. Update 패밀리 비교 (`Update` vs `FixedUpdate` vs `LateUpdate`)

유니티는 실행 목적에 따라 세 가지 Update 메서드를 분리해 제공합니다.

|**메서드**|**호출 주기**|**주 사용처**|**주의 사항**|
|---|---|---|---|
|**`Update`**|매 프레임 (가변)|입력 감지, 일반 타이머, UI 갱신, 카메라 외 로직|연산량이 많은 무거운 루프문이나 탐색(`Find`) 작성 금지|
|**`FixedUpdate`**|고정 시간 간격 (기본 0.02초)|`Rigidbody` 물리 연산 (`AddForce`, `velocity`)|순간 입력을 감지하면 입력을 씹거나 중복 처리할 수 있음|
|**`LateUpdate`**|매 프레임 (`Update` 완료 후)|카메라 추적, 조준 후 애니메이션 보정|타깃 오브젝트가 `Update`에서 이동을 끝낸 후 추적해야 화면 떨림이 없음|

### 4. 실무 최적화 팁

1. **빈 `Update()` 메서드는 삭제하기:**
    
    함수 내용이 비어 있더라도 스크립트에 `void Update() {}`가 선언되어 있으면 C++ 엔진 코어에서 C# 매니지드 영역으로 매 프레임 불필요한 P/Invoke 호출 오버헤드가 발생합니다.
    
2. **매 프레임 무거운 API 호출 피하기:**
    
    `FindObjectOfType`, `GameObject.Find`, `GetComponent` 등을 `Update` 안에서 매 프레임 호출하면 심각한 성능 저하가 발생합니다. `Awake`나 `Start`에서 미리 캐싱해 두고 사용해야 합니다.
    
3. **가비지 컬렉션(GC) 유발 코드 자제:**
    
    `Update` 내에서 `new` 키워드로 객체를 생성하거나 문자열 연결(`str + "abc"`)을 반복하면 GC 스파이크(프레임 드롭)의 주원인이 됩니다.


### 벡터와 스칼라
```text
- 프로그래밍에서 기본적으로 벡터란 숫자를 여러개 가진것
- 양의 숫자하나는 Scale에서 어원을 딴 스칼라(Scalar)라고 함
- Vector의 크기(Magnitude)
  
- 길이가 1인 벡터를 노멀라이즈드 벡터(Normalized Vector) 정규화된 벡터
	  또는 방향벡터라고도 함
  ex) (3,4) 좌표의 대각선의 길이는 5가 나오는데 이 대각선 길이를 1로만듬(나머지 변은 나누기 5하면 됨)
- 이를 이용해 방향벡터 x 스피드를 해주면 해당 프레임에 가야되는 이동량을 구할 수 있다.
  
```

- 정규화된 벡터는 아래 느낌
![[Pasted image 20261007153347.png|346]]

- 정리하자면 정규화된 벡터 Nomalized Vector는 방향벡터라 불리고 방향을 표현, 점벡터는	위치를 표현한다.
- 단위 벡터(스피드를 포함)는 방향벡터에 일정한 스피드량을 곱해준다면 스피드와 방향 둘다 나타낼수도 있는거다.

### Lock View to Selected(Shift + F)
- 하이레키에 있는 객체에 shift + f 를 누르면 선택한 대상에 뷰가 고정된다.

### Translate 
- transfrom.position

 - 트랜스폼을 숏컷으로 가져오고 position 프로퍼티의 벡터값을 수정하면 객체의
	 위치를 바꿀수 있다 Vector 클래스에는 Vector.up, Vector.down 등 방향벡터도 얻어올 수 있다.
	 ![[Pasted image 20261007161834.png]]
```text
또는 transform.Translate(Vector3(0,1,0)); 이런식으로 이동
```
### Rotate

- transform.rotate()
- transform.rotation

-  Space.World(월드기준 좌표) , Space.Self(객체 기준 자기자신)으로 보겠다는 것 transform.Translate(Vector3 a, Space space); 함수에서 기준점을 정해주는 용도의 값
![[Pasted image 20261007163703.png]]

```text
로컬기준 즉 Space.Self로 y방향으로 이동하면 자기자신의 Y방향으로 이동한다.
이미지 처럼
```

### 델타타임과 프레임 (DeltaTime, Frame)

- DeltaTIme은 시간의 변화를 말하며 이전프레임과 다음프레임까지 도달하기까지 연산시간이 얼마나 걸렸는지를 의미함 프레임과 프레임사이의 타임이 델타타임
- Time.DeltaTime 을 통해 얻을 수도 있다.

- Application.targetFrameRate = 1; <- 게임에 프레임제한을 강제로 건것 이렇게 하면 1초당 1프레임이 돌아간다.

- 1초당 1도 만큼 각도를 움직이고 싶다면
```cs

// 1프레임마다 1도 x 누적시간  누적시간이 1초가 되면 1도 x 1초 시간별로 움직일 수 있게 되는거다
transform.Rotate(Vector3.up * Time.deltaTime);

transform.Translate(Vector3.up * Time.deltaTime);
```