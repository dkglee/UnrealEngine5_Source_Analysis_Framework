```cpp
// 엔진에서 자주 보이는 패턴 (경과한 시간 계산)
    double EngineInitializationTime = FPlatformTime::Seconds() - GStartTime;
```

- 언리얼에서 경과 시간을 계산할 때 자주 사용하는 패턴