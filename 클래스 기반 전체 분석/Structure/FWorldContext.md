```cpp
// 엔진이 여러 UWorld를 다룰 때, 각 월드가 어떤 컨텍스트에 속하는지를 명확히 구분하기 위한 구조체.
struct FWorldContext
{
    // FWorldContext가 관리하는 World를 할당
    void SetCurrentWorld(UWorld* World);

    // 본인의 WorldType을 나타냄
    TEnumAsByte<EWorldType::Type> WorldType;
    
    // 월드 컨텍스트의 이름을 UWorld의 이름과 다르게 지정
    FName ContextHandle;

    // GameInstance는 한 개의 WorldContext를 소유한다.
    // NOTE : 그렇다고 World의 생성과 제거의 주체는 아니다.
    TObjectPtr<class UGameInstance> OwningGameInstance;

    // UWorld를 가리키는 포인터 변수
    TObjectPtr<UWorld> ThisCurrentWorld;
};
```

## FWorldContext 요약

- **목적**:  
    엔진이 여러 UWorld를 다룰 때, 각 월드가 어떤 컨텍스트에 속하는지를 명확히 구분하기 위한 구조체.
    
- **구조적 개념**:  
    하나의 WorldContext는 하나의 "트랙"처럼 동작하며, 여러 레벨의 로드/언로드를 관리.  
    → 새로운 WorldContext를 추가하는 것은 "다른 트랙"을 만드는 것과 같음.
    
- **사용 예**:
    
    - `GameEngine`: 기본적으로 하나의 WorldContext만 사용.
        
    - `EditorEngine`:
        
        - 에디터 월드용 WorldContext
            
        - PIE(Play In Editor) 월드용 WorldContext
            
- **기능**:
    
    - PIE의 현재 UWorld 포인터 관리
        
    - 새로운 월드로 연결/이동할 때 상태 정보 유지
        
- **사용 제약**:
    
    - 외부 코드에서는 FWorldContext를 직접 다루면 안 되며, UWorld*만 사용 가능.
        
    - 엔진 내부에서 UWorld\*를 기반으로 적절한 WorldContext를 자동 조회함.
        
- **추가 기능**:
    
    - 외부 포인터(UWorld*)와 연동 가능 (예: PIE에서 `PlayWorld` 포인터 자동 업데이트)
        
    - 관련 함수: `AddRef()`, `SetCurrentWorld()`

## 왜 필요할까?

- `UEngine`은 `UWorld`의 생명 주기를 관리하는 객체이다.
- 하지만 `UWorld`는 `UObject`를 상속 받고 있다.
- 즉, 실질적으로 메모리 측면에서 생성과 소멸을 관리하고 있는 건 `GC`다
- 그래서 `UEngine`이 직접적으로 `UWorld`를 들고 있는 것 보다, `FWorldContext`를 일종의 `FileDescriptor`처럼 사용하는 것이다.
- 그렇게 함으로써 의존도를 낮춘다.

## 참고
[[EWorldType]]
