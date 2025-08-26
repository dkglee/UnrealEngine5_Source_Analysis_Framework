```cpp
/** 새로 생성된 월드를 초기화함 */
void UWorld::InitializeNewWorld(const InitializationValues IVS = InitializationValues(), bool bInSkipInitWorld = false)
{
    if (!IVS.bTransactional)
    {
        // UObjectBase의 ObjectFlags를 지움(비트 연산)
        ClearFlags(RF_Transactional);
    }

    // 새 월드를 위한 기본 퍼시스턴트 레벨을 생성
    PersistentLevel = NewObject<ULevel>(this, TEXT("PersistentLevel"));
    PersistentLevel->OwningWorld = this;

#if WITH_EDITORONLY_DATA || 1
    CurrentLevel = PersistentLevel;
#endif

    // WorldInfo 액터를 생성
    // AWorldSettings를 자세히 들여다보지는 않지만, 예시로 이해해보자
    // [ ] 에디터에서 AWorldSettings 설명
    AWorldSettings* WorldSettings = nullptr;
    {
        // 액터를 스폰할 때 FActorSpawnParameters를 전달해야 함
        // FActorSpawnParameters 참고
        FActorSpawnParameters SpawnInfo;
        SpawnInfo.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

        // 호스트와 클라이언트의 새 월드 간 네트워크 복제가 동작하도록 WorldSettings에 상수 이름을 설정함
        // UEngine의 WorldSettingClass를 사용해서 WorldSettings의 클래스를 오버라이드할 수 있음
        SpawnInfo.Name = GEngine->WorldSettingsClass->GetFName();

        // SpawnActor는 나중에 다룸
        WorldSettings = SpawnActor<AWorldSettings>(GEngine->WorldSettingsClass, SpawnInfo);
    }
    
    // 레벨을 로드할 계획이 없는 경우를 대비해 월드 생성자가 기본 게임 모드를 오버라이드할 수 있도록 허용
    if (IVS.DefaultGameMode)
    {
        WorldSettings->DefaultGameMode = IVS.DefaultGameMode;
    }

    // 퍼시스턴트 레벨은 GameMode 같은 월드 정보를 담는 AWorldSettings를 생성함
    PersistentLevel->SetWorldSettings(WorldSettings);

#if WITH_EDITOR || 1
    WorldSettings->SetIsTemporaryHiddenInEditor(true);

    if (IVS.bCreateWorldParitition)
    {
        // 지금은 월드 파티션을 건너뜀
        // - 하지만 Lyra는 World-Partition 기반임
    }
#endif

    if (!bInSkipInitWorld)
    {
        // 월드를 초기화함
        // UWorld::InitWorld 참고
        InitWorld(IVS);

        // 컴포넌트를 업데이트함
        // UWorld::UpdateWorldComponents 참고
        const bool bRerunConstructionScripts = !FPlatformProperties::RequiresCookedData();
        UpdateWorldComponents(bRerunConstructionScripts, false);
    }
}
```
---
## 상세 설명
### 1. IVS의 bTransactional이 `false`일 경우 제거
```cpp
    if (!IVS.bTransactional)
    {
        // UObjectBase의 ObjectFlags를 지움(비트 연산)
        ClearFlags(RF_Transactional);
    }
```

---

### 2. Persistent Level 생성
```cpp
	// 새 월드를 위한 기본 퍼시스턴트 레벨을 생성
    PersistentLevel = NewObject<ULevel>(this, TEXT("PersistentLevel"));
    PersistentLevel->OwningWorld = this;

#if WITH_EDITORONLY_DATA || 1
    CurrentLevel = PersistentLevel;
#endif
```
- `PersistentLevel`의 `Outer`로 `this` 즉, `World`를 집어넣음
	  
- `PersistentLevel`의 `OuterPrivate`과 `OwningWorld`의 **주소가 서로 같은 걸 볼 수 있음**.
	  
- **새로 만든 월드**와 **Persistent Level**은 똑같은 UPackage에 저장되게 될 것이다.

---

### 3. World Setting 생성 후 `PersistentLevel`에 할당
```cpp
    // WorldInfo 액터를 생성
    // AWorldSettings를 자세히 들여다보지는 않지만, 예시로 이해해보자
    // [ ] 에디터에서 AWorldSettings 설명
    AWorldSettings* WorldSettings = nullptr;
    {
        // 액터를 스폰할 때 FActorSpawnParameters를 전달해야 함
        // FActorSpawnParameters 참고
        FActorSpawnParameters SpawnInfo;
        SpawnInfo.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

        // 호스트와 클라이언트의 새 월드 간 네트워크 복제가 동작하도록 WorldSettings에 상수 이름을 설정함
        // UEngine의 WorldSettingClass를 사용해서 WorldSettings의 클래스를 오버라이드할 수 있음
        SpawnInfo.Name = GEngine->WorldSettingsClass->GetFName();

        // SpawnActor는 나중에 다룸
        WorldSettings = SpawnActor<AWorldSettings>(GEngine->WorldSettingsClass, SpawnInfo);
    }
    
    // 레벨을 로드할 계획이 없는 경우를 대비해 월드 생성자가 기본 게임 모드를 오버라이드할 수 있도록 허용
    if (IVS.DefaultGameMode)
    {
        WorldSettings->DefaultGameMode = IVS.DefaultGameMode;
    }

    // 퍼시스턴트 레벨은 GameMode 같은 월드 정보를 담는 AWorldSettings를 생성함
    PersistentLevel->SetWorldSettings(WorldSettings);

```
- [[KeyWord (World Settings)]] 참고
	  
- `FActorSpawnParamters`를 바탕으로 스폰 정보를 꾸림
	  
- `UEngine`의 `WorldSettingClass`를 통해서 `AWorldSetting`의 실제 타입을 들고옴
	  
	- 오버라이딩 가능
		  
- `SpawnActor`로 생성
	  
	- 이 액터는 현재 `World`에 종속되어 있음. 정확히는 `PersistentLevel`에 종속되어 있음
		
	- 별도로 `FActorSpawnParameters`에 등록하지 않을 경우 `CurrentLevel` 즉, `PersistentLevel`에 종속되게 된다.
		  
- 이후 `PersistentLevel`에 생성한 `WorldSettings`를 할당

---

### 4. 월드 초기화
```cpp
        // 월드를 초기화함
        // UWorld::InitWorld 참고
        InitWorld(IVS);
```
- 실질적인 월드 초기화
	 
	- 여기서 WorldSubsystem 등을 초기화 함

---

### 5. 컴포넌트 업데이트
```cpp
        // 컴포넌트를 업데이트함
        // UWorld::UpdateWorldComponents 참고
        const bool bRerunConstructionScripts = !FPlatformProperties::RequiresCookedData();
        UpdateWorldComponents(bRerunConstructionScripts, false);
```
- 월드 컴포넌트를 업데이트 함
