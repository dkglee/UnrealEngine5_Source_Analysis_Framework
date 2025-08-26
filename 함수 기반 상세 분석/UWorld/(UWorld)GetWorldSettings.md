```cpp
/** 이 월드에 연관된 AWorldSettings 액터를 반환한다 */
AWorldSettings* GetWorldSettings(bool bCheckStreamingPersistent = false, bool bChecked = true) const
{
    AWorldSettings* WorldSettings = nullptr;
    if (PersistentLevel)
    {
        // AWorldSettings를 만들 때 Outer를 PersistentLevel로 설정했던 것 기억
        WorldSettings = PersistentLevel->GetWorldSettings(bChecked);
        if (bCheckStreamingPersistent)
        {
            //...
        }
    }
    return WorldSettings;
}
```
---
- 이전에 `PersistentLevel`을 생성하고 `WordSettings`를 할당했던 것을 기억하자
	  
- 그리고 그것을 반환하는 것을 볼 수 있다.