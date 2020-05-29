---
title: "[Unreal] Asset Path Picking Dialog"
categories:
  - Editor
  - AssetPath
  - Picking
---



#### source
다음 코드로 asset path를 가져오는 dialog를 띄울 수 있다.
if문 내부에서 dialog가 닫히고 가져온 path를 사용할 수 있다.

```c++
FString NewNameSuggestion = FString(TEXT("Suggestion"));
FString PackageNameSuggestion = FString(TEXT("/Game/")) + NewNameSuggestion;
FString Name;

TSharedPtr<SDlgPickAssetPath> PickAssetPathWidget =
  SNew(SDlgPickAssetPath)
  .Title(LOCTEXT("MyTitleName", "Choose Your Location"))
  .DefaultAssetPath(FText::FromString(PackageNameSuggestion));

if (PickAssetPathWidget->ShowModal() == EAppReturnType::Ok)
{
  // Get the full name of where we want to create the mesh asset.
  FString PackageName = PickAssetPathWidget->GetFullAssetPath().ToString();
  UE_LOG(LogTemp, Warning, TEXT("##### Asset Path Here : %s  #####"), *PackageName);
}
```


#### result
![image-center](/assets/images/unreal-assetpath-picker-dialog.png){: .align-left}


#### customized class
#### header

```c++
#pragma once

#include "CoreMinimal.h"
#include "Widgets/DeclarativeSyntaxSupport.h"
#include "Input/Reply.h"
#include "Widgets/SWindow.h"

#define LOCTEXT_NAMESPACE "MyAssetPathPicker"

class SMyAssetPathPicker : public SWindow
{
public:
	SLATE_BEGIN_ARGS(SMyAssetPathPicker)
	{
	}
	SLATE_ARGUMENT(FText, Title)
	SLATE_ARGUMENT(FText, DefaultAssetPath)
	SLATE_END_ARGS()

	SMyAssetPathPicker()
		: UserResponse(EAppReturnType::Cancel)
	{
	}

	void Construct(const FArguments& InArgs);

	/** Displays the dialog in a blocking fashion */
	EAppReturnType::Type ShowModal();

	/** Gets the resulting asset path */
	const FText& GetAssetPath();

	/** Gets the resulting asset name */
	const FText& GetAssetName();

	/** Gets the resulting full asset path (path+'/'+name) */
	FText GetFullAssetPath();

protected:
	void OnPathChange(const FString& NewPath);
	void OnNameChange(const FText& NewName, ETextCommit::Type CommitInfo);
	FReply OnButtonClick(EAppReturnType::Type ButtonID);

	bool ValidatePackage();

	EAppReturnType::Type UserResponse;
	FText AssetPath;
	FText AssetName;
};

#undef LOCTEXT_NAMESPACE

```

#### source
```c++
// Copyright 1998-2019 Epic Games, Inc. All Rights Reserved.

#include "SMyAssetPathPicker.h"
#include "Misc/MessageDialog.h"
#include "Modules/ModuleManager.h"
#include "Misc/PackageName.h"
#include "Widgets/Layout/SBorder.h"
#include "Widgets/Text/STextBlock.h"
#include "Widgets/Layout/SUniformGridPanel.h"
#include "Widgets/Input/SEditableTextBox.h"
#include "Widgets/Input/SButton.h"
#include "EditorStyleSet.h"
#include "Editor.h"
#include "IContentBrowserSingleton.h"
#include "ContentBrowserModule.h"

#define LOCTEXT_NAMESPACE "MyAssetPathPicker"

void SMyAssetPathPicker::Construct(const FArguments& InArgs)
{
	AssetPath = FText::FromString(FPackageName::GetLongPackagePath(InArgs._DefaultAssetPath.ToString()));
	AssetName = FText::FromString(FPackageName::GetLongPackageAssetName(InArgs._DefaultAssetPath.ToString()));

	FPathPickerConfig PathPickerConfig;
	PathPickerConfig.DefaultPath = AssetPath.ToString();
	PathPickerConfig.OnPathSelected = FOnPathSelected::CreateSP(this, &SMyAssetPathPicker::OnPathChange);
	PathPickerConfig.bAddDefaultPath = true;

	FContentBrowserModule& ContentBrowserModule = 
		FModuleManager::LoadModuleChecked<FContentBrowserModule>("ContentBrowser");

	SWindow::Construct(
		SWindow::FArguments()
		.Title(InArgs._Title)
		.SupportsMinimize(false)
		.SupportsMaximize(false)
		//.SizingRule( ESizingRule::Autosized )
		.ClientSize(FVector2D(450, 450))
		[
			SNew(SVerticalBox)

			+ SVerticalBox::Slot() // Add user input block
			.Padding(2, 2, 2, 4)
			[
				SNew(SBorder)
				.BorderImage(FEditorStyle::GetBrush("ToolPanel.GroupBorder"))
				[
					SNew(SVerticalBox)

					+ SVerticalBox::Slot()
					.FillHeight(1)
					.Padding(3)
					[
						ContentBrowserModule.Get().CreatePathPicker(PathPickerConfig)
					]
				]
			]
	

			+ SVerticalBox::Slot()
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
					.Text(LOCTEXT("OK", "OK"))
					.HAlign(HAlign_Center)
					.ContentPadding(FEditorStyle::GetMargin("StandardDialog.ContentPadding"))
					.OnClicked(this, &SMyAssetPathPicker::OnButtonClick, EAppReturnType::Ok)
				]
				+ SUniformGridPanel::Slot(1, 0)
				[
					SNew(SButton)
					.Text(LOCTEXT("Cancel", "Cancel"))
					.HAlign(HAlign_Center)
					.ContentPadding(FEditorStyle::GetMargin("StandardDialog.ContentPadding"))
					.OnClicked(this, &SMyAssetPathPicker::OnButtonClick, EAppReturnType::Cancel)
				]
			]
		]);
}


void SMyAssetPathPicker::OnPathChange(const FString& NewPath)
{
	AssetPath = FText::FromString(NewPath);
}

FReply SMyAssetPathPicker::OnButtonClick(EAppReturnType::Type ButtonID)
{
	UserResponse = ButtonID;

	if (ButtonID == EAppReturnType::Cancel || ValidatePackage())
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

/** Ensures supplied package name information is valid */
bool SMyAssetPathPicker::ValidatePackage()
{
	FText Reason;
	if (!FPackageName::IsValidLongPackageName(GetFullAssetPath().ToString(), false, &Reason)
		|| !FName(*AssetName.ToString()).IsValidObjectName(Reason))
	{
		FMessageDialog::Open(EAppMsgType::Ok, Reason);
		return false;
	}

	if (FPackageName::DoesPackageExist(GetFullAssetPath().ToString()) || 
		FindObject<UObject>(NULL, *(AssetPath.ToString() + "/" + AssetName.ToString() + "." + AssetName.ToString())) != NULL)
	{
		FMessageDialog::Open(EAppMsgType::Ok, FText::Format(LOCTEXT("AssetAlreadyExists", "Asset {0} already exists."), GetFullAssetPath()));
		return false;
	}

	return true;
}

EAppReturnType::Type SMyAssetPathPicker::ShowModal()
{
	GEditor->EditorAddModalWindow(SharedThis(this));
	return UserResponse;
}

const FText& SMyAssetPathPicker::GetAssetPath()
{
	return AssetPath;
}

const FText& SMyAssetPathPicker::GetAssetName()
{
	return AssetName;
}

FText SMyAssetPathPicker::GetFullAssetPath()
{
	return FText::FromString(AssetPath.ToString() + "/" + AssetName.ToString());
}


#undef LOCTEXT_NAMESPACE

```

