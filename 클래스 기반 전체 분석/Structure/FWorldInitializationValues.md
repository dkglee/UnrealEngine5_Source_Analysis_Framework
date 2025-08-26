```cpp
/** 월드 초기화를 위한 선택적 매개변수들을 모아둔 구조체 */
// 이 패턴은 코드 가독성을 위해 여러 매개변수를 하나의 구조체에 캡슐화한 것으로 생각하면 된다
// - 이 구조체는 월드를 생성하는 데 필요한 모든 옵션을 포함한다
struct FWorldInitializationValues
{
    /** 씬들(물리, 렌더링 등)을 생성할지 여부 */
    // 렌더 월드, 물리 월드 등과 같은 월드들을 생성할지 여부
    uint32 bInitializeScenes:1;

    /** 물리 씬을 생성할지 여부. 이 옵션이 고려되기 위해서는 bInitializeScenes가 true여야 함. */
    uint32 bCreatePhysicsScene:1;

    /** 이 월드 내에서 충돌 트레이스 호출이 유효한지 여부 */
    uint32 bEnableTraceCollision:1;

    //...
};
```
---
### 🧩 역할 / 책임

- `UWorld` 생성 시 필요한 초기 설정 값을 담는 구조체
    
- 여러 개별 매개변수를 구조체 하나로 **캡슐화**하여 코드 가독성을 높임
    

### ⚙️ 기능

- 렌더링/물리/트레이스 시스템 초기화 여부를 세부적으로 설정 가능
    
- `UWorld::CreateWorld()` 또는 유사한 초기화 함수에서 사용
    

### 🔗 관계

#### 직접 포함

- `UWorld` 생성 루틴 내에서 `FWorldInitializationValues` 사용됨
    

#### 상속 계보

- 일반적인 POD(Plain Old Data) 스타일 구조체, 상속 없음
    

### 🧬 멤버 변수

|이름|타입|설명|
|---|---|---|
|`bInitializeScenes`|`uint32:1`|물리, 렌더링 등 씬들을 생성할지 여부|
|`bCreatePhysicsScene`|`uint32:1`|물리 씬을 생성할지 여부  <br>※ `bInitializeScenes = true`일 때만 유효|
|`bEnableTraceCollision`|`uint32:1`|충돌 트레이스 호출이 이 월드 내에서 유효한지 여부|
