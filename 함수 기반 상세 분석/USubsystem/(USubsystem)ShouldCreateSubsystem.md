```cpp
virtual bool USubsystem::ShouldCreateSubsystem(UObject* Outer) const
{
	return true;
}
```
- `virtual` 키워드를 보면 알 수 있지만, 이를 **오버라이딩**하여 **생성 여부를 제어**할 수 있다.
	  
- **`CDO`** 객체에서 불린다고 생각하면 된다.