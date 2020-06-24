---
title: "[Unreal] Detail Editor Customization"
categories:
  - Editor
  - Detail
  - Customization
---

#### Modules
다음 모듈들을 미리 추가해두자.
```c#
PrivateDependencyModuleNames.AddRange(
			new string[]
			{
				"Projects",
				"InputCore",
				"UnrealEd",
				"LevelEditor",
				"CoreUObject",
				"Engine",
				"Slate",
				"SlateCore",
				"EditorScriptingUtilities",
				"EditorStyle",
				"DesktopPlatform",
				// ... add private dependencies that you statically link with here ...	
			}
			);
```


#### UMyObject
디테일 패널의 표시될 데이터를 갖는 타겟 오브젝트로 일반적인 UObject를 상속한 클래스이다.
디테일 패널에서 수정하고 싶은 프로퍼티들을 정의해주면 된다.

```c++
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "Animation/Skeleton.h"
#include "MyObject.generated.h"


UENUM(BlueprintType)
enum class EType : uint8
{
	Type1,
	Type2
};

UCLASS()
class MYPLUGIN_API UMyObject : public UObject
{
	GENERATED_BODY()
	
public:
	UMyObject() {}


	UPROPERTY(EditAnywhere, Category = "Voice Characteristics")
		EType Language;

	UPROPERTY(EditAnywhere, Category = "Conversion")
		FString SaveAssetTo;

	UPROPERTY(EditAnywhere, Category = "Conversion")
		USkeleton* TargetSkeleton;

	UPROPERTY(EditAnywhere, Category = "Conversion")
		bool bImportAudio;

	UPROPERTY(EditAnywhere, Category = "Conversion")
		bool bInsertPlaySoundNotify;

	UPROPERTY(EditAnywhere, Category = "FilesToDump")
		bool bPhoneme;

	UPROPERTY(EditAnywhere, Category = "FilesToDump")
		bool bAnimClip;

	UPROPERTY(EditAnywhere, Category = "FilesToDump")
		bool bConversionLog;
};

```


#### MyCustomization
UMyObject에 정의된 데이터로 디테일 패널을 실질적으로 꾸며주는 부분이다.
CustomizeDetails 함수가 호출에서 디테일 패널을 꾸며주는 코드를 작성하면 된다.
CustomizeDetails는 디테일 패널이 생성되거나 수정사항이 생길 때 호출된다.

#### header
```c++
#pragma once

#include "CoreMinimal.h"
#include "IDetailCustomization.h"
#include "SlateBasics.h"

class UMyObject;
class MYPLUGIN_API MyCustomization : public IDetailCustomization
{
public:
	const FString DefaultPath = TEXT("/Game");

public:

	static TSharedRef<IDetailCustomization> MakeInsance();

	virtual void CustomizeDetails(IDetailLayoutBuilder& DetailBuilder) override;

	FReply OnSaveAssetPathBtnClicked();

	EAppReturnType::Type CreatePickAssetPathWidget(FString& AssetPath);

	void OnCheckImportAudio(ECheckBoxState State);

	void OnCheckInsertPlaySoundNotify(ECheckBoxState State);

private:
	TSharedPtr<STextBlock> SaveAssetPathBtnText;

	TSharedPtr<SPanel> InsertPlaySoundNotifyNameWidget;

	TSharedPtr<SPanel> InsertPlaySoundNotifyValueWidget;

	UMyObject* Target;

};

```

#### source
```c++
#include "MyCustomization.h"
#include "PropertyEditing.h"
#include "Engine/StaticMesh.h"
#include "MyObject.h"
#include "SMyAssetPathPicker.h"
#include "Widgets/Input/SCheckBox.h"

#define LOCTEXT_NAMESPACE "MyCustomization"

TSharedRef<IDetailCustomization> MyCustomization::MakeInsance()
{
	UE_LOG(LogTemp, Warning, TEXT("######################MakeInsance"));
	return MakeShareable(new MyCustomization);
}


void MyCustomization::CustomizeDetails(IDetailLayoutBuilder& DetailBuilder)
{
	UE_LOG(LogTemp, Warning, TEXT("###################### CustomizeDetails"));

	TArray<TWeakObjectPtr<UObject>> CustomizedObjects;
	DetailBuilder.GetObjectsBeingCustomized(CustomizedObjects);

	for (TWeakObjectPtr<UObject> Object : CustomizedObjects)
	{
		if (Object.IsValid())
		{
			Target = Cast<UMyObject>(Object);
			if (Target)
				break;
		}
	}

	if (Target)
	{
		UE_LOG(LogTemp, Warning, TEXT("##### Target is here!! #####"));
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("##### Target is null!! #####"));
	}

	TSharedRef<IPropertyHandle> LanguageProp = DetailBuilder.GetProperty("Language");
	TSharedRef<IPropertyHandle> SaveAssetToProp = DetailBuilder.GetProperty("SaveAssetTo");
	TSharedRef<IPropertyHandle> TargetSkeletonProp = DetailBuilder.GetProperty("TargetSkeleton");
	TSharedRef<IPropertyHandle> ImportAudioProp = DetailBuilder.GetProperty("bImportAudio");
	TSharedRef<IPropertyHandle> InsertPlaySoundNotifyProp = DetailBuilder.GetProperty("bInsertPlaySoundNotify");
	TSharedRef<IPropertyHandle> PhonemeProp = DetailBuilder.GetProperty("bPhoneme");
	TSharedRef<IPropertyHandle> AnimClipProp = DetailBuilder.GetProperty("bAnimClip");
	TSharedRef<IPropertyHandle> ConversionLogProp = DetailBuilder.GetProperty("bConversionLog");

	IDetailCategoryBuilder& VoiceCharacteristicsCategory =
		DetailBuilder.EditCategory("Voice Characteristics", FText::GetEmpty());

	IDetailCategoryBuilder& ConversionCategory = DetailBuilder.EditCategory("Conversion", FText::GetEmpty());

	DetailBuilder.HideProperty("SaveAssetTo");
	ConversionCategory.AddCustomRow(FText::FromString("ConversionSaveAssetTo"))
		.NameContent()
		[
			SaveAssetToProp->CreatePropertyNameWidget()
		]
	.ValueContent()
		[
			SNew(SButton)
			.OnClicked(this, &MyCustomization::OnSaveAssetPathBtnClicked)
		[
			SAssignNew(SaveAssetPathBtnText, STextBlock)
			.Text(FText::FromString(DefaultPath))
		.ToolTipText(FText::FromString(DefaultPath))
		.Font(IDetailLayoutBuilder::GetDetailFont())
		]
		];


	DetailBuilder.AddPropertyToCategory(TargetSkeletonProp);

	IDetailPropertyRow& ImportAudioRow = DetailBuilder.AddPropertyToCategory(ImportAudioProp);
	FDetailWidgetRow& ImportAudioWidgetRow = ImportAudioRow.CustomWidget();
	ImportAudioWidgetRow
		.NameWidget
		[
			ImportAudioProp->CreatePropertyNameWidget()
		]
	.ValueWidget
		[
			SNew(SCheckBox)
			.OnCheckStateChanged(this, &MyCustomization::OnCheckImportAudio)
		];

	IDetailPropertyRow& InsertPlaySoundNotifyRow = DetailBuilder.AddPropertyToCategory(InsertPlaySoundNotifyProp);

	FDetailWidgetRow& InsertPlaySoundNotifyWidgetRow = InsertPlaySoundNotifyRow.CustomWidget();
	InsertPlaySoundNotifyWidgetRow
		.NameWidget
		[
			SAssignNew(InsertPlaySoundNotifyNameWidget, SHorizontalBox)
			+ SHorizontalBox::Slot()
		.AutoWidth()
		[
			InsertPlaySoundNotifyProp->CreatePropertyNameWidget()
		]
		]
	.ValueWidget
		[
			SAssignNew(InsertPlaySoundNotifyValueWidget, SHorizontalBox)
			+ SHorizontalBox::Slot()
		.AutoWidth()
		[
			SNew(SCheckBox)
			.OnCheckStateChanged(this, &MyCustomization::OnCheckImportAudio)
		]
		];
	InsertPlaySoundNotifyNameWidget->SetVisibility(EVisibility::Hidden);
	InsertPlaySoundNotifyValueWidget->SetVisibility(EVisibility::Hidden);

	IDetailCategoryBuilder& FilesToDumpCategory = DetailBuilder.EditCategory("FilesToDump", FText::GetEmpty());


}

FReply MyCustomization::OnSaveAssetPathBtnClicked()
{
	FString Str = SaveAssetPathBtnText->GetText().ToString();
	UE_LOG(LogTemp, Warning, TEXT("#### OnSaveAssetPathBtnClicked ####"));
	EAppReturnType::Type RetType = CreatePickAssetPathWidget(Str);

	UE_LOG(LogTemp, Warning, TEXT("#### OnSaveAssetToBtnClicked ####  %s  !"), *Str);

	Target->SaveAssetTo = Str;
	SaveAssetPathBtnText->SetText(FText::FromString(Str));
	SaveAssetPathBtnText->SetToolTipText(FText::FromString(Str));

	return FReply::Handled();
}

EAppReturnType::Type MyCustomization::CreatePickAssetPathWidget(FString& AssetPath)
{
	FString PackageNameSuggestion = DefaultPath;
	FString Name;

	UE_LOG(LogTemp, Warning, TEXT("#### CreatePickAssetPathWidget ####"));
	TSharedPtr<SMyAssetPathPicker> PickAssetPathWidget =
		SNew(SMyAssetPathPicker)
		.Title(LOCTEXT("AssetPathPickerTitle", "Choose Your Location"))
		.DefaultAssetPath(FText::FromString(PackageNameSuggestion));

	EAppReturnType::Type RetType = PickAssetPathWidget->ShowModal();

	if (RetType == EAppReturnType::Ok)
	{
		AssetPath = PickAssetPathWidget->GetAssetPath().ToString();
	}

	UE_LOG(LogTemp, Warning, TEXT("#### CreatePickAssetPathWidget Return ####"));
	return RetType;
}


void MyCustomization::OnCheckImportAudio(ECheckBoxState State)
{
	if (State == ECheckBoxState::Checked)
	{
		InsertPlaySoundNotifyNameWidget->SetVisibility(EVisibility::All);
		InsertPlaySoundNotifyValueWidget->SetVisibility(EVisibility::All);
		Target->bImportAudio = true;
	}
	else if (State == ECheckBoxState::Unchecked)
	{
		InsertPlaySoundNotifyNameWidget->SetVisibility(EVisibility::Hidden);
		InsertPlaySoundNotifyValueWidget->SetVisibility(EVisibility::Hidden);
		Target->bImportAudio = false;
	}
}

void MyCustomization::OnCheckInsertPlaySoundNotify(ECheckBoxState State)
{
	if (State == ECheckBoxState::Checked)
	{
		Target->bInsertPlaySoundNotify = true;
	}
	else if (State == ECheckBoxState::Unchecked)
	{
		Target->bInsertPlaySoundNotify = false;
	}
}


#undef LOCTEXT_NAMESPACE

```

#### SMyWindow
위에 정의한 UMyObject의 데이터를 표시해줄 수 있는 윈도우를 언리얼 에디터 상에 띄우기 위해,
프로퍼티 에디터를 이용하여 SWindow 클래스를 제작한다.

#### header
```c++
#pragma once

#include "CoreMinimal.h"
#include "Input/Reply.h"
#include "Widgets/SWindow.h"
#include "UObject/WeakObjectPtrTemplates.h"

class UMyObject;

class MYPLUGIN_API SMyWindow : public SWindow
{
public:
	SLATE_BEGIN_ARGS(SMyWindow) {}
	SLATE_END_ARGS()

	void Construct(const FArguments& InArgs);

	EAppReturnType::Type ShowModal();
private:

	FReply OnClickCancel();
	FReply OnClickAddFiles();
	void OnClosed(const TSharedRef<SWindow>& Window);

private:
	EAppReturnType::Type UserResponse;
	UMyObject* TargetObject;
};

```

#### source
```c++
#include "SMyWindow.h"
#include "MyObject.h"

#include "PropertyEditing.h"
#include "Modules/ModuleManager.h"
#include "Framework/Application/SlateApplication.h"
#include "SlateBasics.h"
#include "SlateExtras.h"
#include "Editor/EditorEngine.h"
#include "DesktopPlatformModule.h"	//"DesktopPlatform",
#include "IDesktopPlatform.h"
#include "Editor.h"

#define LOCTEXT_NAMESPACE "SNFaceConvertWindow"

void SMyWindow::Construct(const FArguments& InArgs)
{
	// create detail view
	FDetailsViewArgs Args;
	Args.bHideSelectionTip = true;
	Args.bAllowSearch = false;

	// UMyObject를 프로퍼티 에디터 커스터마이징 타겟으로 잡는다.
	TargetObject = NewObject<UMyObject>();
	TargetObject->AddToRoot();
	TArray<UObject*> ObjectToView;
	ObjectToView.Add(TargetObject);

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
		.OnClicked(this, &SMyWindow::OnClickAddFiles)
		]
	+ SUniformGridPanel::Slot(1, 0)
		[
			SNew(SButton)
			.Text(LOCTEXT("Cancel", "Cancel"))
		.HAlign(HAlign_Center)
		.ContentPadding(FEditorStyle::GetMargin("StandardDialog.ContentPadding"))
		.OnClicked(this, &SMyWindow::OnClickCancel)
		]
		]
		]
		]
		]
	);


	this->SetOnWindowClosed(FOnWindowClosed::CreateRaw(this, &SMyWindow::OnClosed));
}


FReply SMyWindow::OnClickCancel()
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnClickCancel ####"));
	RequestDestroyWindow();
	return FReply::Handled();
}

FReply SMyWindow::OnClickAddFiles()
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnClickAddFiles ####"));
	
	return FReply::Handled();
}


void SMyWindow::OnClosed(const TSharedRef<SWindow>& Window)
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnClosed ####"));
	TargetObject->RemoveFromRoot();
}

EAppReturnType::Type SMyWindow::ShowModal()
{
	GEditor->EditorAddModalWindow(SharedThis(this));
	return UserResponse;
}


#undef LOCTEXT_NAMESPACE
```


#### MyPlugin.cpp
MyPlugin 코드에서 플러그인 버튼이 눌러졌을 때 SMyWindow를 띄우도록 한다.

```c++
// Copyright 1998-2019 Epic Games, Inc. All Rights Reserved.

#include "MyPlugin.h"
#include "MyPluginStyle.h"
#include "MyPluginCommands.h"
#include "Misc/MessageDialog.h"
#include "Framework/MultiBox/MultiBoxBuilder.h"

#include "MyPrimaryDataAsset.h"
#include "AssetRegistryModule.h"
#include "UObject/ConstructorHelpers.h"

#include "LevelEditor.h"
#include "EngineUtils.h"

#include "EditorAssetLibrary.h"
#include "MyCustomization.h"
#include "SMyWindow.h"

static const FName MyPluginTabName("MyPlugin");

#define LOCTEXT_NAMESPACE "FMyPluginModule"

FMyPluginModule::FMyPluginModule()
{
	
}

void FMyPluginModule::StartupModule()
{	
	// This code will execute after your module is loaded into memory; the exact timing is specified in the .uplugin file per-module
	FMyPluginStyle::Initialize();
	FMyPluginStyle::ReloadTextures();

	FMyPluginCommands::Register();
	
	PluginCommands = MakeShareable(new FUICommandList);

	PluginCommands->MapAction(
		FMyPluginCommands::Get().PluginAction,
		FExecuteAction::CreateRaw(this, &FMyPluginModule::PluginButtonClicked),
		FCanExecuteAction());
		
	FLevelEditorModule& LevelEditorModule = FModuleManager::LoadModuleChecked<FLevelEditorModule>("LevelEditor");
	
	{
		TSharedPtr<FExtender> MenuExtender = MakeShareable(new FExtender());
		MenuExtender->AddMenuExtension("WindowLayout", EExtensionHook::After, PluginCommands, FMenuExtensionDelegate::CreateRaw(this, &FMyPluginModule::AddMenuExtension));

		LevelEditorModule.GetMenuExtensibilityManager()->AddExtender(MenuExtender);
	}
	
	{
		TSharedPtr<FExtender> ToolbarExtender = MakeShareable(new FExtender);
		ToolbarExtender->AddToolBarExtension("Settings", EExtensionHook::After, PluginCommands, FToolBarExtensionDelegate::CreateRaw(this, &FMyPluginModule::AddToolbarExtension));
		
		LevelEditorModule.GetToolBarExtensibilityManager()->AddExtender(ToolbarExtender);
	}


	FPropertyEditorModule& PropertyModule = 
		FModuleManager::LoadModuleChecked<FPropertyEditorModule>("PropertyEditor");

	PropertyModule.RegisterCustomClassLayout(
		"MyObject", FOnGetDetailCustomizationInstance::CreateStatic(&MyCustomization::MakeInsance));

	InitConsoleCommand();
}

void FMyPluginModule::ShutdownModule()
{
	// This function may be called during shutdown to clean up your module.  For modules that support dynamic reloading,
	// we call this function before unloading the module.
	FMyPluginStyle::Shutdown();

	FMyPluginCommands::Unregister();
}

void FMyPluginModule::PluginButtonClicked()
{
	// Put your "OnButtonClicked" stuff here
	FText DialogText = FText::Format(
							LOCTEXT("PluginButtonDialogText", "Add code to {0} in {1} to override this button's actions"),
							FText::FromString(TEXT("FMyPluginModule::PluginButtonClicked()")),
							FText::FromString(TEXT("MyPlugin.cpp"))
					   );


	// SMyWindow를 띄운다.
	TSharedRef<SMyWindow> Window = SNew(SMyWindow);

	if (Window->ShowModal() == EAppReturnType::Ok)
	{
		UE_LOG(LogTemp, Warning, TEXT("#### OnExecuteFromAudioAction ShowModal Ok ####"));
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("#### OnExecuteFromAudioAction ShowModal Not Ok ####"));
	}

	//FMessageDialog::Open(EAppMsgType::Ok, DialogText);
}

void FMyPluginModule::AddMenuExtension(FMenuBuilder& Builder)
{
	Builder.AddMenuEntry(FMyPluginCommands::Get().PluginAction);
}

void FMyPluginModule::AddToolbarExtension(FToolBarBuilder& Builder)
{
	Builder.AddToolBarButton(FMyPluginCommands::Get().PluginAction);
}

// call this from start - ex)  gamemode being
void FMyPluginModule::InitConsoleCommand()
{
	FConsoleCommandWithArgsDelegate Delegate;

	Delegate.BindRaw(this, &FMyPluginModule::ConsoleCommand);

	IConsoleManager::Get().RegisterConsoleCommand(
		TEXT("insooneelifeCmd"),
		TEXT("insooneelife test cmd"), Delegate);
}

// this is static function
void FMyPluginModule::ConsoleCommand(const TArray<FString>& Args)
{
	UE_LOG(LogTemp, Warning, TEXT("#################"));

	FAssetRegistryModule& AssetRegistryModule =
		FModuleManager::LoadModuleChecked<FAssetRegistryModule>("AssetRegistry");
	FAssetData AssetData =
		AssetRegistryModule.Get().GetAssetByObjectPath(TEXT("/Game/Temp/MyPrimaryDataAsset.MyPrimaryDataAsset"));


	if (AssetData.IsValid())
	{
		UMyPrimaryDataAsset* Data = Cast<UMyPrimaryDataAsset> (AssetData.GetAsset());
		UE_LOG(LogTemp, Warning, TEXT("#######  Success %s #######"), *Data->ItemName);
	}
	else
	{
		CreateAsset();
		UE_LOG(LogTemp, Warning, TEXT("#######  Failed  #######"));
	}
}

void FMyPluginModule::CreateAsset()
{
	
	FString AssetName = TEXT("MyPrimaryDataAsset");
	FString PackageName = TEXT("/Game/Temp/");
	PackageName += AssetName;

	UPackage* Package = CreatePackage(NULL, *PackageName);
	Package->FullyLoad();

	UMyPrimaryDataAsset* NewAsset = NewObject<UMyPrimaryDataAsset>(
		Package, *AssetName, RF_Public | RF_Standalone | RF_MarkAsRootSet);

	NewAsset->ItemName = TEXT("NewAssetName");

	Package->MarkPackageDirty();
	FAssetRegistryModule::AssetCreated(NewAsset);

	FString PackageFileName = FPackageName::LongPackageNameToFilename(
		PackageName, FPackageName::GetAssetPackageExtension());

	bool bSaved = UPackage::SavePackage(
		Package,
		NewAsset,
		EObjectFlags::RF_Public | EObjectFlags::RF_Standalone,
		*PackageFileName,
		GError, nullptr, true, true, SAVE_NoError);

	TArray<UObject*> ObjectsToSync;
	ObjectsToSync.Add(NewAsset);
	GEditor->SyncBrowserToObjects(ObjectsToSync);
	

}

#undef LOCTEXT_NAMESPACE
	
IMPLEMENT_MODULE(FMyPluginModule, MyPlugin)
```

#### 결과 화면

![image-center](/assets/images/unreal-detail-editor-customization.png){: .align-left}



