
### Scriptable Object

![[Pasted image 20261021144657.png|379]]

- 프로젝트에서 우클릭으로 생성

스크립터블 오브젝트(ScriptableObject, 이하 SO)는 유니티에서 대량의 데이터를 저장하고 관리하기 위해 사용하는 특수한 데이터 컨테이너 클래스

일반적인 `MonoBehaviour`와 달리 **게임오브젝트(GameObject)에 컴포넌트로 부착되지 않으며, 프로젝트 창에 독립적인 에셋(`.asset`) 파일 형태로 존재**합니다.

## 1. 왜 사용하는가? (핵심 장점)

- **메모리 절약 (인스턴스 공유):**
    
    `MonoBehaviour`에 몬스터 능력치(공격력, 이동속도, 프리팹 등)를 직접 넣으면 몬스터 1,000마리가 생성될 때마다 동일한 데이터 복사본이 1,000개 생깁니다. 반면 SO를 참조하게 만들면 **단 1개의 데이터 에셋을 1,000마리가 공유**하므로 메모리 사용량이 크게 줄어듭니다.
    
- **데이터와 로직의 분리:**
    
    기획 데이터(아이템 스펙, 스킬 정보, 사운드 설정 등)를 코드 로직과 완전히 분리할 수 있습니다.
    
- **직관적인 에디터 워크플로:**
    
    인스펙터 창에서 기획자나 디자이너가 스크립트 수정 없이 우클릭 메뉴(`CreateAssetMenu`)를 통해 새로운 아이템이나 몬스터 에셋을 쉽게 만들고 편집할 수 있습니다.
    
- **씬(Scene) 독립성:**
    
    씬이 전환되거나 언로드되어도 데이터가 파괴되지 않고 유지됩니다

## . 대표적인 활용 사례

| **활용 분야**                | **설명**                                                       |
| ------------------------ | ------------------------------------------------------------ |
| **정적 데이터 관리**            | 아이템 도감, 몬스터 스펙, 스킬 툴팁, 퀘스트 정보                                |
| **게임 설정 (Settings)**     | 오디오 믹서 볼륨 기본값, 그래픽 옵션 프리셋                                    |
| **이벤트 기반 아키텍처**          | `GameEvent` SO를 만들어 씬 간 또는 서로 모르는 객체 간의 결합도 낮은 이벤트 브로드캐스팅 구현 |
| **런타임 셋 (Runtime Sets)** | 현재 씬에 살아있는 모든 적(Enemy) 리스트를 전역 싱글톤 없이 SO에 등록해 추적             |

### 코루틴 중복 호출 시 주의할 점

```cs
using System.Collections;
using UnityEngine;

public class FadeController : MonoBehaviour
{
    private Coroutine _fadeCoroutine;

    public void TriggerFade(float targetAlpha)
    {
        // 1. 이미 실행 중인 코루틴이 있다면 먼저 정지
        if (_fadeCoroutine != null)
        {
            StopCoroutine(_fadeCoroutine);
        }

        // 2. 새 코루틴 시작 후 참조 보관
        _fadeCoroutine = StartCoroutine(FadeRoutine(targetAlpha));
    }

    private IEnumerator FadeRoutine(float targetAlpha)
    {
        // ... 페이드 보간 로직 ...
        yield return null;

        // 3. 작업이 완전히 끝나면 핸들을 null로 초기화
        _fadeCoroutine = null;
    }
}
```
- 즉 코루틴 실행중에 똑같은 코루틴이 또 호출된다면 로직이 중첩될 수 있어 문제
- 똑같은 코루틴을 계속 호출하면 힙에 할당 되니까 GC부하

### 스크립트에서 머티리얼 접근

- 기본적으로 메시랜더러에 있는 머티리얼 객체를 가져오면 됨
```cs
using UnityEngine;

public class MaterialChanger : MonoBehaviour
{
    private Renderer _renderer;

    private void Awake()
    {
	    // 메시 렌더러 가져오기
        _renderer = GetComponent<Renderer>();
    }

    public void ChangeColorToRed()
    {
        // 주의: 이 순간 머티리얼이 복제되어 새 인스턴스가 됨
        _renderer.material.color = Color.red;
    }

    private void OnDestroy()
    {
        // renderer.material로 생성된 복제본은 가비지 컬렉터가 자동 정리해주지 않으므로 수동 파괴 권장
        if (_renderer != null && _renderer.material != null)
        {
            Destroy(_renderer.material);
        }
    }
}

```

### 머티리얼 공유 문제

- 스크립트에서 머티리얼에 접근해 값을 변경할 때 함수마다 차이가 발생

```cs
private Renderer _renderer;

_renderer.material.color = Color.red; // 머티리얼 인스턴스를 가져옴 즉 깊은 복사
_renderer.Sharedmaterial.color = Color.blue; // 머티리얼 원본을 직접참조함
//인스펙터에서 변경하는 거랑 같은 효과(즉 인스펙터에서 머티리얼 변경은 얕은 복사로 원본 영향감)

```