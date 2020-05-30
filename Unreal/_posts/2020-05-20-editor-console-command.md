---
title: "[Unreal] Editor Console Command"
categories:
  - Editor
  - Console Command
---

#### header
```c++
//...
class FMyPluginModule : public IModuleInterface
{
//...
	virtual void StartupModule() override;
	void OnConsoleCommand();
//...
};
```

#### source
```c++
void FMyPluginModule::StartupModule()
{
	//...
  
	IConsoleManager::Get().RegisterConsoleCommand(
		TEXT("MyCommand"),
		TEXT("My Example Command"),
		FConsoleCommandDelegate::CreateRaw(this, &FMyPluginModule::OnConsoleCommand),
		ECVF_Default
	);
}
  
void FMyPluginModule::OnConsoleCommand()
{
	UE_LOG(LogTemp, Warning, TEXT("##### OnConsoleCommand #####"));
}
//...
```

#### notification
다음 방법으로 notification info popup을 띄울 수 있다.
우측 하단에 작게 뜨는 팝업이다.


```c++
void FMyPluginModule::OnConsoleCommand2()
{
	const FText NotificationErrorText = LOCTEXT("MyNotification", "write some text here.");
		
	FNotificationInfo Info(NotificationErrorText);
	Info.ExpireDuration = 5.0f;
	FSlateNotificationManager::Get().AddNotification(Info);
}
```
