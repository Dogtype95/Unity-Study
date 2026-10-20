
### 파티클 프리팹/ContactPoint 자료형/시네머신(가상) 카메라/Baked 애니메이션, 절차적 애니메이션/FK와 IK 애니메이션


### 1. 유니티 파티클 프리팹 (Particle Prefab)

파티클 시스템(`ParticleSystem`) 컴포넌트를 구성한 뒤 프로젝트 창으로 드래그해 에셋화한 재사용 가능한 이펙트 단위입니다. 주로 타격, 폭발, 피격, 스킬 연출에 사용됩니다.

#### 핵심 생성 및 회수 패턴

런타임에 파티클을 단순 `Instantiate` / `Destroy`하면 가비지 컬렉션(GC)과 프레임 드랍을 유발하므로, 오브젝트 풀링(Object Pooling)을 사용하는 것이 정석입니다.

- **Main 모듈 설정:**
    
    - **Stop Action:** 파티클 재생이 끝났을 때의 동작(`Disable`, `Destroy`, `Callback`). 풀링 사용 시 `Disable`로 두고 `OnDisable` 시점에 풀로 반환하는 방식을 주로 사용합니다.
        
    - **Play On Awake:** 풀링 생성 시 즉시 터지지 않게 제어하려면 체크를 해제하고, 코드에서 `ps.Play()`로 제어합니다.

```cs
// 피격 지점에 파티클 생성 및 법선 벡터 방향으로 정렬
public void SpawnHitEffect(GameObject effectPrefab, Vector3 hitPoint, Vector3 hitNormal)
{
    // hitNormal 방향을 바라보도록 회전값 생성
    Quaternion hitRotation = Quaternion.LookRotation(hitNormal);
    
    // 풀링 또는 생성
    GameObject effect = Instantiate(effectPrefab, hitPoint, hitRotation);
}
```

### 2. ContactPoint 자료형

물리 충돌(`OnCollisionEnter`, `OnCollisionStay`) 발생 시 **두 콜라이더가 실제로 맞닿은 지점의 물리적 접촉 정보**를 담는 `struct`(값 형식)입니다.

#### 핵심 프로퍼티

| **프로퍼티**            | **타입**     | **설명**                        |
| ------------------- | ---------- | ----------------------------- |
| **`point`**         | `Vector3`  | 충돌이 발생한 월드 좌표 (이펙트 생성 위치)     |
| **`normal`**        | `Vector3`  | 접촉면의 수직 방향 벡터 (이펙트가 튕겨 나갈 각도) |
| **`thisCollider`**  | `Collider` | 이 충돌을 감지한 내 오브젝트의 콜라이더        |
| **`otherCollider`** | `Collider` | 부딪힌 상대방 오브젝트의 콜라이더            |
| **`separation`**    | `float`    | 두 콜라이더 간의 거리 (음수면 파고든 깊이)     |
```cs
private void OnCollisionEnter(Collision collision)
{
    // 첫 번째 접촉점 가져오기
    ContactPoint contact = collision.GetContact(0);

    Vector3 spawnPos = contact.point;
    Vector3 surfaceNormal = contact.normal;

    // 표면 법선 방향으로 불꽃 이펙트 스폰
    SpawnHitEffect(hitSparkPrefab, spawnPos, surfaceNormal);
}
```

**성능 팁:** `collision.contacts[0]` 대신 **`collision.GetContact(0)`**을 사용해야 힙 메모리 할당(GC Garbage) 없이 캐시된 데이터를 직접 참조합니다.

### 3. 시네머신(Cinemachine) 가상 카메라 (Virtual Camera)

씬 전체를 렌더링하는 무거운 `Unity Camera`를 여러 대 두는 대신, 실제 카메라는 단 1대(Cinemachine Brain 부착)만 유지하고 가상 카메라(Virtual Camera)의 파라미터를 실시간으로 블렌딩(보간)하여 비추는 모듈형 카메라 시스템입니다.

***unity package registry에서 추가해줘야함***

#### 핵심 요소 및 설정

- **Cinemachine Brain:** 실제 메인 카메라에 붙어 어떤 가상 카메라를 비출지(Cut, Ease In/Out 등) 전환을 총괄합니다.
    
- **Virtual Camera 파라미터:**
    
    - **Follow:** 카메라가 위치상 따라다닐 대상 (`Transform`)
        
    - **Look At:** 카메라가 렌더링 각도상 쳐다볼 대상 (`Transform`)
        
    - **Body (추적 방식):**
        
        - `3rd Person Follow`: 플레이어 어깨 뒤 추적 (충돌 회피 내장)
            
        - `Framing Transpose`: 2D/2.5D 고정 거리 추적
            
    - **Aim (조준 방식):**
        
        - `Composer`: 타겟을 화면 내 특정 박스(Dead Zone, Soft Zone) 안에 머물게 유지
            
        - `Hard Look At`: 타겟 중심점을 무조건 정면 응시
            
    - **Priority:** 정수 값이 가장 높은 가상 카메라가 화면을 차지 (런타임에 0에서 10으로 올려 컷씬 전환 구현)

```text

시네머신은 게임 뷰 가이드를 켰을 때 나오는 노란 점(Tracking Point) 기준으로 이동과 회전을 조절하게 됩니다. 

이때 노란 점의 위치는 Tracking Target에서 세팅해둔 오브젝트의 Transform Pivot을 기준으로 잡히게 됩니다. (플레이어 모델이라면 발가락으로 잡힘)

 Camera Distance : 노란 점으로부터 얼마나 Z축으로 멀리 떨어질 지(카메라 로컬 축 기준) 

 Screen Position : 노란 점이 게임 뷰에서 어느 위치에 놓을지 (0, 0이 화면 중앙, 보통 GTA 같은 게임에서 캐릭터 어깨를 화면 중앙으로 두고 싶을 때 세팅) 
 
 Target Offset : 노란 점의 위치를 실제 Tracking Target의 트랜스폼에서 어느 정도 오차를 둘 지 (타겟 오브젝트의 로컬 축 기준) - 플레이어 모델을 타겟팅했다면 발가락을 기준으로 잡히기 때문에 이때 Offset의 y를 조정해서 몸통을 중심으로 배치할 수 있습니다.
```

### Noise

- 화면상 떨림 효과를 주기 위한 옵션
- 이미지 상의 1, 2, 3으로 설정해주면 댐 3번은 그떨림 정도를 나타냄 2번은 프리셋 미리만들어둔 떨림 효과에 대한

![[Pasted image 20261019164841.png|351]]

### Position Composer (Cinemachine)

Cinemachine 3.x 버전부터 기존의 `Transposer`와 `Composer`가 **Position Composer**와 **Rotation Composer** 등으로 체계화되었습니다. 카메라의 월드 위치를 타깃(Follow) 기준으로 제어할 때 사용합니다.

- **핵심 역할:** Follow 타깃을 기준으로 카메라의 거리, 높이, 오프셋을 일정하게 유지하며 따라가는 위치 알고리즘입니다.
    
- **주요 세팅 항목:**
    
    - **Camera Distance:** 타깃과 카메라 사이의 기본 거리.
        
    - **Target Offset:** 피벗 위치(보통 캐릭터의 발 대신 가슴/머리 위치로 올리기 위해 Y축을 1.2~1.6m 정도로 설정).
        
    - **Damping (X, Y, Z):** 타깃이 이동할 때 카메라가 반응하는 지연 시간(부드러움 정도). 0이면 즉시 고정되며, 0.5~1.0 정도로 두면 부드러운 유체감이 생깁니다.
        

### 2. 게임 뷰 가이드 (Game View Guides / Framing Overlay)

카메라 조준 영역을 시각적으로 모니터링할 수 있는 오버레이 격자선입니다.

- **확인 방법:** 가상 카메라 인스펙터 상단 또는 컴포넌트 옵션에서 **Game View Guides**를 체크하면 Game 뷰에 반투명한 색상 영역이 표시됩니다.
    
- **구역별 의미:**
    
    - **Dead Zone (노란색/투명 영역 중앙):** 타깃이 이 박스 안에서 움직일 때는 카메라가 회전하거나 위치를 틀지 않고 가만히 유지됩니다. (불필요한 미세 떨림 방지)
        
    - **Soft Zone (파란색 영역):** 타깃이 Dead Zone을 벗어나 이 영역에 진입하면 카메라가 부드럽게(Damping 적용) 따라붙습니다.
        
    - **Hard Limit (빨간색 영역/경계):** 타깃이 이 경계를 절대 벗어나지 못하도록 카메라가 강제로 즉시 추적합니다.
    - ### 3인칭 회전 (3rd Person Follow / FreeLook Orbit)

플레이어 마우스 또는 패드 입력에 맞춰 캐릭터 주위를 공전(Orbit)하는 3인칭 시점 구현 방식입니다.

- **Cinemachine 3rd Person Follow:**
    
    - 어깨 너머 뷰(Over-the-shoulder)나 일반 액션 RPG 뷰를 만들 때 가장 직관적입니다.
        
    - **Shoulder Offset / Vertical Arm Length:** 카메라가 플레이어의 오른쪽 어깨 너머를 비출지, 중앙을 비출지 결정합니다.
        
    - **Obstacle Handling (충돌 처리):** 벽에 카메라가 파묻히지 않도록 `Cinemachine Deoccluder`(구 버전의 `Cinemachine Collider`) 컴포넌트를 함께 붙여 카메라 거리를 자동으로 좁히도록 구성합니다.
        
- **FreeLook 방식 (Cinemachine 2.x/3.x 공통 개념):**
    
    - Top, Middle, Bottom의 3단 링(Ring) 반경과 높이를 지정하여 마우스 Y축 입력에 따라 위아래로 자연스럽게 기울어지게 설계할 때 유용합니다.
        

### 4. 펄린 노이즈 (Perlin Noise / Camera Shake)

카메라에 자연스러운 흔들림(손떨림, 달리기 충격, 폭발 등)을 줄 때 사용하는 절차적 노이즈 알고리즘입니다.

- **적용 위치:** 가상 카메라의 **Noise** 슬롯 (Cinemachine Basic Multi-Channel Perlin 선택).
    
- **Noise Profile 선택:**
    
    - `Handheld_tele_mild`: 가벼운 핸드헬드 손떨림.
        
    - `Shake_explosion`: 폭발 시 큰 진동.
        
- **핵심 조절 파라미터:**
    
    - **Amplitude Gain (진폭):** 흔들림의 크기(강도). 폭발 시 일시적으로 수치를 2~5까지 올렸다가 서서히 0으로 복귀시킵니다.
        
    - **Frequency Gain (진동수):** 흔들림의 속도(빈도). 수치가 높을수록 잘게 떱니다.
        
- **Impulse Extension:** 폭발이나 피격 같은 일회성 충격은 노이즈를 수동 조절하는 대신 **Cinemachine Impulse Listener / Source** 시스템을 연동하면 이벤트 기반으로 손쉽게 처리할 수 있습니다.

### Trail 렌더러 (Trail Renderer)

칼의 궤적, 총알 궤적, 이동 잔상 등을 남기는 유니티 기본 시각효과 컴포넌트입니다.

- **동작 원리:** 오브젝트가 공간을 이동할 때 이전 프레임의 위치 기록을 바탕으로 폴리곤 띠(Mesh Ribbon)를 동적으로 생성합니다.
    
- **주요 세팅 항목:**
    
    - **Time:** 잔상이 화면에 남아있는 지속 시간(초 단위). 빠른 검광 효과는 0.1~0.25초 정도로 짧게 설정합니다.
        
    - **Width (Curve):** 시작 지점과 끝 지점의 두께를 커브 그래프로 지정. 보통 시작은 두껍게, 끝으로 갈수록 0으로 수렴하게 만듭니다.
        
    - **Color (Gradient):** 시간에 따른 색상 및 알파 페이드아웃 설정.
        
    - **Min Vertex Distance:** 폴리곤을 생성할 최소 이동 거리. 너무 작으면 드로우콜/버텍스가 급증하고, 너무 크면 곡선 궤적이 각져 보입니다.
        
    - **Material:** 반드시 Particles 셰이더(예: Universal Render Pipeline/Particles/Unlit)나 Additive 계열 셰이더를 사용하여 투명도와 빛 번짐(Bloom)을 연출해야 자연스럽습니다.