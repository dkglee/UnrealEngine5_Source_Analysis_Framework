```cpp
int32 FEngineLoop::Init()
{		
	// We're UnrealEd.
	// deulee : 아래에서 Class이름을 추출하고 해당 클래스를 들고옴
	FString UnrealEdEngineClassName;
	GConfig->GetString(TEXT("/Script/Engine.Engine"), TEXT("UnrealEdEngine"), UnrealEdEngineClassName, GEngineIni);
	EngineClass = StaticLoadClass(UUnrealEdEngine::StaticClass(), nullptr, *UnrealEdEngineClassName);
	// 들고온 EngineClass로 UEngine을 만듬
	// 여기서는 ULyraEditorEngine
	// 상속 구조
	// ULyraEditorEngine -> UUnrealEdEngine -> UEditorEngine -> UEngine(루트) 순서로 상속됨
	if (EngineClass == nullptr)
	{
		UE_LOG(LogInit, Fatal, TEXT("Failed to load UnrealEd Engine class '%s'."), *UnrealEdEngineClassName);
	}
	GEngine = GEditor = GUnrealEd = NewObject<UUnrealEdEngine>(GetTransientPackage(), EngineClass);

	{
		SCOPED_BOOT_TIMING("GEngine->ParseCommandline()");
		GEngine->ParseCommandline();
	}

	// 최대 프레임 수(FPS) 계산
	{
		SCOPED_BOOT_TIMING("InitTime");
		InitTime();
	}

	// UEngine의 초기화 부분 (World 생성)
	{
		SCOPED_BOOT_TIMING("GEngine->Init");
		GEngine->Init(this);
	}

	{
		SCOPED_BOOT_TIMING("GEngine->Start()");
		GEngine->Start();
	}
}
```
---
## 상세 설명서
### 1. GEngine 생성
```cpp
GEngine = GEditor = GUnrealEd = NewObject<UUnrealEdEngine>(GetTransientPackage(), EngineClass);
```
- 이전에 어떤 엔진으로 생성할 지 `StaticLoadClass()`를 통하여 `EngineClass`로 할당한다.
- `NewObject` 최상단은 `UObject`로 관리되는 것을 알 수 있다.
- 즉, GC에 의해 메모리가 관리된다는 것

---
### 2. 최대 프레임 수 계산
```cpp
	// 최대 프레임 수(FPS) 계산
	{
		SCOPED_BOOT_TIMING("InitTime");
		InitTime();
	}
```
- `InitTime()` 내부를 보면 MaxFrameCount를 계산하는 것을 볼 수 있다.

```cpp
void FEngineLoop::InitTime()
{
	// Init variables used for benchmarking and ticking.
	FApp::SetCurrentTime(FPlatformTime::Seconds());
	MaxFrameCounter				= 0;
	MaxTickTime					= 0;
	TotalTickTime				= 0;
	LastFrameCycles				= FPlatformTime::Cycles();

	float FloatMaxTickTime		= 0;
#if (!UE_BUILD_SHIPPING || ENABLE_PGO_PROFILE)
	FParse::Value(FCommandLine::Get(),TEXT("SECONDS="),FloatMaxTickTime);
	MaxTickTime					= FloatMaxTickTime;

	// look of a version of seconds that only is applied if FApp::IsBenchmarking() is set. This makes it easier on
	// say, iOS, where we have a toggle setting to enable benchmarking, but don't want to have to make user
	// also disable the seconds setting as well. -seconds= will exit the app after time even if benchmarking
	// is not enabled
	// NOTE: This will override -seconds= if it's specified
	if (FApp::IsBenchmarking())
	{
		if (FParse::Value(FCommandLine::Get(),TEXT("BENCHMARKSECONDS="),FloatMaxTickTime) && FloatMaxTickTime)
		{
			MaxTickTime			= FloatMaxTickTime;
		}
	}

	// Use -FPS=X to override fixed tick rate if e.g. -BENCHMARK is used.
	float FixedFPS = 0;
	FParse::Value(FCommandLine::Get(),TEXT("FPS="),FixedFPS);
	if( FixedFPS > 0 )
	{
		FApp::SetFixedDeltaTime(1 / FixedFPS);
	}

#endif // !UE_BUILD_SHIPPING

	// convert FloatMaxTickTime into number of frames (using 1 / FApp::GetFixedDeltaTime() to convert fps to seconds )
	MaxFrameCounter = FMath::TruncToInt(MaxTickTime / FApp::GetFixedDeltaTime());
}
```

---

### 3. `GEngine` 초기화
```cpp
	// UEngine의 초기화 부분 (World 생성)
	{
		SCOPED_BOOT_TIMING("GEngine->Init");
		GEngine->Init(this);
	}
```
