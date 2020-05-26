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