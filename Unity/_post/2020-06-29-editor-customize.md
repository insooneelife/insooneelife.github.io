---
title: "[Unity] Editor Customize"
categories:
  - Editor
  - Customize
---

#### Unity Editor Customize
유니티에서 에디터 커스터마이징 하는 방법이다.

먼저 커스터마이징 대상 스크립트(DataTable.cs)를 작성한다.

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





