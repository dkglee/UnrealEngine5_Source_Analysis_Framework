```cpp
int32 GuardMain(const TCHAR* CmdLine)
{
	// 'waitforattach'가 붙어 있다면
	// 디버거가 붙을 때까지 계속해서 무한 루프를 돌게 된다.
	// Starting Point를 디버깅할 때 굉장히 유용함
#if !(UE_BUILD_SHIPPING)
    if (FParse::Param(CmdLine, TEXT("waitforattach")))
    {
        while (!FPlatformMisc::IsDebuggerPresent())
        {
            FPlatformProcess::Sleep(0.1f);
        }
        UE_DEBUG_BREAK();
    }
#endif

	// 엔진에서 가장 빠른 초기화 코드 델리게이트
    FCoreDelegates::GetPreMainInitDelegate().Broadcast();

	// 엔진에서 자주 보이는 패턴
	// `EngineExit()`이 항상 호출되도록 보장하는 패턴
    struct EngineLoopCleanupGuard
    {
        ~EngineLoopCleanupGuard()
        {
            EngineExit();
        }
    } CleanupGuard;

	// PreInit 호출
    int32 ErrorLevel = EnginePreInit(CmdLine);

    {
#if WITH_EDITOR || 1
        if (GIsEditor)
        {
	        // 엔진 (에디터) 초기화 부분
            ErrorLevel = EditorInit(GEngineLoop);
        }
        else
#endif
        {
            ErrorLevel = EngineInit();
        }
    }

	// 엔진에서 자주 보이는 패턴 (경과한 시간 계산)
    double EngineInitializationTime = FPlatformTime::Seconds() - GStartTime;
    
    // 본격적인 언리얼 엔진의 메인 루프
    // GIsRequestingEditor가 끝날 때까지 틱을 돈다.
    while (!IsEngineExitRequested()) // GIsRequestingExit
    {
        EngineTick();
    }

#if WITH_EDITOR || 1
    if (GIsEditor)
    {
        EditorExit();
    }
#endif

    return ErrorLevel;
}
```
---


## 상세 분석
### 1. 시작 부분 디버깅 하기 좋은 파라미터
```cpp
	// 'waitforattach'가 붙어 있다면
	// 디버거가 붙을 때까지 계속해서 무한 루프를 돌게 된다.
	// Starting Point를 디버깅할 때 굉장히 유용함
#if !(UE_BUILD_SHIPPING)
    if (FParse::Param(CmdLine, TEXT("waitforattach")))
    {
        while (!FPlatformMisc::IsDebuggerPresent())
        {
            FPlatformProcess::Sleep(0.1f);
        }
        UE_DEBUG_BREAK();
    }
#endif
```
- `-waitforattach`를 넣어서 디버깅을 attach 한 뒤 완전히 시작 부분을 디버깅해보자!
- `UE_DEBUG_BREAK()`를 넣으면 해당 부분에서 BreakPoint를 걸 수 있음

---

### 2. 엔진 초기화 전 델리게이트 호출
```cpp
	// 엔진에서 가장 빠른 초기화 코드 델리게이트
    FCoreDelegates::GetPreMainInitDelegate().Broadcast();
```
- 엔진의 수정 없이 함수를 바인딩 함으로써 델리게이트를 걸 수 있다.
- `FCoreDelegates`를 한번 보는걸 추천
 
---

### 3. 엔진 초기화
```cpp
	// PreInit 호출
    int32 ErrorLevel = EnginePreInit(CmdLine);

#if WITH_EDITOR || 1
        if (GIsEditor)
        {
	        // 엔진 (에디터) 초기화 부분
            ErrorLevel = EditorInit(GEngineLoop);
        }
#endif
```
- 실질적인 엔진의 초기화 부분
- 우선은 에디터 초기화 부분을 탐색할 것이기 때문에 이 이 부분을 집중 연구! 
- `EnginePreInit()` 우선 탐구해보자
- `EditorInit()`도 탐구하도록 하자

---

### 4. 엔진 루프
```cpp
	// 본격적인 언리얼 엔진의 메인 루프
    // GIsRequestingEditor가 끝날 때까지 틱을 돈다.
    while (!IsEngineExitRequested()) // GIsRequestingExit
    {
        EngineTick();
    }
```
- 본격적인 엔진의 메인 루프이다.
- 모든 동작은 해당 Tick에서 이루어진다고 볼 수 있다.
- `GIsRequestingExit`가 true가 될 때까지 계속 루프를 돌면서 엔진을 틱한다.

---

### 5. 엔진 리소스 회수
```cpp
#if WITH_EDITOR || 1
    if (GIsEditor)
    {
        EditorExit();
    }
#endif
```
- 이전의 엔진 루프에서 빠져나오고 실행됨

---

### 구조

- 초기화 -> 루프 -> 리소스 회수
- 전형적인 프로그램의 구조를 보여주는듯 하다.
