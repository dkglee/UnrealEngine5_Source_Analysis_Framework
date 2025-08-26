```cpp
/** 컴포넌트에서 월드로의 변환 값을 재계산한다 */
// 구체적인 예시는 USceneComponent를 상속한 모든 클래스들이다
virtual void UpdateComponentToWorld(EUpdateTransformFlags UpdateTransformFlags = EUpdateTransformFlags::None, ETeleportType Teleport = ETeleportType::None) {}
```
---
- **부모(AttachParent)의 월드 변환** × **자신의 상대 변환(RelativeLocation/Rotation/Scale)** → **ComponentToWorld** 갱신
    
- 절대 플래그(bAbsoluteLocation/Rotation/Scale)와 소켓 오프셋 등을 반영