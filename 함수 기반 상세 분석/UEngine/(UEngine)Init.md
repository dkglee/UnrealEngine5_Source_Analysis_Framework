```cpp
/** initialize the game engine */
void UEngine::Init(IEngineLoop* InEngineLoop)
{
	// Engine의 Loop를 GEngineLoop(FEngineLoop)로 등록
	EngineLoop = InEngineLoop;

	// Engine SubSystem 생성
	EngineSubsystemCollection.Initialize(this);
    
    if (GIsEditor)
   	{
		// create a WorldContext for the editor to use and create an initially empty world
		// 비어있는 WorldContext 생성
		FWorldConext& InitialWorldContext = CreateNewWorldContext(EWorldType::Editor);
	
		// UWorld(Type::Editor) 생성 후 WorldContext에 할당
		// see CreateWorld
   		InitialWorldContext.SetCurrentWorld(UWorld::CreateWorld(EWorldType::Editor, true));
       	GWorld = InitialWorldContext.World();
   	}
}
```
---
### 상세 설명서
### 1. WorldContext 생성
```cpp
		// create a WorldContext for the editor to use and create an initially empty world
		// 비어있는 WorldContext 생성
		FWorldConext& InitialWorldContext = CreateNewWorldContext(EWorldType::Editor);
```
- 여기서 생각해볼 것은 왜 `FWorldContext`가 필요하다는 것이다.

### 2. `UWorld` 생성
```cpp
		// UWorld(Type::Editor) 생성 후 WorldContext에 할당
		// see CreateWorld
   		InitialWorldContext.SetCurrentWorld(UWorld::CreateWorld(EWorldType::Editor, true));
```
- `UWorld`를 생성하고 `WorldContext`에 할당하는 것을 볼 수 있다.