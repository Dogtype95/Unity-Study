
### enabled vs activeSelf/activeInHierarchy

Unity에서 `enabled`와 `active` (정확히는 `activeSelf` / `activeInHierarchy`)는 제어하는 대상의 계층(컴포넌트 vs 게임오브젝트)과 동작 방식에서 명확한 차이가 있습니다.

| **구분**    | **enabled**                                | **activeSelf / activeInHierarchy**    |
| --------- | ------------------------------------------ | ------------------------------------- |
| **제어 대상** | **Component** (스크립트, Collider, Renderer 등) | **GameObject** (오브젝트 자체)              |
| **개념**    | 컴포넌트의 기능 켜기/끄기 (체크박스)                      | 씬 내 게임오브젝트의 존재 활성화/비활성화               |
| **설정 방법** | `component.enabled = true / false;`        | `gameObject.SetActive(true / false);` |
| **확인 방법** | `component.enabled` (읽기/쓰기 가능)             | `acti`                                |


`Destroy`와 `DestroyImmediate`의 가장 핵심적인 차이는 **객체가 메모리 및 씬에서 실제로 파괴되는 시점**과 사용 목적(런타임 게임 플레이 vs 에디터 스크립팅)에 있습니다.

| **구분**       | **Destroy(obj)**                 | **DestroyImmediate(obj)**           |
| ------------ | -------------------------------- | ----------------------------------- |
| **파괴 시점**    | **지연 삭제** (현재 프레임의 모든 업데이트 종료 후) | **즉시 삭제** (코드 호출 순간 동기적으로 삭제)       |
| **사용 환경**    | **게임 플레이 런타임** (일반적인 인게임 로직)     | **에디터 스크립트** (`Editor`, `EditMode`) |
| **지연 시간 옵션** | 가능 (`Destroy(obj, 2.0f);`)       | 불가능                                 |
| **참조 안전성**   | 안전 (해당 프레임의 다른 스크립트 실행 보호)       | 위험 (같은 프레임 내 후속 코드에서 널 참조 에러 유발 가능) |

### 1. `Destroy` (권장: 일반 런타임 게임 플레이)

호출 즉시 오브젝트를 지우지 않고, **현재 프레임의 모든 렌더링/업데이트 주기가 끝난 시점**에 한꺼번에 파괴합니다.

- **동작 원리:**
    
    - `Destroy(gameObject)`가 실행되어도 같은 프레임의 다음 코드나 다른 오브젝트의 `Update()`가 끝날 때까지는 메모리에 남아 있습니다.
        
    - C# 레벨에서는 즉시 가짜 `null` 상태(Unity 특유의 오버로딩된 `null` 체크)로 플래그가 세워지지만, 내부 엔진 C++ 객체 파괴와 `OnDestroy()` 호출은 프레임 끝에 안전하게 처리됩니다.
        
- **장점:**
    
    - 한 프레임 내에서 여러 스크립트가 해당 오브젝트를 참조하고 있을 때, 중간에 사라져서 발생하는 예기치 못한 크래시나 참조 예외를 방지합니다.
        
    - `Destroy(bullet, 3.0f);` 처럼 타이머를 두어 일정 시간 뒤 삭제하는 기능이 기본 지원됩니다.
        

### 2. `DestroyImmediate` (권장: 에디터 툴 제작 시)

코드가 실행되는 **그 즉시(동기적으로)** 씬과 메모리에서 오브젝트를 파괴합니다.

- **동작 원리:**
    
    - 함수가 호출된 그 라인에서 C++ 엔진 객체까지 완전히 삭제되며, `OnDestroy()`도 그 즉시 실행됩니다.
        
- **사용하는 이유:**
    
    - 유니티 에디터 편집 모드(플레이 버튼을 누르지 않은 상태)에서는 프레임 업데이트 루프가 계속 돌지 않기 때문에, 일반 `Destroy`를 쓰면 객체가 제때 지워지지 않고 에러가 발생합니다.
        
    - 따라서 커스텀 인스펙터, 에디터 윈도우 스크립트 등 에디터 모드에서 생성한 프리팹이나 에셋을 정리할 때 반드시 사용합니다.
        
- **주의점 (런타임 사용 금지):**
    
    - 일반 게임 실행 중 `DestroyImmediate`를 호출하면 콘솔에 경고(`Destroying GameObjects immediately is not permitted to avoid data corruption...`)가 발생하거나 물리/렌더링 루프 중에 객체가 증발해 버그가 생길 수 있습니다.
    - 그리고 Fake Null은 Unity에서 쓰는 개념이기에 당연히 ?문법으로 Null체크를 해 감지가 안된다.(연산자 오버로딩을 게임오브젝트단에서 정의했기 때문에 FakeNull을 쓸 수 없기 떄문 )
    - Action, Func 등 델리게이트는 게임오브젝트가 아니기 때문에 Action?.InVoke()
	    같이 사용해도 괜찮다.

### 핵심 요약

- **게임 플레이 중 몬스터, 총알, UI 등을 없앨 때:** 무조건 **`Destroy()`** 사용
    
- **에디터 확장 툴, 커스텀 에디터 스크립트 작성 중 에셋/오브젝트를 지울 때:**



### Instance ID(Entity ID)

- Instance ID는 런타임 도중에 인스턴스화 된 개체의 ID다.

- UnityEngine.Object 소속의 GetInstanceID 는 C#의 GetHashCode와 대응하는 함수다.
개체끼리 비교할때는 GetInstanceID로 비교하면 빠르게 비교할 수 있다.

- 에디터 인스펙터 상의 EntityId와 같은 것임

### 컴포넌트의 숏컷 멤버
- name 프로퍼티
- transform 프로퍼티
- GetComponent 메서드

### Find 계열 함수 3개
- Object.FindAnyObjectByType : (Object.FindObjectOfType) : 타입 기반
- GameObject.Find : 이름 기반 탐색
- Transform.Find : 자식 개체도 찾고 싶을때 


Unity에서 오브젝트나 컴포넌트를 탐색할 때 사용하는 `Find` 계열 함수는 크게 4가지가 있습니다. 각 함수는 탐색 기준(이름, 경로, 타입, 태그)과 성능 특성이 다릅니다.

|**함수**|**탐색 기준**|**비활성화 오브젝트 탐색 여부**|**연산 비용**|**주 사용 목적**|
|---|---|---|---|---|
|**`GameObject.Find`**|전체 씬 내 **이름/경로**|❌ 불가|매우 높음|씬 전역에서 고유 이름으로 탐색|
|**`Transform.Find`**|직계 자식의 **이름/경로**|⭕ **가능**|낮음|특정 부모 아래 자식 오브젝트 탐색|
|**`FindFirstObjectByType`**|**타입(클래스)**|기본 ❌ (옵션으로 ⭕ 가능)|높음|매니저 등 싱글톤 객체 1개 탐색|
|**`GameObject.FindWithTag`**|**태그(Tag)**|❌ 불가|중간|Player 등 태그 지정 객체 탐색|

### 1. `GameObject.Find` (이름으로 전역 탐색)

씬의 루트부터 전체 계층 구조를 순회하여 지정한 이름의 게임오브젝트를 찾습니다.

C#

```
// 씬 전체에서 이름이 "BossMonster"인 오브젝트 검색
GameObject boss = GameObject.Find("BossMonster");

// 슬래시(/)를 사용해 경로로 검색 가능
GameObject sword = GameObject.Find("Player/WeaponHolder/Sword");
```

- **특징:**
    
    - **비활성화된 오브젝트는 찾지 못합니다.**
        
    - 씬의 오브젝트가 많을수록 전체를 순회하고 문자열을 비교하므로 연산 비용이 매우 큽니다.
        
    - 오브젝트 이름이 씬 안에서 바뀌면 곧바로 `NullReferenceException`이 발생합니다.
        

### 2. `Transform.Find` (자식 계층 탐색)

호출한 오브젝트의 **자식 계층 구조 내에서만** 탐색합니다.

C#

```
// 현재 오브젝트의 직계 자식 중 "Hand" 탐색
Transform hand = transform.Find("Hand");

// 하위 경로 지정 가능
Transform ring = transform.Find("Hand/Finger/Ring");
```

- **특징:**
    
    - **자식 오브젝트가 비활성화(`SetActive(false)`)되어 있어도 찾아냅니다.**
        
    - 전체 씬을 뒤지지 않고 자식 목록만 검사하므로 `GameObject.Find`보다 훨씬 빠릅니다.
        
    - 단, 경로 없이 이름만 입력할 경우 손자/증손자 계층은 건너뛰고 직계 자식 중에서만 찾습니다.
        

### 3. `FindFirstObjectByType<T>` / `FindAnyObjectByType<T>` (타입 탐색)

지정한 컴포넌트 타입을 가진 오브젝트를 탐색합니다. (구 버전의 `FindObjectOfType<T>` 대체)

C#

```
// 씬에 존재하는 GameManager 컴포넌트 탐색
GameManager gm = Object.FindFirstObjectByType<GameManager>();

// 비활성화된 오브젝트까지 포함하여 탐색할 때
GameManager inactiveGm = Object.FindFirstObjectByType<GameManager>(
    FindObjectsInactive.Include
);
```

- **`FindFirst` vs `FindAny`:**
    
    - `FindFirstObjectByType`: 여러 개가 있을 때 계층 구조상 가장 앞선 객체를 확정적으로 반환합니다.
        
    - `FindAnyObjectByType`: 순서 상관없이 아무거나 1개를 빠르게 반환합니다. (싱글톤 매니저처럼 씬에 딱 하나만 존재하는 경우 권장)
        
- **특징:** Unity 2023부터 `FindObjectOfType`이 Deprecated되고 이 두 함수로 세분화되었습니다.
    

### 4. `GameObject.FindWithTag` (태그 탐색)

Unity의 Tag 시스템을 기반으로 오브젝트를 찾습니다.

C#

```
// "Player" 태그를 가진 오브젝트 1개 탐색
GameObject player = GameObject.FindWithTag("Player");

// "Enemy" 태그를 가진 모든 오브젝트를 배열로 가져오기
GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
```

- **특징:**
    
    - `GameObject.Find(name)`보다 내부 인덱싱 처리가 되어 있어 상대적으로 빠릅니다.
        
    - 비활성화된 오브젝트는 탐색하지 못합니다.
        

### 실무 최적화 주의점

1. **`Update()`, `FixedUpdate()`에서 호출 금지:**
    
    - 매 프레임 수천 개의 계층 구조를 순회하게 되어 심각한 프레임 드롭(GC 및 CPU 스파이크)을 유발합니다.
        
    - 반드시 `Awake()`나 `Start()`에서 1회만 호출해 변수에 캐싱해 두어야 합니다.
        
2. **최우선 권장 방식 (인스펙터 할당 & 의존성 주입):**
    
    - 코드로 동적 탐색(`Find`)을 남발하기보다는, `[SerializeField]`를 선언해 인스펙터에서 직접 드래그 앤 드롭으로 연결하는 것이 런타임 비용 0이자 안정적인 구조입니다.

### transform.SetAsLastSibling()


`transform.SetAsLastSibling()`은 해당 게임오브젝트를 부모 오브젝트의 **자식 목록(계층 구조)에서 가장 마지막 순번(맨 아래)으로 이동**시키는 함수입니다.

주로 UI 렌더링 순서(가장 앞에 표시하기)나 **레이아웃 정렬**을 제어할 때 핵심적으로 사용됩니다.

### 1. 주요 동작 원리

Unity 인스펙터의 **Hierarchy(계층 구조) 창**에서 오브젝트를 마우스로 끌어 자식들 중 맨 밑으로 옮기는 것과 동일하게 작동합니다.

- **UGUI 렌더링 순서 결정:**
    
    - Unity의 UI(Canvas)는 기본적으로 **위에서 아래 순서**로 렌더링합니다.
        
    - 계층 구조에서 **아래쪽에 있을수록 나중에 그려지므로, 화면상에서는 가장 위에(앞에)** 보이게 됩니다.
        
    - 따라서 `SetAsLastSibling()`을 호출하면 해당 UI 요소가 겹쳐 있는 다른 UI들을 덮고 **맨 앞으로 올라옵니다.**
        

### 2. 대표적인 사용 사례

- **팝업 창 열기 / 탭 전환:**
    
    - 여러 개의 창(인벤토리, 상점 등)이 겹쳐 있을 때, 클릭한 창을 화면 가장 앞으로 가져올 때 사용합니다.
        
    
    C#
    
    ```
    public void OpenPopup()
    {
        gameObject.SetActive(true);
        // 부모 UI 내에서 맨 아래로 내려 화면 맨 앞에 렌더링
        transform.SetAsLastSibling();
    }
    ```
    
- **마우스 드래그 앤 드롭 아이템:**
    
    - 인벤토리 슬롯에서 아이템 아이콘을 드래그할 때 다른 슬롯에 가려지지 않도록 드래그 시작 시 맨 앞으로 올립니다.
        
    
    C#
    
    ```
    public void OnBeginDrag(PointerEventData eventData)
    {
        transform.SetAsLastSibling();
    }
    ```
    
- **수직/수평 레이아웃 그룹 (Layout Group):**
    
    - `VerticalLayoutGroup`이나 `HorizontalLayoutGroup`을 사용할 때 새로 추가된 요소를 리스트의 맨 끝으로 보낼 때 사용됩니다.
        

### 3. 관련 Sibling 메서드 세트

자식 순서를 다루는 연관 함수 4가지입니다.

| **메서드**                          | **동작**                                         |
| -------------------------------- | ---------------------------------------------- |
| **`SetAsLastSibling()`**         | 자식 목록의 **맨 마지막**으로 이동 (UI 화면 **맨 앞**)          |
| **`SetAsFirstSibling()`**        | 자식 목록의 **맨 처음(인덱스 0)**으로 이동 (UI 화면 **맨 뒤/배경**) |
| **`SetSiblingIndex(int index)`** | 원하는 특정 순번(0부터 시작)으로 직접 지정                      |
| **`GetSiblingIndex()`**          | 현재 자신이 몇 번째 순번인지 인덱스 값 반환                      |



### Component의 상속구조
![[Pasted image 20261002163529.png]]

![[Pasted image 20261002164952.png]]

### Object -> 리소스 에셋

![[Pasted image 20261002165337.png]]