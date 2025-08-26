```cpp
/** 기본 물리 볼륨을 반환하며, 필요 시 생성한다 */
// InternalGetDefaultPhysicsVolume 참고
APhysicsVolume* UWorld::GetDefaultPhysicsVolume() const { return DefaultPhysicsVolume ? ToRawPtr(DefaultPhysicsVolume) : InternalGetDefaultPhysicsVolume(); }
```
---
- `DefaultPhysicsVolume`이 없으면 생성해줌
	  
	- `InternalGetDefaultPhysicsVolume()` 함수 호출
		  
- 해당 범위에서 **LineTrace** 및 **충돌 검사**들이 이루어진다.

## 주목 할 점
- 물리 씬 생성 여부와 상관없이 이 함수가 호출된다는 것이다. 즉, 시뮬레이션이 없더라도 반드시 필요한 객체라는 것!