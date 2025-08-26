```cpp
// WorldInfo 액터 생성
// AWorldSettings를 자세히 보지는 않고, 예시로 이해해 보자
// [ ] 에디터에서 AWorldSettings 설명 추가
AWorldSettings* WorldSettings = nullptr;
{
    // 액터를 스폰할 때 FActorSpawnParameters를 전달해야 함
    // FActorSpawnParameters 참고
    FActorSpawnParameters SpawnInfo;
    SpawnInfo.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

    // WorldSettings의 이름을 고정하여 호스트와 클라이언트의 새 월드 간 네트워크 복제가 동작하도록 함
    // UEngine의 WorldSettingsClass로 WorldSettings의 클래스를 오버라이드할 수 있음
    SpawnInfo.Name = GEngine->WorldSettingsClass->GetFName();

    // SpawnActor는 나중에 다룸(할 일 목록에 남겨 둠)
    WorldSettings = SpawnActor<AWorldSettings>(GEngine->WorldSettingsClass, SpawnInfo);
}

// 월드 생성자가 레벨을 로드할 계획이 없을 경우를 대비해 기본 게임 모드를 오버라이드할 수 있도록 허용
if (IVS.DefaultGameMode)
{
    WorldSettings->DefaultGameMode = IVS.DefaultGameMode;
}
```
---
- 우리가 World를 생성하고 World Setting에서 GameMode 설정하고 하는 그 부분을 말하는 것
	  
- `Persistent Level`이 해당 정보를 가지고 있음
