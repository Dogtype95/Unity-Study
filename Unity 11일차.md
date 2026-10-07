

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




64~ 59 6
57~52 6
46 1
40~35 6