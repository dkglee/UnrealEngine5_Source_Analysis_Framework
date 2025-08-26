```cpp
	// 엔진에서 자주 보이는 패턴
	// `EngineExit()`이 항상 호출되도록 보장하는 패턴
    struct EngineLoopCleanupGuard
    {
        ~EngineLoopCleanupGuard()
        {
            EngineExit();
        }
    } CleanupGuard;
```

- 언리얼에서 자주 보이는 패턴
- 지역 객체를 만들어 두고, 스코프가 끝날 때(정상 종료든 예외든) 소멸자에서 정리 코드를 실행하도록 하는 기법