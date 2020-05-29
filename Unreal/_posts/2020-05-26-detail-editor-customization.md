---
title: "[Unreal] Detail Editor Customization"
categories:
  - Editor
  - Detail
  - Customization
---


#### BaseEditorTool
디테일 패널의 타겟 오브젝트로 일반적인 UObject를 상속한 클래스이다.
디테일 패널에서 수정하고 싶은 프로퍼티들을 정의해주면 된다.

```c++
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "BaseEditorTool.generated.h"

UCLASS()
class MYPLUGIN_API UBaseEditorTool : public UObject
{
	GENERATED_BODY()
	
public:
	UPROPERTY(EditAnywhere, Category = "Settings")
	FPlane MirrorPlane;
};
```


#### BaseEditorToolCustomization
디테일 패널을 실질적으로 꾸며주는 부분이다.
CustomizeDetails 함수가 호출에서 디테일 패널을 꾸며주는 코드를 작성하면 된다.
CustomizeDetails는 디테일 패널이 생성되거나 수정사항이 생길 때 호출된다.

#### header
```c++
#pragma once

#include "CoreMinimal.h"
#include "IDetailCustomization.h"
#include "SlateBasics.h"

class MYPLUGIN_API BaseEditorToolCustomization : public IDetailCustomization
{
public:

	virtual void CustomizeDetails(IDetailLayoutBuilder& DetailBuilder) override;

	static TSharedRef<IDetailCustomization> MakeInsance();
};
```

#### source
```c++
#include "BaseEditorToolCustomization.h"
#include "PropertyEditing.h"

TSharedRef<IDetailCustomization> BaseEditorToolCustomization::MakeInsance()
{
	return MakeShareable(new BaseEditorToolCustomization);
}


void BaseEditorToolCustomization::CustomizeDetails(IDetailLayoutBuilder& DetailBuilder)
{
  UE_LOG(LogTemp, Warning, TEXT("######################CustomizeDetails#####################"));
}
```


#### source

다음 코드를 통해 BaseEditorToolCustomization을 커스터마이즈 클래스로 등록시킨다.
이 코드 이후로 BaseEditorToolCustomization 클래스는 이벤트를 받을 수 있게 된다.
StartupModule과 같은 함수에서 호출해주면 된다.

```c++
FPropertyEditorModule& PropertyModule = FModuleManager::LoadModuleChecked<FPropertyEditorModule>("PropertyEditor");
PropertyModule.RegisterCustomClassLayout(
  "BaseEditorTool", FOnGetDetailCustomizationInstance::CreateStatic(&BaseEditorToolCustomization::MakeInsance));

PropertyModule.NotifyCustomizationModuleChanged();
```

다음 코드를 호출하여 detail panel을 생성한다.
Ex) 에디터 모듈 버튼 클릭 이벤트에서 호출
```c++
UBaseEditorTool* ToolInstance = NewObject<UBaseEditorTool>(GetTransientPackage(), ToolClass);
ToolInstance->AddToRoot();

FPropertyEditorModule& PropertyModule = FModuleManager::LoadModuleChecked<FPropertyEditorModule>("PropertyEditor");
	
TArray<UObject*> ObjectToView;
ObjectToView.Add(ToolInstance);
TSharedRef<SWindow> Window = PropertyModule.CreateFloatingDetailsView(ObjectToView, true);
```

#### result
![image-center](/assets/images/unreal-detail-editor-customize.png){: .align-left}




#### customized class
다음과 같이 윈도우 내에 detail view만 넣은 형태로 새로운 SWindow class 형태로 제작할수도 있다.

#### header
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "Input/Reply.h"
#include "Widgets/SWindow.h"

class UBaseEditorTool;
class MYPLUGIN_API SMyDetailViewWindow : public SWindow
{
public:

	SLATE_BEGIN_ARGS(SMyDetailViewWindow)
	{
	}
	SLATE_ARGUMENT(UClass*, ToolClass)
	SLATE_ARGUMENT(FOnClickedOutside, OnClose)
	SLATE_END_ARGS()

	void Construct(const FArguments& InArgs);
private:

	FReply OnClick(EAppReturnType::Type ButtonID);
	void OnDetailViewWindowClosed(const TSharedRef<SWindow>& Window);

private:
	UBaseEditorTool* ToolInstance;
	EAppReturnType::Type UserResponse;

	FOnClickedOutside OnCloseDelegate;
};

```

#### source
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#include "SMyDetailViewWindow.h"
#include "BaseEditorTool.h"
#include "PropertyEditing.h"
#include "Modules/ModuleManager.h"
#include "Interfaces/IMainFrameModule.h"
#include "Framework/Application/SlateApplication.h"
#include "SlateBasics.h"
#include "SlateExtras.h"
#include "Editor/EditorEngine.h"

#define LOCTEXT_NAMESPACE "MyDetailViewWindow"

void SMyDetailViewWindow::Construct(const FArguments& InArgs)
{
	UClass* InToolClass = InArgs._ToolClass;
	OnCloseDelegate = InArgs._OnClose;

	// create detail view
	FDetailsViewArgs Args;
	Args.bHideSelectionTip = true;
	Args.bAllowSearch = false;

	ToolInstance = NewObject<UBaseEditorTool>(GetTransientPackage(), InToolClass);
	ToolInstance->AddToRoot();

	TArray<UObject*> ObjectToView;
	ObjectToView.Add(ToolInstance);

	FPropertyEditorModule& PropertyEditorModule =
		FModuleManager::GetModuleChecked<FPropertyEditorModule>("PropertyEditor");
	TSharedRef<IDetailsView> DetailView = PropertyEditorModule.CreateDetailView(Args);

	DetailView->SetObjects(ObjectToView);

	SWindow::Construct(
		SWindow::FArguments()
		.Title(NSLOCTEXT("PropertyEditor", "WindowTitle", "Property Editor"))
		.ClientSize(FVector2D(500, 650))
		[
			SNew(SBorder)
			.BorderImage(FEditorStyle::GetBrush(TEXT("PropertyWindow.WindowBorder")))
			[
				SNew(SVerticalBox)
				+ SVerticalBox::Slot()
				.FillHeight(1)
				.Padding(3)
				[
					DetailView
				]
				+ SVerticalBox::Slot()
				.AutoHeight()
				.Padding(3)
				[
					SNew(SVerticalBox)
					+ SVerticalBox::Slot() // Add user input block
					.Padding(2, 2, 2, 4)
					.AutoHeight()
					.HAlign(HAlign_Right)
					.VAlign(VAlign_Bottom)
					[
						SNew(SUniformGridPanel)
						.SlotPadding(FEditorStyle::GetMargin("StandardDialog.SlotPadding"))
						.MinDesiredSlotWidth(FEditorStyle::GetFloat("StandardDialog.MinDesiredSlotWidth"))
						.MinDesiredSlotHeight(FEditorStyle::GetFloat("StandardDialog.MinDesiredSlotHeight"))
						+ SUniformGridPanel::Slot(0, 0)
						[
							SNew(SButton)
							.Text(LOCTEXT("AddFiles", "Add Files(s)..."))
							.HAlign(HAlign_Center)
							.ContentPadding(FEditorStyle::GetMargin("StandardDialog.ContentPadding"))
							.OnClicked(this, &SMyDetailViewWindow::OnClick, EAppReturnType::Ok)
						]
						+ SUniformGridPanel::Slot(1, 0)
						[
							SNew(SButton)
							.Text(LOCTEXT("Cancel", "Cancel"))
							.HAlign(HAlign_Center)
							.ContentPadding(FEditorStyle::GetMargin("StandardDialog.ContentPadding"))

							.OnClicked(this, &SMyDetailViewWindow::OnClick, EAppReturnType::Cancel)
						]
					]
				]
			]
		]
	);


	this->SetOnWindowClosed(
		FOnWindowClosed::CreateRaw(this, &SMyDetailViewWindow::OnDetailViewWindowClosed));
	
	// If the main frame exists parent the window to it
	TSharedPtr< SWindow > ParentWindow;
	if (FModuleManager::Get().IsModuleLoaded("MainFrame"))
	{
		IMainFrameModule& MainFrame = FModuleManager::GetModuleChecked<IMainFrameModule>("MainFrame");
		ParentWindow = MainFrame.GetParentWindow();
	}

	if (ParentWindow.IsValid())
	{
		// Parent the window to the main frame 
		FSlateApplication::Get().AddWindowAsNativeChild(SharedThis(this), ParentWindow.ToSharedRef());
	}
	else
	{
		FSlateApplication::Get().AddWindow(SharedThis(this));
	}
}


FReply SMyDetailViewWindow::OnClick(EAppReturnType::Type ButtonID)
{
	int val = ButtonID;

	UE_LOG(LogTemp, Warning, TEXT("#### OnClickCancelButton ####  %d"), val);

	UserResponse = ButtonID;

	if (ButtonID == EAppReturnType::Cancel)
	{


		// Only close the window if canceling or if the asset name is valid
		RequestDestroyWindow();
	}
	else
	{
		// reset the user response in case the window is closed using 'x'.
		UserResponse = EAppReturnType::Cancel;
	}

	return FReply::Handled();
}


void SMyDetailViewWindow::OnDetailViewWindowClosed(const TSharedRef<SWindow>& Window)
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnDetailViewWindowClosed ####"));

	OnCloseDelegate.Execute();

	ToolInstance->RemoveFromRoot();
	ToolInstance = nullptr;
}

#undef LOCTEXT_NAMESPACE
```

