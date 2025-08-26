```cpp
/** UWorld와 동일한 수명을 가지며 자동으로 인스턴스화·초기화되는 시스템의 기반 클래스 */
// USubsystem 참고
class UWorldSubsystem : public USubsystem
{
    /** 라인 배처 등 월드 컴포넌트(레벨 컴포넌트 포함) 갱신이 모두 끝난 뒤 호출됨 */
    virtual void OnWorldComponentsUpdated(UWorld& World) {}
};
```
---
### 역할 / 책임
- **월드-스코프 상태 관리자**
    
    - `UWorld`가 생성될 때 자동으로 인스턴스화되어, 해당 월드가 파괴될 때까지 함께 존재한다. 따라서 “월드마다 하나”인 시스템(예: 데이터 레이어, 파티션, 부력 시스템 등)을 구현할 때 기반 클래스로 사용된다.

### 참고
[[USubsystem]]
