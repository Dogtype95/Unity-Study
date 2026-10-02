
### UPM( Unity Package Manager)

- 유니티 프로젝트 공유에 필요한 유니티 폴더목록은 3개다.
```text
Assets
Packages
ProjectSettings
```
![[Pasted image 20260930142351.png]]


***

### Unity Scripts 

- Unity C# 파일은 MonoBehaivior를 상속받아야 컴포넌트가 되며 오브젝트에 부착이 가능해진다.


Unity `MonoBehaviour` 라이프사이클(Life Cycle)의 가장 핵심이 되는 세 이벤트 함수의 호출 시점, 역할, 실무 활용 차이점입니다.

### 1. Awake()

- **호출 시점:** 스크립트 인스턴스가 로드될 때 **단 한 번** 호출됩니다.
    
- **실행 조건:** 게임 오브젝트(GameObject)가 활성화(`Active`) 상태라면, **컴포넌트(Script) 활성화 체크박스가 꺼져(`enabled = false`) 있어도 무조건 실행**됩니다. (단, 게임 오브젝트 자체가 비활성화되어 있으면 켜지는 순간 실행됩니다.)
    
- **주요 용도:**
    
    - 자기 자신 내부의 컴포넌트 캐싱 (`GetComponent<Rigidbody>()`, `GetComponent<Animator>()` 등)
        
    - 싱글톤(Singleton) 인스턴스 초기화 (`instance = this;`)
        
    - 다른 오브젝트와 상관없는 독립적인 변수 및 데이터 구조 할당
        
- **주의점:** 다른 스크립트의 `Awake`가 실행되기 전일 수 있으므로, 다른 오브젝트의 상태를 조회하거나 참조하는 코드는 `NullReferenceException`이 발생할 위험이 있습니다.

★Awake는 생성자와 비슷한 느낌
★부모자식 관계에서 Awake를 호출하면 자식이 호출됨
★어웨이크는 우리가 부착한 컴포넌트 기준으로 호출된다.
★오브젝트 기준
★ 어웨이크는 객체별로 돌기 때문에 다른 객체의 Awake와는 순서가 보장이 안되는걸 항상 염두해야 한다.
****** 

★★★OnEnable은 아래에 정리
순서는 Awake() OnEnable() Start() 순서임

### 2. Start()

- **호출 시점:** 첫 번째 프레임 업데이트(`Update`)가 실행되기 직전에 **단 한 번** 호출됩니다.
    
- **실행 조건:** 게임 오브젝트와 해당 컴포넌트(Script)가 모두 활성화(`enabled = true`)되어 있어야 실행됩니다.
    
- **주요 용도:**
    
    - 모든 오브젝트의 `Awake`가 이미 끝난 시점이므로, **외부 오브젝트나 다른 컴포넌트 간의 상호작용/참조 초기화**에 안전합니다.
        
    - 예: `Awake`에서 플레이어가 체력 컴포넌트를 캐싱해 두면, UI 매니저는 `Start`에서 플레이어의 현재 체력을 읽어와 슬라이더 바에 반영.
        
- **주의점:** 오브젝트가 생성된 첫 프레임에 비활성화 상태였다가 나중에 켜지면, 켜지는 그 시점에 `Start`가 늦게 실행됩니다.
    

### 3. Update()

- **호출 시점:** 매 프레임마다 한 번씩 계속 호출됩니다.
    
- **실행 조건:** 게임 오브젝트와 컴포넌트가 모두 활성화되어 있는 동안 지속 실행됩니다.
    
- **주요 용도:**
    
    - 키보드/마우스 입력 감지 (`Input.GetKeyDown`, `Input.GetAxis`)
        
    - 논-물리(Non-Physics) 기반 일반 타이머, 이동 로직
        
    - 실시간 상태 체크 및 회전 연출
        
- **주의점:**
    
    - 하드웨어 성능 및 렌더링 부하에 따라 초당 프레임 수(FPS)가 변동되므로, 거리를 이동하거나 시간을 계산할 때는 프레임 보정 계수인 `Time.deltaTime`을 반드시 곱해주어야 기기 사양과 무관하게 일정한 속도가 유지됩니다.
        
    - 물리 연산(Rigidbody 힘 적용, 충돌 감지)은 프레임 레이트와 무관하게 고정 주기로 실행되는 `FixedUpdate()`에서 처리해야 합니다.
        

### 핵심 요약 비교

|**함수**|**호출 빈도**|**스크립트 비활성화 시 실행 여부**|**주 실행 목적**|
|---|---|---|---|
|**Awake**|1회 (생성/로드 즉시)|**실행됨** (오브젝트가 켜져 있다면)|자체 컴포넌트 캐싱, 싱글톤 선언|
|**Start**|1회 (첫 업데이트 직전)|**실행 안 됨** (컴포넌트 켜질 때까지 대기)|타 오브젝트 참조, 초기 데이터 연동|
|**Update**|매 프레임 반복|**실행 안 됨**|사용자 입력 처리, 타이머, 이동 연산 (`Time.deltaT`|

- ★유니티 MonoBehavior 생명주기 실행순서★
![[Pasted image 20260930152551.png]]

기본적으로 Awake() -> OnEnable() -> Start() 순서로 호출
Awake : 오브젝트의 생성자같은놈
OnEnable : 스크립트의 생성자 같은놈
Start : 스크립트 시작

### 1. OnEnable()

오브젝트나 스크립트가 **활성화(Active / Enabled) 상태가 되는 즉시 호출**됩니다.

- **호출 시점:**
    
    - 씬 시작 시: `Awake()` 직후, `Start()` 직전에 1회 실행됩니다.
        
    - 런타임 도중:
        
        - `gameObject.SetActive(true)`로 비활성 오브젝트가 켜질 때
            
        - 스크립트 컴포넌트 체크박스를 켤 때 (`enabled = true`)
            
        - `Instantiate`로 생성된 프리팹이 활성화 상태일 때
            
- **주요 용도:**
    
    - **C# 이벤트 및 델리게이트 구독:** `GameEvents.OnPlayerDead += HandleDeath;`
        
    - **오브젝트 풀링(Object Pooling) 재사용 초기화:** 몬스터나 탄환이 풀에서 꺼내져 활성화될 때 체력, 이동 방향, 타이머 등을 리셋
        
    - **코루틴 시작:** 활성화와 동시에 동작해야 하는 연출 루프 실행
        
    - **UI 컴포넌트 갱신:** 인벤토리나 팝업 창이 화면에 열리는 순간의 데이터 최신화
        

### 2. OnDisable()

오브젝트나 스크립트가 **비활성화(Inactive / Disabled) 상태가 되는 즉시 호출**됩니다.

- **호출 시점:**
    
    - `gameObject.SetActive(false)`로 오브젝트를 끌 때
        
    - 스크립트 컴포넌트 체크박스를 끌 때 (`enabled = false`)
        
    - `Destroy()`로 파괴될 때 (`OnDestroy` 직전에 먼저 호출)
        
    - 씬이 전환되어 기존 씬의 오브젝트들이 언로드될 때
        
    - 애플리케이션 종료 시
        
- **주요 용도:**
    
    - **C# 이벤트 구독 해제 (메모리 누수 방지 필수):** `GameEvents.OnPlayerDead -= HandleDeath;`
        
    - **진행 중인 코루틴 및 트윈 애니메이션 정지:** 의도치 않은 백그라운드 연산 방지
        
    - **임시 데이터 정리:** 타이머 리셋, 상태 플래그 초기화

***
### 1. `Debug` 클래스 (로그 출력 및 시각화)

`UnityEngine.Debug`는 콘솔 창(Console Window)에 메시지를 띄우거나 씬 뷰(Scene View)에 레이/선을 그려 시각적으로 디버깅할 때 사용합니다.

C#

```
using UnityEngine;

public class DebugExample : MonoBehaviour
{
    void Start()
    {
        // 1. 일반 정보 로그
        Debug.Log("게임이 시작되었습니다.");

        // 2. 경고 로그 (노란색 느낌표 아이콘)
        Debug.LogWarning("체력이 낮습니다!");

        // 3. 에러 로그 (빨간색 정지 아이콘)
        Debug.LogError("플레이어 프리팹을 찾을 수 없습니다.");

        // 4. 특정 오브젝트 강조 (콘솔 더블클릭 시 하이러키에서 해당 오브젝트 포커스)
        Debug.Log("타깃 오브젝트 참조 확인", gameObject);

        // 5. 텍스트 서식 (Rich Text 지원)
        Debug.Log("<color=cyan><b>속도 증가:</b></color> 100%");
    }

    void Update()
    {
        // 6. 씬 뷰 시각화 (선 / 레이 그리기)
        // Transform 기준 전방으로 5m 빨간색 광선 표시 (1프레임 동안 유지)
        Debug.DrawRay(transform.position, transform.forward * 5f, Color.red);
        
        // 두 지점 사이 선 그리기 (지속 시간 duration 지정 가능)
        Debug.DrawLine(transform.position, Vector3.zero, Color.green, 2.0f);
    }
}
```

- **콘솔 하이라이트 기능:** `Debug.Log(message, context)`의 두 번째 인자에 `this` 또는 `gameObject`를 전달하면, 콘솔 창에서 해당 로그를 더블클릭했을 때 하이러키 창에서 해당 오브젝트를 즉시 하이라이트해 줍니다.
    
- **성능 주의점:** `Debug.Log`는 문자열 생성(String Allocation)으로 인한 가비지 컬렉션(GC Alloc)과 I/O 비용이 발생하므로, `Update` 내부에서 매 프레임 대량으로 호출하는 것은 피해야 합니다.
    

### 2. `Assert` 클래스 (가정 검증 및 단언문)

`UnityEngine.Assertions.Assert`는 "이 변수나 조건은 무조건 참(True)이어야 한다"는 가정을 검증하는 도구입니다. 조건이 거짓(`false`)이면 콘솔에 즉각 `AssertionException` 형태의 에러를 출력합니다.

C#

```
using UnityEngine;
using UnityEngine.Assertions; // Assert 사용을 위한 필수 네임스페이스

public class AssertExample : MonoBehaviour
{
    [SerializeField] private Rigidbody rb;
    private int playerHealth = 100;

    void Awake()
    {
        // 1. 참조 유효성 검증 (null이면 에러 출력)
        Assert.IsNotNull(rb, "Rigidbody 컴포넌트가 인스펙터에 할당되지 않았습니다!");
    }

    public void TakeDamage(int damage)
    {
        // 2. 입력값 범위 검증 (데미지는 0보다 커야 함)
        Assert.IsTrue(damage > 0, "데미지는 0보다 커야 합니다.");

        playerHealth -= damage;

        // 3. 음수 방지 논리 검증
        Assert.IsTrue(playerHealth >= 0, "체력은 음수가 될 수 없습니다.");
    }
}
```

#### 자주 쓰는 `Assert` 메서드

- **`Assert.IsTrue(condition, message)` / `Assert.IsFalse(...)`**: 조건식의 참/거짓 평가
    
- **`Assert.IsNotNull(obj, message)` / `Assert.IsNull(...)`**: 인스펙터 참조나 동적 할당의 `null` 여부 즉시 검증
    
- **`Assert.AreEqual(expected, actual)` / `Assert.AreNotEqual(...)`**: 기대치와 실제값 일치 여부 비교
    
- **`Assert.AreApproximatelyEqual(a, b, tolerance)`**: 부동소수점(`float`) 오차를 고려한 수치 비교
    

### 3. Debug와 Assert의 핵심 차이점

|**구분**|**Debug.Log 계열**|**Assert 클래스**|
|---|---|---|
|**목적**|값 확인, 이벤트 발생 추적, 데이터 흐름 모니터링|논리적 오류 사전 차단, 필수 참조/값의 유효성 강제|
|**조건 분기**|보통 `if`문 안에서 수동 호출|조건식 자체를 메서드 인자로 전달하여 자동 평가|
|**빌드 스트립 (제거)**|상용 릴리스 빌드에도 코드가 기본 포함됨 (스트립 설정 필요)|**`UNITY_ASSERTIONS` 기호 기반으로 자동 비활성화 가능**|
|**코드 가독성**|`if (rb == null) Debug.LogError(...);` (다소 김)|`Assert.IsNotNull(rb);` (단 한 줄로 간결화)|

### 4. 릴리스 빌드 시 로그 최적화 팁

상용 빌드(Release Build)에서 디버그 로그가 계속 출력되면 불필요한 CPU 성능 저하와 로그 파일 용량 증가가 발생합니다.

- **Assert 자동 제거:** `UnityEngine.Assertions.Assert`는 배포용 릴리스 빌드(`Development Build` 체크 해제 시)에서 호출 오버헤드가 자동으로 제거되거나 최소화됩니다.
    
- **Debug.Log 일괄 비활성화:** 게임 시작 진입점(예: 초기 로더의 `Awake`)에서 다음 코드를 적용하면 릴리스 환경에서 모든 일반 로그 출력을 차단할 수 있습니다.
    
    C#
    
    ```
    #if !UNITY_EDITOR && !DEVELOPMENT_BUILD
    Debug.unityLogger.logEnabled = false; // 빌드에서 로그 비활성화 (GC 방지)
    #endif
    ```

### Unity Assembly Definition

어셈블리 정의(Assembly Definition, asmdef)는 프로젝트 내 C# 스크립트들을 독립된 컴파일 단위(별도의 관리되는 C# 어셈블리 `.dll`)로 분할하여 컴파일 시간 단축, 모듈화, 의존성 제어를 달성하는 기능입니다.

### 1. 기본 컴파일 방식과 도입 배경

#### 기본 컴파일 (asmdef 미사용 시)

- Unity는 프로젝트 내 모든 스크립트를 기본적으로 `Assembly-CSharp.dll`이라는 단 하나의 거대한 어셈블리로 한꺼번에 묶어 컴파일합니다.
    
- **문제점:** 스크립트 하나에서 쉼표 하나만 수정해도 프로젝트 전체의 수백~수천 개 스크립트를 매번 다시 재컴파일하므로 프로젝트 규모가 커질수록 에디터 대기 시간이 극단적으로 늘어납니다.
    

#### Assembly Definition 적용 시

- 특정 폴더에 `asmdef` 파일을 생성하면, 해당 폴더와 하위 폴더의 모든 스크립트는 독립된 별도의 `.dll`로 컴파일됩니다.
    
- 해당 모듈을 수정할 때 **해당 어셈블리 및 이를 직접 참조하는 모듈만 다시 컴파일**되므로 컴파일 시간이 대폭 감소합니다.
    

### 2. Assembly Definition의 4가지 핵심 이점

1. **컴파일 시간 단축 (증분 컴파일 최적화):**
    
    - 코드가 자주 바뀌지 않는 공용 라이브러리, 코어 시스템, 플러그인을 독립 어셈블리로 분리해 두면 변경 사항이 생겨도 불필요한 재컴파일이 발생하지 않습니다.
        
2. **명시적 의존성 관리 및 스파게티 코드 차단:**
    
    - 어셈블리 간의 참조 관계를 명시적으로 등록해야만 서로의 타입을 사용할 수 있습니다.
        
    - **순환 참조(Circular Dependency) 방지:** A 어셈블리가 B를 참조하고, B가 다시 A를 참조하는 구조를 엔진 차원에서 강제로 에러로 차단하여 아키텍처 결합도를 낮춥니다.
        
3. **플랫폼별 컴파일 분리 (Platforms):**
    
    - 특정 모듈이 Standalone, Android, iOS 또는 `Editor` 환경에서만 빌드/실행되도록 인스펙터 체크박스로 타깃 플랫폼을 제한할 수 있습니다.
        
4. **단위 테스트(Unit Test) 분리:**
    
    - 유니티 테스트 러너(`UTF`) 전용 테스트 스크립트들을 별도 어셈블리로 묶어 상용 릴리스 빌드 바이너리에서 완전히 제외시킬 수 있습니다.
        

### 3. 생성 및 설정 방법

**1.asmdef 파일 생성:**

Project 창에서 분리할 스크립트들이 위치한 상위 폴더를 우클릭한 후 **Create → Scripting → Assembly Definition**을 선택합니다.

**2.어셈블리 이름 지정:**

생성된 파일의 이름을 모듈의 네임스페이스 규칙에 맞게 지정합니다 (예: `Game.Core`, `Game.UI`, `Game.Network`).

**3.인스펙터에서 의존성(References) 추가:**

해당 어셈블리 파일(`.asmdef`)을 선택하고 Inspector의 **Assembly Definition References** 항목에 이 모듈이 참조해야 할 다른 어셈블리를 드래그 앤 드롭으로 등록합니다.

**4.Apply 클릭:**

인스펙터 하단의 **Apply**를 누르면 컴파일이 다시 진행되고, 솔루션 파일(`.csproj`)에 해당 프로젝트가 독립적으로 분리 반영됩니다.

### 4. 주요 인스펙터 설정 항목

| **설정 항목**                | **설명 및 실무 권장 사항**                                                                                       |
| ------------------------ | ------------------------------------------------------------------------------------------------------- |
| **Name**                 | 어셈블리의 실제 식별 이름 (C# 프로젝트 및 빌드 결과 `.dll` 파일명)                                                             |
| **Auto Referenced**      | **체크 시** `Assembly-CSharp` 같은 기본 어셈블리가 이 모듈을 자동 참조합니다. 코어 라이브러리나 툴 모듈은 의존성을 명확히 하기 위해 체크를 해제하는 것이 좋습니다. |
| **No Engine References** | 순수 C# 로직(엔티티 모델, 수학 라이브러리 등)에서 `UnityEngine` 참조를 완전히 배제하여 엔진 의존성을 차단하고 컴파일을 극대화합니다.                     |
| **Platforms**            | Any Platform이 기본이며, 에디터 전용 툴은 `Editor`만 체크하여 런타임 빌드 스트립을 보장합니다.                                         |
| **Root Namespace**       | 해당 폴더 내에서 새로 생성하는 C# 스크립트에 기본으로 부여될 네임스페이스를 지정합니다.                                                      |

### 5. Assembly Definition Reference (asmref)

- **역할:** 이미 존재하는 `.asmdef` 어셈블리의 범위를 **물리적으로 다른 폴더에 있는 스크립트들까지 확장**할 때 사용합니다.
    
- **사용 예시:** 여러 폴더에 흩어져 있는 테스트 코드들을 하나의 Test 어셈블리 범위로 묶고 싶을 때, 각 폴더에 `asmref`를 두고 타깃 `asmdef`를 지정하면 물리적 폴더 구조를 재배치하지 않고도 동일 어셈블리로 취급됩니다.