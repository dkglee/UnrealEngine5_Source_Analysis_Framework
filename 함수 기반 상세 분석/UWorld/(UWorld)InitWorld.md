```cpp
/** 월드를 초기화하고, 퍼시스턴트 레벨을 연결하며, 적절한 존을 설정한다 */
void UWorld::InitWorld(const FWorldInitializationValues IVS = FWorldInitializationValues())
{
    // CoreUObjectDelegates에는 GC 이벤트와 관련된 델리게이트가 있음 -> 유용!
    FCoreUObjectDelegates::GetPostGarbageCollect().AddUObject(this, &UWorld::OnPostGC);

    // UWorldSubsystem들을 초기화한다
    // UWorld::InitializeSubsystems 참조
    InitializeSubsystems();

    FWorldDelegates::OnPreWorldInitialization.Broadcast(this, IVS);

    // UWorld::GetWorldSettings 참조
    AWorldSettings* WorldSettings = GetWorldSettings();
    if (IVS.bInitializeScenes)
    {
        if (IVS.bCreatePhysicsScene)
        {
            // 물리 씬의 상세는 생략
            // - 물리 월드 생성
            CreatePhysicsScene(WorldSettings);
        }

        bShouldSimulatePhysics = IVS.bShouldSimulatePhysics;

        // 렌더 월드(== FScene) 생성
        // - 렌더 월드는 FScene. 이 부분은 나중에 다룸
        bRequiresHitProxies = IVS.bRequiresHitProxies;
        GetRendererModule().AllocateScene(this, bRequiresHitProxies, IVS.bCreateFXSystem, GetFeatureLevel());
    }
    
    // 퍼시스턴트 레벨 추가
    // - 퍼시스턴트 레벨이 월드 정보(AWorldSettings)를 가진다고 보면 됨
    Levels.Empty(1);
    Levels.Add(PersistentLevel);

    PersistentLevel->OwningWorld = this;
    PersistentLevel->bIsVisible = true;

    // 월드의 DefaultPhysicsVolume 초기화
    // 이 함수가 필요 시 스폰함
    
    // GetDefaultPhysicsVolume 참조
    DefaultPhysicsVolume = GetDefaultPhysicsVolume();

    // 중력 설정
    if (GetPhysicsScene())
    {
        FVector Gravity = FVector( 0.f, 0.f, GetGravityZ() );
        GetPhysicsScene()->SetUpForFrame( &Gravity, 0, 0, 0, 0, 0, false);
    }

    // 물리 충돌 핸들러 생성
    if (IVS.bCreatePhysicsScene)
    {
        //...
    }

    // 여기부터는 UWorld의 URL이 퍼시스턴트 레벨의 URL에서 온다
    URL = PersistentLevel->URL;
#if WITH_EDITORONLY_DATA || 1
	CurrentLevel = PersistentLevel;
#endif

    // ConditionallyCreateDefaultLevelCollections 참조
    // 모든 레벨 컬렉션 타입들이 인스턴스화되어 있는지 보장
    ConditionallyCreateDefaultLevelCollections();

    // 이제 초기화됨:
    bIsWorldInitialized = true;
    bHasEverBeenInitialized = true;

    FWorldDelegate::OnPostWorldInitialization.Broadcast(this, IVS);

    PersistentLevel->OnLevelLoaded();

    // 새로운 레벨이 추가되었음을 스트리밍 매니저에 알림
    IStreamingManager::Get().AddLevel(PersistentLevel);

    // PostInitializeSubsystems 참조
    // 각 UWorldSubsystem에 대해 PostInitialize() 호출
    PostInitializeSubsystems();

    BroadcastLevelsChanged();

    // InitWorld()가 수행한 작업 정리:
    // 1. WorldSubsystemCollection 초기화
    // 2. 렌더 월드(FScene)와 물리 월드(FPhysScene) 할당(생성)
}
```
---
## 상세 설명
### 1. GC 이후 콜백 등록
```cpp
    // CoreUObjectDelegates에는 GC 이벤트와 관련된 델리게이트가 있음 -> 유용!
    FCoreUObjectDelegates::GetPostGarbageCollect().AddUObject(this, &UWorld::OnPostGC);
```
- `FCoreUObjectDelegates::GetPostGarbageCollect()`에 `UWorld::OnPostGC`를 바인딩해 GC 후처리를 준비

### 2. 월드 서브시스템 초기화
```cpp
	// UWorldSubsystem들을 초기화한다
    // UWorld::InitializeSubsystems 참조
    InitializeSubsystems();
```
- `InitializeSubsystems()`로 `UWorldSubsystem` 인스턴스들을 생성·초기화
	  
- `UWorld::SubsystemCollection`을 구축

### 3. Pre-World 초기화 브로드캐스트
```cpp
	FWorldDelegates::OnPreWorldInitialization.Broadcast(this, IVS);
```
- `FWorldDelegates::OnPreWorldInitialization.Broadcast(this, IVS)`로 초기화 직전 훅을 알림

### 4. 월드 세팅 확보
```cpp
    // UWorld::GetWorldSettings 참조
    AWorldSettings* WorldSettings = GetWorldSettings();
```
- `GetWorldSettings()`로 `AWorldSettings` 포인터를 가져옴

### 5. 씬(렌더/물리) 초기화(옵션)
```cpp
    // UWorld::GetWorldSettings 참조
    AWorldSettings* WorldSettings = GetWorldSettings();
    if (IVS.bInitializeScenes)
    {
        if (IVS.bCreatePhysicsScene)
        {
            // 물리 씬의 상세는 생략
            // - 물리 월드 생성
            CreatePhysicsScene(WorldSettings);
        }

        bShouldSimulatePhysics = IVS.bShouldSimulatePhysics;

        // 렌더 월드(== FScene) 생성
        // - 렌더 월드는 FScene. 이 부분은 나중에 다룸
        bRequiresHitProxies = IVS.bRequiresHitProxies;
        GetRendererModule().AllocateScene(this, bRequiresHitProxies, IVS.bCreateFXSystem, GetFeatureLevel());
    }
```
- `IVS.bInitializeScenes`가 `true`면 렌더 월드(`FScene`) 생성
	  
- `IVS.bCreatePhysicsScene`이 `true`면 물리 월드(`FPhysScene`) 생성

### 6. 퍼시스턴트 레벨 연결·가시화
```cpp
    // 퍼시스턴트 레벨 추가
    // - 퍼시스턴트 레벨이 월드 정보(AWorldSettings)를 가진다고 보면 됨
    Levels.Empty(1);
    Levels.Add(PersistentLevel);

    PersistentLevel->OwningWorld = this;
    PersistentLevel->bIsVisible = true;
```
- `Levels` 배열 초기화 후 `PersistentLevel` 추가
	  
- `OwningWorld` 지정, `bIsVisible = true`로 설정

### 7. 기본 물리 볼륨 확보
```cpp
	// GetDefaultPhysicsVolume 참조
    DefaultPhysicsVolume = GetDefaultPhysicsVolume();
```
- `DefaultPhysicsVolume = GetDefaultPhysicsVolume()` 생성 혹은 참조
	  
- `IVS.bCreatePhysicsScene`가 `false`이더라도 생성해줘야 함.
	  
	- 추측건데, `LineTrace` 및 `Collision`등의 충돌 검사도 이거를 통해서 이루어지는 것으로 보임
	  
	- 혹은 `PhysicsScene`이 반드시 있어야 할 듯?

### 8. 중력 설정(물리 씬 존재 시) 후 충돌 핸들러 설정

```cpp
    // 중력 설정
    if (GetPhysicsScene())
    {
        FVector Gravity = FVector( 0.f, 0.f, GetGravityZ() );
        GetPhysicsScene()->SetUpForFrame( &Gravity, 0, 0, 0, 0, 0, false);
    }

    // 물리 충돌 핸들러 생성
    if (IVS.bCreatePhysicsScene)
    {
        //...
    }
```
- `GetGravityZ()` 기반 중력을 적용
	  
- `IVS.bCreatePhysicsScene`가 `true`면 충돌 핸들러 등 물리 관련 후속 객체 구성

### 9. 월드 URL 동기화·현재 레벨 지정
```cpp
    // 여기부터는 UWorld의 URL이 퍼시스턴트 레벨의 URL에서 온다
    URL = PersistentLevel->URL;
#if WITH_EDITORONLY_DATA || 1
	CurrentLevel = PersistentLevel;
#endif
```
- `URL = PersistentLevel->URL;`로 동기화
	  
- `CurrentLevel = PersistentLevel;`로 현재 레벨 지정

### 10. 기본 레벨 컬렉션 보장
```cpp
    // ConditionallyCreateDefaultLevelCollections 참조
    // 모든 레벨 컬렉션 타입들이 인스턴스화되어 있는지 보장
    ConditionallyCreateDefaultLevelCollections();
```
- `ConditionallyCreateDefaultLevelCollections()`로 필요한 `ELevelCollectionType` 컬렉션들을 모두 갖춤
	  
- 이전에 [[FLevelCollection]]에 대해서 다루었는데 모든 `Dynamic` 혹은 `Static` 레벨 컬렉션들을 생성·참조하여 `LevelCollections` 배열을 구축함

```cpp
TArray<FLevelCollection> LevelCollections;
```

### 11. 초기화 상태 플래그 설정
```cpp
    // 이제 초기화됨:
    bIsWorldInitialized = true;
    bHasEverBeenInitialized = true;
```
- `bIsWorldInitialized = true;
- `bHasEverBeenInitialized = true;

### 12. Post-World 초기화 브로드캐스트
```cpp
FWorldDelegate::OnPostWorldInitialization.Broadcast(this, IVS);
```
- `FWorldDelegate::OnPostWorldInitialization.Broadcast(this, IVS)`로 초기화 직후 이벤트 날림
	  
- 여기서 `WorldSubSystem`을 전부 초기화 한 후에 추후 동작이 필요한 경우 여기다가 바인딩하면 될 듯

### 14. 퍼시스턴트 레벨 로드 통지
```cpp
    PersistentLevel->OnLevelLoaded();
```
- `PersistentLevel->OnLevelLoaded();`를 호출해 `PersistentLevel`이 `Load`되었음을 알림.

### 15. 스트리밍 매니저에 레벨 추가
```cpp
    // 새로운 레벨이 추가되었음을 스트리밍 매니저에 알림
    IStreamingManager::Get().AddLevel(PersistentLevel);
```
- `IStreamingManager::Get().AddLevel(PersistentLevel);`로 스트리밍 시스템에 등록
	  
- 이를 통해, `PersistentLevel`은 이미 `Load`되었음을 알려줌


### 16. 월드 서브시스템 후처리
```cpp
    // PostInitializeSubsystems 참조
    // 각 UWorldSubsystem에 대해 PostInitialize() 호출
    PostInitializeSubsystems();
```
- 각 `UWorldSubsystem::PostInitialize()`를 호출

### 17. 레벨 변경 브로드캐스트
```cpp
    BroadcastLevelsChanged();
```
- 레벨 목록/상태 변경을 시스템에 알림

---

## 결론 (핵심)
1. WorldSubsystemCollection 초기화
	   
2. 렌더 월드(FScene)와 물리 월드(FPhysScene) 할당(생성)