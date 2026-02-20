# 박태훈 | 유니티 게임 클라이언트 프로그래머

> 1998년생, 군필 (의경 만기제대)  
> 📧 hunobas.dev@gmail.com

> Monster Hunter: World를 500시간 이상 플레이하며 헌팅 액션의 손맛이 무엇인지 몸으로 익혔습니다.  
> 그 경험을 코드로 구현하고 싶어 지원합니다.

---

## 해긴 지원동기

<Monster Hunter: World HR 259, Monster Hunter: Wilds 52h>

몬스터 헌터 시리즈를 장시간 플레이하며 느낀 건, '손맛'은 단순히 타격 이펙트가 아니라 **모션, 충돌 판정, 사운드, 카메라가 0.1초 안에 맞아떨어지는 정밀한 조율**이라는 점이었습니다.

와치독스 헌팅 액션이라는 컨셉과, 무기 기반 전투 연출을 목표로 한다는 공고를 읽고 제가 플레이어로서 느꼈던 바를 개발자 관점으로 구현할 수 있는 팀이라 생각했습니다.

---

## 핵심 역량

### Unity 최적화

**1인칭 퍼즐 게임 목성의 노래** 프로젝트에서 400만 버텍스 + 300개 머터리얼 우주 정거장 씬이 30~60 FPS로 불안정했습니다. MeshBaker 텍스처 아틀라스 + 콤바인 메쉬, 오클루전 컬링을 적용해 **배칭 2,650 → 601개, FPS 120+ 안정화**를 달성했습니다.

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

### 셰이더 구현

- [모션 벡터 없는 카메라 모션블러 셰이더 구현](https://github.com/Hunobas/Song-Of-Jupitor/tree/main#6%EF%B8%8F%E2%83%A3-%EB%AA%A8%EC%85%98%EB%B2%A1%ED%84%B0-%EC%97%86%EB%8A%94-%EC%B9%B4%EB%A9%94%EB%9D%BC-%EB%AA%A8%EC%85%98%EB%B8%94%EB%9F%AC-%EC%85%B0%EC%9D%B4%EB%8D%94-%EA%B5%AC%ED%98%84)
- C++ 레이 트레이싱 직접 구현 [[Github]](https://github.com/Hunobas/RayTracingClass_OneWeek)

---

### 애니메이션 & 물리 기반 전투 구현

**My Little Puppy (드림모션 인턴)** 에서 루트모션 및 FSM 관련 버그 3건을 해결했습니다. 공통적으로 외부 모듈이 캐릭터 트랜스폼 업데이트를 오염시키는 패턴이었으며, 이를 바탕으로 코드 컨벤션 개선을 제안했습니다.

슈퍼점프 콘텐츠 구현 시 **차징 시간 기반 높이 보간 + 런타임 커브 기반 속도 제어**를 적용해 물리적으로 자연스러운 포물선 움직임을 구현했습니다.

<details>
<summary><b>루트모션 버그 사례 (3건)</b></summary>

**사례 1 — 루트모션 회전이 프레임 컨디션에 따라 달라지는 문제**  
루트모션 각도와 `ActorPosDir` 보간이 중복 적용되고 있었음. 루트모션 전용 위치/각도 업데이트 메서드를 분리하여 해결.

**사례 2 — 루트모션 종료 시 앞으로 튀어나가는 문제**  
Walk → RootMotion → Idle 전환 시, Walk 상태의 속도값이 초기화되지 않고 Idle까지 전달됨. 루트모션 상태머신 종료 시점에 블렌딩 속도를 0으로 초기화하여 해결.

**사례 3 — 컷씬 일시정지 시 NPC 위치가 튀는 문제**  
PlayableDirector가 액터 위치를 직접 수정하는데, 일시정지 모드에서 LateUpdate 재조정이 누락됨. 이전 모드가 컷씬이면 `HandleGameActors()` 호출하도록 처리.

</details>

---

### FSM 아키텍처 설계

**목성의 노래** 에서 플레이 모드 전환(탐색 / 퍼즐 / 컷씬 / 일시정지)을 FSM으로 설계했습니다. 상태 간 진입/이탈 조건을 명확히 분리해 컷씬 일시정지 버그 등을 구조적으로 해결했습니다.

---

## 팀 프로젝트 경험

**My Little Puppy (드림모션, 3개월 인턴)**  
38인 팀에서 스팀 데모 빌드를 11개 국어로 출시했습니다. QA 테스트 15회 진행, 버그/폴리싱 103건 제보, 주요 버그 11건 해결했습니다. 기획자용 작업 가이드 문서 2건을 작성해 협업 효율을 높였습니다.

- [신규 무기/아이템 추가 가이드](https://ethereal-judo-1f1.notion.site/223486e2cdb980c5a807f920ebad70a6)
- [신규 몬스터 추가 가이드](https://ethereal-judo-1f1.notion.site/223486e2cdb98001869cef28bb9bfbb5)

---

## 프로젝트 요약

| 프로젝트 | 엔진 | 기간 | 규모 | 역할 |
|----------|------|------|------|------|
| [목성의 노래](https://github.com/Hunobas/Song-Of-Jupitor) | Unity | 2025.06~2026.01 | 5명 | FSM, 렌더링 최적화, 셰이더, 퍼즐 로직 |
| [My Little Puppy](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc) | Unity | 2025.01~03 | 38명 | 슈퍼점프 물리, 애니메이션 버그 해결, 에디터 확장 |
| [TOGU: Planet Survivors](https://github.com/Hunobas/Planet) | Unreal 5.4 | 2025.04~06 | 1명 | 전체 아키텍처 설계 및 구현 |

---

## 플레이 이력

| 게임 | 플레이타임 | 비고 |
|------|------------|------|
| Monster Hunter: World | 564h | HR 259 |
| Monster Hunter: Wilds | 52h | |
| God of War 시리즈 | 70h+ | |
| Elden Ring | 40h+ | |
| Bloodborne | 40h+ | |

---

## Contact

- 📱 010-3702-1279
- 📧 hunobas.dev@gmail.com
- 📝 [Velog](https://velog.io/@po127992/posts)
- 💻 [GitHub](https://github.com/hunobas)
