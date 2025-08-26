```cpp
/** UObject 참조를 자신의 소유 UObject로 위임하는 서브시스템 컬렉션
 *  (오브젝트는 AddReferencedObjects를 구현하고 호출을 Collection으로 전달해야 함)
 */
// FSubsystemCollectionBase 참조
template <typename TBaseType>
class FObjectSubsystemCollection : public FSubsystemCollectionBase
{
    /** FSubsystemCollection을 구성함, 소유 오브젝트(거의 항상 this)를 전달함 */
    FObjectSubsystemCollection();

    template <typename TSubsystemClass>
    const TArray<TSubsystemClass*>& GetSubsystemArray(const TSubclassOf<TSubsystemClass>& SubsystemClass) const;
};
```
---
### 참고
[[FSubsystemCollectionBase]]