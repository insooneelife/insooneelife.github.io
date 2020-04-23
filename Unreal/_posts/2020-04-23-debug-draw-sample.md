
---
title: "[Unreal] Debug Draw"
categories:
  - Debug Draw
---

### Debug Draw
Unreal engine 개발 과정에서 화면상에 원하는 도형을 그려서,
디버깅을 수월하게 도와주는 함수들에 대한 예제이다.


```c++
#include "DrawDebugHelpers.h"

...

void Examples::DebugDrawExample(UWorld* World)
{
	FVector StartOffset = FVector(0, 0, 200);
	FVector LocationOne = FVector(0, 0, 600) + StartOffset;
	FVector LocationTwo = FVector(0, -600, 600) + StartOffset;
	FVector LocationThree = FVector(0, 600, 600) + StartOffset;
	FVector LocationFour = FVector(-300, 0, 600) + StartOffset;
	FVector LocationFive = FVector(-400, -600, 600) + StartOffset;

	DrawDebugPoint(World, LocationOne, 200, FColor(52, 220, 239), true);

	DrawDebugSphere(World, LocationTwo, 200, 26, FColor(181, 0, 0), true, -1, 0, 2);

	DrawDebugCircle(World, LocationFour, 200, 50, FColor(0, 0, 0), true, -1, 0, 10);

	DrawDebugBox(World, LocationFive, FVector(100, 100, 100), FColor::Purple, true, -1, 0, 10);

	DrawDebugLine(World, LocationTwo, LocationThree, FColor::Emerald, true, -1, 0, 10);

	DrawDebugDirectionalArrow(World, FVector(-300, 600, 600), FVector(-300, -600, 600), 120.f, FColor::Magenta, true, -1.f, 0, 5.f);

	DrawDebugCrosshairs(World, FVector(0, 0, 1000), FRotator(0, 0, 0), 500.f, FColor::White, true, -1.f, 0);
}
```

#### 실행 결과
다음과 같이 화면에 도형들이 출력된다.
![image-center](/assets/images/unreal-create-asset-to-editor-result.png){: .align-center}
