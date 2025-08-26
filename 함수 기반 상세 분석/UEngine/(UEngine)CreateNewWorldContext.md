```cpp
FWorldContext& UEngine::CreateNewWorldContext(EWorldType::Type WorldType)
{
	FWorldContext* NewWorldContext = new FWorldContext;
	
	// UEngine의 WorldList에 추가(중요)
	WorldList.Add(NewWorldContext);
	NewWorldContext->WorldType = WorldType;
	NewWorldContext->ContextHandle = FName(*FString::Printf(TEXT("Context_%d"), NextWorldContextHandle++));

	return *NewWorldContext;
}
```
---
## 상세 설명서
### 생성 후 `WorldList`에 추가
```cpp
	FWorldContext* NewWorldContext = new FWorldContext;
	
	// UEngine의 WorldList에 추가(중요)
	WorldList.Add(NewWorldContext);
```
- `new`로 FWorldContext를 생성하는 것을 볼 수 있다. (GC 관리 X)
- 즉, 직접 메모리를 관리해야 함
- WorldList에 추가하는 것을 볼 수 있다.

```cpp
TIndirectArray<FWorldContext>   WorldList;
```
- 여기서 `TArray<>`와 `TIndirectArray<>`의 차이점을 알아두자!