```cpp
class ULevel : UObject
{
	enum class EIncrementalComponentState : uint8
    {
        Init,
        RegisterInitialComponents,
#if WITH_EDITOR || 1
        RunConstructionScripts,
#endif
        Finalize,
    };

    /** 이 레벨에서 점진적으로 액터 컴포넌트를 갱신하는 현재 단계 */
    // 이미 AActor의 초기화 단계를 다룸
    EIncrementalComponentState IncrementalComponentState;
}
```
---
### 역할
- 레벨을 한꺼번에 스트리밍 인이 됐거나 레벨 로드 했을 때 종속되어 있는 모든 액터들을 한꺼번에 로드하게 되면 멈추게 된다.
	  
- 이를 막기 위해 증분적으로 스테이지를 나눠서 한다는 것.
	  
	 1. 초기화
	 2. 등록
	 3. (에디터일 경우) CostructionScript 실행하기
		 - 게임의 경우 이미 쿠킹되어 있기 때문에 블루프린트가 바뀔 경우가 없음
	 4. 완료


