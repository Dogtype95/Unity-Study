
## 1. Static 체크박스 (정적 오브젝트 플래그)

인스펙터 우측 상단의 **Static** 체크박스는 게임 실행 중 위치나 외형이 변하지 않는 정적 오브젝트임을 유니티 엔진에 알려 다양한 사전 연산(베이킹)과 배칭 최적화를 적용하도록 만듭니다.

- **주요 세부 플래그 (드롭다운):**
    
    - **Batching Static:** 드로우콜(Draw Call)을 줄이기 위해 메시들을 하나로 묶어 정적 배칭(Static Batching)을 수행합니다.
        
    - **Contribute GI:** 라이트맵(Lightmap) 생성 시 빛을 반사하고 그림자를 받는 정적 물체로 계산에 포함시킵니다.
        
    - **Reflection Probe Static:** 리플렉션 프로브 베이킹 시 반사 큐브맵 텍스처에 포함됩니다.
        
    - **Occluder / Occludee Static:** 오클루전 컬링(Occlusion Culling) 계산에 사용되어 카메라 뒤나 큰 물체 뒤에 가려진 오브젝트를 렌더링에서 제외합니다.
        
    - **Navigation Static:** AI 길찾기용 NavMesh 베이킹 대상이 됩니다.
        
    - **Off-Mesh Link Generation:** 옥상이나 절벽 간 점프 경로 자동 생성에 포함됩니다.
        

## 2. 실시간 반사 (Realtime Reflection)

유니티에서 물체 표면에 주변 환경을 반사시키는 방식은 크게 세 가지로 나뉩니다.

- **Reflection Probe (Type: Realtime):**
    
    - 프로브 위치에서 씬을 주기적으로 다시 렌더링하여 큐브맵을 갱신합니다.
        
    - **주의점:** 큐브맵 6면을 다시 그려야 하므로 드로우콜과 연산 비용이 매우 큽니다. 모바일/저사양 환경에서는 `Refresh Mode`를 _Every Frame_ 대신 _On Awake_ 또는 _Via Scripting_(스크립트로 필요할 때만 호출)으로 제한해야 합니다.
        
- **Screen Space Reflections (SSR):**
    
    - 화면(카메라)에 렌더링된 픽셀 정보를 기반으로 실시간 레이트레이싱 효과를 흉내 내는 포스트 프로세싱 방식입니다.
        
    - 평면 반사나 젖은 바닥 표현에 우수하지만, 화면 밖으로 벗어난 물체는 반사되지 않는 한계가 있습니다.
        
- **Planar Reflection (평면 반사):**
    
    - 거울이나 잔잔한 수면에 주로 쓰이며, 별도의 가상 카메라로 반사면 반대편을 실시간 렌더링해 텍스처로 입힙니다.
        

## 3. 품질 설정 (Quality Settings)

**Project Settings > Quality**에서 빌드 타깃 기기(PC, 콘솔, 모바일)의 성능에 맞게 그래픽 프리셋(Low, Medium, High 등)을 구성합니다.

- **역할:** 텍스처 해상도 제한, 안티앨리어싱(MSAA), 섀도우 해상도/거리, 프레임 레이트 상한선 등을 일괄 정의합니다.
    
- **SRP/URP 환경과의 관계:** 렌더 파이프라인(URP/HDRP)을 사용할 경우 대부분의 세부 렌더링 옵션(그림자 거리, 포스트 프로세싱 등)이 Quality 창 대신 아래의 **RP 에셋**으로 위임됩니다. 각 품질 레벨마다 서로 다른 URP 에셋을 연결하여 동적으로 그래픽 품질을 전환할 수 있습니다.
    

## 4. RP 에셋과 렌더러 (Universal Render Pipeline Asset & Renderer Data)

URP 환경은 설정을 담는 **URP Asset**과 실제 그리기 방식을 정의하는 **Renderer Feature/Data**가 분리된 구조를 가집니다.

| **구성 요소**                              | **역할**                        | **핵심 설정 항목**                                                                                                    |
| -------------------------------------- | ----------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **URP Asset**                          | 프로젝트 전반의 그래픽 스펙 및 품질 제어       | 그림자 해상도/거리, HDR, 안티앨리어싱(MSAA), 렌더 스케일, 사용할 Renderer 지정                                                          |
| **Renderer Data** (Universal Renderer) | 카메라가 화면을 그리는 구체적인 파이프라인 단계 정의 | Rendering Path (Forward / Deferred), Depth/Opaque Texture 생성 여부, **Renderer Features** (커스텀 셰이더 패스, 아웃라인, 블러 등) |

## 5.RP 컨버터 (Render Pipeline Converter)

빌트인(Built-in) 렌더 파이프라인에서 제작된 프로젝트나 에셋을 URP/HDRP로 마이그레이션할 때 사용하는 공식 변환 도구입니다.

- **접근 경로:** `Window > Rendering > Render Pipeline Converter`
    
- **주요 변환 항목:**
    
    - **Material Upgrade:** 빌트인 기본 셰이더(Standard 등)를 URP용 `Universal Render Pipeline/Lit` 등으로 자동 교체 및 텍스처 재매핑.
        
    - **Lighting Settings:** 라이트맵 및 씬 조명 컴포넌트 파라미터 변환.
        
- **한계:** 타사 커스텀 셰이더나 셰이더 그래프가 아닌 서페이스 셰이더 코드는 자동 변환되지 않으므로 수동 재작성이 필요합니다.


### 애니메이션 레이어

- 애니메이터에서 특정 부위의 파츠만 애니메이션을 적용해주는 영역

### 아바타 마스크
- 그 영역을 지정하는 아바타 마스크 파일이 필요함 눌러보면암

![[Pasted image 20261023152945.png]]

- 레이어별에 마스크를 넣으면 어떤 부위만 애니메이터로 실행시킬지 정할수 있음
### 병행 상태 머신 동기화