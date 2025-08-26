```cpp
/** 컴포넌트가 등록될 때 호출된다. Scene이 설정된 뒤,
 *  CreateRenderState_Concurrent 또는 OnCreatePhysicsState가 호출되기 전에 실행된다 */
virtual void UActorComponent::OnRegister()
{
    bRegistered = true;

    UpdateComponentToWorld();

    // bAutoActivate가 켜져 있으면 등록 단계에서 Activate() 실행
    if (bAutoActivate)
    {
        AActor* Owner = GetOwner();

		// 컴포넌트가 UWorld에 직접 포함된 경우 Owner가 nullptr일 수 있음
        if (!WorldPrivate->IsGameWorld() 
            || Owner == nullptr 
            || Owner->IsActorInitialized)
        {
            // Activate 참고
            Activate(true);
        }
    }
}
```
---
## 상세 설명
### 1. Register 상태 전환
```cpp
bRegistered = true;
```
- `bRegistered`를 `true`로 할당

---

### 2. 트랜스폼 갱신
```cpp
UpdateComponentToWorld();
```
- `UpdateComponentToWorld()` 호출로 **월드 변환 갱신**.
	  
- `USceneComponent`인 경우 **자식 컴포넌트 트랜스폼 전파**까지 포함.
	  
- 바로 뒤이어 발생할 **렌더/물리 상태 생성**이 **최신 트랜스폼**을 캡처해야 함.

---

### 3. 자동 활성화
```cpp
// bAutoActivate가 켜져 있으면 등록 단계에서 Activate() 실행
if (bAutoActivate)
{
	AActor* Owner = GetOwner();

	// 컴포넌트가 UWorld에 직접 포함된 경우 Owner가 nullptr일 수 있음
	if (!WorldPrivate->IsGameWorld() 
		|| Owner == nullptr 
		|| Owner->IsActorInitialized)
	{
		// Activate 참고
		Activate(true);
	}
}
```
- `bAutoActivate`가 활성화 되어 있으면 조건 검사 후 `Activate` 실행
	  
	- `bAutoActivate`: 보통 파티클 시스템에서 주로 본적이 있을 것이다.
		
	- `!WorldPrivate->IsGameWorld()` : 게임 월드가 아닌 경우 (Editor)
		  
	- `Owner == nullptr`: **Owner가 없음**(예: `UWorld`가 직접 보유한 라인 배처)
		  (현재 이 조건에 만족한다고 보면 됨)
		  
	- `Owner->IsActorInitialized`: **Owner가 이미 초기화됨**(`Owner->IsActorInitialized`)
		
- `Activate()` 실행