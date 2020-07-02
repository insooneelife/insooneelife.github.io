---
title: "[Unity] Runtime NavMesh Baking"
categories:
  - Runtime
  - NavMesh
  - NavMeshComponents
---

#### Runtime NavMesh Baking
NavMesh를 런타임에 생성해야 하는 상황이 생길 수 있다.
맵 자체를 제작하는 크래프트류의 게임들은 게임 중에 갑자기 새로운 맵이 생성될 수 있고,
그에 따라 NavMesh로 새로 생성되어야 할 수 있다.
Unity 엔진 내부에서 이런 상황을 위해 제공해주는 기능은 없지만, 
유니티에서 오픈소스로 제공해주는 코드가 있다. 
https://github.com/Unity-Technologies/NavMeshComponents

먼저 Download ZIP 으로  이 프로젝트를 받는다.
그 후 프로젝트에 이 코드를 포함시킨다.
NavMeshComponents-master\NavMeshComponents-master\Assets\NavMeshComponents 폴더를 해당 프로젝트에 Drag & Drop 하면 된다.
그 후 NavMesh를 위한 Floor로 사용할 GameObject에 NavMeshSurface 컴포넌트를 붙인다.

나는 기본 Plane을 사용하였다.
이 GameObject를 Prefab으로 만든다.
![image-center](/assets/images/unity-editor-customize-script-position.png){: .align-left}
이제 이 GameObject를 생성시킬 스크립트를 작성한다.



#### MapManager.cs
BulidNavMesh 함수를 통해 런타임에 NavMesh를 생성시킬 수 있다.
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class MapManager : MonoBehaviour
{
	[SerializeField]
	private GameObject _mapPrefab;

	private Vector3 _generatePos = new Vector3(50, 0, 50);

	public void Init()
	{
	}

    private void Awake()
	{
		GenerateNavmesh();
	}

	private void GenerateNavmesh()
	{
		GameObject obj = Instantiate(_mapPrefab, _generatePos, Quaternion.identity, transform);
		_generatePos += new Vector3(50, 0, 50);

		NavMeshSurface[] surfaces = gameObject.GetComponentsInChildren<NavMeshSurface>();

		foreach (var s in surfaces)
		{
			s.RemoveData();
			s.BuildNavMesh();
		}
		
	}

	// Update is called once per frame
	void Update()
    {
        if(Input.GetKeyDown(KeyCode.P))
		{
			GenerateNavmesh();
		}
    }
}
```

코드 작성 후 inspector에서 _mapPrefab에 Plane Prefab을 넣는다.

![image-center](/assets/images/unity-editor-customize-script-position.png){: .align-left}


코드 실행 후 P 키를 누르면 실시간으로 NavMesh가 생성됨을 확인할 수 있다.

