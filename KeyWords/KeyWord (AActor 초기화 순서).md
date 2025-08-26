```cpp
/** 액터 초기화는 여러 단계를 거치며, 호출되는 중요한 가상 함수들의 순서는 다음과 같음:
 * - UObject::PostLoad:
 *     레벨에 정적으로 배치된 액터의 경우, 일반적인 UObject의 PostLoad가 에디터와 게임플레이 모두에서 호출됨
 *     새로 스폰된 액터에 대해서는 호출되지 않음
 * 
 * - UActorComponent::OnComponentCreated:
 *     액터가 에디터에서 또는 게임플레이 중에 스폰될 때, 네이티브 컴포넌트들에 대해 이 함수가 호출됨
 *     블루프린트로 생성된 컴포넌트의 경우, 그 컴포넌트의 Construction 중에 호출됨
 *     레벨에서 로드된 컴포넌트에 대해서는 호출되지 않음
 * 
 * - AActor::PreRegisterAllComponents:
 *     정적으로 배치된 액터와 네이티브 루트 컴포넌트를 가진 스폰된 액터에 대해서는 이 시점에 호출됨
 *     네이티브 루트 컴포넌트가 없는 블루프린트 액터의 경우, 이러한 등록 함수들은 이후 Construction 중에 호출됨
 * 
 * - UActorComponent::RegisterComponent:
 *     모든 컴포넌트는 에디터와 런타임에서 등록되며, 이 과정에서 물리/시각적 표현이 생성됨
 *     이러한 호출은 여러 프레임에 걸쳐 분산될 수 있으나, 항상 PreRegisterAllComponents 이후에 이루어짐
 * 
 * - AActor::PostRegisterAllComponents:
 *     에디터와 게임플레이 모두에서 모든 액터에 대해 호출되며, 모든 경우에서 마지막으로 호출되는 함수임
 * 
 * - AActor::PostActorCreated:
 *     액터가 에디터나 게임플레이에서 생성될 때, Construction 직전에 호출됨
 *     레벨에서 로드된 컴포넌트에 대해서는 호출되지 않음
 * 
 * - AActor::UserConstructionScript:
 *     Construction Script를 구현한 블루프린트에 대해 호출됨
 * 
 * - AActor::OnConstruction:
 *     블루프린트 Construction Script를 호출하는 ExecuteConstruction의 끝에서 호출됨
 *     블루프린트로 생성된 모든 컴포넌트가 완전히 생성되고 등록된 이후에 호출됨
 *     게임플레이 중에는 스폰된 액터에 대해서만 호출되며, 에디터에서 블루프린트를 변경할 때 재실행될 수 있음
 * 
 * - AActor::PreInitializeComponents:
 *     액터의 컴포넌트에서 InitializeComponent가 호출되기 전에 호출됨
 *     게임플레이 중과 특정 에디터 프리뷰 창에서만 호출됨
 * 
 * - UActorComponent::Activate:
 *     이 컴포넌트가 bAutoActivate를 설정한 경우에만 호출됨
 *     컴포넌트가 bAutoActivate를 설정했다면 이후에도 호출될 수 있음
 * 
 * - UActorComponent::InitializeComponent:
 *     이 컴포넌트가 bWantsInitializeComponent를 설정한 경우에만 호출됨
 *     게임플레이 세션당 한 번만 일어남
 * 
 * - AActor::PostInitializeComponents:
 *     액터의 컴포넌트가 초기화된 후에 호출되며, 게임플레이와 일부 에디터 프리뷰에서만 호출됨
 * 
 * - AActor::BeginPlay:
 *     레벨이 틱을 시작할 때 호출되며, 실제 게임플레이에서만 호출됨
 *     일반적으로 PostInitializeComponents 직후에 발생하지만, 네트워크 환경이나 자식 액터의 경우 지연될 수 있음
 */

// 12 - Foundation - CreateWorld - AActor
// 해설: 
// - Actor:
//   - AActor == ActorComponent들의 컬렉션(-> 엔티티-컴포넌트 구조)
//   - ActorComponent의 예시:
//     - UStaticMeshComponent, USkeletalMeshComponent, UAudioComponent, ... 등
//   - (네트워킹) 상태 업데이트와 RPC 호출을 전파하기 위한 복제의 단위
//
// - Actor의 초기화:
//   1. UObject::PostLoad:
//     - AActor를 ULevel에 배치하면, AActor는 ULevel의 패키지에 저장됨
//     - 게임 빌드에서 ULevel이 스트리밍(혹은 로드)될 때, 배치된 AActor가 로드되고 UObject::PostLoad()가 호출됨
//       - 참고: UObject::PostLoad()는 [스폰되어야 할 오브젝트]→[로드되어야 할 에셋]→[로드 완료 후 후처리 이벤트인 UObject::PostLoad() 호출] 순서로 이루어짐
//   
//   2. UActorComponent::OnComponentCreated:
//     - AActor는 이미 ‘스폰’된 상태임
//     - UActorComponent별로 알림이 호출됨
//     - 기억할 점: ‘컴포넌트 생성’이 먼저!
//   
//   3. AActor::PreRegisterAllComponents:
//     - UActorComponent를 AActor에 등록하기 전 단계의 이벤트
//     - 기억할 점:
//       - *** [UActorComponent 생성] ---그 다음---> [UActorComponent 등록] ***
//   
//   4. UActorComponent::RegisterComponent:
//     - 점진적 등록: 여러 프레임에 걸쳐 UActorComponent를 AActor에 등록함
//     - 등록 단계가 완료되면 무엇이 이루어지는가?
//       - UActorComponent를 월드들(GameWorld[UWorld], RenderWorld[FScene], PhysicsWorld[FPhysScene], ...)에 등록
//       - 각 월드에서 요구하는 ***상태 초기화*** 수행
//   
//   5. AActor::PostRegisterAllComponents:
//     - UActorComponent 등록 이후의 후처리 이벤트
//
//   6. AActor::UserConstructionScript:
//     [ ] 에디터에서의 UserConstructionScript vs. SimpleConstructionScript 설명:
//        - UserConstructionScript: 블루프린트 이벤트 그래프에서 UActorComponent를 생성하는 함수가 호출됨
//        - SimpleConstructionScript: 블루프린트 뷰포트에서 UActorComponent를 구성(계층적)
//
/*   7. AActor::PreInitializeComponents:
 *     - UActor의 UActorComponent들을 초기화하기 전 단계의 이벤트
 *     - [UActorComponent 생성]→[UActorComponent 등록]→[UActorComponent 초기화]
 *
 *   8. UActorComponent::Activate:
 *     - UActorComponent를 초기화하기 전에, UActorComponent의 활성화가 먼저 호출됨
 *
 *   9. UActorComponent::InitializeComponent:
 *     - UActorComponent 초기화를 두 단계로 볼 수 있음:
 *       1. Activate
 *       2. InitializeComponent
 *
 *  10. AActor::PostInitializeComponents:
 *     - UActorComponent 초기화가 완료되었을 때의 후처리 이벤트
 * 
 *  11. AActor::BeginPlay:
 *     - 레벨이 틱을 시작하면, AActor는 BeginPlay()를 호출함
 */

// 다이어그램:
// ┌─────────────────────┐                                                                    
// │ UObject::PostLoad() │                                                                    
// └─────────┬───────────┘                                                                    
//           │                                                                                
//           │                                                                                
// ┌─────────▼───────────┐      ┌────────────────────────────────────────────────────────────┐
// │ AActor 스폰         ├─────►│                                                             │
// └─────────┬───────────┘      │  UActorComponent 생성:                                      │
//           │                  │   │                                                        │
//           │                  │   └──UActorComponent::OnComponentCreated()                 │
//           │                  │                                                            │
//           │                  ├────────────────────────────────────────────────────────────┤
//           │                  │                                                            │
//           │                  │  UActorComponent 등록:                                      │
//           │                  │   │                                                        │
/*           │                  │   ├──AActor::PreRegisterAllComponents()                    │
 *           │                  │   │                                                        │
 *           │                  │   ├──액터의 모든 UActorComponent에 대해                       │
 *           │                  │   │   │                                                    │
 *           │                  │   │   └──AActor::RegisterComponent()                       │
 *           │                  │   │                                                        │
 *           │                  │   └──AActor::PostRegisterAllComponents()                   │
 *           │                  │                                                            │
 *           │                  ├────────────────────────────────────────────────────────────┤
 *           │                  │  AActor::UserConstructionScript()                          │
 *           │                  ├────────────────────────────────────────────────────────────┤
 *           │                  │                                                            │
 *           │                  │  UActorComponent 초기화:                                    │
 *           │                  │   │                                                        │
 *           │                  │   ├──AActor::PreInitializeComponents()                     │
 *           │                  │   │                                                        │
 *           │                  │   ├──액터의 모든 UActorComponent에 대해                       │
 *           │                  │   │   │                                                    │
 *           │                  │   │   ├──UActorComponent::Activate()                       │
 *           │                  │   │   │                                                    │
 *           │                  │   │   └──UActorComponent::InitializeComponent()            │
 *           │                  │   │                                                        │
 *           │                  │   └──AActor::PostInitializeComponents                      │
 *           │                  │                                                            │
 *           │                  └────────────────────────────────────────────────────────────┘
 *           │                                                                                
 *  ┌────────▼───────────┐      ┌────────────────────┐                                        
 *  │ AActor 준비 완료    ├─────►│ AActor::BeginPlay()│                                        
 *  └────────────────────┘      └────────────────────┘                                        
 */
```


## 1) 전체 호출 순서(원문 흐름 유지)

1. `UObject::PostLoad`
    
2. `UActorComponent::OnComponentCreated`
    
3. `AActor::PreRegisterAllComponents`
    
4. `UActorComponent::RegisterComponent` ← _(원문 목록에 “AActor::RegisterComponent”로 적혀 있으나, 실제 등록 주체는 컴포넌트임)_
    
5. `AActor::PostRegisterAllComponents`
    
6. `AActor::PostActorCreated`
    
7. `AActor::UserConstructionScript`
    
8. `AActor::OnConstruction`
    
9. `AActor::PreInitializeComponents`
    
10. `UActorComponent::Activate` _(조건: `bAutoActivate`)_
    
11. `UActorComponent::InitializeComponent` _(조건: `bWantsInitializeComponent` / 세션당 1회)_
    
12. `AActor::PostInitializeComponents`
    
13. `AActor::BeginPlay`
    

> 원문 주의: `UActorComponent::InitializeComponent` 조건 플래그는 `bWantsInitializeComponent`이며, 제공 텍스트의 “bWantsInitializeComponentSet”은 오타로 보임.

---

## 2) 단계별 상세 (주석/해설/메모 포함)

### 1) `UObject::PostLoad`

- **역할**: 레벨에 **정적으로 배치된 액터**가 디스크에서 로드된 뒤의 후처리(직렬화 데이터 정합화).
    
- **언제**: 에디터/게임 공통, **배치 액터가 로드/스트리밍될 때**. **스폰된 액터에는 호출되지 않음**.
    
- **해설**: “배치된 AActor는 ULevel의 패키지에 저장 → ULevel 로드 시 AActor 로드 → `PostLoad()` 호출.”
    

---

### 2) `UActorComponent::OnComponentCreated`

- **역할**: **막 생성된 컴포넌트**에 대한 1회성 생성 알림.
    
- **언제**:
    
    - 스폰 경로: 액터 스폰 시 **네이티브 컴포넌트** 대상 호출.
        
    - 블루프린트 컴포넌트: **해당 컴포넌트의 Construction 중** 호출.
        
    - **레벨에서 로드된 컴포넌트**에는 호출되지 않음.
        
- **해설**: “기억할 점: **컴포넌트 생성이 먼저**! 이후에 등록 단계가 뒤따름.”
    

---

### 3) `AActor::PreRegisterAllComponents`

- **역할**: 액터 레벨에서 **컴포넌트 등록 직전** 사전 훅.
    
- **언제**:
    
    - 정적 배치 액터, 또는 **네이티브 루트 컴포넌트를 가진 스폰 액터**에서 **등록 직전**.
        
    - **네이티브 루트 없는 블루프린트 액터**는 **컨스트럭션 중**(뒤 단계)로 밀릴 수 있음.
        
- **해설**: “순서 핵심: **[컴포넌트 생성] → [컴포넌트 등록]**.”
    

---

### 4) `UActorComponent::RegisterComponent`

- **역할**: 컴포넌트의 **씬/물리/오디오 등 실제 월드 객체 생성 및 연결**. 내부적으로 `OnRegister()` 수행.
    
- **언제**: 에디터/런타임 모두. **여러 프레임에 분산**될 수 있음(레벨 스트리밍/대규모 등록 최적화).
    
- **규칙/주의**:
    
    - **루트 컴포넌트 먼저** 등록 → 자식이 안전하게 부모 정보(트랜스폼 등) 참조 가능.
        
    - 미등록 부모가 있으면 **부모부터** 등록(부모-자식 순서 보장).
        
    - **에디터 월드**: 이 시점에 액터 틱 함수를 **즉시 등록**하는 흐름이 흔함.  
        **게임 월드**: 보통 **BeginPlay 직전까지 틱 등록/활성 지연**(네트워크 타이밍 문제 방지).
        
- **해설**:
    
    - “등록이 끝나면 컴포넌트가 **GameWorld(UWorld) / RenderWorld(FScene) / PhysicsWorld(FPhysScene)** 등에 연결되고, 각 월드가 요구하는 **상태 초기화**가 수행됨.”
        
    - “점진적(Incremental) 등록 지원: 여러 프레임에 걸쳐 등록 가능.”
        

> 정정: 목록의 “AActor::RegisterComponent” 표기는 혼동을 유발. 실제 API 핵심은 **`UActorComponent::RegisterComponent`**임. 액터는 오케스트레이션(순서/루트 우선/부모 확인 등)을 담당.

---

### 5) `AActor::PostRegisterAllComponents`

- **역할**: **모든 컴포넌트 등록 완료 후** 액터 레벨 후처리.
    
- **언제**: 등록 분산이 모두 끝난 **완료 시점**.
    
- **해설**: “등록 완료 시점 표시 및 월드에 알림(예: `NotifyPostRegisterAllActorComponents`) 수행.”
    

---

### 6) `AActor::PostActorCreated`

- **역할**: **스폰된 액터**에서 Construction 직전 호출되는 1회성 초기 훅.
    
- **언제**: 에디터/런타임 공통의 **스폰 경로**.  
    **레벨에서 로드된(배치) 액터에는 호출되지 않음**.
    
- **원문 주석 이슈**: 제공 텍스트의 “this is not called for components loaded from a level”은 맥락상 **액터**를 가리킴. 의미는 “레벨에서 로드된 액터에는 호출되지 않음.”
    

---

### 7) `AActor::UserConstructionScript` (UCS, 블루프린트)

- **역할**: 블루프린트 **Construction Script** 실행. BP 생성 컴포넌트의 생성·설정·부착.
    
- **언제**: `ExecuteConstruction` 내부.  
    **에디터**에선 BP 변경/프로퍼티 변경/부모 트랜스폼 변경 시 **여러 번 재실행**될 수 있음.  
    **런타임**에선 **스폰된 액터에만** 호출(배치 액터는 이미 완성 상태로 로드됨).
    
- **해설**: “에디터에서의 **UserConstructionScript vs. SimpleConstructionScript(SCS)**:
    
    - **UCS**: 이벤트 그래프에서 컴포넌트 생성 함수 호출
        
    - **SCS**: BP 뷰포트의 계층에서 컴포넌트 구성”
        

---

### 8) `AActor::OnConstruction`

- **역할**: `ExecuteConstruction` **끝에서** 호출되는 C++ 훅. UCS 이후 **최종 구성 지점**.
    
- **언제**: UCS 직후.  
    **에디터**: BP/프로퍼티 변경 시 **재실행 가능**.  
    **런타임**: **스폰된 액터에 한해** 호출.
    
- **주의**: **반복 재실행 가능**하므로, 무거운 런타임 리소스 할당/네트워크 초기화 등은 여기서 지양(아래 Initialize/BeginPlay로 이동).
    

---

### 9) `AActor::PreInitializeComponents`

- **역할**: 컴포넌트의 `InitializeComponent` 호출 **직전** 액터 차원의 프리 훅.
    
- **언제**: **게임플레이**(및 일부 에디터 프리뷰)에서만.
    
- **해설**: “초기화 3단계 흐름 재강조: **[생성] → [등록] → [초기화]**.”
    

---

### 10) `UActorComponent::Activate`

- **역할**: **상태 On/Off**를 전환. 틱/입력/오디오 등을 **켜는** 스위치.
    
- **언제**: `bAutoActivate=true`이면 **자동으로**. 이후 상황 변화로 **여러 번** 호출될 수 있음.
    
- **해설**: “컴포넌트 **초기화 전에** 활성화가 선행될 수 있음.”
    

---

### 11) `UActorComponent::InitializeComponent`

- **역할**: **런타임 세션 1회성 초기화**(핸들 취득, 캐시 빌드 등).
    
- **언제**: `bWantsInitializeComponent=true`인 경우에만, **게임 세션당 1회**.
    
- **해설**: “초기화를 **두 단계**로 볼 수 있음: (1) Activate, (2) InitializeComponent.”
    

---

### 12) `AActor::PostInitializeComponents`

- **역할**: **모든 컴포넌트 InitializeComponent 완료 후** 액터 레벨 **최종 초기화 마감**.
    
- **언제**: 게임플레이/일부 에디터 프리뷰.
    
- **해설**: “여기서 `bActorInitialized` 플래그가 **true**로 굳어짐.”
    

---

### 13) `AActor::BeginPlay`

- **역할**: **게임 로직 시작**. 입력/AI/타이머 등 가동.
    
- **언제**: **레벨이 틱 시작**할 때. 보통 `PostInitializeComponents` 직후.  
    단, **네트워크/자식 액터/스트리밍** 사유로 **지연**될 수 있음.
    
- **해설**: “네트워크나 자식 액터 연결 상황에서 BeginPlay 진입 시점이 뒤로 밀릴 수 있음.”

---

### 참고
[액터 라이프 사이클 (Actor LifeCycle)](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle?application_version=5.4)
