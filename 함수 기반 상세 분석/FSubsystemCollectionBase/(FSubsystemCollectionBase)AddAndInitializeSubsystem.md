```cpp
USubsystem* AddAndInitializeSubsystem(UClass* SubsystemClass)
{
	// 클래스 당 한개만 만들어야 함
    if (!SubsystemMap.Contains(SubsystemClass))
    {
        if (SubsystemClass && !SubsystemClass->HasAllClassFlags(CLASS_Abstract))
        {
            check(SubsystemClass->IsChildOf(BaseType));

			// CDO를 통하여 생성 여부 판단
            const USubsystem* CDO = SubsystemClass->GetDefaultObject<USubsystem>();
            if (CDO->ShouldCreateSubsystem(Outer))
            {
	            // 실제로 만드는 부분
                USubsystem* Subsystem = NewObject<USubsystem>(Outer, SubsystemClass);
                SubsystemMap.Add(SubsystemClass, Subsystem);
                Subsystem->InternalOwningSubsystem = this;
                Subsystem->Initialize(*this);
                return Subsystem;
            }
        }
        return nullptr;
    }
    return SubsystemMap.FindRef(SubsystemClass);
}
```
---
## 상세 설명
### 1. 중복 체크
```cpp
if (!SubsystemMap.Contains(SubsystemClass))
```
- 클래스 타입당 오직 하나의 인스턴스만 허용

---

### 2. 유효·비추상 검사
```cpp
if (SubsystemClass && !SubsystemClass->HasAllClassFlags(CLASS_Abstract))
```
- 추상(Abstract)이 아닌 Subsystem 클래스만 인스턴스를 생성
	  
- `UClass::ClassFlags`에 클래스 정보를 담고 있음

---

### 3. 타입 계층 확인
```cpp
check(SubsystemClass->IsChildOf(BaseType));
```
- 만들고자 하는 Subsystem은 `BaseType`으로부터 파생되어야 함.
	  
- 그 이외의 것이 들어오는 경우 엔진 터트림

---

### 4. 생성 필요 여부 판단
```cpp
const USubsystem* CDO = SubsystemClass->GetDefaultObject<USubsystem>();
if (CDO->ShouldCreateSubsystem(Outer))
```
- `ShouldCreateSubsystem()` 함수로 Subsystem 생성 여부를 결정함
	 
	-  이 함수를 오버라이딩하여 생성 흐름을 제어
	  
- **CDO**를 통하여 **ShouldCreateSubsystem()** 함수를 호출

---

### 5. Subsystem 생성
```cpp
USubsystem* Subsystem = NewObject<USubsystem>(Outer, SubsystemClass);
SubsystemMap.Add(SubsystemClass, Subsystem);
Subsystem->InternalOwningSubsystem = this;
Subsystem->Initialize(*this);
return Subsystem;
```
- `SubsystemClass`로 새로운 `USubsystem` 생성
	  
- 이 경우 **`UWorldSubsystem`** 객체
	  
- 초기화 후 `SubsystemMap`에 추가 후 반환

---

### 6. 이외의 경우
```cpp
return SubsystemMap.FindRef(SubsystemClass);
```
- 이미 존재하는 것이면 `SubsystemMap`에서 참조 반환

