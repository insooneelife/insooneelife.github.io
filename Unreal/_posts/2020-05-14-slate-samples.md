---
title: "[Unreal] Slate Samples"
categories:
  - Slate
---


### State
Unreal ui framework인 slate에 대한 예제이다.


#### SButton
```c++
#include "Widgets/Input/SButton.h"

TSharedPtr<SWidget> FMyPluginModule::Create()
{
	return SNew(SButton)
		.Text(WidgetText)
		.OnClicked(FOnClicked::CreateRaw(this, &FMyPluginModule::OnClickButton));
}


FReply FMyPluginModule::OnClickButton()
{
	UE_LOG(LogTemp, Warning, TEXT("OnClickButton"));

	return FReply::Handled();
}


```

#### SAssetSearchBox
```c++
// "EditorWidgets" Module
#include "SAssetSearchBox.h"

TSharedPtr<SWidget> FMyPluginModule::Create()
{
	return SNew(SAssetSearchBox)
    	.OnTextChanged(FOnTextChanged::CreateRaw(this, &FMyPluginModule::OnSearchBoxChanged))
    	.OnTextCommitted(FOnTextCommitted::CreateRaw(this, &FMyPluginModule::OnSearchBoxCommitted))
    	.DelayChangeNotificationsWhileTyping(true)
    	.HintText(LOCTEXT("SearchHint", "Search..."));
}


// searchbox 내부 텍스트 변경 시 호출
void FMyPluginModule::OnSearchBoxChanged(const FText& InSearchText)
{
	UE_LOG(LogTemp, Warning, TEXT("OnSearchBoxChanged  %s"), *InSearchText.ToString());
}

// searchbox 내부 텍스트 커밋 시 호출
void FMyPluginModule::OnSearchBoxCommitted(const FText& InSearchText, ETextCommit::Type CommitInfo)
{
	UE_LOG(LogTemp, Warning, TEXT("OnSearchBoxCommitted  %s"), *InSearchText.ToString());
}
```

#### AssetSelector
```c++
// "AssetManagerEditor" Module
#include "AssetManagerEditorModule.h"


TSharedPtr<SWidget> FMyPluginModule::Create()
{
	return IAssetManagerEditorModule::MakePrimaryAssetTypeSelector(
		FOnGetPrimaryAssetDisplayText::CreateRaw(this, &FMyPluginModule::GetDisplayText),
		FOnSetPrimaryAssetType::CreateRaw(this, &FMyPluginModule::OnTypeSelected));
}


FText FMyPluginModule::GetDisplayText() const
{
	return FText::FromString(FString(TEXT("ABCDEFG!!")));
}

void FMyPluginModule::OnIdSelected(FPrimaryAssetId AssetId)
{
	UE_LOG(LogTemp, Warning, TEXT("OnIdSelected !!!!!!!!!!!!!!!!!!!!!!!   %s"), *AssetId.ToString());
}

```

#### Notification Popup
```c++
void FMyPluginModule::OnConsoleCommand2()
{
	const FText NotificationErrorText = LOCTEXT("MyNotification", "write some text here.");
		
	FNotificationInfo Info(NotificationErrorText);
	Info.ExpireDuration = 5.0f;
	FSlateNotificationManager::Get().AddNotification(Info);
}
```


#### Message Dialog
```c++
...
	FMessageDialog::Open(EAppMsgType::Ok, FText::FromString(TEXT("message here!")));
...
```

