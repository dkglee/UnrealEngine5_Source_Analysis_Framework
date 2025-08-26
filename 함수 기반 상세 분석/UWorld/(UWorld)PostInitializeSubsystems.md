```cpp
/** 모든 월드 서브시스템의 초기화를 마무리한다 */
void PostInitializeSubsystems()
{
    const TArray<UWorldSubsystem*>& WorldSubsystems = SubsystemCollection.GetSubsystemArray<UWorldSubsystem>(UWorldSubsystem::StaticClass());
    for (UWorldSubsystem* WorldSubsystem : WorldSubsystems)
    {
        WorldSubsystem->PostInitialize();
    }
}
```
---
- `SubsystemCollection`으로부터 생성된 `SubSystem`들의 배열인 `SubSystemArray`를 받음
	  
- 배열을 순회하며 `PostInitialize()` 호출하며 초기화 마무리