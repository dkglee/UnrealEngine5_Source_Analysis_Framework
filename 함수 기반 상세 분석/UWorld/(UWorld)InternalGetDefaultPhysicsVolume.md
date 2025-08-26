```cpp
APhysicsVolume* UWorld::InternalGetDefaultPhysicsVolume() const
{
    // 아직 DefaultPhysicsVolume이 없으면 새로 생성한다
    if (DefaultPhysicsVolume == nullptr)
    {
        // 인스턴스화할 PhysicsVolume의 클래스를 가져온다:
        AWorldSettings* WorldSettings = GetWorldSettings(false, false);
        UClass* DefaultPhysicsVolumeClass = (WorldSettings ? WorldSettings->DefaultPhysicsVolumeClass : nullptr);

        if (DefaultPhysicsVolumeClass == nullptr)
        {
            DefaultPhysicsVolumeClass = ADefaultPhysicsVolume::StaticClass();
        }

        // 볼륨 스폰: 여기서 FActorSpawnParameters를 사용한다
        FActorSpawnParameters SpawnParams;
        SpawnParams.bAllowDuringConstructionScript = true;
        {
            UWorld* MutableThis = const_cast<UWorld*>(this);
            MutableThis->DefaultPhysicsVolume = MutableThis->SpawnActor<APhysicsVolume>(DefaultPhysicsVolumeClass, SpawnParams);
            MutableThis->DefaultPhysicsVolume->Priority = -1000000;
        }
    }
    return DefaultPhysicsVolume;
}
```
---
## 상세 설명
### 1. 기존 `DefaultPhysicsVolume` 존재 여부 확인
```cpp
if (DefaultPhysicsVolume == nullptr)
```
- 검사 후 이미 있으면 그대로 반환

---

### 2. 사용할 클래스 결정
```cpp
// 인스턴스화할 PhysicsVolume의 클래스를 가져온다:
AWorldSettings* WorldSettings = GetWorldSettings(false, false);
UClass* DefaultPhysicsVolumeClass = (WorldSettings ? WorldSettings->DefaultPhysicsVolumeClass : nullptr);

if (DefaultPhysicsVolumeClass == nullptr)
{
	DefaultPhysicsVolumeClass = ADefaultPhysicsVolume::StaticClass();
}
```
- `WorldSettings`에는 월드에 필요한 클래스 정의들이 있으며, 이는 월드의 전반적 동작을 바꿀 수 있다.
	  
- `WorldSettings`에서 이 클래스를 오버라이드할 수도 있다.
	  
- 없으면 기본 클래스로 설정

---

### 3. 생성 후 반환
```cpp
// 볼륨 스폰: 여기서 FActorSpawnParameters를 사용한다
FActorSpawnParameters SpawnParams;
SpawnParams.bAllowDuringConstructionScript = true;
{
	UWorld* MutableThis = const_cast<UWorld*>(this);
	MutableThis->DefaultPhysicsVolume = MutableThis->SpawnActor<APhysicsVolume>(DefaultPhysicsVolumeClass, SpawnParams);
	MutableThis->DefaultPhysicsVolume->Priority = -1000000;
}

return DefaultPhysicsVolume;
```
- 생성 후 반환

