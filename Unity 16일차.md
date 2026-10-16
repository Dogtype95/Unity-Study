
### animator 스크립트로 값 가져오기

```cs

private Animator _animator;

private void Awake()
{
	_animator = GetComponent<Animator>();
}

private void Update()
{
	_animaotr.SetFloat("MoveX", dir.x); // 애니메이터 변수 세팅 
	_animaotr.SetFloat("MoveY", dir.y);
	
}

```


### Animator.StringToHash

![[Pasted image 20261016105111.png]]

```text
문자열 기반으로 찾으면 뭔가 성능이 좋지 않다...
해당 문자열에 Alt + Enter를 누르면 '캐시 처리된 프로퍼티 색인사용이 뜨는데' 이게좋다 

```

![[Pasted image 20261016105224.png]]


```text
그러면 위와 같이 변수가 하나 만들어진다. MoveX 의 해쉬값을 미리 만들어서 캐싱 해두는 거다
```


![[Pasted image 20261016105324.png]]

```text
만들어진 변수를 그대로 사용하면 끝
```

### Animator Blend Tree(블렌드 트리)

블렌드 트리(Blend Tree)는 Unity 메카님(Mecanim) 애니메이션 시스템에서 **두 개 이상의 애니메이션 클립을 파라미터 값에 따라 부드럽게 섞어(Blending) 하나의 동작으로 출력하는 기능**
### 블렌드 타입 (Blend Type) 종류

블렌드 트리 인스펙터 창에서 제어 파라미터의 개수와 목적에 따라 타입을 선택합니다.

|**타입**|**파라미터 수**|**주요 활용 사례**|**특징**|
|---|---|---|---|
|**1D**|1개 (Float)|정면 걷기 $\rightarrow$ 달리기, 체력 저하에 따른 걷기 폼 변화|단일 축(예: `Speed`)을 기준으로 선형 보간|
|**2D Simple Directional**|2개 (Float, Float)|4방향/8방향 걷기 (전/후/좌/우)|같은 방향에 여러 속도 클립이 없을 때 적합 (예: 걷기만 존재)|
|**2D Freeform Directional**|2개 (Float, Float)|방향별 걷기 + 달리기 (전방 걷기, 전방 달리기 동시 포함)|같은 방향에 서로 다른 속도 단계가 공존할 때 사용|
|**2D Freeform Cartesian**|2개 (Float, Float)|무기 조준(Aiming), 자동차 드리프트, 기울기|극좌표(방향/각도)가 아닌 직교 좌표계(X, Y) 기반 위치 보간|
|**Direct**|N개|얼굴 표정(모프 타겟), 손가락 관절 세부 제어|각 클립마다 독립적인 가중치 파라미터를 1:1로 직접 매핑|

### Animation  익스펙터의 Apply Root Motion
- 코드로 우리가 직접 트랜스폼을 조작하는게 아니라 애니메이션으로 트랜스폼을 조정하게 해주겠다 라는 설정

```text
파쿠르나 복잡한 동작을 하는데 애니메이션이 아닌 우리가 트랜스폼을 조정하면 이상하게 움직일 것이다 하지만 애니메이터가 루트애니메이션으로 제작한게 아니면 그 값을 우리가 직접 조정하는건 합리적이지 않음
```

-  루트전용 애니메이션인지 확인 하는 방법은 인스펙터 창의 `Average Velocity` 수치 확인 

프로젝트 창에서 FBX 파일 또는 애니메이션 클립을 선택한 뒤 **Inspector $\rightarrow$ Animation 탭**의 하단 정보를 확인합니다.

- **루트 모션 포함 여부 판별:**
    
    - 애니메이션 클립 정보 맨 아래 박스에 있는 **`Average Velocity`** 값을 확인합니다.
        
    - `(0.000, 0.000, 3.687)`처럼 0이 아닌 수치(이동 속도)가 찍혀 있다면 캐릭터의 중심점(루트)이 실제로 이동하는 **루트 모션 클립**입니다.
        
    - 완전한 제자리(In-Place) 애니메이션이라면 `Average Velocity: (0.000, 0.000, 0.000)`으로 표시됩니다.