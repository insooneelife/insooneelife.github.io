---
title: "[Unreal] Slate Class"
categories:
  - Slate
---

### Make custom slate class
Slate를 이용한 커스텀 클래스 ui 제작 예제이다.

참고 영상
https://www.youtube.com/watch?v=jeK6DPB5weA


#### ~.Build.cs
```c++
...
PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });
...
```

#### SMainMenuWidget.h
```c++

// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "SlateBasics.h"
#include "SlateExtras.h"

class MENU_API SMainMenuWidget : public SCompoundWidget
{
public:

	SLATE_BEGIN_ARGS(SMainMenuWidget) {}
	
	// Construct 시 OwningHUD의 대한 Argument를 앞에 under bar가 붙은 형태로 생성해준다. ex) InArgs._OwningHUD
	SLATE_ARGUMENT(TWeakObjectPtr<class AMenuHUD>, OwningHUD)
	SLATE_ARGUMENT(float, TempVal)
	SLATE_END_ARGS()

	// slate class는 생성자와 소멸자를 사용하지 않고 Construct를 사용하여 생성시키도록 한다.
	void Construct(const FArguments& InArgs);
	virtual bool SupportsKeyboardFocus() const override { return true; }
	FReply OnQuitClicked() const;


private:
	TWeakObjectPtr<class AMenuHUD> OwningHUD;
	float TempVal;

};


```

#### SMainMenuWidget.cpp
```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "SMainMenuWidget.h"
#include "MenuHUD.h"
#include "GameFramework/PlayerController.h"

// localize for all languages
#define LOCTEXT_NAMESPACE "MainMenu"

void SMainMenuWidget::Construct(const FArguments & InArgs)
{
	// 생성될 때 받은 argument들이 InArgs를 통해 들어온다.
	// InArgs._TempVal
	// InArgs._OwningHUD

	OwningHUD = InArgs._OwningHUD;

	const FMargin ContentPadding = FMargin(500.f, 300.f);
	const FMargin ButtonPadding = FMargin(10.f);

	// localize "My Title Name" by key "GameTitle"
	const FText TitleText = LOCTEXT("GameTitle", "My Title Name");
	const FText QuitText = LOCTEXT("QuitGame", "Quit Game");

	// get unreal core style by FCoreStyle
	// FCoreStyle 를 통해 언리얼 코어 스타일을 사용할 수 있다.
	FSlateFontInfo ButtonTextStyle = FCoreStyle::Get().GetFontStyle("EmbossedText");
	ButtonTextStyle.Size = 40.f;

	FSlateFontInfo TitleTextStyle = ButtonTextStyle;
	TitleTextStyle.Size = 40.f;

	ChildSlot
	[
		SNew(SOverlay)
		+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Fill)
		[
			SNew(SImage)
			.ColorAndOpacity(FColor::Black)
		]
		+ SOverlay::Slot()
		.HAlign(HAlign_Fill)
		.VAlign(VAlign_Fill)
		.Padding(ContentPadding)
		[
			SNew(SVerticalBox)
			+ SVerticalBox::Slot()
			[
				SNew(STextBlock)
				.Font(TitleTextStyle)
				.Text(TitleText)
				.Justification(ETextJustify::Center)
			]
			+ SVerticalBox::Slot()
			.Padding(ButtonPadding)
			[
				SNew(SButton)
				.OnClicked(this, &SMainMenuWidget::OnQuitClicked)
				[
					SNew(STextBlock)
					.Font(ButtonTextStyle)
					.Text(QuitText)
					.Justification(ETextJustify::Center)
				]
			]
		]

	];
}

FReply SMainMenuWidget::OnQuitClicked() const
{
	if (OwningHUD.IsValid())
	{
		if (APlayerController* PC = OwningHUD->PlayerOwner)
		{
			PC->ConsoleCommand("quit");
		}
	}

	return FReply::Handled();
}

#undef LOCTEXT_NAMESPACE

```


#### MenuHUD.h
```c++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/HUD.h"
#include "MenuHUD.generated.h"

UCLASS()
class MENU_API AMenuHUD : public AHUD
{
	GENERATED_BODY()
	
protected:
	TSharedPtr<class SMainMenuWidget> MenuWidget;
	TSharedPtr<class SWidget> MenuWidgetContainer;
	
	virtual void BeginPlay() override;
};

```

#### MenuHUD.cpp
```c++
#include "MenuHUD.h"

#include "SMainMenuWidget.h"
#include "Widgets/SWeakWidget.h"
#include "Engine/Engine.h"

void AMenuHUD::BeginPlay()
{
	Super::BeginPlay();

	if (GEngine && GEngine->GameViewport)
	{
		// slate 생성
		// .OwningHUD(this).TempVal(5)를 통해 Construct 함수의 InArgs에 파라메터를 넘겨준다.
		MenuWidget = SNew(SMainMenuWidget).OwningHUD(this).TempVal(5);
		GEngine->GameViewport->AddViewportWidgetContent(
			SAssignNew(MenuWidgetContainer, SWeakWidget).PossiblyNullContent(MenuWidget.ToSharedRef()));
	}
}

```


