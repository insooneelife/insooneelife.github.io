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

unreal-detail-editor-customization

