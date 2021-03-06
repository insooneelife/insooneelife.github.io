---
title: "[Unity] Editor Customize"
categories:
  - Editor
  - Customize
---

#### Unity Editor Customize
유니티에서 에디터 커스터마이징 하는 방법이다.

먼저 커스터마이징 대상 스크립트(DataTable.cs)를 작성한다.

#### DataTable.cs
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;


[Serializable]
public struct WeaponData
{
	public enum Types
	{
		Bow, Spear, Sword, Staff, TwoHandSword
	}

	[SerializeField]
	private string name;

	[SerializeField]
	private Types type;


	public WeaponData(string name, Types type)
	{
		this.name = name;
		this.type = type;
	}

}

public class DataTable : MonoBehaviour
{
	[SerializeField]
	public List<WeaponData> weaponDataList;

}
```

그리고 에디터 커스터마이즈 클래스(DataTableEditor)를 작성한다.
일반적으로 커스터마이즈 클래스는 대상이름+Editor 로 네이밍 한다.
그리고 이 클래스는 Editor라는 폴더 하위에 위치해야 한다.

![image-center](/assets/images/unity-editor-customize-script-position.png){: .align-left}



#### DataTableEditor.cs
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(DataTable))]
[CanEditMultipleObjects]
public class DataTableEditor : Editor
{
	private SerializedProperty weaponDataList;
	private SerializedProperty armorDataList;
	private DataTable dataTable;

	void OnEnable()
	{
		weaponDataList = serializedObject.FindProperty("weaponDataList");
		armorDataList = serializedObject.FindProperty("armorDataList");
		dataTable = (target as DataTable);
	}

	public override void OnInspectorGUI()
	{
		serializedObject.Update();

		GUIStyle style = EditorStyles.helpBox;
		GUILayout.BeginVertical(style);
		EditorGUILayout.PropertyField(weaponDataList, true);
		if (GUILayout.Button("Generate WeaponData"))
		{
			dataTable.weaponDataList.Add(new WeaponData("a", WeaponData.Types.Bow));
			dataTable.weaponDataList.Add(new WeaponData("b", WeaponData.Types.Spear));
			dataTable.weaponDataList.Add(new WeaponData("c", WeaponData.Types.TwoHandSword));
		}
		GUILayout.EndVertical();
		
		
		GUILayout.BeginVertical(EditorStyles.helpBox);
		EditorGUILayout.PropertyField(armorDataList, true);
		if (GUILayout.Button("Generate ArmorData"))
		{
			dataTable.armorDataList.Add(new ArmorData("aa", ArmorData.Types.Armor));
			dataTable.armorDataList.Add(new ArmorData("bb", ArmorData.Types.Shield));
			dataTable.armorDataList.Add(new ArmorData("cc", ArmorData.Types.Shield));
		}
		GUILayout.EndVertical();
		

		serializedObject.ApplyModifiedProperties();


	}

}

```

#### 결과 화면
![image-center](/assets/images/unity-editor-customize-box-layout.png){: .align-left}

