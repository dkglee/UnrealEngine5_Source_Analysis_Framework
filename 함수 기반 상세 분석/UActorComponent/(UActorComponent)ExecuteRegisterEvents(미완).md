```cpp
/** OnRegister, CreateRenderState_Concurrent, OnCreatePhysicsState를 호출한다 */
void UActorComponent::ExecuteRegisterEvents(FRegisterComponentContext* Context = nullptr)
{
    if (!bRegistered)
    {
        // OnRegister 참고
        OnRegister();
    }

    if (FApp::CanEverRender() && !bRenderStateCreated && WorldPrivate->Scene 
        // ShouldCreateRenderState() 확인
        && ShouldCreateRenderState())
    {
        // 이 지점을 기억할 것
        // - 여기서 렌더 월드(월드의 Scene == FScene)에 프리미티브를 추가한다
        CreateRenderState_Concurrent(Context);
    }

    CreatePhysicsState(/*bAllowDeferral=*/true);

    // create[render|physics]state는 '메인 월드의 상태를 각 렌더 월드와 물리 월드에 반영(투영)하는 것'을 의미
}
```
---
## 상세 설명
### 1. 