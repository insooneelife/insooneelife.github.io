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
FString NewNameSuggestion = FString(TEXT("Suggestion~~"));
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
