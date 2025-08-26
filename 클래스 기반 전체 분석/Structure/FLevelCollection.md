```cpp
/**
 * UWorld 내부에서 특정 ELevelCollectionType에 속하는 레벨들의 그룹과
 * 해당 레벨들을 올바르게 틱/업데이트하기 위한 컨텍스트를 보유한다.
 * move-only 객체다.
 */
 // FLevelCollection은 ELevelCollectionType 기반의 컬렉션이다
struct FLevelCollection
{
    /** 이 컬렉션의 유형 */
    // ELevelCollectionType 참고
    ELevelCollectionType CollectionType;

    /**
     * 이 컬렉션과 연관된 퍼시스턴트 레벨
     * 원본 컬렉션과 복제된 컬렉션은 각자 고유한 인스턴스를 가진다
     */
    // 보통 OwnerWorld의 PersistentLevel
    TObjectPtr<class ULevel> PersistentLevel;

    /** 이 컬렉션에 포함된 모든 레벨 */
    TSet<TObjectPtr<ULevel>> Levels;
};
```
---
### FLevelCollection의 역할과 책임

- **레벨 그룹 보관**  
    `UWorld` 안에서 [[ELevelCollectionType]]에 따라 묶인 `ULevel`들을 한데 관리한다. 각 레벨은 자기 컬렉션 포인터를 캐시하고, 언로드 시 캐시를 깨끗하게 비워서 댕글링 포인터를 막는다.
    
- **Persistent Level 참조**  
    컬렉션의 기준이 되는 `PersistentLevel`을 별도로 저장한다. `SetPersistentLevel()`을 호출하면 자동으로 `Levels` 집합에도 포함된다.
    
- **Levels 집합 관리**  
    `TSet<TObjectPtr<ULevel>> Levels`에 모든 레벨을 보관하며, `AddLevel()`과 `RemoveLevel()`로 원자적으로 추가·삭제할 수 있다.
    
- **틱/업데이트 컨텍스트 분리**  
    컬렉션마다 고유한
    
    - `UNetDriver`(복제)
        
    - `UDemoNetDriver`(리플레이)
        
    - `AGameStateBase`(게임 전역 상태)  
        를 갖는다. 한 프레임 동안 `UWorld`가 활성 컬렉션을 바꿔 가며 Tick을 돌리므로, 서로 다른 네트워크 파이프라인과 게임 상태를 안전하게 격리할 수 있다.
        
- **가시성 토글**  
    `SetIsVisible()` / `IsVisible()`로 렌더링·충돌·틱 처리 범위를 즉시 전환할 수 있다. 리플레이나 스펙테이터 뷰처럼 복제 레벨만 잠시 보여주고 싶을 때 유용하다.
    
- **Move-Only 설계**  
    복사 생성자와 복사 대입 연산자가 삭제되어 있고 이동만 허용된다. 얕은 복사로 인한 컨텍스트 공유 오류를 사전에 차단하려는 의도다.


---

### 결론
**Level Collection의 역할**

- 월드에 포함된 여러 레벨을 **_타입_** 기준으로 구분한 집합.
    
- 각 컬렉션 안에는 동일한 용도를 가진 레벨들이 `Set` 형태로 저장됨.
    
- 계층 구조(부모-자식)로 묶는 것은 아니고, 논리적 분류용 컨테이너라고 보면 됨.


### 참고
[[ELevelCollectionType]]
