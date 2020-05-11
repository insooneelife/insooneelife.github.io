---
title: "[Unreal] Plugin Start"
categories:
  - Plugin
---


### Plugin start
Unreal plugin 시작방법 예제이다.

1. 메뉴바 -> 편집 -> 플러그인

2. 오른쪽 하단 새 플러그인 클릭
![image-center](/assets/images/unreal-plugin-start-newplugin.png){: .align-left}

3. 에디터 툴바를 사용하는 "에디터 툴바 버튼"으로 플러그인 생성 (MyPlugin)

4. 프로젝트 재시작 시 에디터 툴바에 MyPlugin을 확인할 수 있음
![image-center](/assets/images/unreal-plugin-start-editor-toolbar.png){: .align-left}


5. 플러그인 소스는 visual studio 프로젝트에서 수정할 수 있음

![image-center](/assets/images/unreal-plugin-start-visual-plugin-source.png){: .align-left}








### Plugin ui
이후 에디터 tool bar의 플러그인 버튼을 누르면 생성되는 플러그인 ui를 꾸미고 싶은 경우,
OnSpawnPluginTab 함수에서 slate를 수정하면 된다.

```c++
...
#include "Widgets/Input/SButton.h"
...

TSharedRef<SDockTab> FMyPluginModule::OnSpawnPluginTab(const FSpawnTabArgs& SpawnTabArgs)
{
	FText WidgetText = FText::Format(
		LOCTEXT("WindowWidgetText", "Add code to {0} in {1} to override this window's contents"),
		FText::FromString(TEXT("FMyPluginModule::OnSpawnPluginTab")),
		FText::FromString(TEXT("MyPlugin.cpp"))
		);
	
	return SNew(SDockTab)
		.TabRole(ETabRole::NomadTab)
		[
			// Put your tab content here!
			SNew(SBox)
			.HAlign(HAlign_Center)
			.VAlign(VAlign_Center)
			[
				SNew(SButton)
				.Text(WidgetText)
				// CreateRaw로 하지 않으면 컴파일러가 인식하지 못하는 문제가 있음
				.OnClicked(FOnClicked::CreateRaw(this, &FMyPluginModule::OnClickButton))
			]
		];
		
}

...

FReply FMyPluginModule::OnClickButton()
{
	UE_LOG(LogTemp, Warning, TEXT("###############################"));

	return FReply::Handled();
}

```
플러그인 코드는 수정 후 에디터에서 컴파일을 하더라도 바로 적용되지 않는다는 문제점이 있다.
에디터를 끄고 다시 실행시키면 플러그인 모듈 소스가 적용된다.

#### result
에디터 툴바 플러그인 버튼 클릭 시 다음과 같이 ui 창이 생성된다.
![image-center](/assets/images/unreal-plugin-start-pluginui.png){: .align-left}




### slate ui 추가 
```c++
...
#include "Widgets/Layout/SUniformGridPanel.h"

...

TSharedRef<SDockTab> FMyPluginModule::OnSpawnPluginTab(const FSpawnTabArgs& SpawnTabArgs)
{
	FText WidgetText = FText::Format(
		LOCTEXT("#####WindowWidgetText", "Add code to {0} in {1} to override this window's contents"),
		FText::FromString(TEXT("FMyPluginModule::OnSpawnPluginTab")),
		FText::FromString(TEXT("MyPlugin.cpp"))
		);
	
	FText SearchBoxText = FText::FromString(FString(TEXT("SearchBox")));


	auto t = SNew(STextBlock)
		.Text(FText::FromString(FString(TEXT("FileDirectory"))));
	

	return SNew(SDockTab)
		.TabRole(ETabRole::NomadTab)
		[
			SNew(SUniformGridPanel)
			.SlotPadding(FMargin(5.0f))
			+ SUniformGridPanel::Slot(0, 0)
			.HAlign(HAlign_Right)
			[
				SNew(SUniformGridPanel)
				.SlotPadding(FMargin(5.0f))
				+ SUniformGridPanel::Slot(0, 0)
				.HAlign(HAlign_Center)
				[
					SNew(SButton)
					.Text(FText::FromString(FString(TEXT("select file .wav"))))
					.OnClicked(FOnClicked::CreateRaw(this, &FMyPluginModule::OnClickButton))
				]
				+ SUniformGridPanel::Slot(0, 1)
				.HAlign(HAlign_Center)
				[
					SNew(STextBlock)
					.Text(FText::FromString(FString(TEXT("FileDirectory"))))
				]
			]
			+ SUniformGridPanel::Slot(0, 1)
			.HAlign(HAlign_Right)
			[
				SNew(SUniformGridPanel)
				.SlotPadding(FMargin(5.0f))
				+ SUniformGridPanel::Slot(0, 0)
				.HAlign(HAlign_Center)
				[
					SNew(SButton)
					.Text(FText::FromString(FString(TEXT("select file .script"))))
					.OnClicked(FOnClicked::CreateRaw(this, &FMyPluginModule::OnClickButton))
				]
				+ SUniformGridPanel::Slot(0, 1)
				.HAlign(HAlign_Center)
				[
					SNew(STextBlock)
					.Text(FText::FromString(FString(TEXT("FileDirectory"))))
				]
			]
			+ SUniformGridPanel::Slot(0, 2)
			.HAlign(HAlign_Center)
			[
				SNew(SBox)
				.HAlign(HAlign_Center)
				.VAlign(VAlign_Center)
				[
					SNew(SButton)
					.Text(WidgetText)
					.OnClicked(FOnClicked::CreateRaw(this, &FMyPluginModule::OnClickButton))
				]
			]
			+ SUniformGridPanel::Slot(1, 0)
			[
				SNew(SBox)
				.HAlign(HAlign_Center)
				.VAlign(VAlign_Center)
				[
					SNew(SButton)
					.Text(WidgetText)
					.OnClicked(FOnClicked::CreateRaw(this, &FMyPluginModule::OnClickButton))
				]
			]
			+ SUniformGridPanel::Slot(1, 1)
			[
				SNew(SBox)
				.HAlign(HAlign_Center)
				.VAlign(VAlign_Center)
				[
					SNew(SSearchBox)
				]
			]
		];		
}
```

#### result
사진


### 파일 탐색기 창 띄우기 (File Dialog)
탐색기 창을 띄우기 위해서 DesktopPlatformModule을 사용해야 하는데,
이는 언리얼 내 다른 모듈의 헤더이기 때문에 사용하기 위해 따로 모듈을 활성화 시켜주어야 한다.

창 -> 개발자 툴 -> 모듈
모듈 창이 켜지면 DesktopPlatform을 검색하고 로드해준다.

모듈 활성화 후 에디터를 다시 켜주어야 한다.

```c++
...

#include "DesktopPlatformModule.h"
#include "IDesktopPlatform.h"

...

FReply FMyPluginModule::OnClickButton()
{
	UE_LOG(LogTemp, Warning, TEXT("###############################"));

	auto DesktopPlatform = FDesktopPlatformModule::Get();
	if (DesktopPlatform == nullptr)
	{
		UE_LOG(LogTemp, Warning, TEXT("DesktopPlatform == nullptr"));
	}

	auto FileNames = TArray<FString>();

	DesktopPlatform->OpenFileDialog(
		nullptr,
		TEXT("Open SoFace Weight Curve"),
		TEXT(""),
		TEXT(""),
		TEXT("Text (*.txt)|*.txt|Data (*.wav; *.script)|*.wav; *.script"),
		EFileDialogFlags::Type::Multiple,
		FileNames
	);

	return FReply::Handled();
}
```


