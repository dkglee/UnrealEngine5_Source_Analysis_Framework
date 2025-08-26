```cpp
/** UObject를 위한 유틸리티 함수들을 제공합니다.  
 * 이 클래스는 직접 사용되어서는 안 됩니다.
 */
class UObjectBaseUtility : public UObjectBase
{
    /** 
     * 주어진 UClass* 타입을 기준으로, 외부(Outer) 체인을 따라가며 해당 타입의 UObject를 찾습니다.
     * UObject::GetOuter()를 재귀적으로 호출하며, 일치하는 타입이 나오면 반환합니다.
     */
    UObject* GetTypedOuter(UClass* Target) const;

    /**
     * 템플릿 버전의 GetTypedOuter.
     * T는 UObject를 상속한 타입이어야 하며, T::StaticClass()를 이용해 타입 탐색을 수행합니다.
     */
    template <typename T>
    T* GetTypedOuter() const;

    /**
     * 이 객체가 템플릿 객체(CDO 또는 Archetype)인지 확인합니다.
     * 외부 객체 중 하나라도 Template 플래그(RF_ArchetypeObject, RF_ClassDefaultObject)를 가지고 있다면 true를 반환합니다.
     */
    bool IsTemplate(EObjectFlags TemplateTypes = RF_ArchetypeObject|RF_ClassDefaultObject) const;
};
```
---
## 1) 클래스의 역할/책임
- `UObject` 계열 공통 유틸리티를 제공하는 **중간 베이스 클래스**.
- Outer 체인 탐색과 템플릿 판정(Archetype/CDO)을 비롯한 **Reflection 보조 기능**을 캡슐화.

## 2) 클래스가 제공하는 기능
- **`UObject* GetTypedOuter(UClass* Target) const`**:
  - 현재 객체의 **Outer 체인**을 순회하여 `Target` 타입(`UClass*`)과 일치하는 첫 Outer 반환.
  - 일치하는 것이 없으면 `nullptr`.
  
- `**template <typename T> T* GetTypedOuter() const**`:
  - `T`가 `UObject` 파생 타입일 때, `T::StaticClass()`를 사용해 **타입 안전한 Outer 탐색**을 제공.
  
- **`bool IsTemplate(EObjectFlags TemplateTypes = RF_ArchetypeObject | RF_ClassDefaultObject) const`**:
  - **자기 자신 또는 어떤 Outer라도** `TemplateTypes` 플래그를 가지면 `true` 반환.
  - CDO(Class Default Object) 및 다양한 Archetype 판정에 사용.


> 성능/제약:
> - Outer 탐색은 **체인 길이에 비례(O(depth))**하는 선형 비용.
> - 템플릿 버전은 `T`가 `UObject` 파생임이 전제(`StaticClass()` 존재).
> - 플래그 판정은 누적 Outer까지 확인하므로 **객체의 컨텍스트(Outer 구성)**에 영향.

## 3) 관계
### 3.1 직접 상속( IsA )
- **Direct Subclasses**
  - `UObject` (바로 위 상속자이자 대부분의 엔진 타입의 공통 루트로 이어짐)

> 대표적인 간접 상속자(예시):
> - `AActor`, `UActorComponent`/`USceneComponent`
> - `UClass`, `UBlueprintGeneratedClass`
> - `UTexture`, `UMaterial`, `USkeletalMesh` … (대부분의 `UObject` 파생형)

### 3.2 포함( Has )
- **Direct Containment**: 일반적으로 **없음** (상속 베이스로만 사용).

### 3.3 기타 구조적 관계
- **Outer 체인**: `UPackage`, `UWorld`, `ULevel`, `AActor`, `UActorComponent` 등 다양한 소유자 유형이 Outer로 구성될 수 있으며, 본 클래스의 유틸리티는 이 **Outer 기반 컨텍스트**를 전제로 동작.

## 4) 상속 계보(참고)
- Base Classes:
  - `UObjectBase` ← `UObjectBaseUtility`
- Interfaces/Mixins: (해당 없음)

### 참고
[[UObjectBase]]
