```cpp
void FWorldContext::SetCurrentWorld(UWorld* World)
{
	UWorld* OldWorld = ThisCurrentWorld;
	ThisCurrentWorld = World;

	// GameInstance의 World를 변경
	if (OwningGameInstance)
	{
		OwningGameInstance->OnWorldChanged(OldWorld, ThisCurrentWorld);
	}
}
```
---
