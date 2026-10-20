
### 파티클 쉐이더

- Unity의 시각 특수효과(VFX)와 오디오 시스템을 구성하는 핵심 개념 5가지의 원리와 실전 설정 가이드입니다.

### 1. 파티클 쉐이더 (Particle Shader)

연기, 불꽃, 마법 효과 등 수많은 입자를 가볍고 빠르게 그리기 위해 특화된 쉐이더입니다.

- **동작 원리:** 일반 3D 오브젝트와 달리 깊이 쓰기(Z-Write)를 끄고 투명도를 겹쳐 표현하는 블렌딩 연산 위주로 작동합니다.
    
- **주요 블렌드 모드:**
    
    - **Additive (가산):** 파티클이 겹칠수록 색상이 더해져 밝게 빛납니다. (폭발 중심부, 빛나는 스파크, 섬광 연출)
        
    - **Alpha Blended (알파 블렌드):** 텍스처의 알파 채널을 기준으로 부드럽게 배경과 합성됩니다. (연기, 먼지, 구름)
        
- **주요 기능 (URP 기준 `Universal Render Pipeline/Particles/Unlit` 등):**
    
    - **Soft Particles (소프트 파티클):** 파티클이 바닥이나 벽에 닿을 때 교차선이 칼로 자른 듯 각지지 않고, 카메라 뎁스 텍스처를 참조해 경계면을 부드럽게 페이드아웃 처리합니다.
        
    - **Flip-Book Blending:** 스프라이트 시트 애니메이션이 다음 프레임으로 넘어갈 때 뚝뚝 끊기지 않고 모핑되듯 부드럽게 보간합니다.

- 파티클 쉐이더 UnLit(빛의 영향을 안받는 통상적으로 파티클은 빛의 영향을 안받아야함)

- Rendering Mode 

![[Pasted image 20261020132849.png]]

### 2. 빌보드 렌더링 (Billboard Rendering)

평면 사각 폴리곤(Quad)이 카메라를 향해 항상 정면을 바라보도록 실시간 회전시키는 기법입니다.

- **사용 목적:** 파티클 시스템의 입자, 잔디, 원경의 나무, 캐릭터 머리 위의 체력바/닉네임 UI 등을 저비용으로 입체감 있게 표현할 때 씁니다.
    
- **Particle System Renderer 모드:**
    
    - **Billboard:** 카메라 평면과 완전히 평행하게 마주봅니다. (기본 연기/먼지)
        
    - **Stretched Billboard:** 입자의 이동 속도나 진행 방향에 맞춰 늘어나며 카메라를 바라봅니다. (스파크, 빗줄기, 레이저)
        
    - **Horizontal / Vertical Billboard:** 각각 XZ 바닥 평면이나 Y축 축 고정 회전만 허용합니다. (바닥 마법진, 기둥형 이펙트)
        

### 3. PlayOneShot 함수 (AudioSource.PlayOneShot)

단발성 효과음(발소리, 총소리, 버튼 클릭음 등)을 겹쳐서 출력할 때 사용하는 `AudioSource`의 메서드입니다.

- **`Play()`와의 결정적 차이:**
    
    - `audioSource.Play()`는 호출 시 현재 재생 중이던 클립을 중단하고 처음부터 다시 시작합니다.
        
    - `audioSource.PlayOneShot(clip, volumeScale)`은 **새로운 오디오 음성을 별도 트랙으로 독립 재생**합니다. 총을 연발로 쏘거나 칼을 빠르게 여러 번 휘둘러도 이전 소리가 끊기지 않고 자연스럽게 중첩됩니다.
        
- **주의점:** 한 번 재생을 시작하면 도중에 스크립트로 특정 사운드만 일시정지하거나 멈출 수 없으므로, BGM이나 루프 사운드에는 사용하지 않습니다.
    

### 4. AudioMixer 에셋 (오디오 믹서)

게임 내 모든 사운드의 볼륨 그룹, 라우팅, 오디오 이펙트를 관리하는 통합 믹싱 콘솔 에셋입니다.

- **핵심 구조:**
    
    - **Audio Group 계층:** `Master` 그룹 아래에 `BGM`, `SFX`, `UI`, `Voice` 등으로 그룹을 분기하여 카테고리별 마스터 볼륨을 개별 제어합니다.
        
    - **DSP 이펙트 플러그인:** 각 그룹에 `Reverb`(동굴/실내 잔향), `Lowpass Filter`(물속에 잠겼을 때 먹먹한 소리, UI 일시정지 효과), `Duck Volume`(나레이션 음성이 나올 때 BGM 볼륨 자동 감소) 등을 손쉽게 체인 형태로 적용할 수 있습니다.
        
- **스크립트 연동 (Exposed Parameters):**
    
    - 볼륨 페이더를 우클릭하여 "Expose to Script"를 지정해야 C# 스크립트(`mixer.SetFloat(...)`)에서 환경설정 슬라이더와 연동할 수 있습니다.
        
    - _참고:_ AudioMixer 볼륨은 데시벨(dB, 보통 -80dB ~ 0dB) 체계를 따르므로, 일반 슬라이더(0.0001 ~ 1) 값을 넘길 때는 로그 변환(`Mathf.Log10(val) * 20`)을 거쳐야 청각적으로 균등한 볼륨 조절이 됩니다.
        

### 5. 3D 사운드 (Spatial Audio / 3D Sound Settings)

월드 내 소리 발생지점(AudioSource)과 플레이어 귀(AudioListener) 사이의 거리와 방향에 따라 소리의 크기와 좌우 팬(Pan)을 시뮬레이션하는 기능입니다.

- **Spatial Blend:**
    
    - 슬라이더를 `0(2D)`으로 두면 리스너 위치와 상관없이 일정하게 들립니다. (BGM, UI 클릭음)
        
    - 슬라이더를 `1(3D)`로 두면 3D 공간 연산이 적용됩니다. (폭발, 적 발소리, 환경음)
        
- **3D Sound Settings 핵심 파라미터:**
    
    - **Min Distance:** 소리가 감쇠 없이 100% 최대 음량으로 들리는 반경.
        
    - **Max Distance:** 소리 감쇠가 완전히 끝나는 거리.
        
    - **Volume Rolloff:** 거리에 따른 볼륨 감소 곡선.
        
        - _Logarithmic Rolloff:_ 현실 물리에 가까운 감쇠 (가까울 때 급격히 줄어듦).
            
        - _Linear Rolloff:_ 거리 비례 일정 감쇠 (게임 밸런싱 및 가청 범위 예측에 용이).
            
    - **Spread:** 3D 입체 음향이 스피커 공간으로 퍼지는 각도

### RigidBody _rb ->  _rb.AddExplosionForce()

```cs
private RigidBody _rb;

_rb = GetComponent<RigidBody>();

private void OnCollisonEnter(Collision other)
{
// 순서대로 폭발 힘, 폭발 위치, 폭발 반경, 폭발반경(위방향), 포스모드로 구성
_rb.AddExplosionForce(float explosionForce, Vector3 explosionPosition, float explosionRadius, float upwardsModifier = 0.0f, ForceMode mode = ForceMode.Force)	
	
	//폭발 반경은 이걸 써서 넣어주면 됨 보통
	other.GetContact(0);

}
```

### `hysics.Overlap` 계열 함수

특정 가상 영역(구, 박스, 캡슐 등)을 씬에 생성하여, 그 영역과 **겹치거나 내부에 들어온 모든 콜라이더(`Collider`)를 일괄 감지**할 때 사용하는 물리 쿼리 함수입니다.

폭발 범위 감지, 근접 공격 피격 판정, 주변 상호작용 오브젝트 탐색 등에 주로 사용됩니다.

| **함수명**                      | **형태**        | **주요 매개변수**                            | **용도**                     |
| ---------------------------- | ------------- | -------------------------------------- | -------------------------- |
| **`Physics.OverlapSphere`**  | 구형 (Sphere)   | `center`, `radius`                     | 폭발 반경, 원형 오라, 주변 탐색        |
| **`Physics.OverlapBox`**     | 박스형 (Box)     | `center`, `halfExtents`, `orientation` | 직사각형 근접 공격 판정, 사각 영역 진입 체크 |
| **`Physics.OverlapCapsule`** | 캡슐형 (Capsule) | `point0`, `point1`, `radius`           | 캐릭터 형태의 충돌 영역 검사, 기둥형 판정   |
```cs
// 기본 사용 형태 (반경 5m 내 'Monster' 레이어 콜라이더 감지)
int monsterLayerMask = 1 << LayerMask.NameToLayer("Monster");
Collider[] hits = Physics.OverlapSphere(transform.position, 5f, monsterLayerMask);

foreach (Collider hit in hits)
{
    Debug.Log($"감지된 대상: {hit.name}");
}
```

### `NonAlloc` 최적화 (가비지 컬렉션(GC) 제거)

기본 `Physics.OverlapSphere`는 호출할 때마다 감지된 콜라이더들을 담을 **새로운 배열(`Collider[]`)을 힙(Heap) 메모리에 할당**하여 반환합니다.

매 프레임(또는 빈번한 공격/폭발마다) 호출하면 수많은 가비지 컬렉션(GC Alloc)이 쌓여 주기적으로 프레임 드랍(GC Spike)이 발생합니다. 이를 방지하기 위해 **`NonAlloc` 접미사가 붙은 함수**를 사용합니다.

#### 핵심 원리

- 배열을 미리 1회만 할당해두고 재사용합니다.
    
- 함수는 새로운 배열을 만들지 않고, **미리 전달한 배열에 결과를 채운 뒤 감지된 개수(`int`)만 반환**합니다.
    

#### 기본형 vs `NonAlloc` 비교


```C#
public class AttackHitScanner : MonoBehaviour
{
    [SerializeField] private float attackRadius = 3f;
    [SerializeField] private LayerMask targetLayer;

    // 1. 재사용할 결과 버퍼 배열을 미리 할당 (최대 감지 개수 크기로 설정)
    private readonly Collider[] hitResults = new Collider[10];

    public void ScanTargets()
    {
        // 2. NonAlloc 버전 호출: 버퍼를 넘겨주고 감지된 개수(hitCount)를 반환받음
        int hitCount = Physics.OverlapSphereNonAlloc(
            transform.position, 
            attackRadius, 
            hitResults, 
            targetLayer
        );

        // 3. 주의: hitResults.Length가 아니라 실제 감지된 'hitCount'까지만 순회
        for (int i = 0; i < hitCount; i++)
        {
            Collider col = hitResults[i];
            Debug.Log($"피격 대상: {col.name}");
        }

        // 4. (선택) 이전 참조 누수를 방지하기 위해 사용 후 배열 비우기
        System.Array.Clear(hitResults, 0, hitCount);
    }
}
```

> **`NonAlloc` 사용 시 주의사항:** 감지된 오브젝트가 버퍼 크기(`10`)보다 많으면, 10개를 초과하는 대상은 **감지되지 않고 무시**됩니다. 따라서 예상 가능한 최대치보다 약간 여유 있게 버퍼 크기를 잡아야 합니다.


### `OnDrawGizmos` 이벤트 (물리 범위 시각화)

`Physics.Overlap`은 보이지 않는 가상의 도형을 사용하므로, 에디터 씬(Scene) 뷰에 기즈모(Gizmo)로 범위를 그려주지 않으면 크기나 방향이 올바른지 확인하기 어렵습니다.

유니티는 이를 위해 2가지 기즈모 콜백을 제공합니다.

| **이벤트명**                     | **동작 조건**                         | **주요 용도**                         |
| ---------------------------- | --------------------------------- | --------------------------------- |
| **`OnDrawGizmos()`**         | 씬 뷰가 렌더링될 때 **항상** 실행             | 기본 영역, 순찰 경로, 고정 감지 구역            |
| **`OnDrawGizmosSelected()`** | 해당 오브젝트를 **하이어라키/씬에서 선택했을 때만** 실행 | 공격 반경, 폭발 범위 등 화면이 복잡해지는 것을 방지할 때 |

```cs
using UnityEngine;

public class AreaVisualizer : MonoBehaviour
{
    [Header("Sphere 설정")]
    [SerializeField] private float sphereRadius = 3f;

    [Header("Box 설정")]
    [SerializeField] private Vector3 boxHalfExtents = new Vector3(1f, 1f, 2f); // 중심에서 각 축 방향 절반 크기

    // 선택되었을 때만 씬 뷰에 표시
    private void OnDrawGizmosSelected()
    {
        // 1. 구형 범위 시각화 (빨간색 와이어프레임)
        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, sphereRadius);

        // 2. 반투명 내부 구체 (속이 찬 형태)
        Gizmos.color = new Color(1f, 0f, 0f, 0.15f);
        Gizmos.DrawSphere(transform.position, sphereRadius);

        // 3. 회전이 적용된 박스 범위 시각화 (초록색)
        Gizmos.color = Color.green;
        // 박스 기즈모는 기본적으로 월드 축에 정렬되므로, 
        // 오브젝트의 회전을 반영하려면 기즈모 매트릭스를 설정해야 합니다.
        Matrix4x4 originalMatrix = Gizmos.matrix;
        Gizmos.matrix = Matrix4x4.TRS(transform.position, transform.rotation, Vector3.one);

        // 로컬 (0,0,0) 기준으로 halfExtents * 2 크기의 박스 렌더링
        Gizmos.DrawWireCube(Vector3.zero, boxHalfExtents * 2f);

        // 매트릭스 원복 (필수: 다른 기즈모에 영향을 주지 않도록 복구)
        Gizmos.matrix = originalMatrix;
    }
}
```