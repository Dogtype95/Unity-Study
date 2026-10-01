
### IDE 디버깅

디버깅을 위해서는 IDE툴과 연결이 필요한데 기본적으로 Unity Package와 VS코드나 라이더에 라이브러리가 설치되었는지 체크해야함

Unity에서는 우측하단 벌레모양으로 릴리즈와 디버그 모드를 전환할 수 있음
![[22.png]]

에디터를 켰을때 디버그 모드 릴리즈 모드인지 설정하고 싶다면 Preference세팅에서 설정할 수 있음

Preference -> General -> Code Optimization On StartUp 에서 설정
### Instantiate
```text
`Object.Instantiate`는 Unity에서 프리팹(Prefab)이나 기존 게임 오브젝트의 복제본(클론)을 런타임에 동적으로 생성하는 메서드

반대되는 개념으로 Destroy(Object);
함수가 있다.
```



- 유니티에서 오브젝트를 생성할 땐 기본적으로 New를 사용하지 않고 (불가능은 아님)
	기존 하이레키에 있던 오브젝트나 Prefab을 참조해서 생성한다.
	Script에 하이레키나 Prefab을 드래그 드롭 해서 변수에 담을 수 도있다
	
![[20.png]]

### 유니티 이벤트함수의 브로드캐스트 시스템

Unity에서 **이벤트 함수의 브로드캐스트(Broadcast) 시스템**은 특정 게임 오브젝트 계층(Hierarchy) 또는 씬 전체에 걸쳐 수신자의 명시적인 컴포넌트 타입 참조 없이 함수 이름을 기반으로 메시지를 일괄 전파·호출하는 메커니즘을 의미합니다.

크게 **① Unity 내장 메시지 전송 API(`SendMessage` 계열)**, **② UI EventSystem 기반의 계층 전파(`ExecuteEvents`)**, ③ 실무 표준 대안(C# Event/옵저버 패턴)으로 구분됩니다.

### 1. Unity 내장 메시지 전파 API

`GameObject` 및 `Component` 클래스에 내장된 메시지 전파 함수는 하이러키 계층 트리를 따라 탐색 방향이 다릅니다.

```
                  [Parent Object]
                         │
                   (SendMessageUpwards) ▲
                         │
                 [Target GameObject]  ◄─── (SendMessage: 대상 오브젝트의 컴포넌트들만)
                         │
                   (BroadcastMessage)  ▼
                         │
                  [Child Objects...]
```

- **`BroadcastMessage(string methodName, object value, SendMessageOptions)`:**
    
    - **전파 방향:** 대상 게임 오브젝트 본인과 **모든 자식(하위 계층) 오브젝트**의 모든 `MonoBehaviour` 컴포넌트로 메시지를 아래 방향(Downwards)으로 일괄 전송합니다.
        
    - 하위 트리에 붙은 특정 함수(예: `"ApplyExplosion"`, `"OnReset"`)를 한 번에 실행시킬 때 사용됩니다.
        
- **`SendMessage(string methodName, ...)`:**
    
    - **전파 범위:** 오직 **해당 게임 오브젝트 본인**에 붙어 있는 컴포넌트들에게만 메시지를 보냅니다 (자식이나 부모는 탐색하지 않음).
        
- **`SendMessageUpwards(string methodName, ...)`:**
    
    - **전파 방향:** 대상 게임 오브젝트 본인부터 부모 계층(Upwards)을 타고 루트까지 올라가며 메시지를 전달합니다.
        

#### 주요 옵션: `SendMessageOptions`

- `SendMessageOptions.RequireReceiver` (기본값): 메시지를 받는 컴포넌트나 해당 메서드가 하나도 없으면 콘솔에 에러(`SendMessage has no receiver!`)를 출력합니다.
    
- `SendMessageOptions.DontRequireReceiver`: 수신할 함수가 없어도 에러 없이 조용히 무시합니다.
    

C#

```
// 사용 예시: 보스 몬스터 오브젝트 하위의 모든 부속 파츠에 피격 메시지 일괄 전파
gameObject.BroadcastMessage("TakeDamage", 50, SendMessageOptions.DontRequireReceiver);
```

### 2. 내장 BroadcastMessage의 치명적인 한계점

유니티 초창기부터 지원된 레거시 기능이지만, 실무 프로덕션에서는 다음 문제들로 인해 사용을 엄격히 지양합니다.

1. **리플렉션(Reflection) 비용:** 문자열(String) 이름으로 런타임에 메서드를 탐색하므로 CPU 연산 부하가 매우 큽니다.
    
2. **타입 안정성 부재 (Type Safety):** 메서드 이름의 오타나 파라미터 타입 불일치가 컴파일 타임에 잡히지 않고 런타임 버그로 이어집니다.
    
3. **가비지 컬렉션(GC Alloc):** 기본값 타입(int, float 등)을 파라미터로 넘길 때 `object` 형변환으로 인한 박싱(Boxing)이 발생합니다.
    
4. **추적/디버깅 어려움:** IDE(Visual Studio, Rider)에서 "모든 참조 찾기"가 되지 않아 코드 유지보수성이 급격히 떨어집니다.