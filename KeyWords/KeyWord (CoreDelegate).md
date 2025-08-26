```cpp
FCoreDelegates
```
- Unreal Engine에서 **엔진 전체의 전역적인 이벤트 처리 및 콜백 시스템을 제공하는 정적 구조체**
- 내부에 유용한 델리게이트가 많으므로 한번 확인해보면 좋다.

---
### 유용한 것들
```cpp
// 엔진에서 가장 빠른 초기화 코드 델리게이트
FCoreDelegates::GetPreMainInitDelegate().Broadcast();
```
