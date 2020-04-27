


---
title: "[Unreal] Console Command"
categories:
  - Console Command
---

### Console Command
Unreal engine 콘솔 커맨드 관련 예제이다.


```c++
#include "HAL/IConsoleManager.h"

...

void Utility::InitConsoleCommand()
{
	FConsoleCommandWithArgsDelegate CurveByPathDelegate;

	CurveByPathDelegate.BindStatic(&Utility::ConsoleCommand);

	IConsoleManager::Get().RegisterConsoleCommand(
		TEXT("insooneelifeCmd"),
		TEXT("insooneelife test cmd"), CurveByPathDelegate);
}

void Utility::ConsoleCommand(const TArray<FString>& Args)
{
	UE_LOG(LogTemp, Warning, TEXT("#################"));
}
```

#### 실행 결과
