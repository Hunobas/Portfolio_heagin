# 박태훈 | 유니티 게임 클라이언트 프로그래머

> 1998년생, 군필 (의경 만기제대)  
> 📧 hunobas.dev@gmail.com

> Monster Hunter: World를 500시간 이상 플레이하며 헌팅 액션의 손맛이 무엇인지 몸으로 익혔습니다.  
> 그 경험을 개발자로써 구현하고 싶어 지원합니다.

---

## 핵심 역량

### Unity 최적화

**1인칭 퍼즐 게임 목성의 노래** 프로젝트에서 400만 버텍스 + 300개 머터리얼 우주 정거장 씬이 30~60 FPS로 불안정했습니다. MeshBaker 텍스처 아틀라스 + 콤바인 메쉬, 오클루전 컬링을 적용해 **배칭 2,650 → 601개, FPS 120+ 안정화**를 달성했습니다.

<img width="1548" height="591" alt="최적화 전후" src="https://github.com/user-attachments/assets/6b35a453-6a45-4258-9635-3bcff6062e97" />

| 지표 | Before | After |
|------|--------|-------|
| Batches | 2,650 | 601 |
| FPS | 30~60 | 120+ |

- 📂 [MeshBaker 에디터 확장 코드](https://github.com/Hunobas/Song-Of-Jupitor/blob/main/Scripts/Editor/MB3_ApplyCombinedMaterialToSourceObjects.cs)
- 📜 [최적화 개발일지](https://velog.io/@po127992/목성의-노래-MeshBaker-최적화-삽질기-텍스처-아틀라스만-vs-콤바인-메쉬까지)

<details>
<summary><b>추가: ASCII 이미지 UGUI 렌더러 성능 최적화</b></summary>

160×90 그리드 × 4×4 슈퍼샘플 = 230,400회 픽셀 접근으로 CPU 점유 27.6ms(프레임 비중 70.4%)를 차지하던 문제를, 비동기 Readback + 색상 변경 구간 태그 최적화 + 다운샘플링으로 해결했습니다.

| 지표 | Before | After |
|------|--------|-------|
| CPU 시간 | 27.6ms | 2.15ms |
| 프레임 비중 | 70.4% | 3.5% |

</details>

---

### 애니메이션 디버깅 & 물리 기반 액션 구현

**My Little Puppy (드림모션 인턴)** 에서 루트모션 및 FSM 관련 버그 3건을 해결했습니다. 공통적으로 외부 모듈이 캐릭터 트랜스폼 업데이트를 오염시키는 패턴이었으며, 이를 바탕으로 코드 컨벤션 개선을 제안했습니다.

이외에도 슈퍼점프 콘텐츠 구현 시 **차징 시간 기반 높이 보간 + 런타임 커브 기반 속도 제어**를 적용해 물리적으로 자연스러운 포물선 움직임을 구현했습니다.

<details>
<summary><b>사례 1: 루트모션 회전이 프레임 컨디션에 따라 달라지는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![ezgif com-video-to-gif-converter (4)](https://github.com/user-attachments/assets/c038982c-4e66-4c04-a3c1-a4878d3d48c5) | ![ezgif com-video-to-gif-converter (5)](https://github.com/user-attachments/assets/f0d4974a-a54d-4b7d-870c-e7b6ad794d5c) |

**증상:** 게임 FPS가 낮으면 정상 회전, 정상 FPS에서는 회전이 중간에 멈춤

**원인 추적:**
1. 소수점 오차 가설 → 검증 결과 0.1도 미만으로 육안 차이와 무관
2. `ActorPosDir` 내부에 목표 각도로 선형 보간하는 로직 발견
3. 루트모션 각도와 `ActorPosDir` 보간이 중복 적용되고 있었음

**해결:** 루트모션 전용 위치/각도 업데이트 메서드를 분리하여 외부 보간 로직 제외

</details>

<details>
<summary><b>사례 2: 루트모션 종료 시 앞으로 튀어나가는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![02_-ezgif com-video-to-gif-converter (3)](https://github.com/user-attachments/assets/4e19abda-d7a0-41ea-a84e-c6ffce35ca67) | ![02_-ezgif com-video-to-gif-converter (4)](https://github.com/user-attachments/assets/53eb6482-83c2-4f4d-bbe5-2e215c2b7f15) |

**원인:** Walk → RootMotion → Idle 전환 시, Walk 상태의 `WalkToIdle` 속도값이 초기화되지 않고 Idle까지 전달됨

**해결:** 루트모션 상태머신 종료 시점에 Idle 블렌딩 속도를 0으로 초기화

</details>

<details>
<summary><b>사례 3: 컷씬 일시정지 시 NPC 위치가 튀는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![PlayableDirector_-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/436f5f08-5091-4b73-bb91-3871885ca447) | ![ezgif com-video-to-gif-converter (6)](https://github.com/user-attachments/assets/9bb6594b-afca-43f8-b29a-31fe61e320e3) |

**원인:** PlayableDirector가 액터 위치를 직접 수정하는데, 컷씬 모드는 `LateUpdate`에서 위치를 재조정하지만 일시정지 모드는 이 처리가 없었음

**해결:** 일시정지 모드에서도 이전 모드가 컷씬이면 `HandleGameActors()` 호출

</details>

---

### FSM 아키텍처 설계

**문제:** 5가지 플레이 모드(Normal/Panel/Cinema/Dialog/Pause)가 상호배타적이어야 하는데, 패널을 여는 도중 컷씬이 재생되면 컷씬이 끝나도 **조작 불가 상태**가 되는 치명적 버그 발생. 상태 관련 코드가 여러 파일에 분산되어 디버깅 평균 **60분 소요**.

![GameState 버그 영상](https://github.com/user-attachments/assets/fa973d2f-df58-483d-ae3b-05d5104e9bc6)

*패널 모드 진입 중 시네마 모드가 끼어들면 발생하는 조작 불가 문제*

**해결:** 중앙 집중식 FSM으로 모든 플레이 모드를 단일 책임 관리.
```csharp
public void ChangePlayMode(IPlayMode next)
{
    if (next == null || ReferenceEquals(_activeMode, next)) return;

    // 시네마 모드는 일시정지 이외 모든 전환 요청 무시
    if (IsPlayingCinema && !ReferenceEquals(next, PauseMode)) return;

    // 패널 모드 강제 종료 후 전환
    if (IsOperatingPanel && PanelMode.Controller != null)
        PanelMode.Controller.EndPanelForcely();

    var prev = _activeMode;
    prev?.OnExit(next);
    _activeMode = next;
    _activeMode.OnEnter(prev);
    InputManager.Instance?.UpdateCursorLock();
}
```

각 모드의 `OnExit` 훅이 UI 상태·입력 바인딩·커서 잠금을 자동으로 정리하므로, 호출부에서 정리 로직을 신경 쓸 필요 없습니다.

| 개선 항목 | Before | After |
|---------|--------|-------|
| 상태 충돌 버그 | 주 2~3건 | **0건** |
| 디버깅 소요 시간 | 평균 60분 | 평균 30분 |
| 신규 모드 추가 | - | `IPlayMode` 구현만으로 20분 이내 |

📂 [GameState.cs 전체 코드](https://github.com/Hunobas/Song-Of-Jupitor/blob/7386ab978fc3115a13a700758c7a618567bc168a/Scripts/System/GameState.cs#L15)

<details>
<summary><b>엣지 케이스 처리: 일시정지 해제 시 이전 모드 복구 / 시네마 중 다이얼로그 전환 방지</b></summary>

- `PauseMode`가 `prevMode`를 저장하고 자체 `Resume()` 메서드로 복구 — [코드 보기](https://github.com/Hunobas/Song-Of-Jupitor/blob/7386ab978fc3115a13a700758c7a618567bc168a/Scripts/System/PauseMode.cs#L34)
- 시네마 모드는 `TimelineController`에서 자체적으로 종료, `ChangePlayMode`의 가드 조건이 외부 중단 차단 — [코드 보기](https://github.com/Hunobas/Song-Of-Jupitor/blob/7386ab978fc3115a13a700758c7a618567bc168a/Scripts/System/CinemaMode.cs#L27)

</details>


---

## 프로젝트 요약

| 프로젝트 | 엔진 | 기간 | 규모 | 역할 |
|----------|------|------|------|------|
| [목성의 노래](https://github.com/Hunobas/Song-Of-Jupitor) | Unity | 2025.06~2026.01 | 5명 | FSM, 렌더링 최적화, 셰이더, 퍼즐 로직 |
| [My Little Puppy](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc) | Unity | 2025.01~03 | 38명 | 슈퍼점프 물리, 애니메이션 버그 해결, 에디터 확장 |
| [TOGU: Planet Survivors](https://github.com/Hunobas/Planet) | Unreal 5.4 | 2025.04~06 | 1명 | 전체 아키텍처 설계 및 구현 |

---

## Contact

- 📱 010-3702-1279
- 📧 hunobas.dev@gmail.com
- 📝 [Velog](https://velog.io/@po127992/posts)
- 💻 [GitHub](https://github.com/hunobas)
