
## 함수

#### 생성
- Instantiate();

### 회전 및 이동
### 1. 이동 함수

|**방식**|**주요 함수 / 프로퍼티**|**특징 및 사용 시점**|
|---|---|---|
|**Transform 직접 이동**|`transform.Translate(Vector3)`|충돌 판정이 덜 중요한 비물리 객체, 배경, UI 오브젝트|
|**등속 보간**|`Vector3.MoveTowards(from, to, maxDistance)`|정해진 속도로 목표 지점까지 정확하게 도달해야 할 때|
|**부드러운 감속**|`Vector3.Lerp(from, to, t)` / `Vector3.SmoothDamp(...)`|카메라 추적, 부드러운 감속 연출|
|**물리 기반 속도 지정**|`rigidbody.linearVelocity = Vector3` _(구 velocity)_|관성/충돌이 필요한 플레이어 또는 물리 오브젝트|
|**물리 기반 힘 추가**|`rigidbody.AddForce(Vector3, ForceMode)`|점프, 밀치기, 발사체 등 물리적 충격/추력 연출|

### 2. 회전 함수

|**방식**|**주요 함수 / 프로퍼티**|**특징 및 사용 시점**|
|---|---|---|
|**Transform 자전/축 회전**|`transform.Rotate(Vector3)`|팽이처럼 계속 회전하거나 키 입력으로 회전할 때|
|**특정 타겟 바라보기**|`transform.LookAt(target)`|즉각적으로 타겟 방향을 정면으로 맞출 때|
|**방향 벡터로 쿼터니언 변환**|`Quaternion.LookRotation(direction)`|이동 방향 등을 바라보게 할 때 기준 각도 계산|
|**구면 선형 보간**|`Quaternion.Slerp(from, to, t)`|두 각도 사이를 최단 경로로 부드럽게 회전|
|**등속 회전 보간**|`Quaternion.RotateTowards(from, to, maxDegrees)`|정해진 각속도로 목표 각도까지 등속 회전|
|**물리 기반 회전**|`rigidbody.MoveRotation(Quaternion)`|물리 충돌체를 다른 오브젝트를 뚫지 않고 회전시킬 때|

- GameObject
- 