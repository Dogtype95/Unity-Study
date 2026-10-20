
## Colider(강체, 충돌체)

- 충돌 범위를 보여주는 것 물리적인 연산은 없다 물리적인 중력 가속 연산은 리지드 바디

- 충돌을 감지하려면 서로 콜라이더가 있어야함
- 리지드 바디는 한쪽만 있어도 된다 충돌을 감지하기 위해서 리지드 바디는 움직이는 동적 오브젝트에 붙여주는게 좋다

Unity의 콜라이더(Collider)는 3D와 2D 물리 시스템으로 나뉘며, 연산 비용과 용도에 따라 기본 형태(원시 콜라이더)부터 **복잡한 메시 형태**, **특수 목적용**까지 다양하게 제공됩니다.

### 1. 3D 콜라이더

| **콜라이더**             | **형태**          | **연산 비용**      | **주요 용도 및 특징**                                                                                                                                                             |
| -------------------- | --------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Box Collider**     | 직육면체            | 매우 낮음          | 벽, 바닥, 상자, 문 등 각진 구조물                                                                                                                                                      |
| **Sphere Collider**  | 구(Sphere)       | 가장 낮음 (반경만 계산) | 공, 구체형 투사체, 방사형 피격 범위                                                                                                                                                      |
| **Capsule Collider** | 원기둥 + 반구        | 낮음             | **휴머노이드 캐릭터**, 기둥, 팔/다리 뼈대 (계단이나 턱을 넘기 유리함)                                                                                                                                |
| **Mesh Collider**    | 실제 3D 모델 메시 모양  | **매우 높음**      | 정밀한 지형이나 복잡한 고정 구조물.<br><br>  <br><br>• 기본은 오목(Concave) 구조로 정적(Static) 오브젝트에만 사용 가능.<br><br>  <br><br>• **`Convex` 체크 시 볼록한 형태로 단순화되어 Rigidbody와 충돌 가능 (최대 255개 트라이앵글 제한). |
| **Terrain Collider** | Unity 터레인 지형 전용 | 높음             | Unity Terrain 시스템의 높이맵(Heightmap) 기반 대규모 지형 충돌                                                                                                                             |
| **Wheel Collider**   | 차량 바퀴 전용        | 중간             | 서스펜션, 마찰력, 토크 등 차량 물리 시뮬레이션 전용 내장 콜라이더                                                                                                                                     |

### 2. 2D 콜라이더 (`Physics 2D`)

2D 게임에서는 `Collider2D` 계열 컴포넌트를 사용합니다.

|**콜라이더**|**주요 특징**|
|---|---|
|**Box Collider 2D**|2D 직사각형 충돌체. 바닥, 플랫폼, 블록에 주로 사용.|
|**Circle Collider 2D**|2D 원형 충돌체. 연산이 가장 가벼움.|
|**Capsule Collider 2D**|2D 캡슐 형태. 횡스크롤 캐릭터 이동에 적합.|
|**Polygon Collider 2D**|스프라이트 외곽선을 따라 정점(Vertex)을 직접 편집 가능한 다각형 충돌체.|
|**Edge Collider 2D**|두께가 없는 선/곡선 충돌체. 굴곡진 바닥이나 경계 벽면 제작에 사용.|
|**Composite Collider 2D**|여러 개의 타일맵(Tilemap) 또는 2D 콜라이더를 하나로 병합해 성능 최적화 및 틈새 걸림 방지.|

### 실무 팁 (성능 및 사용 기준)

1. **원시 콜라이더 조합(Compound Collider):**
    
    - 복잡한 모양의 캐릭터나 몬스터라도 연산 비용이 큰 `Mesh Collider` 대신 **여러 개의 Capsule/Box Collider를 계층 구조로 조합**하여 붙이는 것이 물리 연산 최적화에 훨씬 유리합니다.
        
2. **`isTrigger` 옵션:**
    
    - 물리적 튕김이나 밀림 없이 통과하면서 영역 진입 감지만 필요할 때(아이템 획득, 포탈, 감지 구역)는 모든 콜라이더의 `Is Trigger`를 켜서 `OnTriggerEnter` 이벤트를 활용합니다.
        
3. **움직이는 오브젝트는 반드시 Rigidbody 부착:**
    
    - 콜라이더가 붙은 오브젝트를 코드로 이동시킬 때는 정적 콜라이더(Static Collider)로 인식되어 물리 캐시가 매 프레임 재계산되지 않도록 `Rigidbody`(`isKinematic` 포함)를 함께 달아주어야 합니다.


### 물리 매트릭스
Unity의 물리 매트릭스(Layer Collision Matrix)는 **어떤 레이어(Layer)끼리 물리 충돌을 감지하고 상호작용할지 결정하는 2차원 체크박스 테이블**입니다.

불필요한 충돌 연산을 원천 차단하여 게임의 물리 연산 성능을 최적화하고 충돌 로직을 깔끔하게 유지할 때 핵심적으로 사용됩니다.

### 1. 설정 경로

- **3D 물리**: `Edit` $\rightarrow$ `Project Settings` $\rightarrow$ `Physics` $\rightarrow$ 하단의 **Layer Collision Matrix**
    
- **2D 물리**: `Edit` $\rightarrow$ `Project Settings` $\rightarrow$ `Physics 2D` $\rightarrow$ 하단의 **Layer Collision Matrix**
    

행과 열에 프로젝트에 정의된 레이어(Layer)들이 나열되어 있으며, 체크(ON)되어 있으면 두 레이어 간 충돌(Collider, Trigger)을 감지하고, 체크 해제(OFF)하면 물리 엔진 수준에서 충돌을 완전히 무시하고 통과시킵니다.

![[Pasted image 20261016143609.png]]

### 2. 대표적인 활용 사례

- **아군 발사체 무시**: `Player` 레이어와 `PlayerBullet` 레이어 간 충돌 체크 해제 (플레이어가 쏜 총알이 본인 피격 판정에 걸리지 않음).
    
- **적군끼리의 충돌 배제**: `Enemy`와 `Enemy` 간 충돌을 해제하여 몬스터들이 서로 끼이거나 길을 막는 현상 방지 및 연산 최적화.
    
- **배경/이펙트 충돌 제외**: 파티클이나 투명 벽, 장식용 오브젝트가 물리 충돌 연산(Broadphase)에 포함되지 않도록 방지.
### is Trigger 옵션

### 1. 일반 충돌체 vs 트리거 차이점

| **구분**     | **일반 콜라이더 (Is Trigger OFF)**              | **트리거 (Is Trigger ON)**               |
| ---------- | ----------------------------------------- | ------------------------------------- |
| **물리적 반응** | 통과 불가 (벽처럼 막히거나 튕겨 나감)                    | **자유롭게 통과 가능** (저항 없음)                |
| **호출 이벤트** | `OnCollisionEnter / Stay / Exit`          | **`OnTriggerEnter / Stay / Exit`**    |
| **전달 데이터** | `Collision` 객체 (충돌 지점, 반발력, 접촉면 법선 벡터 포함) | `Collider` 객체 (접촉한 상대방의 콜라이더 정보만 포함)  |
| **주요 용도**  | 바닥, 벽, 장애물, 물리 시뮬레이션                      | 아이템 획득, 포탈, 퀘스트/시네마틱 감지 구역, 시야/공격 사거리 |
![[Pasted image 20261016151759.png]]


| **구분**              | **물리 충돌 (OnCollision*)**                                                                 | **트리거 충돌 (OnTrigger*)**                                                            |
| ------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **`Is Trigger` 상태** | 두 콜라이더 모두 **체크 해제**                                                                      | 둘 중 최소 하나가 **체크됨**                                                                 |
| **물리 반응**           | 튕겨나감, 밀림, 마찰 등 물리 법칙 적용                                                                  | 저항 없이 통과함 (단순 통과 감지용)                                                              |
| **이벤트 메서드**         | `OnCollisionEnter`<br><br>  <br><br>`OnCollisionStay`<br><br>  <br><br>`OnCollisionExit` | `OnTriggerEnter`<br><br>  <br><br>`OnTriggerStay`<br><br>  <br><br>`OnTriggerExit` |
| **매개변수**            | **`Collision`** 객체 전달 (접촉점, 반발력 등 상세 데이터)                                                | **`Collider`** 컴포넌트 전달 (상대방 콜라이더 정보만)                                              |
| **주요 사용처**          | 벽/바닥 충돌, 물리 밀치기, 단단한 물체                                                                  | 포탈, 체크포인트, 코인/아이템 획득 영역, 피격 판정 범위                                                  |

### Collision과 Collider의 차이

이름이 비슷해 혼동하기 쉬우나, **컴포넌트 자체**와 **충돌 순간의 보고서**라는 명확한 차이가 있습니다.

- **`Collider` (컴포넌트/엔티티):**
    
    - 오브젝트의 충돌 영역(외형 형태)을 정의하는 컴포넌트 (`BoxCollider`, `SphereCollider`, `MeshCollider` 등).
        
    - 씬 상에 항상 존재하며 `bounds`, `enabled`, `isTrigger` 등의 설정을 가집니다.
        
- **`Collision` (충돌 결과 데이터 클래스):**
    
    - 물리 충돌이 발생한 순간 물리 엔진이 생성하여 전달하는 **데이터 객체**.
        
    - 부딪힌 상대방(`gameObject`, `transform`, `rigidbody`), 충돌 지점 좌표 및 법선 벡터(`ContactPoint`), 충돌 시 전달된 상대 속도(`relativeVelocity`)와 충격량(`impulse`)을 담고 있습니다.

### . Invoke 함수와 충돌 후 지연 처리

`Invoke`는 지정한 메서드를 일정 시간 뒤 지연 호출하거나 반복 실행할 때 사용합니다. 충돌 후 즉시 처리하지 않고 딜레이를 주어야 하는 상황(예: 함정 발동 후 1초 뒤 폭발, 아이템 획득 후 리스폰 등)에 자주 쓰입니다.


```C#
public class Trap : MonoBehaviour
{
    private Collider trapCollider;

    private void Awake()
    {
        trapCollider = GetComponent<Collider>();
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            // 1.5초 뒤 폭발 실행
            Invoke(nameof(Explode), 1.5f);
        }
    }

    private void Explode()
    {
        Debug.Log("함정 폭발!");
        // 폭발 로직...
    }

    private void OnDisable()
    {
        // 오브젝트가 비활성화될 때 대기 중인 Invoke 취소 (에러 방지 필수)
        CancelInvoke(nameof(Explode));
    }
}
```

> **코루틴(Coroutine)과의 차이:**
> 
> `Invoke`는 문자열 기반(또는 `nameof`)으로 호출되어 가볍고 단순한 지연에 편하지만, 파라미터를 넘길 수 없고 일시 정지/재개가 불가능합니다. 파라미터 전달이나 복잡한 흐름 제어가 필요하다면 코루틴을 사용하는 것이 좋습니다.

### 4. 충돌과 컴포넌트 활성화/비활성화

충돌 시 특정 컴포넌트를 켜거나 끄는 처리는 게임 상태 전환의 기본 패턴입니다.

#### 자주 사용하는 제어 대상

1. **Collider 끄기 (`col.enabled = false`):** 몬스터 사망 시 다단 히트 방지, 수집 아이템의 중복 획득 방지.
    
2. **이동 스크립트 끄기 (`movement.enabled = false`):** 피격 경직(Stun) 구현.
    
3. **오브젝트 전체 비활성화 (`gameObject.SetActive(false)`):** 풀링 시스템으로 반환.
    

```C#
private void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Bullet"))
    {
        // 1. 중복 피격 방지를 위해 피격자 콜라이더 즉시 비활성화
        GetComponent<Collider>().enabled = false;

        // 2. 이동 제어 스크립트 비활성화 (경직 상태)
        PlayerMovement movement = GetComponent<PlayerMovement>();
        if (movement != null) movement.enabled = false;

        // 3. 총알 오브젝트 비활성화 (풀링 회수)
        other.gameObject.SetActive(false);
    }
}
```

