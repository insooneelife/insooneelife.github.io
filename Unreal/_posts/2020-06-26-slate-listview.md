---
title: "[Unreal] Slate ListView"
categories:
  - Slate
  - ListView
---

#### Slate List View
Slate ListView 예제이다.


#### SMyListViewWidget.h
```c++
#pragma once

#include "CoreMinimal.h"
#include "SlateFwd.h"
#include "SlateBasics.h"
#include "Widgets/DeclarativeSyntaxSupport.h"
#include "Input/Reply.h"
#include "Widgets/SWidget.h"
#include "Widgets/SCompoundWidget.h"
#include "Widgets/Views/STableViewBase.h"
#include "Widgets/Views/STableRow.h"
#include "Widgets/Input/SComboButton.h"

#include "Widgets/SWindow.h"

#include "Widgets/Views/SListView.h"

//////////////////////////////////////////////////////////////////////////
// Data

UENUM(BlueprintType)
enum class EStatusMode : uint8
{
	Done,
	Failed,
	Converting,
	Ready,
	Cancelled,
	Cancelling
};

class FMyListRowInfo
{
public:
	int Number;
	FString WaveFileName;
	bool HasScript;
	EStatusMode Status;

	// Static function for creating a new item, but ensures that you can only have a TSharedRef to one
	static TSharedRef<FMyListRowInfo> Make(
		int Number,
		FString WaveFileName,
		bool HasScript,
		EStatusMode Status)
	{
		return MakeShareable(new FMyListRowInfo(Number, WaveFileName, HasScript, Status));
	}

	int GetNumber() const
	{
		return Number;
	}

	FString GetWaveFileName() const
	{
		return WaveFileName;
	}

	bool GetHasScript() const
	{
		return HasScript;
	}

	FString GetHasScriptAsString() const
	{
		return HasScript ? FString(TEXT("true")) : FString(TEXT("false"));
	}

	EStatusMode GetStatus() const
	{
		return Status;
	}

	FString GetStatusAsString() const
	{
		const TEnumAsByte<EStatusMode> StatusEnum = Status;
		FString EnumAsName = UEnum::GetValueAsString(StatusEnum.GetValue());
		return EnumAsName;
	}

protected:
	// Hidden constructor, always use Make above
	FMyListRowInfo(
		int Number,
		FString WaveFileName,
		bool HasScript,
		EStatusMode Status)
		:
		Number(Number),
		WaveFileName(WaveFileName),
		HasScript(HasScript),
		Status(Status)
	{}

	// Hidden constructor, always use Make above
	FMyListRowInfo() {}
};

typedef TSharedPtr< FMyListRowInfo > FMyListRowInfoPtr;
typedef SListView<FMyListRowInfoPtr> SMyListView;

//////////////////////////////////////////////////////////////////////////
// List Row


DECLARE_DELEGATE_RetVal(FText, FOnGetText)
DECLARE_DELEGATE_RetVal(FText&, FOnGetFilteredText)
DECLARE_DELEGATE_TwoParams(FOnRefreshListView, const FString&, TArray<TSharedPtr<FMyListRowInfo>>&)

class SMyListRow
	: public SMultiColumnTableRow< FMyListRowInfoPtr >
{
public:

	SLATE_BEGIN_ARGS(SMyListRow) {}

	// The item for this row
	SLATE_ARGUMENT(FMyListRowInfoPtr, Item)

		// Widget used to display the list of retarget sources
		SLATE_ARGUMENT(TSharedPtr<SMyListView>, ListView)
		SLATE_EVENT(FOnGetFilteredText, OnGetFilteredText)

	SLATE_END_ARGS()

	void Construct(const FArguments& InArgs, const TSharedRef<STableViewBase>& OwnerTableView);

	// Overridden from SMultiColumnTableRow.  
	// Generates a widget for this column of the tree row. 
	virtual TSharedRef<SWidget> GenerateWidgetForColumn(const FName& ColumnName) override;

private:
	FText GetFilterText() const;
private:

	// Widget used to display the list of retarget sources
	TSharedPtr<SMyListView> ListView;

	// The name and weight of the retarget source
	FMyListRowInfoPtr Item;

	FOnGetFilteredText OnGetFilteredTextDelegate;
};

//////////////////////////////////////////////////////////////////////////
// Convert Widget

class MYPLUGIN_API SMyListViewWidget : public SCompoundWidget
{
public:
	SLATE_BEGIN_ARGS(SMyListViewWidget)
		: _ShowViewOptionsComboBtn(false)
	{}
		SLATE_EVENT(FOnRefreshListView, OnRefreshListView)
		SLATE_ARGUMENT(bool, ShowViewOptionsComboBtn)

	SLATE_END_ARGS()

	void Construct(const FArguments& InArgs);

	void RefreshListView();

private:

	FText GetAssetCountText() const;

	//------------------------------------- Add Job Combo Button --------------------------------------//
	TSharedRef<SWidget> CreateAddJobComboBtn() const;

	TSharedRef<SWidget> OnGetAddJobBtnMenuContent() const;

	void OnExecuteFromAudioAction();
	//-------------------------------------------------------------------------------------------------//


	//----------------------------------------- Start Button ------------------------------------------//
	TSharedRef<SWidget> CreateStartBtn();

	FReply OnStartButtonClicked();

	void SetStartButtonText(const FText& Text);
	//-------------------------------------------------------------------------------------------------//


	//-------------------------------------- IP Address EditBox ---------------------------------------//
	TSharedRef<SWidget> CreateIPAddressEditBox() const;

	void OnTextChanged(const FText& NewText);

	void OnTextCommitted(const FText& NewText, ETextCommit::Type CommitType);
	//-------------------------------------------------------------------------------------------------//


	//---------------------------------------- Item SearchBox -----------------------------------------//
	TSharedRef<SWidget> CreateItemSearchBox();

	void OnItemSearchBoxTextChanged(const FText& SearchText);

	void OnItemSearchBoxTextCommitted(const FText& SearchText, ETextCommit::Type CommitInfo);
	//--------------------------------------------------------------------------------------------------//


	//------------------------------------------- List View --------------------------------------------//
	TSharedRef<SWidget> CreateListView();

	TSharedRef<ITableRow> GenerateListRow(
		FMyListRowInfoPtr InInfo, const TSharedRef<STableViewBase>& OwnerTable);

	TSharedPtr<SWidget> OnContextMenuOpening();

	void OnExecuteRemove();
	void OnExecuteLocateAsset();
	void OnExecuteOpenAnimSeq();
	void OnExecuteRecheck();
	void OnExecuteLocateAudio();
	void OnExecuteOpenAudio();
	void OnExecuteOpenScript();

	// search box text for filter row items in list view
	FText& GetFilterListRowText() { return FilterListRowText; }

	//--------------------------------------------------------------------------------------------------//



	//----------------------------------- View Options Combo Button ------------------------------------//
	TSharedRef<SWidget> CreateViewOptionsComboBtn();

	FSlateColor GetViewButtonForegroundColor() const;

	TSharedRef<SWidget> OnGetViewBtnMenuContent() const;

	void SetCurrentViewTypeFromMenu(int NewType);

	bool IsCurrentViewType(int ViewType) const;
	//--------------------------------------------------------------------------------------------------//


private:
	TSharedPtr<SSearchBox>	NameFilterSearchBox;

	TSharedPtr<SMyListView> ListView;

	TArray<FMyListRowInfoPtr> DataList;

	FText FilterListRowText;

	// The button that displays view options
	TSharedPtr<SComboButton> ViewOptionsComboButton;

	TSharedPtr<STextBlock> StartBtnText;

	FOnRefreshListView OnRefreshListViewDelegate;

};
```

#### SMyListViewWidget.cpp

#include "SMyListViewWidget.h"
#include "Misc/MessageDialog.h"
#include "Modules/ModuleManager.h"
#include "Widgets/SWindow.h"
#include "Editor.h"
#include "EditorStyleSet.h"
#include "Widgets/Input/SButton.h"
#include "AssetNotifications.h"
#include "Animation/Rig.h"
#include "Widgets/Input/SSearchBox.h"
#include "Widgets/Images/SImage.h"
#include "Widgets/Text/SInlineEditableTextBlock.h"
#include "Framework/MultiBox/MultiBoxBuilder.h"
#include "Framework/Notifications/NotificationManager.h"
#include "Widgets/Notifications/SNotificationList.h"

#include "EditorFontGlyphs.h" // EditorStyle

#define LOCTEXT_NAMESPACE "ContentBrowser"

static const FName ColumnId_NumberLabel("Number");
static const FName ColumnID_NameLabel("Name");
static const FName ColumnID_ScriptLabel("Script");
static const FName ColumnID_StatusLabel("Status");


void SMyListRow::Construct(const FArguments& InArgs, const TSharedRef<STableViewBase>& InOwnerTableView)
{
	this->Item = InArgs._Item;
	this->ListView = InArgs._ListView;
	this->OnGetFilteredTextDelegate = InArgs._OnGetFilteredText;

	check(this->Item.IsValid());

	SMultiColumnTableRow<FMyListRowInfoPtr>::Construct(
		FSuperRowType::FArguments()
		.Style(FEditorStyle::Get(), "TableView.DarkRow"),
		InOwnerTableView);
}

TSharedRef<SWidget> SMyListRow::GenerateWidgetForColumn(const FName& ColumnName)
{
	TSharedPtr< SInlineEditableTextBlock > InlineWidget;
	TSharedRef< SWidget > NewWidget =
		SNew(SVerticalBox)
		+ SVerticalBox::Slot()
		.AutoHeight()
		.Padding(4.0f, 4.0f)
		.VAlign(VAlign_Center)
		[
			SAssignNew(InlineWidget, SInlineEditableTextBlock)
			.HighlightText(this, &SMyListRow::GetFilterText)
			.IsReadOnly(true)
			.IsSelected(this, &SMultiColumnTableRow<FMyListRowInfoPtr>::IsSelectedExclusively)
		];

	// set color
	if (this->Item->GetStatus() == EStatusMode::Done)
	{
		InlineWidget->SetColorAndOpacity(FLinearColor::Green);
	}
	if (this->Item->GetStatus() == EStatusMode::Failed)
	{
		InlineWidget->SetColorAndOpacity(FLinearColor::Red);
	}
	if (this->Item->GetStatus() == EStatusMode::Converting)
	{
		InlineWidget->SetColorAndOpacity(FLinearColor::Yellow);
	}
	if (this->Item->GetStatus() == EStatusMode::Cancelled)
	{
		InlineWidget->SetColorAndOpacity(FLinearColor::Blue);
	}


	// set text
	if (ColumnName == ColumnId_NumberLabel)
	{
  // Attribute를 이용하면 값만 바뀌는 상황에서 ui를 다시 생성하지 않고,
  // 다시 값을 참조하여 화면에 그려줄 수 있다.
		TAttribute<FText> Attribute = TAttribute<FText>::Create(
			FOnGetText::CreateLambda(
			[this]
			{	
				return FText::FromString(FString::FromInt(this->Item->Number));
			}
		));

		InlineWidget->SetText(Attribute);

		//InlineWidget->SetText(FText::FromString(FString::FromInt(this->Item->Number)));
	}
	else
	{
		FString DisplayText;
		if (ColumnName == ColumnID_NameLabel)
		{
			DisplayText = this->Item->GetWaveFileName();
		}
		else if (ColumnName == ColumnID_ScriptLabel)
		{
			DisplayText = this->Item->GetHasScriptAsString();
		}
		else
		{
			DisplayText = this->Item->GetStatusAsString();
		}
		InlineWidget->SetText(FText::FromString(DisplayText));
	}
	return NewWidget;
}



FText SMyListRow::GetFilterText() const
{
	if (this->OnGetFilteredTextDelegate.IsBound())
	{
		return this->OnGetFilteredTextDelegate.Execute();
	}

	return FText::GetEmpty();
}


//////////////////////////////////////////////////////////////////////////
// SNFaceJobListWidget

void SMyListViewWidget::Construct(const FArguments& InArgs)
{
	this->OnRefreshListViewDelegate = InArgs._OnRefreshListView;

	TSharedPtr<SHorizontalBox> ViewOptionsHBox;

	this->ChildSlot
		[
			SNew(SVerticalBox)
			+ SVerticalBox::Slot()
			.AutoHeight()
			.Padding(5)
		[
			SNew(SHorizontalBox)
			+ SHorizontalBox::Slot()
			.AutoWidth()
			.VAlign(VAlign_Center)
			.HAlign(HAlign_Left)
		[
			CreateAddJobComboBtn()
		]
	+ SHorizontalBox::Slot()
		.AutoWidth()
		.VAlign(VAlign_Center)
		.HAlign(HAlign_Left)
		.Padding(6, 0)
		[
			CreateStartBtn()
		]
	+ SHorizontalBox::Slot()
		.VAlign(VAlign_Center)
		.Padding(40, 0, 0, 0)
		[
			CreateIPAddressEditBox()
		]
		]
	+ SVerticalBox::Slot()
		.FillHeight(1.0f)
		.Padding(5)
		[
			SNew(SVerticalBox)
			+ SVerticalBox::Slot()
		.AutoHeight()
		.Padding(0, 2)
		[
			SNew(SHorizontalBox)
			+ SHorizontalBox::Slot()
		.FillWidth(1)
		[
			CreateItemSearchBox()
		]
		]
	+ SVerticalBox::Slot()
		.FillHeight(1.0f)		// This is required to make the scrollbar work, as content overflows Slate containers by default
		.Padding(0, 5, 0, 0)
		[
			CreateListView()
		]
		]

	// bottom tool bar
	+ SVerticalBox::Slot()
		.Padding(5)
		.AutoHeight()
		[
			SAssignNew(ViewOptionsHBox, SHorizontalBox)
			// Asset count
		+ SHorizontalBox::Slot()
		.FillWidth(1.f)
		.VAlign(VAlign_Center)
		.Padding(8, 0)
		[
			SNew(STextBlock)
			.Text(this, &SMyListViewWidget::GetAssetCountText)
		]
		]
		];


	if (InArgs._ShowViewOptionsComboBtn)
	{
		// View mode combo button
		ViewOptionsHBox->AddSlot()
			.AutoWidth()
			[
				CreateViewOptionsComboBtn()
			];
	}

	RefreshListView();
}



void SMyListViewWidget::RefreshListView()
{
	this->OnRefreshListViewDelegate.ExecuteIfBound(this->FilterListRowText.ToString(), this->DataList);

	this->ListView->RequestListRefresh();
}

FText SMyListViewWidget::GetAssetCountText() const
{
	const int32 NumAssets = this->DataList.Num();

	FText AssetCount = FText::GetEmpty();

	AssetCount = FText::Format(
		LOCTEXT("AssetCountLabelPluralPlusSelection", "{0} items"),
		FText::AsNumber(NumAssets));

	return AssetCount;
}


//------------------------------------- Add Job Combo Button --------------------------------------//
TSharedRef<SWidget> SMyListViewWidget::CreateAddJobComboBtn() const
{
	return SNew(SComboButton)
		.ComboButtonStyle(FEditorStyle::Get(), "ToolbarComboButton")
		.ButtonStyle(FEditorStyle::Get(), "FlatButton.Success")
		.ForegroundColor(FLinearColor::White)
		.ContentPadding(FMargin(6, 2))
		.OnGetMenuContent(this, &SMyListViewWidget::OnGetAddJobBtnMenuContent)
		//.ToolTipText(this, &SContentBrowser::GetAddNewToolTipText)
		.AddMetaData<FTagMetaData>(FTagMetaData(TEXT("ContentBrowserNewAsset")))
		.HasDownArrow(false)
		.ButtonContent()
		[
			SNew(SHorizontalBox)
			// New Icon
		+ SHorizontalBox::Slot()
		.VAlign(VAlign_Center)
		.AutoWidth()
		[
			SNew(STextBlock)
			.TextStyle(FEditorStyle::Get(), "ContentBrowser.TopBar.Font")
		.Font(FEditorStyle::Get().GetFontStyle("FontAwesome.11"))
		.Text(FEditorFontGlyphs::File)
		]

	// New Text
	+ SHorizontalBox::Slot()
		.AutoWidth()
		.VAlign(VAlign_Center)
		.Padding(4, 0, 0, 0)
		[
			SNew(STextBlock)
			.TextStyle(FEditorStyle::Get(), "ContentBrowser.TopBar.Font")
		.Text(LOCTEXT("AddJobButton", "Add Job"))
		]

	// Down Arrow
	+ SHorizontalBox::Slot()
		.VAlign(VAlign_Center)
		.AutoWidth()
		.Padding(4, 0, 0, 0)
		[
			SNew(STextBlock)
			.TextStyle(FEditorStyle::Get(), "ContentBrowser.TopBar.Font")
		.Font(FEditorStyle::Get().GetFontStyle("FontAwesome.10"))
		.Text(FEditorFontGlyphs::Caret_Down)
		]
		];
}


TSharedRef<SWidget> SMyListViewWidget::OnGetAddJobBtnMenuContent() const
{
	TArray<TSharedPtr<FExtender>> Extenders;
	TSharedPtr<FExtender> MenuExtender = FExtender::Combine(Extenders);
	FMenuBuilder MenuBuilder(true, NULL, MenuExtender, true);

	MenuBuilder.BeginSection("ConversionSection", LOCTEXT("ConversionText", "Conversion"));
	{
		MenuBuilder.AddMenuEntry(
			LOCTEXT("FromAudioOption", "From Audio (.wav)"),
			LOCTEXT("FromAudioOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteFromAudioAction)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);
	}
	MenuBuilder.EndSection();

	return MenuBuilder.MakeWidget();
}

void SMyListViewWidget::OnExecuteFromAudioAction()
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnExecuteFromAudioAction ####"));

}

//-------------------------------------------------------------------------------------------------//




//----------------------------------------- Start Button ------------------------------------------//

TSharedRef<SWidget> SMyListViewWidget::CreateStartBtn()
{
	return SNew(SButton)
		.ButtonStyle(FEditorStyle::Get(), "FlatButton")
		//.ToolTipText(this, &SContentBrowser::GetImportTooltipText)
		.OnClicked(this, &SMyListViewWidget::OnStartButtonClicked)
		.ContentPadding(FMargin(6, 2))
		.AddMetaData<FTagMetaData>(FTagMetaData(TEXT("ContentBrowserImportAsset")))
		[
			SNew(SHorizontalBox)

			// Import Icon
		+ SHorizontalBox::Slot()
		.VAlign(VAlign_Center)
		.AutoWidth()
		[
			SNew(STextBlock)
			.TextStyle(FEditorStyle::Get(), "ContentBrowser.TopBar.Font")
		.Font(FEditorStyle::Get().GetFontStyle("FontAwesome.11"))
		.Text(FEditorFontGlyphs::Download)
		]
	// Import Text
	+ SHorizontalBox::Slot()
		.AutoWidth()
		.VAlign(VAlign_Center)
		.Padding(4, 0, 0, 0)
		[
			SAssignNew(StartBtnText, STextBlock)
			.TextStyle(FEditorStyle::Get(), "ContentBrowser.TopBar.Font")
		.Text(LOCTEXT("StartButton", "Start"))
		]
		];
}

FReply SMyListViewWidget::OnStartButtonClicked()
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnStartButtonClicked ####"));

	FMessageDialog::Open(
		EAppMsgType::Ok,
		LOCTEXT("Warning Message", "Write Your Message!!"));

	return FReply::Handled();
}

// 	LOCTEXT("StopButton", "Stop")
// 	LOCTEXT("StartButton", "Start")
void SMyListViewWidget::SetStartButtonText(const FText& Text)
{
	StartBtnText->SetText(Text);
}

//-------------------------------------------------------------------------------------------------//



//-------------------------------------- IP Address EditBox ---------------------------------------//

TSharedRef<SWidget> SMyListViewWidget::CreateIPAddressEditBox() const
{
	return SNew(SEditableTextBox)
		.HintText(LOCTEXT("IPTextBox", "Server IP Address"))
		.OnTextChanged(this, &SMyListViewWidget::OnTextChanged)
		.OnTextCommitted(this, &SMyListViewWidget::OnTextCommitted)
		;
}


void SMyListViewWidget::OnTextChanged(const FText& NewText)
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnTextChanged ####  %s"), *NewText.ToString());
}


void SMyListViewWidget::OnTextCommitted(const FText& NewText, ETextCommit::Type CommitType)
{
	UE_LOG(LogTemp, Warning, TEXT("#### OnTextCommitted ####  %s  %d"), *NewText.ToString(), CommitType);
}

//-------------------------------------------------------------------------------------------------//



//---------------------------------------- Item SearchBox -----------------------------------------//
TSharedRef<SWidget> SMyListViewWidget::CreateItemSearchBox()
{
	return SAssignNew(this->NameFilterSearchBox, SSearchBox)
		.SelectAllTextWhenFocused(true)
		.OnTextChanged(this, &SMyListViewWidget::OnItemSearchBoxTextChanged)
		.OnTextCommitted(this, &SMyListViewWidget::OnItemSearchBoxTextCommitted);
}

void SMyListViewWidget::OnItemSearchBoxTextChanged(const FText& SearchText)
{
	// need to make sure not to have the same text go
	// otherwise, the widget gets recreated multiple times causing 
	// other issue
	if (this->FilterListRowText.CompareToCaseIgnored(SearchText) != 0)
	{
		this->FilterListRowText = SearchText;
		RefreshListView();
	}
}

void SMyListViewWidget::OnItemSearchBoxTextCommitted(const FText& SearchText, ETextCommit::Type CommitInfo)
{
	// Just do the same as if the user typed in the box
	OnItemSearchBoxTextChanged(SearchText);
}

//--------------------------------------------------------------------------------------------------//



//------------------------------------------- List View --------------------------------------------//


TSharedRef<SWidget> SMyListViewWidget::CreateListView()
{
	return SAssignNew(this->ListView, SMyListView)
		.OnGenerateRow(this, &SMyListViewWidget::GenerateListRow)
		.ListItemsSource(&this->DataList)
		.OnContextMenuOpening(this, &SMyListViewWidget::OnContextMenuOpening)
		.ItemHeight(24)
		.Visibility(EVisibility::Visible)
		.HeaderRow(
			SNew(SHeaderRow)
			+ SHeaderRow::Column(ColumnId_NumberLabel)
			.DefaultLabel(LOCTEXT("MyFaceConvert_NumberLabel", "No"))
			.ManualWidth(50.f)

			+ SHeaderRow::Column(ColumnID_NameLabel)
			.DefaultLabel(LOCTEXT("MyFaceConvert_NameLabel", "Name"))
			.ManualWidth(100.f)

			+ SHeaderRow::Column(ColumnID_ScriptLabel)
			.DefaultLabel(LOCTEXT("MyFaceConvert_ScriptLabel", "HasScript"))
			.ManualWidth(100.f)

			+ SHeaderRow::Column(ColumnID_StatusLabel)
			.DefaultLabel(LOCTEXT("MyFaceConvert_StatusLabel", "Status"))
		);
}

TSharedRef<ITableRow> SMyListViewWidget::GenerateListRow(
	FMyListRowInfoPtr InInfo,
	const TSharedRef<STableViewBase>& OwnerTable)
{
	check(InInfo.IsValid());

	return SNew(SMyListRow, OwnerTable)
		.Item(InInfo)
		.ListView(ListView)
		.OnGetFilteredText(this, &SMyListViewWidget::GetFilterListRowText)
		;
}

TSharedPtr<SWidget> SMyListViewWidget::OnContextMenuOpening()
{
	TArray<TSharedPtr<FExtender>> Extenders;
	TSharedPtr<FExtender> MenuExtender = FExtender::Combine(Extenders);
	FMenuBuilder MenuBuilder(true, NULL, MenuExtender, true);

	MenuBuilder.BeginSection("JobListSection", LOCTEXT("JobListText", "Job List"));
	{
		MenuBuilder.AddMenuEntry(
			LOCTEXT("RemoveOption", "Remove from List"),
			LOCTEXT("RemoveOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteRemove)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);
	}
	MenuBuilder.EndSection();


	MenuBuilder.BeginSection("OutputSection", LOCTEXT("OutputText", "Output"));
	{
		MenuBuilder.AddMenuEntry(
			LOCTEXT("LocateAssetOption", "Locate Asset in Content Browser"),
			LOCTEXT("LocateAssetOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteLocateAsset)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);

		MenuBuilder.AddMenuEntry(
			LOCTEXT("OpenAnimSeqOption", "Open AnimSequence"),
			LOCTEXT("OpenAnimSeqOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteOpenAnimSeq)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);
	}
	MenuBuilder.EndSection();


	MenuBuilder.BeginSection("SourceSection", LOCTEXT("SourceText", "Source"));
	{
		MenuBuilder.AddMenuEntry(
			LOCTEXT("RecheckOption", "Recheck Files"),
			LOCTEXT("RecheckOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteRecheck)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);

		MenuBuilder.AddMenuEntry(
			LOCTEXT("LocateAudioOption", "Locate Audio in Explorer"),
			LOCTEXT("LocateAudioOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteLocateAudio)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);

		MenuBuilder.AddMenuEntry(
			LOCTEXT("OpenAudioOption", "Open Audio (.wav)"),
			LOCTEXT("OpenAudioOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteOpenAudio)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);

		MenuBuilder.AddMenuEntry(
			LOCTEXT("OpenScriptOption", "Open Script (.txt)"),
			LOCTEXT("OpenScriptOptionToolTip", "~~~~~~~~~~~ tips here ~~~~~~~~~~~~"),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::OnExecuteOpenScript)
			),
			NAME_None,
			EUserInterfaceActionType::Button
		);
	}
	MenuBuilder.EndSection();

	return MenuBuilder.MakeWidget();
}

void SMyListViewWidget::OnExecuteRemove()
{
	//this->ListView->GetSelectedItems();

}

void SMyListViewWidget::OnExecuteLocateAsset()
{
}

void SMyListViewWidget::OnExecuteOpenAnimSeq()
{
}

void SMyListViewWidget::OnExecuteRecheck()
{
}

void SMyListViewWidget::OnExecuteLocateAudio()
{
}

void SMyListViewWidget::OnExecuteOpenAudio()
{
}

void SMyListViewWidget::OnExecuteOpenScript()
{
}

//--------------------------------------------------------------------------------------------------//



//----------------------------------- View Options Combo Button ------------------------------------//

TSharedRef<SWidget> SMyListViewWidget::CreateViewOptionsComboBtn()
{
	return SAssignNew(this->ViewOptionsComboButton, SComboButton)
		.ContentPadding(0)
		.ForegroundColor(this, &SMyListViewWidget::GetViewButtonForegroundColor)
		.ButtonStyle(FEditorStyle::Get(), "ToggleButton") // Use the tool bar item style for this button
		.OnGetMenuContent(this, &SMyListViewWidget::OnGetViewBtnMenuContent)
		.ButtonContent()
		[
			SNew(SHorizontalBox)
			+ SHorizontalBox::Slot()
		.AutoWidth()
		.VAlign(VAlign_Center)
		[
			SNew(SImage).Image(FEditorStyle::GetBrush("GenericViewButton"))
		]
	+ SHorizontalBox::Slot()
		.AutoWidth()
		.Padding(2, 0, 0, 0)
		.VAlign(VAlign_Center)
		[
			SNew(STextBlock).Text(LOCTEXT("ViewButton", "View Options"))
		]
		];
}

FSlateColor SMyListViewWidget::GetViewButtonForegroundColor() const
{
	static const FName InvertedForegroundName("InvertedForeground");
	static const FName DefaultForegroundName("DefaultForeground");

	return this->ViewOptionsComboButton->IsHovered() ?
		FEditorStyle::GetSlateColor(InvertedForegroundName) : FEditorStyle::GetSlateColor(DefaultForegroundName);
}

TSharedRef<SWidget> SMyListViewWidget::OnGetViewBtnMenuContent() const
{
	TArray<TSharedPtr<FExtender>> Extenders;

	TSharedPtr<FExtender> MenuExtender = FExtender::Combine(Extenders);

	FMenuBuilder MenuBuilder(true, NULL, MenuExtender, true);

	MenuBuilder.BeginSection("AssetViewType", LOCTEXT("ViewTypeHeading", "View Type"));
	{
		MenuBuilder.AddMenuEntry(
			LOCTEXT("TileViewOption", "Tiles"),
			LOCTEXT("TileViewOptionToolTip", "View assets as tiles in a grid."),
			FSlateIcon(),
			FUIAction(
				FExecuteAction::CreateSP(this, &SMyListViewWidget::SetCurrentViewTypeFromMenu, 0),
				FCanExecuteAction(),
				FIsActionChecked::CreateSP(this, &SMyListViewWidget::IsCurrentViewType, 0)
			),
			NAME_None,
			EUserInterfaceActionType::RadioButton
		);
	}
	MenuBuilder.EndSection();

	return MenuBuilder.MakeWidget();
}

void SMyListViewWidget::SetCurrentViewTypeFromMenu(int NewType)
{
	UE_LOG(LogTemp, Warning, TEXT("#### SetCurrentViewTypeFromMenu ####"));
}

bool SMyListViewWidget::IsCurrentViewType(int ViewType) const
{
	UE_LOG(LogTemp, Warning, TEXT("#### IsCurrentViewType ####"));
	return true;
}

//--------------------------------------------------------------------------------------------------//


#undef LOCTEXT_NAMESPACE



#### MyPlugin.h
```c++
#pragma once

#include "CoreMinimal.h"
#include "Modules/ModuleManager.h"
#include "SMyListViewWidget.h"

class FToolBarBuilder;
class FMenuBuilder;

class FMyPluginModule : public IModuleInterface
{
public:

	/** IModuleInterface implementation */
	virtual void StartupModule() override;
	virtual void ShutdownModule() override;
	
	TSharedRef<SDockTab> OnSpawnPluginTab(const FSpawnTabArgs& SpawnTabArgs);

	/** This function will be bound to Command. */
	void PluginButtonClicked();
	
private:

	void AddToolbarExtension(FToolBarBuilder& Builder);
	void AddMenuExtension(FMenuBuilder& Builder);

	void InitConsoleCommand();
	void ConsoleCommand(const TArray<FString>& Args);

	void OnRefreshListView(
		const FString& FilterListRowText, TArray<TSharedPtr<FMyListRowInfo>>& DataList);

private:
	TSharedPtr<class FUICommandList> PluginCommands;
	TArray<FMyListRowInfoPtr> RealData;
	TSharedPtr<SMyListViewWidget> ListView;
};

```



#### MyPlugin.cpp
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
#include "Widgets/Docking/SDockTab.h"
#include "SMyListViewWidget.h"

static const FName MyPluginTabName("MyPlugin");

#define LOCTEXT_NAMESPACE "FMyPluginModule"


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

	FGlobalTabmanager::Get()->RegisterNomadTabSpawner(
		FName("Test"), 
		FOnSpawnTab::CreateRaw(this, &FMyPluginModule::OnSpawnPluginTab))
		.SetDisplayName(LOCTEXT("FNFaceTabTitle", "NFace"))
		.SetMenuType(ETabSpawnerMenuType::Hidden);

		
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


	RealData = {
		FMyListRowInfo::Make(1, FString(TEXT("file1.txt")), true, EStatusMode::Done),
		FMyListRowInfo::Make(2, FString(TEXT("file2.txt")), false, EStatusMode::Failed),
		FMyListRowInfo::Make(3, FString(TEXT("file3.txt")), true, EStatusMode::Converting),
		FMyListRowInfo::Make(4, FString(TEXT("file4.txt")), true, EStatusMode::Cancelled),
		FMyListRowInfo::Make(5, FString(TEXT("file5.txt")), true, EStatusMode::Ready),
		FMyListRowInfo::Make(6, FString(TEXT("file6.txt")), true, EStatusMode::Done),
		FMyListRowInfo::Make(7, FString(TEXT("file7.wav")), true, EStatusMode::Ready),
		FMyListRowInfo::Make(1, FString(TEXT("file1.wav")), true, EStatusMode::Done),
		FMyListRowInfo::Make(2, FString(TEXT("file2.wav")), false, EStatusMode::Failed),
		FMyListRowInfo::Make(3, FString(TEXT("file3.wav")), true, EStatusMode::Converting),
		FMyListRowInfo::Make(4, FString(TEXT("file4.wav")), true, EStatusMode::Cancelled),
		FMyListRowInfo::Make(5, FString(TEXT("file5.wav")), true, EStatusMode::Ready),
		FMyListRowInfo::Make(6, FString(TEXT("file6.wav")), true, EStatusMode::Done),
		FMyListRowInfo::Make(7, FString(TEXT("file7.wav")), true, EStatusMode::Ready),
		FMyListRowInfo::Make(1, FString(TEXT("file1.wav")), true, EStatusMode::Done),
		FMyListRowInfo::Make(2, FString(TEXT("file2.wav")), false, EStatusMode::Failed),
		FMyListRowInfo::Make(3, FString(TEXT("file3.wav")), true, EStatusMode::Converting),
		FMyListRowInfo::Make(4, FString(TEXT("file4.wav")), true, EStatusMode::Cancelled),
		FMyListRowInfo::Make(5, FString(TEXT("file5.wav")), true, EStatusMode::Ready),
		FMyListRowInfo::Make(6, FString(TEXT("file6.wav")), true, EStatusMode::Done),
		FMyListRowInfo::Make(7, FString(TEXT("file7.wav")), true, EStatusMode::Ready),
	};

	InitConsoleCommand();
}

void FMyPluginModule::ShutdownModule()
{
	// This function may be called during shutdown to clean up your module.  For modules that support dynamic reloading,
	// we call this function before unloading the module.
	FMyPluginStyle::Shutdown();

	FMyPluginCommands::Unregister();
}

TSharedRef<SDockTab> FMyPluginModule::OnSpawnPluginTab(const FSpawnTabArgs& SpawnTabArgs)
{
	return SNew(SDockTab)
		.TabRole(ETabRole::NomadTab)
		[
			// Put your tab content here!
			SAssignNew(ListView, SMyListViewWidget)
			.OnRefreshListView(FOnRefreshListView::CreateRaw(this, &FMyPluginModule::OnRefreshListView))
		];
}


void FMyPluginModule::PluginButtonClicked()
{
	FGlobalTabmanager::Get()->InvokeTab(FName("Test"));
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
	for (auto& Data : this->RealData)
	{
		Data->WaveFileName = FString(TEXT("Modified"));
	}

	ListView->RefreshListView();
}


void FMyPluginModule::OnRefreshListView(
	const FString& FilterListRowText, TArray<TSharedPtr<FMyListRowInfo>>& DataList)
{
	FString SearchText = FilterListRowText;

	DataList.Empty();

	bool bDoFiltering = !SearchText.IsEmpty();

	for (const auto Data : RealData)
	{
		const int Number = Data->GetNumber();
		const FString WaveFileName = Data->GetWaveFileName();
		const bool HasScript = Data->GetHasScript();
		const EStatusMode Status = Data->GetStatus();

		const FString HasScriptAsString = Data->GetHasScriptAsString();
		const FString StatusAsString = Data->GetStatusAsString();

		if (bDoFiltering)
		{
			// make sure it doens't fit any of them
			if (!WaveFileName.Contains(SearchText) &&
				!StatusAsString.Contains(SearchText) &&
				!HasScriptAsString.Contains(SearchText))
			{
				continue; // Skip items that don't match our filter
			}
		}

		FMyListRowInfoPtr Info = FMyListRowInfo::Make(Number, WaveFileName, HasScript, Status);

		DataList.Add(Info);
	}

	// Sort
	DataList.Sort([](const FMyListRowInfoPtr& A, const FMyListRowInfoPtr& B) {
		return A->GetNumber() < B->GetNumber();
	});

}

#undef LOCTEXT_NAMESPACE
	
IMPLEMENT_MODULE(FMyPluginModule, MyPlugin)
```

