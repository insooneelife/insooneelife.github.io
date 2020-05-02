


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

// call this from start - ex)  gamemode being
void Utility::InitConsoleCommand()
{
	FConsoleCommandWithArgsDelegate CurveByPathDelegate;

	CurveByPathDelegate.BindStatic(&Utility::ConsoleCommand);

	IConsoleManager::Get().RegisterConsoleCommand(
		TEXT("insooneelifeCmd"),
		TEXT("insooneelife test cmd"), CurveByPathDelegate);
}

// this is static function
void Utility::ConsoleCommand(const TArray<FString>& Args)
{
	UE_LOG(LogTemp, Warning, TEXT("#################"));
}
```

#### 실행 결과
![image-center](/assets/images/unreal-console-command.png){: .align-center}
