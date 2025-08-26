```cpp
/** 특정 월드에 컴포넌트를 등록한다. 이 과정에서 시각/물리 상태가 생성된다 */
void UActorComponent::RegisterComponentWithWorld(UWorld* InWorld, FRegisterComponentContext* Context = nullptr)
{
    // 이미 등록된 컴포넌트라면 아무 것도 하지 않음
    if (IsRegistered())
    {
        return;
    }

    // 등록할 월드가 없으면 일찍 반환하는 것이 자연스러움
    if (InWorld == nullptr)
    {
        return;
    }

    // 등록되지 않았다면 씬을 가지고 있지 않아야 함
    // WorldPrivate의 존재 여부로 등록 상태를 알 수 있다는 의미
    check(WorldPrivate == nullptr);

    // UWorld::LineBatcher(ULineBatchComponent)의 Owner는 nullptr
    AActor* MyOwner = GetOwner();
    check(MyOwner == nullptr || MyOwner->OwnsComponent(this));

    if (!HasBeenCreated)
    {
        // AActor에서 다뤘던 OnComponentCreated() 이벤트 기억
        // - ActorComponent가 생성될 때 호출됨
        OnComponentCreated();
    }

    WorldPrivate = InWorld;

    // ExecuteRegisterEvents 참고
    ExecuteRegisterEvents(Context);

    // 게임 월드가 아니면 지금 즉시 Tick을 등록한다.
    // Owner가 없으면 BeginPlay()도 트리거되지 않으므로 이 경우에도 지금 등록한다.
    if (!InWorld->IsGameWorld())
    {
        // 월드가 게임 월드가 아니면 InitializeComponent()를 호출하지 않음
        //   아래 조건(MyOwner == nullptr) 참조
        RegisterAllComponentTickFunctions(true);
    }
    else if (MyOwner == nullptr)
    {
        // AActor 초기화 단계에서 다뤘던 InitializeComponent() 호출 지점
        if (!bHasBeenInitialized && bWantsInitializeComponent) // bWantsInitializeComponents
        {
            // InitializeComponent() 참고
            InitializeComponent();
        }

        // RegisterAllComponentTickFunctions 참고 (goto 60)
        RegisterAllComponentTickFunctions(true);

        // haker: 이 분기가 바로 UWorld::LineBatcher에 정확히 해당함
    }
    else
    {
        // **일반적으로 기대되는 정상 경로**
        // HandleRegisterComponentWithWorld 참고 (goto 65)
        MyOwner->HandleRegisterComponentWithWorld(this);
    }

    // 블루프린트로 생성된 컴포넌트이고 자식 컴포넌트가 있는 경우,
    // 일부 시나리오에서 자식들이 등록을 놓칠 수 있음
    if (IsCreatedByConstructionScript())
    {
        // haker: SCS(Simple Construction Script)와 CCS(Custom Construction Script)를 예시(또는 에디터)로 설명
        // - 자식들은 SCS/CCS에서 수집되며, Outer는 이 UActorComponent가 됨
        TArray<UObject*> Children;
        GetObjectsWithOuter(this, Children, true, RF_NoFlags, EInternalObjectFlags::Garbage);

        // haker: Children을 순회하며, 필요한 경우 재귀적으로 RegisterComponentWithWorld 호출
        for (UObject* Child : Children)
        {
            if (UActorComponent* ChildComponent = Cast<UActorComponent>(Child))
            {
                if (ChildComponent->bAutoRegister && !ChildComponent->IsRegistered() && ChildComponent->GetOwner() == MyOwner)
                {
                    ChildComponent->RegisterComponentWithWorld(InWorld);
                }
            }
        }
    }
}
```
---
### 1. 초기 가드 & 정합성 체크
```cpp
// 이미 등록된 컴포넌트라면 아무 것도 하지 않음
if (IsRegistered())
{
	return;
}

// 등록할 월드가 없으면 일찍 반환하는 것이 자연스러움
if (InWorld == nullptr)
{
	return;
}

// 등록되지 않았다면 씬을 가지고 있지 않아야 함
// WorldPrivate의 존재 여부로 등록 상태를 알 수 있다는 의미
check(WorldPrivate == nullptr);

// UWorld::LineBatcher(ULineBatchComponent)의 Owner는 nullptr
AActor* MyOwner = GetOwner();
check(MyOwner == nullptr || MyOwner->OwnsComponent(this));
```
- 이미 등록됐으면 종료.
	  
- `InWorld == nullptr`이면 종료.
    
- 미등록이라면 `WorldPrivate == nullptr`이어야 함.
    
- `MyOwner == nullptr` **또는** `MyOwner->OwnsComponent(this)` 여야 함.
	
	- `MyOwner == nullptr` : `NewObject`로 동적 생성된 경우
		  
	- `MyOwner->OwnsComponent(this)` : `AActor`에 SubObject로써 붙여져 있는 경우
		
### 2) 생성 이벤트 및 월드 결속
```cpp
if (!HasBeenCreated)
{
	// AActor에서 다뤘던 OnComponentCreated() 이벤트 기억
	// - ActorComponent가 생성될 때 호출됨
	OnComponentCreated();
}

WorldPrivate = InWorld;
```
- 아직 생성 전이면 `OnComponentCreated()` 호출.
    
- `WorldPrivate = InWorld`로 대상 월드에 결속.
    
### 3) 등록 이벤트 실행
```cpp
// ExecuteRegisterEvents 참고
ExecuteRegisterEvents(Context);
```
- `ExecuteRegisterEvents(Context)`로 등록 시그널을 발행
	  
- 컴포넌트가 씬/리소스를 실제로 붙이기 시작.