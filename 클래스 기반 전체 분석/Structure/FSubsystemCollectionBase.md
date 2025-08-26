```cpp
// FSubsystemCollectionBase의 멤버 변수 보기
class FSubsystemCollectionBase
{
    FSubsystemCollectionBase(UClass* InBaseType);
    
    USubsystem* AddAndInitializeSubsystem(UClass* SubsystemClass);

    /** 시스템 컬렉션을 초기화한다; 시스템이 생성되고 초기화됨 */
	void Initialize(UObject* NewOuter);

    // 내부 배열을 갱신
    // 각 엔트리를 순회하며 KeyClass 가 SubsystemClass 의 자식이면 해당 인스턴스를 SubsystemArray 에 추가
    void UpdateSubsystemArrayInternal(UClass* SubsystemClass, TArray<USubsystem*>& SubsystemArray) const;

    // 내부 배열을 얻음
    // SubsystemArrayMap 에 키가 없으면 새 리스트를 추가하고 UpdateSubsystemArrayInternal 로 채운 뒤 반환
    // 이미 존재하면 FindChecked 로 찾아 반환
    const TArray<USubsystem*>& GetSubsystemArrayInternal(UClass* SubsystemClass) const;

    // UWorldSubsystem 을 다루는 경우:
    // - Outer 는 UWorld
    // - BaseType 은 UWorldSubsystem
    UObject* Outer;
    UClass* BaseType;

    // UClass(서브시스템의 UClass) -> UObject(서브시스템 인스턴스) 매퍼
    // - FObjectSubsystemCollection<SubsystemType> 초기화 시점에는 클래스별 1:1 매핑이 지원됨
    TMap<TObjectPtr<UClass>, TObjectPtr<USubsystem>> SubsystemMap;

	// 타입별 서브시스템 인덱스 캐시.
	//   Key  : 조회 기준 타입(구체 클래스/부모 클래스/인터페이스의 UClass*)
	//   Value: 해당 타입으로 '대입 가능한' USubsystem*들의 배열
	// 목적  : GetSubsystemArray<T>() / ForEachSubsystemOfClass() 호출 시
	//         전 컬렉션을 순회하지 않고 즉시 반환하기 위한 가속용 인덱스.
	// 주의  : 한 오너(Engine/GameInstance/World/LocalPlayer) 기준으로
	//         동일 서브시스템 클래스는 보통 1개가 원칙이며,
	//         이 맵은 '같은 기반 타입(부모/인터페이스)을 공유하는 서로 다른 구현들'을
	//         한데 모으기 위한 것이다(동일 클래스의 다중 인스턴스용 아님).
	mutable TMap<UClass*, TArray<USubsystem*>> SubsystemArrayMap;
};
```
---
### 1. 역할 및 책임

- **서브시스템(Low-level 서비스 객체) 수명 관리자**
    
    - 지정된 `BaseType` (예: `UWorldSubsystem`, `UGameInstanceSubsystem`)을 상속한 모든 서브시스템을 찾아 **한 엔진 콘텍스트(예: UWorld, UEngine 등) 안에서 단 하나의 인스턴스만** 생성·보유한다.
	    
	- 예로 World에는 `FObjectSubsystemCollection`으로 `WorldSubSystem`들을 관리
		
- **초기화 오케스트레이터**
    
    - `Initialize()`에서 파생 클래스를 열거→`AddAndInitializeSubsystem()`으로 각각을 생성→`USubsystem::Initialize()`까지 호출해 서브시스템을 완전한 실행 상태로 만든다.
        
- **조회 및 캐싱 허브**
    
    - 클래스→인스턴스 매핑(`SubsystemMap`)과  
        클래스→배열(`SubsystemArrayMap`)을 유지해 빠르게 서브시스템 또는 그 그룹을 찾을 수 있도록 한다.
        

### 2. 클래스가 제공하는 주요 기능

- **`AddAndInitializeSubsystem(UClass*)`**
    
    - 이미 존재 여부 검사→없으면 CDO로 `ShouldCreateSubsystem()` 체크→생성·초기화→`SubsystemMap`에 저장.
        
- **`Initialize(UObject* NewOuter)`**
    
    - `Outer`를 설정하고, `BaseType`의 모든 파생 클래스를 열거해 위의 함수로 등록한다.
        
    - 재호출 안전성(이미 초기화되면 즉시 return)과 **게임 스레드 보장**(`check(IsInGameThread())`) 로직 포함.
        
- **`GetSubsystemArrayInternal(UClass*)` / `UpdateSubsystemArrayInternal()`**
    
    - 특정 부모 타입의 모든 서브시스템을 반환(캐시 존재 시 재사용, 없으면 생성).
        
- **전역 레지스트리 등록**
    
    - `GlobalSubsystemCollections.Add(this)`로 런타임 동안 다른 시스템이 모든 컬렉션에 접근할 수 있도록 연결한다.
        

### 3. 관계 (IsA / 소유 관계)

- **보유자(Outer)**
    
    - `UWorld`, `UGameInstance`, `UEngine`, `UEditorSubsystemCollection` 등 **각 엔진 콘텍스트 객체**가 자신만의 `FSubsystemCollection<…>`를 가진다.
        
    - 따라서 이 클래스는 “콘텍스트 객체의 서브시스템 컨테이너” 역할.
        
- **포함(Is-Owned)**
    
    - 서브시스템 인스턴스는 컨테이너에 소유(Soft Reference 없음). GC는 `TObjectPtr` 경유로 Outer → Subsystem 참조 사슬을 따라 동작한다.
        

### 4. 상속 계보
(없음) ← `FSubsystemCollectionBase`

### 5. 멤버 변수 요약

- **`UObject* Outer`** : 컬렉션이 귀속된 콘텍스트 객체(예: `UWorld`).
    
- **`UClass* BaseType`** : 등록 가능한 서브시스템의 루트 클래스.
    
- **`TMap<TObjectPtr<UClass>, TObjectPtr<USubsystem>> SubsystemMap`**
    
    - “클래스 당 단일 인스턴스” 보장 용 주 컨테이너.
        
- **`mutable TMap<UClass*, TArray<USubsystem*>> SubsystemArrayMap`**
	  
	- 부모/인터페이스/구체 **타입 기준**으로 대입 가능한 `USubsystem*`들을 모아 둔 **인덱스 캐시**. (`ForEachSubsystemOfClass`, `GetSubsystemArray` 가속)
	    
	- **배열인 이유:** 같은 부모/인터페이스를 공유하는 여러 구현 클래스의 인스턴스를 한 번에 반환하기 위해서임.  
	    → **동일 클래스의 다중 인스턴스(클래스 1 : 인스턴스 N) 수용 목적이 아님.**

### 6. 특징 및 주의점

- **싱글턴 보장**: `SubsystemMap`이 키가 있으면 새로 만들지 않고 기존 객체를 돌려주므로, 동일 클래스 중복·생성을 막는다.
    
- **런타임 확장성**: `GetDerivedClasses()`를 사용해 릴리즈 빌드에서도 리플렉션 기반으로 **플러그인·모듈이 추가한 서브시스템**을 자동 인식한다.
    
- **게임 스레드 안전성**: 모든 공개 API는 게임 스레드에서만 호출된다는 전제를 내부 `check()`로 강제한다.
    
- **GC-친화적 설계**: `TObjectPtr` 사용으로 참조 수집기와 직접 연동, 외부가 `Outer`를 잃으면 컬렉션과 서브시스템도 함께 파기된다.
    
- **동적 서브시스템 지원**: `SubsystemArrayMap`이 클래스를 넘어선 그룹 조회·다중 인스턴스 패턴(예: `UDynamicSubsystem`)을 수용한다.
    
- **전역 조회 가능**: `GlobalSubsystemCollections`에 자신을 등록해, 다른 엔진 모듈이 “모든 월드 서브시스템”을 순회할 수 있다.
    
- **확장 포인트**: 서브시스템 자체에서 `ShouldCreateSubsystem()`을 오버라이드하면 **콘텍스트·플랫폼·빌드 구성**에 따라 선택적 생성이 가능하다.
	  
- **`UDynamicSubsystem`**: 사용을 웬만하면 하지 말자. 혼란을 야기함. 왜냐하면 SubSystem은 보통 컨텍스트 당 한 개 존재하는 것을 중요로 하는데 이는 복수 개를 지원하기 때문에 코드에 혼란을 야기할 수 있음
