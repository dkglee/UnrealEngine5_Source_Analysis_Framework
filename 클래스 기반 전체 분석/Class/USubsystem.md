```cpp
/**
 * 서브시스템(Subsystem)은 특정 엔진 구성 요소와 동일한 수명(Lifetime)을 공유하며
 * 자동으로 인스턴스화되는 클래스다.
 *
 * 현재 지원되는 서브시스템 수명 종류:
 *  Engine       -> UEngineSubsystem을 상속
 *  Editor       -> UEditorSubsystem을 상속
 *  GameInstance -> UGameInstanceSubsystem을 상속
 *  World        -> UWorldSubsystem을 상속
 *  LocalPlayer  -> ULocalPlayerSubsystem을 상속
 *
 * 일반적인 예제:
 *  class UMySystem : public UGameInstanceSubsystem
 * 접근 방법:
 *  UGameInstance* GameInstance = ...;
 *  UMySystem* MySystem = GameInstance->GetSubsystem<UMySystem>();
 *
 * GameInstance가 null일 수도 있는 상황을 대비하려면 다음처럼 사용할 수 있다:
 *  UGameInstance* GameInstance = ...;
 *  UMyGameSubsystem* MySubsystem =
 *      UGameInstance::GetSubsystem<MyGameSubsystem>(GameInstance);
 *
 * 또한 하나의 인터페이스에 여러 구현체를 둘 수도 있다.
 * 인터페이스 예제:
 *      MySystemInterface
 *  두 개의 구체 파생 클래스:
 *      MyA : public MySystemInterface
 *      MyB : public MySystemInterface
 *
 * 접근 방법:
 *      UGameInstance* GameInstance = ...;
 *      const TArray<UMyGameSubsystem*>& MySubsystems =
 *          GameInstance->GetSubsystemArray<MyGameSubsystem>();
 */

// - USubsystem은 언리얼 엔진 구성 요소(UWorld 등)의 생성/소멸과 동일한 수명을 따르는 시스템이다.
// - 서브시스템의 종류:
//   1. Engine        -> UEngineSubsystem 상속
//   2. Editor        -> UEditorSubsystem 상속
//   3. GameInstance  -> UGameInstanceSubsystem 상속
//   4. World         -> UWorldSubsystem 상속
//   5. LocalPlayer   -> ULocalPlayerSubsystem 상속
// - **UWorld 같은 엔진 구성 요소의 수명 관리**는 복잡하다.
//   - 서브시스템을 사용하면 훨씬 쉽고 편리하게 다룰 수 있다!
// - 서브시스템 접근 예제를 읽어볼 것

class USubsystem : public UObject
{
    /**
     * 서브시스템을 생성할지 여부를 제어하려면 이 함수를 오버라이드한다.
     * 예를 들어 서버에서만 시스템을 생성하도록 제한할 수 있다.
     * 이 기능을 사용할 경우, 어디에서든 서브시스템을 가져올 때 null 체크가 매우 중요해진다.
     *
     * 주의: 이 함수는 인스턴스가 생성되기 전에 CDO(Class Default Object)에서 호출된다!!!
     */
    // 주석에서 설명한 대로, 이 멤버 함수는 CDO(클래스 디폴트 오브젝트)를 통해 호출된다
    virtual bool ShouldCreateSubsystem(UObject* Outer) const;

    /** 시스템 인스턴스의 초기화/해제를 구현한다 */
    virtual void Initialize(FSubsystemCollectionBase& Collection);
    virtual void Deinitialize();

    // UWorld의 [FObjectSubsystemCollection<UWorldSubsystem>]를 살펴볼 것이다.
    // - 각 서브시스템은 이와 같은 오너(Owner)를 갖는다
    // see FObjectSubsystemCollection
    FSubsystemCollectionBase* InternalOwningSubsystem;
};
```
---
### 1. 역할 및 책임

- **수명 관리 일원화**  
    엔진 핵심 객체(UEngine, UGameInstance, UWorld 등)와 **동일한 라이프사이클**을 갖는 서비스 클래스를 작성하기 위한 공통 기반을 제공한다.
    
- **초기화·해제 훅 제공**  
    `Initialize()` / `Deinitialize()` 가상 함수를 통해, 오너 객체가 준비된 직후와 파괴 직전에 필요한 로직을 삽입할 수 있다.
    
- **조건부 생성 지원**  
    `ShouldCreateSubsystem()`을 오버라이드해 “전용 서버 전용”, “에디터 전용” 같은 환경별 활성화/비활성화를 제어한다.
    

---

### 2. 클래스가 제공하는 기능

- `bool ShouldCreateSubsystem(UObject* Outer) const`
    
    - **CDO 단계**에서 호출되어 서브시스템 인스턴스를 생성할지 여부를 판단한다.
        
- `void Initialize(FSubsystemCollectionBase& Collection)`
    
    - 다른 서브시스템 의존성을 확보하거나 초기 상태를 세팅한다.
        
- `void Deinitialize()`
    
    - 게임 종료·월드 언로드 등으로 오너가 파괴될 때 정리 작업을 수행한다.
        
- **정적 헬퍼(파생 클래스 쪽 기능)**
    
    - `Outer->GetSubsystem<T>()`, `UObjectType::GetSubsystem<T>(Outer)` 등 템플릿 래퍼로 안전한 인스턴스 조회를 지원한다.
        

---

### 3. 관계

- **Owner → USubsystem**
    
    - 오너(UEngine, UWorld 등)는 `FSubsystemCollectionBase`를 통해 서브시스템 인스턴스를 **소유**한다.
        
- **USubsystem ↔ GC**
    
    - 라이프사이클이 오너에 귀속되므로, 일반 UObject와 달리 별도 레퍼런스 유지 없이도 GC 대상에서 보호된다.

### 4. 상속 계보
```
UObject
 └─ USubsystem
     ├─ UEngineSubsystem
     ├─ UEditorSubsystem
     ├─ UGameInstanceSubsystem
     ├─ UWorldSubsystem
     └─ ULocalPlayerSubsystem
```
- 각 파생 클래스는 대응하는 오너 객체 내부에서만 인스턴스화된다.
    

---

### 5. 멤버 변수

- `FSubsystemCollectionBase* InternalOwningSubsystem`
    
    - 자신을 보관하고 있는 컬렉션에 대한 역참조 포인터.
        
    - 의존성 초기화와 파괴 순서를 조정할 때 사용된다.
        

---

### 6. 특징 및 주의점

1. **컨텍스트별 다중 인스턴스**  
    PIE, 시뮬레이션, 에디터 월드 등 컨텍스트가 늘어나면 동일 서브시스템 클래스가 여러 개 생성될 수 있다.
    
2. **Null-safe 접근 필수**  
    `ShouldCreateSubsystem()`에서 조건부로 생성이 막히면 `GetSubsystem<T>()`가 nullptr를 반환한다.
    
3. **틱(FTickable)·Timer는 명시적 등록**  
    서브시스템 자체는 틱되지 않으므로, 주기 작업이 필요하면 `FTicker`나 `FTickableGameObject`를 이용해야 한다.
    
4. **네트워크 모드 분기**  
    `IsRunningDedicatedServer()`, `GetNetMode()` 등을 활용해 서버·클라이언트 전용 로직을 분리하면 오버헤드를 줄일 수 있다.
    
5. **장기 캐시 데이터 주의**  
    월드 스트리밍 언로드 시 서브시스템도 함께 사라지므로, 엔진 전역 캐시가 필요하면 `UEngineSubsystem` 계층을 고려한다.