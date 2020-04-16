---
title: "[Unreal] Profile"
categories:
  - Profile
---

### Profile

[![foo](https://github.com/insooneelife/insooneelife.github.io/blob/master/assets/images/unreal-profile-sample1.png)](https://flic.kr/p/dNiUYB)


프로파일링을 위한 그룹 선언
```c++
DECLARE_STATS_GROUP(TEXT("Sample"), STATGROUP_Sample, STATCAT_Advanced);
```

프로파일링을 위한 개체 선언
```c++
DECLARE_CYCLE_STAT(TEXT("Agent [Tick]"), STAT_AgentTick, STATGROUP_Sample);
```

프로파일링 구간 설정

```c++
void ACharacterAgent::Tick(float DeltaTime)
{
	SCOPE_CYCLE_COUNTER(STAT_AgentTick)
	;

	Super::Tick(DeltaTime);
	UpdateAgent(DeltaTime);
}
```
