```cpp
// Static 함수임을 알아야 한다.
// 아래의 함수 시그니쳐를 보면 두 개의 변수에만 집중하면 되는 것을 알 수 있다.
// EWorldType, bInformEngineOfWorld
// UWorld를 생성하고 초기화해주는 함수 : UWorld 반환
static UWorld* UWorld::CreateWorld(
        const EWorldType::Type InWorldType, 
        bool bInformEngineOfWorld,
        FName WorldName = NAME_None, 
        UPackage* InWorldPackage = NULL, 
        bool bAddToRoot = true, 
        ERHIFeatureLevel::Type InFeatureLevel = ERHIFeatureLevel::Num,
        // InitializationValues 참고 
        const InitializationValues* InIVS = nullptr, 
        bool bInSkipInitWorld = false
    )
{
	// 패키지 생성하는 부분
	UPackage* WorldPackage = InWorldPackage;
	if (!WorldPackage)
	{
        WorldPackage = CreatePackage(nullptr);
    }

    if (InWorldType == EWorldType::PIE)
    {
        WorldPackage->SetPackageFlags(PKG_PlayInEditor);
    }

      
    if (WorldPackage != GetTransientPackage())
    {
        WorldPackage->ThisContainsMap();
    }

	const FString WorldNameString = (WorldName != NAME_None) ? WorldName.ToString() : TEXT("Untitled");
	// UWorld 인스턴스 생성하는 부분
    // 이때 WorldPackage(UPackage)가 OuterPrivate이 되는 것을 볼 수 있음
    UWorld* NewWorld = NewObject<UWorld>(WorldPackage, *WorldNameString);

    NewWorld->SetFlags(RF_Transactional);
    NewWorld->WorldType = InWorldType;
    NewWorld->SetFeatureLevel(InFeatureLevel);

	// 초기화 부분
    NewWorld->InitializeNewWorld(
        InIVS ? *InIVS : UWorld::InitializationValues()
            .CreatePhysicsScene(InWorldType != EWorldType::Inactive)
            .ShouldSimulatePhysics(false)
            .EnableTraceCollision(true)
            .CreateNavigation(InWorldType == EWorldType::Editor)
            .CreateAISystem(InWorldType == EWorldType::Editor)
       , bInSkipInitWorld);

    WorldPackage->SetDirtyFlag(false);

    if (bAddToRoot)
    {
       NewWorld->AddToRoot();
    }

    if ((GEngine) && (bInformEngineOfWorld == true))
    {
        GEngine->WorldAdded(NewWorld);
    }

    return NewWorld;
}
```
---
## 상세 설명
### 1. 함수 원형 분석
```cpp
// Static 함수임을 알아야 한다.
// 아래의 함수 시그니쳐를 보면 두 개의 변수에만 집중하면 되는 것을 알 수 있다.
// EWorldType, bInformEngineOfWorld
static UWorld* UWorld::CreateWorld(
        const EWorldType::Type InWorldType, 
        bool bInformEngineOfWorld,
        FName WorldName = NAME_None, 
        UPackage* InWorldPackage = NULL, 
        bool bAddToRoot = true, 
        ERHIFeatureLevel::Type InFeatureLevel = ERHIFeatureLevel::Num,
        // InitializationValues 참고 
        const InitializationValues* InIVS = nullptr, 
        bool bInSkipInitWorld = false
    )
```
- `static` 함수인 것을 눈치 채야 함
	  
-  나머지 매개변수들을 보면 `default`값이 지정된 것을 알 수 있다.
	  
	- 그 말은 EWorldType하고 bInformEngineOfWorld만 주의깊게 보면 된다.
		  
- [[FWorldInitializationValues]] 참고

---

### 2. `UPackage` 생성
```cpp
// UPackage는 나중에 AsyncLoading을 다룰 때 자세히 설명함
// - 지금은 그냥 UWorld의 파일 포맷을 설명하는 개념이라고 생각하면 됨
// - 기억할 점은, 각 월드는 개별 패키지와 1:1로 매핑된다는 거임
//   - 즉, 월드에 있는 모든 데이터는 파일로 직렬화돼야 함
//   - 언리얼 엔진에서 파일을 저장한다는 건 '패키지', 즉 UPackage를 의미함
// - 또 하나는 UObject가 OuterPrivate를 가진다는 점임:
//   - OuterPrivate는 객체가 어디에 소속되는지 나타냄
//   - 보통 OuterPrivate는 UPackage로 설정됨:
//     - 객체가 소속된 위치 == 객체가 저장된 파일
// - 이건 이해를 돕기 위한 설명이고, 외우려고 하지 말고 그냥 ‘아 그렇구나’ 하면 됨
UPackage* WorldPackage = InWorldPackage;
if (!WorldPackage)
{
	// UWorld는 직렬화되어 저장되기 위해 OuterPrivate으로써 UPackage를 필요로 함
	WorldPackage = CreatePackage(nullptr);
}
```
- `UPackage`는 언리얼에서 **파일 저장 단위**를 의미하며, **월드와 1:1로 매핑됨**
    
- **월드**의 모든 데이터는 **직렬화**되어 패키지(파일)로 저장됨
    
- `UObject`는 `OuterPrivate`를 통해 **자신이 속한 패키지**\(**파일**\)를 참조함
    
- 따라서, 객체가 속한 위치는 곧 객체가 저장된 파일을 의미함
    
- 지금은 깊이 외울 필요 없고, 월드는 별도 파일에 저장된다’는 개념만 이해하면 충분함

---

### 3. `UPackage` 세팅
```cpp
// UObjectBase의 ObjectFlags처럼, UPackage도 플래그를 통해 속성을 설정할 수 있음
// - 지금은 자세히 읽지 않을 거고, 나중에 다시 마주칠 기회가 있을 수 있음
if (InWorldType == EWorldType::PIE)
{
    WorldPackage->SetPackageFlags(PKG_PlayInEditor);
}


// 해당 패키지가 월드를 포함하고 있음을 표시함
// 언리얼 엔진에서 'Transient'란 무엇인가?
// - 직렬화되지 않는 속성이나 에셋이 있다면, 이를 'Transient'로 표시함
// - Transient == 직렬화되지 않도록 표시하는 메타데이터
// - 즉, Transient(직렬화X)가 아닌 경우 해당 패키지를 맵을 포함하는 패키지로 표시!
if (WorldPackage != GetTransientPackage())
{
    WorldPackage->ThisContainsMap();
}
```
- `UPackage`도 `UObject`처럼 **플래그(PackageFlags)** 를 통해 속성을 설정함
    
- `PIE(Play In Editor)` 월드일 경우, 해당 패키지에 `PKG_PlayInEditor` 플래그를 설정함
    
- **Transient**란 **직렬화되지 않아야 할 속성/에셋**을 의미하며, 이를 통해 저장 대상에서 제외됨
    
- `GetTransientPackage()`가 아닌 경우, 해당 패키지를 **맵(월드)을 포함하는 패키지**로 표시함 (`ThisContainsMap()` 호출)

---

### 4. `UWorld` 생성 후 세팅
```cpp
// NewWorld의 Outer를 WorldPackage로 설정함
// - 일반적으로 Outer 객체를 따라가다 보면 결국 이 UObject를 포함하는 패키지(에셋 파일)에 도달하게 됨
UWorld* NewWorld = NewObject<UWorld>(WorldPackage, *WorldNameString);

// UObject::SetFlags → 비트 연산자(AND(&), OR(|), SHIFT(>>, <<) 등)를 사용해 언리얼 객체의 속성을 설정함
// - EObjectFlags 및 UObject::ObjectFlags를 참고할 것
NewWorld->SetFlags(RF_Transactional);
NewWorld->WorldType = InWorldType;
NewWorld->SetFeatureLevel(InFeatureLevel);
```
- `NewWorld` 객체는 `WorldPackage`를 **Outer**로 하여 생성됨  
    → 즉, 해당 월드는 이 패키지(파일)에 포함됨을 의미함
    
- `SetFlags(RF_Transactional)`로 **변경 이력 추적 가능**하도록 플래그 설정  
    → Undo/Redo 등을 위한 설정
    
- `WorldType`, `FeatureLevel` 등 월드의 **기본 속성**을 초기화함


---


### 5. `UWorld` 초기화
```cpp
// 생성한 `UWorld`를 초기화 함
// 만약 *InIVS*가 제공되지 않으면 직접 생성하여 전달
NewWorld->InitializeNewWorld(
    InIVS ? *InIVS : UWorld::InitializationValues()
        // 이전에 봤던 FWorldInitializationValues처럼,
        // 아래에 이어지는 함수들은 각각 플래그를 설정하여,
        // 새로운 월드를 생성할 때 참고할 수 있도록 만듦
        .CreatePhysicsScene(InWorldType != EWorldType::Inactive)
        .ShouldSimulatePhysics(false)
        .EnableTraceCollision(true)
        .CreateNavigation(InWorldType == EWorldType::Editor)
        .CreateAISystem(InWorldType == EWorldType::Editor)
    , bInSkipInitWorld);
```
- `UWorld::InitializeNewWorld()`를 호출하여 새로 생성한 월드를 **초기화**함
    
- 초기화 옵션(`FWorldInitializationValues`)은
    
    - 전달된 `InIVS`가 있으면 그것을 사용하고
        
    - 없으면 **직접 생성하여 설정값을 체이닝** 방식으로 지정함
        
- 설정되는 주요 플래그:
    
    - 물리 씬 생성 여부
        
    - 물리 시뮬레이션 사용 여부
        
    - 충돌 트레이스 활성화
        
    - 내비게이션 시스템 생성 여부 (에디터에서만)
        
    - AI 시스템 생성 여부 (에디터에서만)


---


### 6. 초기화 이후의 세팅 (GC 및 브로드캐스트) 
```cpp
// SpawnActor 및 UpdateLevelComponents 과정에서 설정된 dirty 플래그를 초기화함
WorldPackage->SetDirtyFlag(false);

if (bAddToRoot)
{
    // 루트 세트에 추가하여 GC(가비지 컬렉션)되지 않도록 함
    NewWorld->AddToRoot();
}

// 엔진에 월드를 추가한다고 알림 (단, 알리지 않도록 요청된 경우는 제외)
if ((GEngine) && (bInformEngineOfWorld == true))
{
    GEngine->WorldAdded(NewWorld);
}

return NewWorld;
```
- **SpawnActor 등으로 인해 변경된 패키지 상태(dirty flag)를 초기화**
    
- `bAddToRoot`가 true인 경우, `NewWorld`를 **루트에 추가**해 GC 대상에서 제외
    
- `bInformEngineOfWorld`가 true이고 `GEngine`이 유효하면  
    → **엔진에 새 월드가 생성되었음을 알림**


---

### 참고
[[KeyWord (Transient)]]