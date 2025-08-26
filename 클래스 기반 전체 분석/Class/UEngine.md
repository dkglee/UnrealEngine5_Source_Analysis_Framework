```cpp
/**  
* Global engine pointer. Can be 0 so don't use without checking.  
*/  
ENGINE_API UEngine* GEngine = NULL;

// UObject를 상속 받고 있음
// UWorld의 생성과 소멸을 관리함 물론 WorldContext로써 간접적으로 관리
class UEngine : public UObject, public FExec
{
    // WorldContext 생성
    FWorldContext& CreateNewWorldContext(EWorldType::Type WorldType);
    // Engine을 초기화 하는 함수
    virtual void Init(IEngineLoop* InEngineLoop);

    // WorldContext를 관리하는 배열
    TIndirectArray<FWorldContext> WorldList;
    int32 NextWorldContextHandle;
};
```
---
## 1) 클래스의 역할/책임
- `UWorld`의 **생성·소멸** 및 **전반적인 관리**를 담당하는 전역 엔진 싱글톤.  
- **`FEngineLoop`**가 부팅 시 생성하며, 전역 포인터 **`GEngine`, `GEditor`, `GUnrealEd`** 등으로 참조됩니다.

## 2) 클래스가 제공하는 기능
<!-- 상세 API 목록은 아직 제공되지 않았으므로 비워 둡니다. -->

## 3) 관계
### 3.1 직접 상속(IsA)
- `UEditorEngine`
- `UUnrealEdEngine`

## 4) 상속 계보(참고)
- `UEngine`  
  ↳ `UEditorEngine`  
  ↳ `UUnrealEdEngine`

## 5) 멤버 변수
| 멤버 | 타입 | 설명 |
|------|------|------|
| `WorldList` | `TIndirectArray<FWorldContext>` | 엔진이 관리하는 **월드 컨텍스트 배열** |
| `NextWorldContextHandle` | `int32` | 새 `WorldContext` 할당 시 사용하는 **증가형 핸들 값** |

### 참고 자료
- [[KeyWord (TArray vs. TIndirectArray)]]
- [[FWorldContext]]
