```cpp
/**
 * 레벨 객체:
 * 레벨의 액터 목록, BSP 정보, 브러시 목록을 포함함
 * 모든 레벨은 자신의 Outer로 World를 가지며 PersistentLevel로 사용될 수 있다. 다만,
 * 레벨이 스트리밍되어 로드되었을 때는 OwningWorld가 그 레벨이 속한 World를 나타낸다
 */

/**
 * 레벨은 액터(라이트, 볼륨, 메시 인스턴스 등)의 컬렉션이다
 * 여러 레벨을 월드에 로드/언로드하여 스트리밍 경험을 만들 수 있다
 */

// - level == actors의 컬렉션:
//   - actors의 예시:
//     - light, static-mesh, volume, brush(예: BSP brush: binary-search-partitioning), ...
//       [ ] 에디터를 통해 BSP 브러시에 대해 설명하기
// - 나머지 내용은 지금은 건너뜀:
//   - world-partition, streaming 등 다른 토픽을 다룰 때 다시 살펴볼 것
class ULevel : public UObject
{
    // ULevel::ULevel() constructor
    ULevel(const FObjectInitializer& ObjectInitializer)
        : UObject(ObjectInitializer)
        // 함수 이름이 말해주듯, ULevel은 새로운 FTickTaskLevel을 할당함
        , TickTaskLevel(FTickTaskManagerInterface::Get().AllocateTickTaskLevel())
    {
    }

    /** 이 레벨이 PersistentLevel인지 여부를 반환 */
    bool IsPersistent() const;

    /**
     * 컴포넌트 등록을 점진적으로 수행
     * @param bPreRegisterComponents   PreRegisterAllComponents 호출 여부
     * @param NumComponentsToUpdate    이번 틱에 갱신(등록)할 컴포넌트 수(0이면 전부)
     * @param Context                  등록 컨텍스트(보류 중 추가 처리 등)
     * @return 모든 액터의 컴포넌트 등록이 끝났으면 true, 아니면 false
     *
     * 동작 개요:
     * - 다음으로 처리할 유효한 액터를 찾아서 컴포넌트 등록을 진행함
     * - bPreRegisterComponents가 true이고 해당 액터에 대해 사전 등록을 아직 안 했다면
     *   AActor::PreRegisterAllComponents()를 호출
     * - Actor->IncrementalRegisterComponents(...)로 실제 점진적 등록 수행
     * - 해당 액터의 모든 컴포넌트가 등록되면 다음 액터로 넘어감
     * - 점진적 등록(NumComponentsToUpdate != 0)인 경우, 각 액터 처리 후 바깥 루프로 제어를 돌려
     *   이번 프레임에서 계속 진행할지 결정할 수 있게 함
     * - 모든 액터를 다 처리하면 Context->Process()로 보류 중 추가를 먼저 처리하고 true 반환
     */
    bool IncrementalRegisterComponents(bool bPreRegisterComponents, int32 NumComponentsToUpdate, FRegisterComponentContext* Context);

    /**
     * 이 레벨에 속한 액터들의 모든 컴포넌트를 점진적으로 갱신
     * @param NumComponentsToUpdate        이번 틱에 갱신할 컴포넌트 수(0이면 전부)
     * @param bRerunConstructionScripts    Construction Script 재실행 여부
     * @param Context                      등록 컨텍스트(기본값 nullptr)
     *
     * 동작 개요:
     * - NumComponentsToUpdate가 0이 아니면(점진적 갱신), 게임 월드에서만 허용됨(Editor는 항상 0)
     * - 상태 머신(IncrementalComponentState)에 따라 다음 단계 수행:
     *   1) Init:
     *      - 부모 액터가 자식 액터보다 먼저 등록되도록 Actors를 계층 순서로 정렬
     *      - 다음 단계로 RegisterInitialComponents 설정
     *   2) RegisterInitialComponents:
     *      - IncrementalRegisterComponents(true, ...)로 초기 컴포넌트 등록을 진행
     *      - 에디터 기준(SCS 관련)으로, 아직 Construction Script를 재실행하지 않았고,
     *        bRerunConstructionScripts가 true이며 템플릿이 아니면
     *        다음 단계로 RunConstructionScripts, 아니면 Finalize로 이동
     *   3) RunConstructionScripts (에디터 전용 로직):
     *      - IncrementalRunConstructionScripts(...)가 완료되면 Finalize로 이동
     *   4) Finalize:
     *      - 상태 초기화(Init로 되돌림), 인덱스/플래그 리셋, bAreComponentsCurrentlyRegistered = true
     *      - CreateCluster() 호출(GC 관련 기능, 여기서는 세부 구현 생략)
     * - bFullyUpdateComponents(= NumComponentsToUpdate == 0)인 경우,
     *   모든 컴포넌트가 등록 완료될 때까지 루프
     * - 마지막에 OwningWorld의 물리 씬을 가져와 ProcessDeferredCreatePhysicsState()로
     *   보류 중 물리 상태 생성 처리
     */
    void IncrementalUpdateComponents(int32 NumComponentsToUpdate, bool bRerunConstructionScripts, FRegisterComponentContext* Context = nullptr);

    /**
     * 이 레벨(Actors 배열)에 연결된 액터들의 모든 컴포넌트를 갱신하고
     * BSP 모델 컴포넌트를 생성
     * @param bRerunConstructionScripts    Construction Script 재실행 여부
     * @param Context                      등록 컨텍스트(기본값 nullptr)
     *
     * 동작 개요:
     * - 한 번에 전체 컴포넌트를 갱신하기 위해 IncrementalUpdateComponents(0, ...)를 호출
     */
    void UpdateLevelComponents(bool bRerunConstructionScripts, FRegisterComponentContext* Context = nullptr);

    // 11 - Foundation - CreateWorld - ULevel's member variables

    /** 이 레벨의 모든 액터 배열로, FActorIteratorBase 및 파생 클래스에서 사용됨 */
    // haker: AActor 목록을 담는 멤버 변수
    // AActor 참조(goto 12)
    TArray<TObjectPtr<AActor>> Actors;

    /** 이 레벨이 포함된 캐시된 레벨 컬렉션 */
    // FLevelCollection 참조(goto 19)
    FLevelCollection* CachedLevelCollection;

    /**
     * 이 레벨을 Levels 배열에 보유하고 있는 월드
     * 스트리밍 레벨의 경우 GetOuter()는 사용되지 않는 잔존(vestigial) 월드를 가리키므로
     * 이것은 GetOuter()와 동일하지 않음
     * GC가 어떤 순서로든 일어날 수 있으므로 BeginDestroy() 중에는
     * 다른 UObject 참조와 마찬가지로 접근해서는 안 됨
     */
    // OwningWorld vs. OuterPrivate 이해
    // - 설명은 WorldComposition 기반 레벨 스트리밍이나 LevelBlueprint의 레벨 로드/언로드 조작을 바탕으로 함
    //   - World Partition은 보통 OwningWorld와 OuterPrivate가 동일한 다른 개념을 가짐
    // - 다이어그램:
    //      World0(OwningWorld)──[OuterPrivate]──►Package0(World.umap)
    //       ▲
    //       │
    // [OuterPrivate]
    //       │
    //       │
    //      Level0(PersistentLevel)
    //       │
    //       │
    //       ├────Level1──[OuterPrivate]──►World1───[OuterPrivate]───►Package1(Level1.umap)
    //       │
    //       └────Level2───────►World2───────►Package2(Level2.umap)
    TObjectPtr<UWorld> OwningWorld;

    enum class EIncrementalComponentState : uint8
    {
        Init,
        RegisterInitialComponents,
#if WITH_EDITOR || 1
        RunConstructionScripts,
#endif
        Finalize,
    };

    /** 이 레벨에서 점진적으로 액터 컴포넌트를 갱신하는 현재 단계 */
    // 이미 AActor의 초기화 단계를 다룸
    EIncrementalComponentState IncrementalComponentState;

    /** 현재 컴포넌트 갱신 대상으로 가리키는 액터가 PreRegisterAllComponents를 호출했는지 여부 */
    uint8 bHasCurrentActorCalledPreRegister : 1;

    /** 현재 컴포넌트들이 등록되어 있는지 여부 */
    uint8 bAreComponentsCurrentlyRegistered : 1;

    /** 컴포넌트 갱신을 위한 액터 배열의 현재 인덱스 */
    // 점진적 갱신을 지원하기 위해 ULevel의 ActorList에서 액터 인덱스를 추적
    int32 CurrentActorIndexForIncrementalUpdate;

    /** 틱 함수들을 보관하는 데이터 구조 */
    // 지금은 틱 함수 관련 멤버 변수는 건너뜀
    FTickTaskLevel* TickTaskLevel;
};
```
---

## 1) 클래스의 역할/책임
- **레벨(맵)의 핵심 데이터 컨테이너**로, 액터 목록·BSP/브러시 정보 등을 보유합니다.  
- 모든 레벨은 `UWorld`를 Outer로 가지며, **퍼시스턴트 레벨** 또는 **스트리밍 레벨**로 로드될 수 있습니다.  
- 스트리밍 시스템을 통해 여러 레벨을 월드에 동적 로드/언로드하여 **시 seamless**(지역 전환)·**월드 파티션** 등 대규모 맵 기능을 지원합니다.

## 2) 클래스가 제공하는 기능
| API | 개요(주석 기준) |
|-----|------------------|
| `bool IsPersistent() const` | 이 레벨이 **퍼시스턴트 레벨인지** 여부 반환 |
| `bool IncrementalRegisterComponents(...)` | **점진적 컴포넌트 등록** 수행 |
| `void IncrementalUpdateComponents(...)` | 액터 컴포넌트 **점진적 갱신** |
| `void UpdateLevelComponents(...)` | 한 번에 **모든 컴포넌트 갱신** 및 BSP 모델 컴포넌트 생성 |

## 3) 관계
### 3.1 직접 상속(IsA)
- `ULevel`을 직접 파생하는 클래스는 **없음** (엔진 기본 구현에서 `final` 아님이지만 실질적 서브클래스 미존재)

### 3.2 포함(Has) ― **`ULevel`을 멤버/포인터로 보유하는 대표 코드**
| 포함하는 주체              | 형태                                                          | 비고                    |
| -------------------- | ----------------------------------------------------------- | --------------------- |
| `UWorld`             | `PersistentLevel`, `CurrentLevel`, `TArray<ULevel*> Levels` | 월드가 여러 레벨을 **소유·관리**  |
| `ULevelStreaming` 계열 | `ULevel* LoadedLevel`                                       | 스트리밍 시스템이 로드한 레벨 인스턴스 |

## 4) 상속 계보(참고)
- `UObjectBase` ← `UObjectBaseUtility` ← `UObject` ← **`ULevel`**

## 5) 멤버 변수
| 멤버                                      | 타입                           | 설명                                         |
| --------------------------------------- | ---------------------------- | ------------------------------------------ |
| `Actors`                                | `TArray<TObjectPtr<AActor>>` | 이 레벨에 배치된 **액터 배열**                        |
| `CachedLevelCollection`                 | `FLevelCollection*`          | 레벨이 속한 **레벨 컬렉션**(캐시)                      |
| `OwningWorld`                           | `TObjectPtr<UWorld>`         | 레벨을 **소유하는 월드** 포인터                        |
| `IncrementalComponentState`             | `EIncrementalComponentState` | 점진적 컴포넌트 갱신 **상태 머신 단계**                   |
| `bHasCurrentActorCalledPreRegister`     | `uint8 : 1`                  | 현재 액터가 `PreRegisterAllComponents` 호출 완료 여부 |
| `bAreComponentsCurrentlyRegistered`     | `uint8 : 1`                  | 현재 컴포넌트가 **등록 완료** 상태인지                    |
| `CurrentActorIndexForIncrementalUpdate` | `int32`                      | 점진적 갱신 시 **Actors 배열 인덱스**                 |
| `TickTaskLevel`                         | `FTickTaskLevel*`            | 틱 함수들을 보관하는 **레벨별 TickTask 데이터**           |

### 참고
[[KeyWord (OwningWorld vs. OuterPrivate)]]
