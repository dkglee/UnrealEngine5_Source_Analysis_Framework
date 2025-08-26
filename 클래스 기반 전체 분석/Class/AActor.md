```cpp
/**
 * 액터(Actor)는 레벨에 배치되거나 스폰될 수 있는 오브젝트의 기본 클래스임
 * 액터는 ActorComponent의 컬렉션을 포함할 수 있으며, 이를 통해 액터가 어떻게 이동하고 어떻게 렌더링되는지 등을 제어할 수 있음
 * 액터의 또 다른 주요 기능은 플레이 중 네트워크 전반에 걸쳐 속성과 함수 호출을 복제(Replication)하는 것임
 */

// 액터의 멤버 변수 참고
class AActor : public UObject
{
    /** 이 액터의 루트 컴포넌트를 반환함 */
    USceneComponent* GetRootComponent() const;

    /** 
     * RootComponent로부터 부착 체인을 따라 올라가다가 다른 액터를 만나면 그 액터를 반환함 
     * 다른 액터의 컴포넌트에 부착되어 있지 않다면 nullptr을 반환함
     */
    AActor* GetAttachParentActor() const;

    /** Components 배열의 모든 컴포넌트가 등록되기 전에 호출됨 */
    virtual void PreRegisterAllComponents();

    /** 이 액터가 속한 ULevel을 반환함 */
    ULevel* GetLevel() const;

    /** 
     * null 포인터가 제거된 복사본이 아닌, 컴포넌트 집합에 대한 직접 참조를 가져옴 
     * 경고: 소유권 변경 또는 파괴를 유발할 수 있는 어떤 작업도 이 배열을 무효화하므로,
     * 이 집합을 순회할 때 주의할 것!
     */
    const TSet<UActorComponent*>& GetComponents() const;

    /** 캐시된 월드 포인터의 게터. 액터가 실제로 레벨에 스폰되지 않았다면 null을 반환함 */
    virtual UWorld* GetWorld() const override final;

    /** 액터 클래스 계층에 대한 모든 틱 함수 등록을 위한 가상 호출 체인 */
    virtual void RegisterActorTickFunctions(bool bRegister);

    /** 호출 시, 액터 및 선택적으로 모든 컴포넌트의 모든 틱 함수를 등록하는 가상 호출 체인을 호출함 */
    void RegisterAllActorTickFunctions(bool bRegister, bool bDoComponents);

    /** 액터가 BeginPlay()를 호출받았는지(이후 EndPlay()로 되돌려지지 않았는지) 여부를 반환함 */
    bool HasActorBegunPlay() const;

    /** 액터가 BeginPlay 중인지 여부를 반환함 */
    bool IsActorBeginningPlay() const;

    /** 
     * 컴포넌트의 초기화를 마무리하고, 적절한 시점이라면 틱 함수 등록과 BeginPlay()를 수행함
     */
    void HandleRegisterComponentWithWorld(UActorComponent* Component);

    /**
     * 주어진 타입의 모든 컴포넌트에 대해 컴파일 타임 람다를 호출하는 내부 헬퍼 함수
     * ComponentClass가 정확히 UActorComponent인 경우 불필요한 IsA 체크를 피하기 위해
     * 템플릿 매개변수 bClassIsActorComponent를 사용할 것
     * 적절한 타입의 컴포넌트를 자식 액터에서도 검색하도록 하려면
     * 템플릿 매개변수 bIncludeFromChildActors를 사용하여 ChildActor 컴포넌트로 재귀 접근할 것
     */
    template <class ComponentType, bool bClassIsActorComponent, bool bIncludeFromChildActors, typename Func>
    void ForEachComponent_Internal(TSubclassOf<UActorComponent> ComponentClass, Func InFunc) const;

    /**
     * UActorComponent 특수화 버전의 ForEachComponent_Internal 오버로드
     * bIncludeFromChildActors 플래그에 따라 자식 액터 포함 여부를 제어함
     */
    template <class ComponentType, typename Func>
    void ForEachComponent_Internal(TSubclassOf<UActorComponent> ComponentClass, bool bIncludeFromChildActors, Func InFunc) const;

    /**
     * UActorComponent에 대한 GetComponents() 특수화로, 불필요한 캐스트를 피하기 위함
     * 메모리 할당 비용을 피하기 위해 TInlineAllocator를 사용하는 TArray 사용을 권장함
     * 이를 쉽게 하기 위해 TInlineComponentArray가 정의되어 있음. 예:
     * {
     *   TInlineComponentArray<UActorComponent*> PrimComponents;
     *   Actor->GetComponents(PrimComponents);
     * }
     */
    template <class AllocatorType>
    void GetComponents(TArray<UActorComponent*, AllocatorType>& OutComponents, bool bIncludeFromChildActors = false);

    /**
     * Components 배열의 모든 컴포넌트가 등록된 후에 호출되며,
     * 에디터와 게임플레이 모두에서 호출됨
     * 이 함수를 호출하기 전에는 bHasRegisteredAllComponents가 true로 설정되어 있어야 함
     */
    virtual void PostRegisterAllComponents();

    /** 레벨 스트리밍 중에 사용되는, 이 액터와 연관된 컴포넌트들을 점진적으로 등록함 */
    bool IncrementalRegisterComponents(int32 NumComponentsToRegister, FRegisterComponentContext* Context = nullptr);

    /** 액터가 게임플레이를 위해 초기화되었는지 여부를 반환함 */
    bool IsActorInitialized() const;

    // 13 - Foundation - CreateWorld - AActor의 멤버 변수

    /** 
     * 기본 액터 틱 함수로, TickActor()를 호출함 
     * - 틱 함수는 틱 활성화 여부, 프레임 내 업데이트 시점, 틱 의존성 설정 등을 구성할 수 있음
     */
    // 해설: 언리얼 엔진 코드는 상태(클래스)와 틱을 FTickFunction으로 분리함
    // - 이는 이후 자세히 다룰 예정
    // FActorTickFunction 참고
    struct FActorTickFunction PrimaryActorTick;

    /** 이 액터가 소유한 모든 ActorComponent; 액터는 다수의 컴포넌트를 가질 수 있으므로 Set으로 보관됨 */
    // 해설: AActor의 모든 컴포넌트는 여기 저장됨
    // UActorComponent 참고
    TSet<TObjectPtr<UActorComponent>> OwnedComponents;

    /**
     * 이 액터에서 PreInitializeComponents/PostInitializeComponents가 호출되었는지 나타냄
     * 레벨 시작 중에 스폰된 액터가 다시 초기화되는 것을 방지함 
     */
    // 해설: PostInitializeComponents가 호출되면 bActorInitialized가 'true'로 설정됨
    uint8 bActorInitialized : 1;

    /** 틱 함수 등록을 시도했는지 여부; 등록이 해제되면 리셋됨 */
    // 해설: 틱 함수가 등록되었는지 여부
    uint8 bTickFunctionsRegistered : 1;

    /** BeginPlay가 시작되었는지 또는 완료되었는지 정의하는 열거형 */
    enum class EActorBeginPlayState : uint8
    {
        HasNotBegunPlay,
        BeginningPlay,
        HasBegunPlay,
    };

    /** 
     * 이 액터에 대해 BeginPlay가 호출되었는지 나타냄 
     * EndPlay가 호출되면 다시 HasNotBegunPlay로 되돌아감
     */
    // 해설: 매우 흥미로운 점!
    // - uint8 비트 할당이 enum 타입의 중간에도 적용됨!
    // - ActorHasBegunPlay는 BeginPlay() 또는 EndPlay() 호출에 의해 갱신됨
    EActorBeginPlayState ActorHasBegunPlay : 2;

    /** 
     * 이 액터의 월드 내 변환(위치, 회전, 스케일)을 정의하는 컴포넌트
     * 다른 모든 컴포넌트는 어떤 방식으로든 이 컴포넌트에 부착되어야 함
     */
    // 해설: AActor는 USceneComponent를 이용한 씬 그래프 형태로 컴포넌트(UActorComponent)를 관리함
    // [ ] 블루프린트 뷰포트(SCS[Simple Construction Script]) 참고
    //
    // 다이어그램:                                                                                       
    // AActor                                                                                
    //  │                                                                                    
    //  ├──OwnedComponents: TSet<TObjectPtr<UActorComponent>>                                 
    //  │    ┌──────────────────────────────────────────────────────────────────────────┐    
    //  │    │ [Component0, Component1, Component2, Component3, Component4, Component5] │    
    //  │    │                                                                          │    
    //  │    └──────────────────────────────────────────────────────선형 형식────────── ─┘    
    //  └──RootComponent: TObjectPtr<USceneComponent>                                        
    //       ┌────────────────────────────┐                                                  
    //       │ Component0 [RootComponent] │                                                  
    //       │  │                         │                                                  
    //       │  ├──Component1             │                                                  
    //       │  │   │                     │                                                  
    //       │  │   ├──Component2         │                                                  
    //       │  │   │                     │                                                  
    //       │  │   └──Component3         │                                                  
    //       │  │       │                 │                                                  
    //       │  │       └──Component4     │                                                  
    //       │  │                         │                                                  
    //       │  └──Component5             │                                                  
    //       │                            │                                                  
    //       └────계층 형식─────────────── ┘                                                  
    //                                                                                       
    // USceneComponent 참고
    TObjectPtr<USceneComponent> RootComponent;

    /** 이 액터를 소유하는 UChildActorComponent */
    // 해설: UChildActorComponent를 자세히 다루지는 않지만, 개념을 설명하면
    // - UChildActorComponent는 액터와 액터 사이의 연결을 지원함
    // - 기본적으로는 액터-액터컴포넌트 연결만 있는데, 액터-액터 연결은 어떻게 지원하는가?
    //   - 다이어그램 참조:                                                                                               
    //     ┌──────────┐                         ┌──────────┐                           ┌──────────┐              
    //     │  Actor0  ├────────────────────────►│  Actor1  ├──────────────────────────►│  Actor2  │              
    //     └──────────┘                         └──────────┘                           └──────────┘              
    //                                                                                                           
    //     RootComponent                        RootComponent ◄──────────┐             RootComponent             
    //      │                                    │                       │              │                        
    //      ├───Component1◄──────────┐           ├───Component1     Actor2-Actor1       └───Component1           
    //      │                        │           │                       │                                       
    //      ├───Component2    Actor1-Actor0      └───Component2          └─────────────ParentComponent           
    //      │    │                   │                                                                           
    //      │    └───Component3      └──────────ParentComponent                                                  
    //      │                                                                                                    
    //      └───Component4                                                                                       
    //                                                                                                           
    //                 ┌──────────────────────────────────────┐                                                  
    //                 │                                      │                                                  
    //                 │      Actor0 ◄───                     │                                                  
    //                 │       │                              │                                                  
    //                 │       └─RootComponent                │                                                  
    //                 │          │                           │                                                  
    //                 │          └─Component1                │                                                  
    //                 │             │                        │                                                  
    //                 │             └─Actor1 ◄───            │                                                  
    //                 │                │                     │                                                  
    //                 │                └─RootComponent       │                                                  
    //                 │                   │                  │                                                  
    //                 │                   └─Actor2 ◄────     │                                                  
    //                 │                                      │                                                  
    //                 └──────────────────────────────────────┘                                                  
    TWeakObjectPtr<UChildActorComponent> ParentComponent;
};

```
---
## 1) 클래스의 역할/책임
- 레벨에 **배치**(정적)되거나 **스폰**(동적)될 수 있는 모든 오브젝트의 **기본 클래스**.  
- **ActorComponent 컬렉션**을 통해 이동·렌더링·물리·오디오 등 다양한 기능을 구성하며, 네트워크 환경에서는 **속성 및 함수 복제(Replication)**의 단위가 됩니다.  
- 액터는 복잡한 **초기화 단계**를 거치며, `PostLoad` → `OnComponentCreated` → `PreRegisterAllComponents` → … → `BeginPlay` 순으로 핵심 가상 함수들이 호출됩니다.
- **액터 - 컴포넌트 연결**로 `UActorComponent`들을 관리한다.

## 2) 클래스가 제공하는 기능 <sup>(일부 발췌)</sup>
| API | 개요(주석 기반) |
|-----|----------------|
| `USceneComponent* GetRootComponent() const` | 액터의 **루트 컴포넌트** 반환 |
| `AActor* GetAttachParentActor() const` | 부착 체인을 따라 **부모 액터** 검색 |
| `virtual void PreRegisterAllComponents()` | 모든 컴포넌트 **등록 전 훅** |
| `ULevel* GetLevel() const` | 이 액터가 속한 **레벨** 반환 |
| `const TSet<UActorComponent*>& GetComponents() const` | **OwnedComponents** 직접 참조 |
| `virtual UWorld* GetWorld() const override` | 현재 **월드** 포인터 반환 |
| `void RegisterActorTickFunctions(bool bRegister)` | 클래스 계층의 **틱 함수** 등록/해제 |
| `bool HasActorBegunPlay() const` / `IsActorBeginningPlay() const` | **BeginPlay 상태** 쿼리 |
| `void HandleRegisterComponentWithWorld(UActorComponent* Comp)` | 컴포넌트 월드 등록 후처리 |
| `bool IncrementalRegisterComponents(int32 Num, FRegisterComponentContext* Ctx)` | **점진적 컴포넌트 등록** |
| `bool IsActorInitialized() const` | `PostInitializeComponents` 완료 여부 |

## 3) 관계
### 3.1 직접 상속(IsA)
- 대표적인 파생 클래스  
  - `APawn`, `ACharacter`  
  - `AStaticMeshActor`, `ASkeletalMeshActor`  
  - `ALight`, `ADirectionalLight`, `APointLight`  
  - `ACameraActor`, `APlayerController`, `AGameModeBase`  
  - (대부분의 게임 플레이 클래스가 `AActor`를 직접 또는 간접 상속)

### 3.2 포함(Has) ― **`AActor`를 멤버/포인터로 보유**
| 포함하는 주체 | 형태 | 비고 |
|---------------|------|------|
| `ULevel` | `TArray<AActor*> Actors` | 레벨이 **액터 배열**로 소유 |
| `UWorld` | `PersistentLevel->Actors`, `Levels[i]->Actors` | 월드를 통해 간접 포함 |

## 4) 상속 계보(참고)
- `UObjectBase` ← `UObjectBaseUtility` ← `UObject` ← **`AActor`**

## 5) 멤버 변수
| 멤버                             | 타입                                     | 설명                                                                                                          |
| ------------------------------ | -------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `PrimaryActorTick`             | `FActorTickFunction`                   | `TickActor()` 호출용 **기본 틱 함수**                                                                               |
| `OwnedComponents`              | `TSet<TObjectPtr<UActorComponent>>`    | 액터가 **소유한 모든 컴포넌트**                                                                                         |
| `bActorInitialized : 1`        | `uint8`                                | `PostInitializeComponents` 완료 플래그                                                                           |
| `bTickFunctionsRegistered : 1` | `uint8`                                | 틱 함수 **등록 여부**                                                                                              |
| `ActorHasBegunPlay : 2`        | `EActorBeginPlayState`                 | `BeginPlay` 진행/완료 상태                                                                                        |
| `RootComponent`                | `TObjectPtr<USceneComponent>`          | 씬 그래프의 **루트 노드**<br><br>액터의 월드 내 **변환**(위치, 회전, 스케일)을 정의하는 컴포넌트<br><br>다른 모든 컴포넌트는 어떤 방식으로든 이 컴포넌트에 부착되어야 함 |
| `ParentComponent`              | `TWeakObjectPtr<UChildActorComponent>` | 이 액터를 소유하는 **ChildActorComponent** (있을 경우)                                                                  |

### 6) 특징

**1. 컴퍼넌트의 계층 구조**
- `ActorComponent` 및 모든 컴퍼넌트는 `OwnedComponents`에 고유하게 소유하게 됨
	  
- 하지만 계층적 구조를 표현(Blueprint에서의 Viewport의 계층 구조)하기 위하여 `RootComponent`를 둠으로써 이를 가능하게 함.
	  
- `USceneComponent`를 통하여 **Scene Graph(씬 그래프)** 의 형태로 모든 `Component`를 관리할 수 있음

**2. 액터의 계층 구조**
- 액터는 레벨에 속해있는 객체임으로 **ActorComponent와 달리** 상속 관계를 가질 수 없음.
	  
- 하지만 `ParentComponent`를 통해 마치 계층 구조를 표현할 수는 있음.

## 참고
[[KeyWord (AActor 초기화 순서)]]
[[KeyWord (ParentComponent)]]
[[KeyWord (Bit Assignment)]]
