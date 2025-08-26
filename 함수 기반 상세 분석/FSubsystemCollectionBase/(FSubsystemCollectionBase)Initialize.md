```cpp
/** 시스템 컬렉션을 초기화한다; 시스템들이 생성되고 초기화된다 */
void Initialize(UObject* NewOuter)
{
    if (Outer)
    {
        return;
    }

    Outer = NewOuter;
    
    if (ensure(BaseType) && ensure(SubSystemMap.Num() == 0))
    {
        check(IsInGameThread());

        {
	        // 단순화를 위하여 UDynamicSubsystem 처리 코드는 제거
            TArray<UClass*> SubsystemClasses;
            GetDerivedClasses(BaseType, SubsystemClasses, true);

            for (UClass* SubsystemClass : SubsystemClasses)
            {
                AddAndInitializeSubsystem(SubsystemClass);
            }
        }

        for (auto& Pair : SubsystemArrayMap)
        {
            Pair.Value.Empty();

            UpdateSubsystemArrayInternal(Pair.Key, Pair.Value);
        }

        GlobalSubsystemCollections.Add(this);
    }
}
```
---
## 상세 설명
### 1. 중복 초기화 방지
```cpp
if (Outer)
{
    return;
}
```
- `Outer`가 설정되었다는 것은 해당 `SubsystemCollection`은 초기화가 되었다는 뜻

---

### 2. `Outer` 할당
```cpp
Outer = NewOuter;
```
- 지금의 경우 `UWorld`의 객체가 할당 되었다고 보면 된다.

---

### 3. 게임 스레드에서만 초기화 진행
```cpp
check(IsInGameThread());
```
- `Game Thread`가 아닐 경우 엔진을 터트린다.
	  
- 오직 `Game Thread`에서만 동작

---

### 4. `WorldSubsystem`을 상속한 모든 클래스 수집
```cpp
TArray<UClass*> SubsystemClasses;
GetDerivedClasses(BaseType, SubsystemClasses, true);
```
- 여기서 `BaseType`은 `UWorldSubsystem`이다.
	  
- `GetDerivedClasses()` 함수를 통해 모든 클래스(`UClass`)를 들고 옴

---

### 5. `WorldSubSystem` 생성
```cpp
for (UClass* SubsystemClass : SubsystemClasses)
{
    AddAndInitializeSubsystem(SubsystemClass);
}
```
- 수집한 모든 클래스를 순회하며 `UWorldSubSystem` 객체들 생성
	  
- 내부적으로 `SubsystemMap`을 채우게 됨

---

### 6. `SubsystemArrayMap` 리프레시
```cpp
for (auto& Pair : SubsystemArrayMap)
{
    Pair.Value.Empty();

    UpdateSubsystemArrayInternal(Pair.Key, Pair.Value);
}
```
- `SubsystemArrayMap`을 순회한다.
    
- 각 엔트리의 배열 **내용만** 비운다: `Pair.Value.Empty()` (배열 객체는 유지).
    
- `UpdateSubsystemArrayInternal()`로 현재 등록된 서브시스템을 기준으로 배열을 다시 채운다.

---

### 7. `GlobalSubsystemCollections`에 현재의 컬렉션 등록
```cpp
GlobalSubsystemCollections.Add(this);
```
- 마지막에 `GlobalSubsystemCollections.Add(this)`로 이 컬렉션을 전역 목록에 등록한다.
	  
```cpp
/** 무조건 GameThread에서만 접근해야 함. 스레드 보호가 없음 */
static TArray<FSubsystemCollectionBase*> GlobalSubsystemCollections;
```
