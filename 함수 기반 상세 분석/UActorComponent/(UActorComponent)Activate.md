```cpp
/** SceneComponent을 활성화한다. 네이티브 자식 클래스에서 오버라이드해야 한다 */
virtual void Activate(bool bReset=false)
{
    // Activate의 세부로 들어가기 전에 FTickFunction 관련 클래스를 먼저 보라:
    // AActor::PrimaryActorTick 참고
    // UActorComponent::PrimaryComponentTick 참고
    if (bReset || ShouldActivate() == true)
    {
        // SetComponentTickEnabled 참고
        SetComponentTickEnabled(true);
        // SetActiveFlag 참고
        SetActiveFlag(true);

        OnComponentActivated.Broadcast(this, bReset);
    }
}
```
---
### 상세 설명

> **주의 :** 내부로 들어가기 전에 `FTickFunction`, `AActor::PrimaryActorTick`, 그리고 `UActorComponent::PrimaryComponentTick`을 참고하자.
