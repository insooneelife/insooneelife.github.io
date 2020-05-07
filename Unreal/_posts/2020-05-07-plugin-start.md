---
title: "[Unreal] Plugin Start"
categories:
  - Plugin
---


### Plugin start
Unreal plugin 시작방법 예제이다.

1. 메뉴바 -> 편집 -> 플러그인

2. 오른쪽 하단 새 플러그인 클릭
![image-center](/assets/images/unreal-plugin-start-newplugin.png){: .align-center}

3. 에디터 툴바를 사용하는 "에디터 툴바 버튼"으로 플러그인 생성 (MyPlugin)

4. 프로젝트 재시작 시 에디터 툴바에 MyPlugin을 확인할 수 있음
![image-center](/assets/images/unreal-plugin-start-editor-toolbar.png){: .align-center}


5. 플러그인 소스는 visual studio 프로젝트에서 수정할 수 있음

![image-center](/assets/images/unreal-plugin-start-visual-plugin-source.png){: .align-center}



이후 플러그인 ui를 꾸미고 싶은 경우 OnSpawnPluginTab 함수에서 slate를 수정하면 된다.
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


