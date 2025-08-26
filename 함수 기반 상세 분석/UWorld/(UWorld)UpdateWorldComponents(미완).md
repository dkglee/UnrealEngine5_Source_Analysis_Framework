```cpp
/** 월드 컴포넌트(예: line batcher 및 모든 레벨 컴포넌트)를 업데이트한다 */
void UWorld::UpdateWorldComponents(bool bRerunConstructionScripts, bool bCurrentLevelOnly, FRegisterComponentContext* Context = nullptr)
{
    if (!IsRunningDedicatedServer())
    {
        // - ULineBatchComponent는 UPrimitiveComponent를 상속함
        // - ULineBatchComponent는 UActorComponent의 일종으로 볼 수 있음
        // - UWorld는 AActor가 아니지만, LineBatcher용으로 동적 UActorComponent를 가짐
        // - LineBatcher들은 개별적으로 등록되어야 함
        if (!LineBatcher)
        {
            LineBatcher = NewObject<ULineBatchComponent>();
            LineBatcher->bCalculateAccurateBounds = false;
        }

        if(!LineBatcher->IsRegistered())
        {	
            // UActorComponent::RegisterComponentWithWorld 참고
            LineBatcher->RegisterComponentWithWorld(this, Context);
        }

        if(!PersistentLineBatcher)
        {
            PersistentLineBatcher = NewObject<ULineBatchComponent>();
            PersistentLineBatcher->bCalculateAccurateBounds = false;
        }

        if(!PersistentLineBatcher->IsRegistered())	
        {
            PersistentLineBatcher->RegisterComponentWithWorld(this, Context);
        }

        if(!ForegroundLineBatcher)
        {
            ForegroundLineBatcher = NewObject<ULineBatchComponent>();
            ForegroundLineBatcher->bCalculateAccurateBounds = false;
        }

        if(!ForegroundLineBatcher->IsRegistered())	
        {
            ForegroundLineBatcher->RegisterComponentWithWorld(this, Context);
        }
    }

    {
        for (int32 LevelIndex = 0; LevelIndex < Levels.Num(); ++LevelIndex)
        {
            ULevel* Level = Levels[LevelIndex];
            ULevelStreaming* StreamingLevel = FLevelUtils::FindStreamingLevel(Level);
            
            // 스트리밍 레벨이 아니거나, 레벨이 가시 상태일 때만 업데이트한다
            if (!StreamingLevel || Level->bIsVisible)
            {
                Level->UpdateLevelComponents(bRerunConstructionScripts, Context);
                IStreamingManager::Get().AddLevel(Level);
            }
        }
    }

    const TArray<UWorldSubsystem*>& WorldSubsystems = SubsystemCollection.GetSubsystemArray<UWorldSubsystem>(UWorldSubsystem::StaticClass());
    for (UWorldSubsystem* WorldSubsystem : WorldSubsystems)
    {
        // 각 서브시스템에 월드 컴포넌트가 업데이트되었음을 통지
        WorldSubsystem->OnWorldComponentsUpdated(*this);
    }
}
```
---
## 상세 설명
### 1. 라인 배처 컴포넌트 생성·등록 
```cpp
// - ULineBatchComponent는 UPrimitiveComponent를 상속함
// - ULineBatchComponent는 UActorComponent의 일종으로 볼 수 있음
// - UWorld는 AActor가 아니지만, LineBatcher용으로 동적 UActorComponent를 가짐
// - LineBatcher들은 개별적으로 등록되어야 함
if (!LineBatcher)
{
	LineBatcher = NewObject<ULineBatchComponent>();
	LineBatcher->bCalculateAccurateBounds = false;
}

```
- `ULineBatchComponent`는 `UActorComponent`이다.
	  
- 현재 함수가 실행되는 객체는 `UWorld`이고 `UObject`를 상속받고 있다.
	  
- `UWorld`는 `AActor`가 아니지만 `NewObject`를 통해서 동적으로 `UActorComponent`를 가짐
	  
- 이 경우 `AActor`의 초기화 순서를 받지 못하기 때문에 수작업으로 **등록**해야 함.

```cpp
if(!LineBatcher->IsRegistered())
{	
	// UActorComponent::RegisterComponentWithWorld 참고
	LineBatcher->RegisterComponentWithWorld(this, Context);
}
// ... 나머지 동일
```
- `RegisterComponentWithWorld()` 함수를 통해서 등록을 한다.
