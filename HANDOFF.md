# HANDOFF

이 프로젝트를 이어받는 사람(또는 에이전트)이 알아야 할 작업 규칙을 기록합니다.

## 커밋 컨벤션

### 형식

```
[작업 태그] YYMMDD 작업내용
```

- **작업 태그**: 대문자 영문, 대괄호로 감쌈 (아래 표 참고)
- **날짜**: 커밋한 날짜, 숫자 6자리 `YYMMDD` (예: 2026년 11월 16일 → `261116`)
- **작업내용**: 한국어로 간결하게. 파일·클래스·에셋 이름은 원래 표기 그대로 써도 됨

### 예시

```
[ADD] 261116 HANDOFF 추가
[FIX] 261117 소총 재장전 중 사격 가능한 버그 수정
[MOD] 261118 BP_ShooterCharacter 이동 속도 조정
```

### 작업 태그

| 태그 | 용도 |
|---|---|
| `ADD` | 새 기능, 파일, 에셋 추가 |
| `MOD` | 기존 기능·에셋·설정 변경 |
| `FIX` | 버그 수정 |
| `DEL` | 기능, 파일, 에셋 삭제 |
| `REFACTOR` | 동작 변화 없는 구조 개선 |
| `DOCS` | 문서 작성·수정 |
| `CHORE` | 빌드, 설정, git 등 잡무 |

### 규칙

- 제목 줄은 위 형식을 반드시 지킵니다. 상세 설명이 필요하면 한 줄 띄우고 본문에 씁니다.
- 한 커밋에는 한 가지 작업만 담습니다. 성격이 다른 변경은 커밋을 나눕니다.

## 프로젝트 개요

- 목적: 언리얼 엔진 연습 및 FPS R&D
- 엔진: Unreal Engine 5.8 (`C:\Program Files\Epic Games\UE_5.8`)
- 베이스: First Person 템플릿 + `Variant_Shooter`
  - 무기(권총·소총·유탄발사기), 투사체, 픽업, `DT_WeaponList`
  - StateTree 기반 적 AI, EQS, NPC 스포너
  - 탄약 HUD, 맵 `Lvl_FirstPerson`(기본 시작 맵) / `Lvl_ArenaShooter`
- C++ 모듈: `Test` (Runtime, 의존성 `Core`, `CoreUObject`, `Engine`, `InputCore`, `EnhancedInput`, 로그 카테고리 `LogTest`)
- 렌더링: Lumen, Virtual Shadow Maps, 레이트레이싱, Substrate, DX12 / SM6

## 작업 콘텐츠 구조 (`Content/FPS/`)

템플릿 원본은 가능한 한 건드리지 않고, 내 작업은 `Content/FPS/`에 두며 템플릿 에셋은 **자식 블루프린트**로 확장한다.

```
Content/FPS/
├─ BP_FPSGameMode                     (부모: BP_ShooterGameMode, Default Pawn = BP_FPSCharacter)
├─ Characters/BP_FPSCharacter         (부모: BP_ShooterCharacter, 상호작용 트레이스)
├─ Input/Actions/IA_Interact          (Digital bool)
├─ Interaction/BPI_Interactable       (Interact(Instigator: Pawn), GetInteractText → Text)
├─ Interaction/BP_WeaponPickup_Interact (부모: BP_ShooterPickup)
└─ Maps/Lvl_FPSTest                   (Lvl_ArenaShooter 복제, GameMode Override = BP_FPSGameMode)
```

템플릿에서 직접 수정한 에셋: `Variant_Shooter/Input/IMC_Weapons` (`IA_Interact` → `E` 매핑 추가)

### 상호작용 무기 픽업 동작 방식

1. `BP_FPSCharacter`: `IA_Interact`(Started) → `FirstPersonCamera` 위치에서 전방 250cm `Line Trace By Channel`(Visibility, Ignore Self)
2. 맞은 액터가 `BPI_Interactable`을 구현하면 `Interact (Message)` 호출 (Target = Hit Actor, Instigator = Self)
3. `BP_WeaponPickup_Interact`: `Event Interact` → Instigator에게 `BPI_Pickups`의 `Add Weapon (Message)` (Target Weapon = 부모 변수 `Weapon Class`, Target Context는 비워둠) → `Destroy Actor`
4. 자동 줍기 방지: 픽업의 `SphereCollision`을 Custom(Query Only, 전부 Ignore, **Visibility만 Block**)으로 변경 → Overlap은 안 일어나고 트레이스에만 맞음

### 템플릿 동작 메모

- `BP_ShooterPickup`은 `Weapon Type`(DataTable 행)의 메시·무기 클래스를 BeginPlay에서 `Load Asset`으로 비동기 로드함 → **에디터 뷰포트에서는 큐브로 보이고 플레이 시 총 메시로 바뀌는 것이 정상**
- 자식 BP에서 BeginPlay를 쓸 경우 반드시 `Parent: BeginPlay`를 호출할 것 (안 하면 메시/무기 로드가 안 됨)
- 무기 입력용 `IMC_Weapons`는 `BP_ShooterPlayerController`가 등록함
- 레벨 `Lvl_ArenaShooter` → GameMode `BP_ShooterGameMode` → Pawn `BP_ShooterCharacter` 구조

## 저장소

- 원격: GitHub `origin` (`https://github.com/LKH1134/Test.git`), 기본 브랜치 `main`
- `.uasset`/`.umap` 및 원본 아트·사운드는 **Git LFS**로 관리, 에셋은 lockable (`.gitattributes` 참고)
- 추적 제외: `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/`, `Content/Developers/`, `Content/Collections/`, `*.sln`/`*.slnx` (`.gitignore` 참고)
- `.vsconfig`는 필요한 VS 구성 요소 목록이므로 추적함

## 개발 환경

- Visual Studio Community 2026 + "C++를 사용한 게임 개발" 워크로드
- Windows SDK 10.0.26100.0 (UE 5.8 최소 요구 10.0.19041.0)
- MSVC 14.52: UE 5.8 권장 버전(14.50)보다 새로워서 엔진 헤더에서 `C4864` 경고가 나오지만 빌드에는 문제 없음. 이상한 컴파일 오류가 나면 MSVC 14.50 구성 요소 설치를 고려

## 빌드 방법

솔루션 파일 생성 (클론 직후 또는 `Source/` 파일 추가·삭제 후):

```powershell
& "C:\Program Files\Epic Games\UE_5.8\Engine\Binaries\DotNET\UnrealBuildTool\UnrealBuildTool.exe" -projectfiles -project="C:\Users\user\Documents\Unreal Projects\Test\Test.uproject" -game -rocket -progress
```

에디터 타깃 빌드:

```powershell
& "C:\Program Files\Epic Games\UE_5.8\Engine\Binaries\DotNET\UnrealBuildTool\UnrealBuildTool.exe" TestEditor Win64 Development -Project="C:\Users\user\Documents\Unreal Projects\Test\Test.uproject" -WaitMutex
```

또는 `Test.uproject` 우클릭 → Generate Visual Studio project files, `Test.sln`을 열어 `Development Editor` / `Win64`로 빌드.

### 주의 사항

- **에디터가 열려 있으면 외부 빌드가 실패함** (`Unable to build while Live Coding is active`). 에디터를 닫고 빌드하거나, 에디터 안에서 Live Coding(`Ctrl+Alt+F11`)을 사용
- 에디터를 닫은 직후에는 프로세스가 잠시 남아 있을 수 있으니 완전히 종료된 뒤 빌드
- Git Bash에서 `Build.bat`을 직접 호출하면 경로 공백(`C:\Program Files`) 때문에 실패함. PowerShell에서 `UnrealBuildTool.exe`를 직접 호출할 것
- 솔루션 생성 시 `Automation_Test.sln`도 함께 생기는데, UE 5.8의 자동화 테스트용 솔루션이므로 무시해도 됨
- `Saved/Crashes/`에 프로젝트 생성 직후 에디터 크래시 기록 1건(`EXCEPTION_ACCESS_VIOLATION`, `UnrealEditor_Core`)이 있음. 원인 미확인, 재현되면 조사 필요

## 작업 이력

| 날짜 | 커밋 | 내용 |
|---|---|---|
| 261116 | `0eb88d3` | 템플릿 초기 커밋, `.gitignore` / `.gitattributes`(LFS) 설정 |
| 261116 | `32f6cdd` | HANDOFF 추가 (커밋 컨벤션) |
| 261116 | `46f3b02` | C++ 모듈 `Test` 추가, `TestEditor` 빌드 확인 |
| 261116 | `144e18f` | HANDOFF 갱신 (환경, 빌드 방법, 이력) |
| 261116 | (이번 커밋) | 상호작용(E키) 무기 픽업 블루프린트, 테스트 맵 추가 |

## 다음 할 일

- 상호작용 줍기 플레이 테스트 마무리: 픽업 메시가 플레이 시 총으로 바뀌는 것까지 확인됨. E키로 주웠을 때 손에 들리고 탄약 HUD가 뜨는지 최종 확인
- 확인 후 `BP_FPSCharacter` 트레이스의 Draw Debug Type을 `None`으로 되돌리기
- 확장 아이디어: 바라볼 때 "E: 줍기" 안내 UI(`GetInteractText` 활용), 이미 가진 무기를 주우면 탄약 보충, 무기 버리기
- `DefaultGame.ini`의 `ProjectName`이 아직 "First Person BP Game Template"이므로 필요하면 변경
- 에디터 재시작 후 C++ 모듈 `Test` 로드 확인
