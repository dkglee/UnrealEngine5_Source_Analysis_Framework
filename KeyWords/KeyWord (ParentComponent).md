```

    /** 이 액터를 소유하는 UChildActorComponent */
    // 해설: UChildActorComponent를 자세히 다루지는 않지만, 개념을 설명하면
    // - UChildActorComponent는 액터와 액터 사이의 연결을 지원함
    // - 기본적으로는 액터-액터컴포넌트 연결만 있는데, 액터-액터 연결은 어떻게 지원하는가?
    //   - 다이어그램 참조:                                                                                               
    //     ┌──────────┐                         ┌──────────┐                           ┌──────────┐              
    //     │  Actor0  ├────────────────────────►│  Actor1  ├──────────────────────────►│  Actor2  │              
    //     └──────────┘                         └──────────┘                           └──────────┘              
    //                                                                                                           
    //     RootComponent                        RootComponent ◄──────────┐             RootComponent             
    //      │                                    │                       │              │                        
    //      ├───Component1◄──────────┐           ├───Component1     Actor2-Actor1       └───Component1           
    //      │                        │           │                       │                                       
    //      ├───Component2    Actor1-Actor0      └───Component2          └─────────────ParentComponent           
    //      │    │                   │                                                                           
    //      │    └───Component3      └──────────ParentComponent                                                  
    //      │                                                                                                    
    //      └───Component4                                                                                       
    //                                                                                                           
    //                 ┌──────────────────────────────────────┐                                                  
    //                 │                                      │                                                  
    //                 │      Actor0 ◄───                     │                                                  
    //                 │       │                              │                                                  
    //                 │       └─RootComponent                │                                                  
    //                 │          │                           │                                                  
    //                 │          └─Component1                │                                                  
    //                 │             │                        │                                                  
    //                 │             └─Actor1 ◄───            │                                                  
    //                 │                │                     │                                                  
    //                 │                └─RootComponent       │                                                  
    //                 │                   │                  │                                                  
    //                 │                   └─Actor2 ◄────     │                                                  
    //                 │                                      │                                                  
    //                 └──────────────────────────────────────┘                                                  
    TWeakObjectPtr<UChildActorComponent> ParentComponent;
```
---
### 📘 `ParentComponent` 멤버 설명

#### 역할 및 책임
- 이 액터가 **`UChildActorComponent`를 통해 생성된 경우**, 자신을 생성한 컴포넌트(`ParentComponent`)를 약참조(`TWeakObjectPtr`)로 저장함.
	  
	- `ParentComponent`에 다른 액터의 `UChildActorComponent`를 지정함으로써, 해당 액터가 **논리적**으로 **그 컴포넌트의 자식**임을 표현할 수 있다.  
		  
	- 하지만 트랜스폼 계층 구조(Attach)는 별도로 설정해야 한다.
	  
- 즉, 이 액터는 **다른 액터의 `UChildActorComponent`에 의해 소유**되고 있다는 의미이며, 해당 소유 컴포넌트를 역으로 추적할 수 있음.
	  
- 이렇게 함으로써 **액터-액터 연결**을 표현할 수 있다.

#### 기능
- **부모 액터의 특정 컴포넌트(UChildActorComponent)**를 가리키며, 이 컴포넌트는 `ChildActor` 속성을 통해 자식 액터(this)를 소유함.
    
- 따라서 이 변수는 다음과 같은 목적으로 활용됨:
    
    - 자식 액터가 자신의 소유자를 알고 싶을 때 (`GetParentComponent()` 등의 함수)
        
    - 부모 컴포넌트와 연동된 특정 로직 처리 시 (예: 부모 파괴 → 자식 제거 등)

#### 관계
- 액터 간 직접적인 소유 관계를 구성할 수 있도록 해주는 Unreal의 구조적 트릭 중 하나
    
- `Actor ↔ UChildActorComponent ↔ Actor` 형태의 **연결 고리**를 구성함

#### `UChildActorComponent` 구조 및 연결 방식
- 기본적으로 Unreal Engine은 액터 안에 컴포넌트들이 존재하는 구조이며, 컴포넌트는 액터가 아니므로 계층 구조를 넘는 액터 간 직접 연결은 불가능

```cpp
// 부모 액터 A의 UChildActorComponent가 자식 액터 B를 생성
// B의 ParentComponent는 A의 UChildActorComponent를 참조
A
└── UChildActorComponent → 생성된 B
                             └── ParentComponent → A의 UChildActorComponent

```

**시각적 예시**
```cpp
Actor0
├── Component1
│   └── ChildActorComponent → Actor1
│                               └── ParentComponent → Component1
│
└── Component2
    └── ChildActorComponent → Actor2
                                └── ParentComponent → Component2
```
