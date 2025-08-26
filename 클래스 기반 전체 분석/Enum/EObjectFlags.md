```cpp
/**
 * 객체 인스턴스의 상태를 기술하는 플래그들
 * 이 enum을 수정할 때는 LexToString 구현도 반드시 업데이트하세요!
 */
enum EObjectFlags
{
	// 정말 이곳에 속한다고 판단되지 않으면 새 플래그를 추가하지 마세요. 다른 대안이 있습니다.
	// RF_Load 계열 플래그의 비트를 변경하면 레거시 직렬화가 필요해집니다.

	RF_NoFlags					= 0x00000000,	///< 플래그 없음. 형변환(cast)을 피하기 위해 사용.

	// 아래 첫 번째 그룹은 주로 객체의 “종류”에 관한 플래그입니다. Transient를 제외하면 지속성 플래그입니다.
	// 가비지 컬렉터(GC)도 주로 이들을 참고합니다.

	RF_Public					= 0x00000001,	///< 객체가 자신의 패키지 밖에서도 보임.
	RF_Standalone				= 0x00000002,	///< 참조가 없어도 편집을 위해 객체를 유지.
	RF_MarkAsNative				= 0x00000004,	///< 객체(UField)가 생성 시 네이티브로 표시됨(HasAnyFlags() 등에서 사용 금지).
	RF_Transactional			= 0x00000008,	///< 트랜잭션 처리 대상.
	RF_ClassDefaultObject		= 0x00000010,	///< 클래스 인스턴스의 기본 템플릿(CDO). 클래스마다 하나 생성됨.
	RF_ArchetypeObject			= 0x00000020,	///< 인스턴싱 템플릿으로 사용할 수 있는 객체. 모든 템플릿에 설정됨.
	RF_Transient				= 0x00000040,	///< 저장하지 않음(비영속).

	// 이 그룹은 주로 가비지 컬렉션과 관련됨.

	RF_MarkAsRootSet			= 0x00000080,	///< 루트 세트로 표시되어 참조가 없어도 GC되지 않음(HasAnyFlags() 등에서 사용 금지).
	RF_TagGarbageTemp			= 0x00000100,	///< GC를 사용하는 유틸리티용 임시 사용자 플래그. GC 자체는 해석하지 않음.

	// 이 그룹은 UObject의 수명주기 단계를 추적함.

	RF_NeedInitialization		= 0x00000200,	///< 초기화가 완료되지 않음. ~FObjectInitializer 완료 시 해제됨.
	RF_NeedLoad					= 0x00000400,	///< 로드 중에 객체 로딩이 필요함을 나타냄.
	RF_KeepForCooker UE_DEPRECATED(5.6, "This flag is now obsolete as the functionality is unnecessary.") = 0x00000800,	///< 쿠커가 사용 중이므로 GC 중에도 유지(UE 5.6부터 불필요하여 폐기).
	RF_NeedPostLoad				= 0x00001000,	///< PostLoad 처리가 필요함.
	RF_NeedPostLoadSubobjects	= 0x00002000,	///< 로드 중 서브오브젝트 인스턴싱 및 직렬화된 컴포넌트 참조 수정이 필요함.
	RF_NewerVersionExists		= 0x00004000,	///< 소유 패키지 리로드로 폐기되었고 더 새로운 버전이 존재함.
	RF_BeginDestroyed			= 0x00008000,	///< BeginDestroy가 호출됨.
	RF_FinishDestroyed			= 0x00010000,	///< FinishDestroy가 호출됨.

	// 기타 플래그

	RF_BeingRegenerated			= 0x00020000,	///< 로드 중 UClass 재생성(예: 블루프린트) 과정에서 UClass를 만드는 데 쓰이는 UObject 및 생성 중인 UClass에 표시됨(FLinkerLoad::CreateExport() 참고).
	RF_DefaultSubObject			= 0x00040000,	///< 클래스 생성자에서 만든 서브오브젝트 템플릿 및 그로부터 생성된 모든 인스턴스에 표시됨.
	RF_WasLoaded				= 0x00080000,	///< 로드된 UObject에 표시됨.
	RF_TextExportTransient		= 0x00100000,	///< 텍스트 형태(복사/붙여넣기 등)로 내보내지 않음. 보통 상위 객체 데이터로 재생성 가능한 서브오브젝트에 사용.
	RF_LoadCompleted			= 0x00200000,	///< 최소 한 번 linkerload에 의해 완전히 직렬화됨. 사용 금지, RF_WasLoaded로 대체해야 함.
	RF_InheritableComponentTemplate = 0x00400000, ///< 클래스 기본 객체가 아니라 “클래스 내부”에 저장된 서브오브젝트 템플릿. 기본 서브오브젝트 이후에 인스턴싱됨.
	RF_DuplicateTransient		= 0x00800000,	///< 어떤 형태의 복제(복사/붙여넣기, 바이너리 복제 등)에도 포함되면 안 됨.
	RF_StrongRefOnFrame			= 0x01000000,	///< 지속적인 함수 프레임에서의 참조를 강한 참조로 취급.
	RF_NonPIEDuplicateTransient	= 0x02000000,	///< PIE 세션을 위한 복제가 아닌 한 복제 대상에 포함되면 안 됨.
	RF_ImmutableDefaultObject	= 0x04000000,	///< 변경될 수 없고 앞으로도 변경되지 않는 클래스 기본 객체의 숨김 버전.
	RF_WillBeLoaded				= 0x08000000,	///< 로드 과정에서 생성되었으며 곧 로드될 예정.
	RF_HasExternalPackage		= 0x10000000,	///< 외부 패키지가 할당되어 있으며 최상위 패키지 획득 시 이를 조회해야 함.
	// RF_Unused				= 0x20000000,   // 사용하지 않음.

	// RF_MirroredGarbage는 EInternalObjectFlags::Garbage에 미러링됨.
	// GC 내부에서는 내부 플래그 확인이 더 빠르고, GC 외부에서는 객체 플래그 확인이 더 빠르기 때문.

	RF_MirroredGarbage			= 0x40000000,	///< 논리적으로 가비지이므로 참조되면 안 됨. 성능을 위해 내부 플래그의 Garbage와 미러링됨.
	RF_AllocatedInSharedPage	= 0x80000000,	///< 다른 UObject들과 공유되는 참조 카운트 기반 페이지에서 할당됨.
};
```
---
## 🗂️ 역할 / 목적
`EObjectFlags`는 **UObject 인스턴스**가 엔진 내부에서 어떤 상태·속성을 갖는지를 *비트 플래그*로 표현하는 핵심 열거형입니다.  
GC 대상 여부, 로딩 단계, 트랜잭션 처리, 리플렉션 가시성 등 **객체 생애주기의 모든 관문**에서 빠르게 조건 분기를 가능하게 합니다.

## 🔖 범주 / 의미 체계
- **가시성/영속성 플래그** : `RF_Public`, `RF_Standalone`, `RF_Transient` …  
- **GC & 루트세트 플래그** : `RF_MarkAsRootSet`, `RF_TagGarbageTemp`, `RF_MirroredGarbage` …  
- **생명주기 단계 플래그** : `RF_NeedInitialization`, `RF_NeedLoad`, `RF_BeginDestroyed`, `RF_FinishDestroyed` …  
- **기타 특수 목적** : `RF_StrongRefOnFrame`, `RF_AllocatedInSharedPage` …

## 🧾 값 상세
| Flag                                  | Hex          | 설명                                | 대표 사용처 / 영향 영역             |
| ------------------------------------- | ------------ | --------------------------------- | -------------------------- |
| **`RF_Public`**                       | `0x00000001` | 패키지 외부 리플렉션에 노출                   | 에디터 Outliner, Blueprint 참조 |
| **`RF_Standalone`**                   | `0x00000002` | 참조 없어도 에디터가 유지                    | 에셋 브라우저 임시 객체              |
| **`RF_MarkAsNative`**                 | `0x00000004` | C++ 네이티브 필드 표시 (조회 전용)            | 리플렉션 메타테이블 구축              |
| **`RF_Transactional`**                | `0x00000008` | Undo/Redo 스택에 기록                  | 에디터 편집 작업                  |
| **`RF_ClassDefaultObject`**           | `0x00000010` | CDO 식별                            | 클래스 템플릿 복제                 |
| **`RF_ArchetypeObject`**              | `0x00000020` | 인스턴싱 템플릿                          | Blueprint 클래스 디폴트          |
| **`RF_Transient`**                    | `0x00000040` | 저장 금지                             | 런타임 생성 임시 자원               |
| **`RF_MarkAsRootSet`**                | `0x00000080` | GC 루트 고정                          | `GEngine`, `GWorld` 등      |
| **`RF_TagGarbageTemp`**               | `0x00000100` | 유틸리티용 임시 마킹                       | 에디터 전용 스캔                  |
| **`RF_NeedInitialization`**           | `0x00000200` | `FObjectInitializer` 완료 전         | 객체 생성 직후                   |
| **`RF_NeedLoad`**                     | `0x00000400` | 디스크 로딩 필요                         | `LinkerLoad` 단계            |
| **`RF_KeepForCooker`**                | `0x00000800` | **(5.6 deprecated)** 쿠커 유지        | 과거 쿠킹                      |
| **`RF_NeedPostLoad`**                 | `0x00001000` | `PostLoad()` 필요                   | 에셋 패치 시                    |
| **`RF_NeedPostLoadSubobjects`**       | `0x00002000` | 서브오브젝트 인스턴싱 대기                    | 컴포넌트 Fix-up                |
| **`RF_NewerVersionExists`**           | `0x00004000` | 패키지 리로드로 폐기됨                      | Hot-Reload 충돌              |
| **`RF_BeginDestroyed`**               | `0x00008000` | `BeginDestroy()` 호출됨              | GC 파이프라인                   |
| **`RF_FinishDestroyed`**              | `0x00010000` | `FinishDestroy()` 완료              | 실제 메모리 해제 직전               |
| **`RF_BeingRegenerated`**             | `0x00020000` | Blueprint UClass 재생성 중            | 에디터 재컴파일                   |
| **`RF_DefaultSubObject`**             | `0x00040000` | CDO 내부 기본 컴포넌트                    | 자동 인스턴스                    |
| **`RF_WasLoaded`**                    | `0x00080000` | 최소 1회 완전 직렬화 완료                   | 에셋 캐싱 판단                   |
| **`RF_TextExportTransient`**          | `0x00100000` | 텍스트 내보내기 제외                       | 복사·붙여넣기                    |
| **`RF_LoadCompleted`**                | `0x00200000` | **(Legacy)** `RF_WasLoaded` 대체 예정 |                            |
| **`RF_InheritableComponentTemplate`** | `0x00400000` | 클래스에 보관된 컴포넌트 템플릿                 | 파생 클래스 자동 포함               |
| **`RF_DuplicateTransient`**           | `0x00800000` | 복제(듀플리케이트) 제외                     | PIE 빠른 복제                  |
| **`RF_StrongRefOnFrame`**             | `0x01000000` | 스크립트 콜스택 강한 참조                    | Blueprint VM               |
| **`RF_NonPIEDuplicateTransient`**     | `0x02000000` | PIE 중엔 복제 허용                      | Live Coding                |
| **`RF_ImmutableDefaultObject`**       | `0x04000000` | 불변 CDO 쉐도우                        | 에디터 비교                     |
| **`RF_WillBeLoaded`**                 | `0x08000000` | 생성됐지만 직후 로드 예정                    | Async 패키지 풀                |
| **`RF_HasExternalPackage`**           | `0x10000000` | 외부 패키지 연결                         | OFPA (One File Per Actor)  |
| **`RF_MirroredGarbage`**              | `0x40000000` | 내부 플래그 `Garbage`와 미러              | GC 성능 최적화                  |
| **`RF_AllocatedInSharedPage`**        | `0x80000000` | 공유 메모리 페이지 할당                     | PS5 메모리 최적화                |
### ⚠️ 주의 / 특이 사항

> 1. `RF_MarkAsNative` / `RF_MarkAsRootSet` 는 HasAnyFlags() 호출용이 아님 (생성 시 내부적으로만 설정).  
> 2. Deprecated 플래그(`RF_KeepForCooker`, `RF_LoadCompleted`)는 읽기 전용 호환성 장치로 남아있으며 신규 코드에서 설정하지 말 것.  
> 3. 멀티스레드 GC 환경에서 플래그 갱신은 원자적이어야 하며, 엔진 외부에서 직접 비트 연산으로 수정하는 것은 금지.

