```cpp
/** 
 * 월드는 맵 또는 샌드박스를 대표하는 최상위 객체로, 그 안에서 액터(Actor)와 컴포넌트(Component)가 존재하고 렌더링된다.
 * 
 * 월드는 하나의 퍼시스턴트 레벨(필요 시 스트리밍 레벨 목록 포함)로 구성될 수 있으며,
 * 스트리밍 레벨은 볼륨이나 블루프린트 함수로 로드/언로드된다.
 * 또는 World Composition(->haker: 오래된 주석...) 방식으로 여러 레벨을 구성할 수도 있다.
 * 
 * 스탠드얼론 게임에서는 일반적으로 하나의 월드만 존재한다.
 * 다만 시리스(seamless) 지역 전환 중에는 목적지 월드와 현재 월드가 동시에 존재할 수 있다.
 * 에디터에서는 여러 월드가 존재한다:
 * - 편집 중인 레벨
 * - 각 PIE 인스턴스
 * - 인터랙티브 렌더 뷰포트를 가진 각 에디터 툴 등
 */
class UWorld final : public UObject, public FNetworkNotify
{
    // FWorldInitializationValues 참조
    using InitializationValues = FWorldInitializationValues;

    /** 새로운 UWorld를 생성하여 포인터를 반환하는 정적 함수 */
    // 이 함수는 정적 함수임
    // - 함수의 파라미터를 볼 때, 타입을 유심히 보자
    // - 우리가 UWorld::CreateWorld()를 사용할 때는 보통 두 개의 파라미터만 넘기므로 나머지는 지금은 신경 쓰지 않아도 됨
    static UWorld* CreateWorld(
        const EWorldType::Type InWorldType, 
        bool bInformEngineOfWorld, 
        FName WorldName = NAME_None, 
        UPackage* InWorldPackage = NULL, 
        bool bAddToRoot = true, 
        ERHIFeatureLevel::Type InFeatureLevel = ERHIFeatureLevel::Num, 
        // haker:
        // InitializationValues 참조
        const InitializationValues* InIVS = nullptr, 
        bool bInSkipInitWorld = false
    );

    /** 모든 월드 서브시스템 초기화 */
    void InitializeSubsystems();

    /** 모든 월드 서브시스템의 초기화 완료 처리 */
    void PostInitializeSubsystems();

    /** 이 월드와 연결된 AWorldSettings 액터 반환 */
    AWorldSettings* GetWorldSettings(bool bCheckStreamingPersistent = false, bool bChecked = true) const;

    APhysicsVolume* InternalGetDefaultPhysicsVolume() const;

    /** 기본 물리 볼륨을 반환하며 필요 시 생성함 */
    // InternalGetDefaultPhysicsVolume 참조
    APhysicsVolume* GetDefaultPhysicsVolume() const;

    FLevelCollection* FindCollectionByType(const ELevelCollectionType InType);

    int32 FindOrAddCollectionByType_Index(const ELevelCollectionType InType);

    /** 아직 존재하지 않으면 동적 소스와 정적 레벨 컬렉션을 생성함 */
    void ConditionallyCreateDefaultLevelCollections();

    /** 월드를 초기화하고, 퍼시스턴트 레벨을 연결하며 적절한 존(영역)을 설정 */
    void InitWorld(const FWorldInitializationValues IVS = FWorldInitializationValues());

    /** 월드 컴포넌트(예: 라인 배처 및 모든 레벨 컴포넌트) 갱신 */
    void UpdateWorldComponents(bool bRerunConstructionScripts, bool bCurrentLevelOnly, FRegisterComponentContext* Context = nullptr);

    /** 새로 생성된 월드를 초기화 */
    void InitializeNewWorld(const InitializationValues IVS = InitializationValues(), bool bInSkipInitWorld = false);


    /** 이 월드를 로드할 때 사용된 URL */
    // URL을 패키지 경로로 생각하자:
    // - 예: Game\Map\Seoul\Seoul.umap
    FURL URL;

    /** 이 월드의 유형. 사용 컨텍스트(에디터, 게임, 프리뷰 등)를 설명함 */
    // 이미 EWorldType을 봤음
    // - TEnumAsByte는 enum 타입에 대한 비트 연산을 지원하는 헬퍼 래퍼 클래스
    // - 구현을 읽어보길 추천
    // 조언: C++ 프로그래머로서 비트 연산을 자유롭게 다루는 것은 매우 중요함!
    TEnumAsByte<EWorldType::Type> WorldType;

    /** 월드 정보, 기본 브러시, 게임플레이 중 스폰된 액터 등을 포함하는 퍼시스턴트 레벨 */
    // ULevel 참조
    // 월드 정보에 대한 간단한 설명
    TObjectPtr<class ULevel> PersistentLevel;

#if WITH_EDITORONLY_DATA || 1
    /** 현재 편집 중인 레벨에 대한 포인터; 레벨은 Levels 배열에 있어야 하며 게임에서는 PersistentLevel과 동일해야 함 */
    TObjectPtr<class ULevel> CurrentLevel;
#endif

    /** 현재 이 월드에 포함된 레벨 배열; 하드 레퍼런스를 피하기 위해 디스크에 직렬화되지 않음 */
    UPROPERTY(Transient)
    TArray<TObjectPtr<class ULevel>> Levels;

    /** 현재 이 월드에 존재하는 레벨 컬렉션 배열 */
    // UWorld는 ULevel에서 다루었던 레벨을 분류한 컬렉션들을 가짐
    TArray<FLevelCollection> LevelCollections;

    /** 현재 틱킹 중인 레벨 컬렉션의 인덱스 */
    int32 ActiveLevelCollectionIndex;

    /** 게임 전체에서 사용하는 기본 물리 볼륨 */
    // 물리 볼륨은 물리 엔진이 동작하는 3D 범위(= 물리 월드가 커버하는 영역)로 생각할 수 있음
    TObjectPtr<APhysicsVolume> DefaultPhysicsVolume;

    /** 이 월드의 물리 씬 */
    FPhysScene* PhysicsScene;

    // UWorldSubsystem 참조
    FObjectSubsystemCollection<UWorldSubsystem> SubsystemCollection;

    /** 라인 배처들: */
    // 디버그 라인
    // - ULineBatchComponent는 UPrimitiveComponent를 상속
    // - ULineBatchComponent는 UActorComponent처럼 볼 수 있음
    // - UWorld는 AActor가 아니지만 라인 배처용 동적 오브젝트로 UActorComponent를 가짐
    // - 라인 배처들은 별도로 등록되어야 함
    TObjectPtr<class ULineBatchComponent> LineBatcher;
    TObjectPtr<class ULineBatchComponent> PersistentLineBatcher;
    TObjectPtr<class ULineBatchComponent> ForegroundLineBatcher;

    //                                                                                ┌───WorldSubsystem0       
    //                                                        ┌────────────────────┐  │                         
    //                                                 World──┤SubsystemCollections├──┼───WorldSubsystem1       
    //                                                   │    └────────────────────┘  │                         
    //                                                   │                            └───WorldSubsystem2       
    //             ┌─────────────────────────────────────┴────┐                                                 
    //             │                                          │                                                 
    //           Level0                                     Level1                                              
    //             │                                          │                                                 
    //         ┌───┴────┐                                 ┌───┴────┐                                            
    //         │ Actor0 ├────Component0(RootComponent)    │ Actor0 ├─────Component0(RootComponent)              
    //         ├────────┤     │                           ├────────┤      │                                     
    //         │ Actor1 │     ├─Component1                │ Actor1 │      │   ┌──────┐                          
    //         ├────────┤     │                           ├────────┤      └───┤Actor2├──RootComponent           
    //         │ Actor2 │     └─Component2                │ Actor2 │          └──────┘   │                      
    //         ├────────┤                                 ├────────┤                     ├──Component0          
    //         │ Actor3 │                                 │ Actor3 │                     │                      
    //         └────────┘                                 └────────┘                     ├──Component1          
    //                                                                                   │   │                  
    //                                                                                   │   └──Component2      
    //                                                                                   │                      
    //                                                                                   └──Component3          
};
```
---
## 1) 클래스의 역할/책임
- **맵 또는 샌드박스 전체를 대표**하는 최상위 객체로, 그 안에서 액터(Actor)와 컴포넌트(Component)가 생성·관리·렌더링됩니다. 
	  
- **퍼시스턴트 레벨**(필요 시 스트리밍 레벨 포함)로 구성되며, 스트리밍 레벨은 볼륨/블루프린트 함수 등으로 로드·언로드됩니다.
	  
- 즉, `UWorld`는 `ULevel`들의 **로드, 언로드, 전환을 조율**하는 상위 컨트롤러 역할을 합니다.
	  
- StandAlone 게임에선 보통 하나의 월드만 존재하지만, **심리스(seamless) 지역 전환** 또는 **에디터 환경**에선 다수의 월드가 동시에 존재할 수 있습니다.

## 2) 클래스가 제공하는 기능
- `static UWorld* CreateWorld(...)` — 월드 인스턴스를 **정적으로 생성**하고 초기 설정을 수행.  
	  
- `void InitializeSubsystems()` / `void PostInitializeSubsystems()` — 월드 서브시스템 초기화 및 후처리.  
	  
- `void InitWorld(const FWorldInitializationValues IVS)` — 퍼시스턴트 레벨 연결 및 존(Zone) 설정 등 **월드 초기화**
	  .  
- `void UpdateWorldComponents(...)` — 라인 배처 및 레벨 컴포넌트 **갱신/재구성**.  
	  
- `AWorldSettings* GetWorldSettings(...)` — 월드에 연결된 **`AWorldSettings`** 액터 참조 반환.  
	  
- `APhysicsVolume* GetDefaultPhysicsVolume()` — **기본 물리 볼륨** 조회(필요 시 생성).  
	  
- 내부 헬퍼: 레벨 컬렉션 관리, 서브시스템 컬렉션 관리 등.

## 3) 관계
### 3.1 직접 상속(IsA)
- `UWorld`은 `UObject`를 **직접 상속**하며, `final` 이므로 추가 파생 클래스 없음.

### 3.2 포함(Has) ― **`UWorld`를 멤버/포인터로 보유하는 대표 코드**
| 포함하는 주체         | 형태                                               | 비고                          |
| --------------- | ------------------------------------------------ | --------------------------- |
| `FWorldContext` | `UWorld* World()`                                | `UEngine`가 관리하는 월드 컨텍스트 구조체 |
| `UEngine`       | `TIndirectArray<FWorldContext> WorldList` *(간접)* | 게임·에디터 전역에서 여러 월드를 보유·관리    |

## 4) 상속 계보(참고)
- `UObjectBase` ← `UObjectBaseUtility` ← `UObject` ← **`UWorld`**

## 5) 멤버 변수

| 멤버                                                                | 타입                                            | 간략 설명                       |
| ----------------------------------------------------------------- | --------------------------------------------- | --------------------------- |
| `FURL URL`                                                        | `FURL`                                        | 월드 로드에 사용된 **맵 URL/패키지 경로** |
| `WorldType`                                                       | `TEnumAsByte<EWorldType::Type>`               | 사용 컨텍스트(게임, 에디터, PIE 등)     |
| `PersistentLevel`                                                 | `TObjectPtr<ULevel>`                          | 월드의 **퍼시스턴트 레벨**            |
| `CurrentLevel`                                                    | `TObjectPtr<ULevel>` *(Editor-only)*          | 에디터에서 편집 중인 레벨              |
| `Levels`                                                          | `TArray<TObjectPtr<ULevel>>`                  | 월드에 포함된 **레벨 배열**           |
| `LevelCollections`                                                | `TArray<FLevelCollection>`                    | 레벨을 유형별로 분류한 **컬렉션들**       |
| `ActiveLevelCollectionIndex`                                      | `int32`                                       | 현재 틱킹 중인 컬렉션 인덱스            |
| `DefaultPhysicsVolume`                                            | `TObjectPtr<APhysicsVolume>`                  | 월드 전역 **기본 물리 볼륨**          |
| `PhysicsScene`                                                    | `FPhysScene*`                                 | 월드의 **물리 씬**                |
| `SubsystemCollection`                                             | `FObjectSubsystemCollection<UWorldSubsystem>` | 월드 서브시스템 모음                 |
| `LineBatcher` / `PersistentLineBatcher` / `ForegroundLineBatcher` | `TObjectPtr<ULineBatchComponent>`             | **디버그 라인 렌더링** 컴포넌트         |
### 특징
- `DefaultPhysicsVolume` - 해당 범위에서만 Physics가 동작한다고 보면 됨
- `ULineBatchComponent` - 이 경우 `UActorComponent`인데 어떻게 `UWorld`에서 선언되었고 어떻게 관리되는지 확인해보자!


### World의 관계 도식화

![[Pasted image 20250729161426.png]]

### 참고
[[ULevel]]
[[FLevelCollection]]
[[KeyWord (Dynamic Allocation_NewObject)]]
