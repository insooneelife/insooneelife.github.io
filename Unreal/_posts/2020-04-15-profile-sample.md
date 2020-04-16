---
title: "[Unreal] Profile"
categories:
  - Profile
---

### Profile




프로파일링을 위한 그룹 선언
```c++
DECLARE_STATS_GROUP(TEXT("Sample"), STATGROUP_Sample, STATCAT_Advanced);
```

프로파일링을 위한 개체 선언
```c++
DECLARE_CYCLE_STAT(TEXT("Agent [Tick]"), STAT_AgentTick, STATGROUP_Sample);
```

프로파일링 대상 구간에 적용

```c++
void ACharacterAgent::Tick(float DeltaTime)
{
	// 현재 포함된 블록에 대한 프로파일링을 시작한다.
	// Actor가 여러번 생성된다면 모든 actor들의 Tick함수 내의 성능을 종합하여 프로파일링 한다.
	SCOPE_CYCLE_COUNTER(STAT_AgentTick)
	;

	Super::Tick(DeltaTime);
	UpdateAgent(DeltaTime);
}
```

Unreal 내부에서 프로파일링 과정을 보기 위해 먼저 ` 키를 눌러 커맨드창을 연다.
다음 커맨드를 입력한다. (Sample은 프로파일링 그룹을 정의할때 작성한 커맨드 이름이다.)
> stat Sample

다음과 같이 표시된다.
[![foo](https://github.com/insooneelife/insooneelife.github.io/blob/master/assets/images/unreal-profile-sample1.png)](https://flic.kr/p/dNiUYB)

주의할 점은 프로파일링 과정을 화면에 ui로 표시해주는 것 자체가 또 어느정도의 성능에 영향을 미치기 때문에
> stat Engine
을 통해 프로파일링 ui로 인해 성능 얼마나 변하는지 미리 확인해둘 필요가 있다.
