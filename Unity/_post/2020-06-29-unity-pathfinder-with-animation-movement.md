---
title: "[Unity] Pathfinder with Animation Movement"
categories:
  - Pathfinding
  - Animation
  - Transform
  - Move
---

#### Pathfinder with Animation Movement
기본적으로 Unity에서는 NavMeshAgent를 지원한다. 
그리고 이 컴포넌트를 통해서 굉장히 쉽게 pathfinding이 가능한 오브젝트를 제작할 수 있다.
하지만, 기본적으로 NavMeshAgent는 GameObject의 Transform을 엔진 코드 내에서 직접 수정하도록 설계되어 있다.

SetDestination 함수를 이용하면 원하는 위치를 줬을 때 Agent가 직접 경로에 맞게 Transform을 이동시키는 것을 확인 할 수 있다.
이것은 Animation 자체에 Transform 이동이 포함되어 있는 경우에 생긴다. 
예를 들어 FPS 게임에서 사람의 걷는 모션을 리얼하게 표현하기 위해, 보폭에 따른 위치의 이동을 Animation 에셋 수준에서 설정했다고 가정해 보자. 이런 에셋을 이용하여 길찾기 에이전트를 제작하려 한다면, Animation과 NavMeshAgent가 서로 Transform을 조작하려 할 것이다. 
이런 상황을 해결해보자.

먼저 NavMeshAgent에는 CalcCalculatePath라는 목표 위치까지의 경로만 구해주는 함수가 있다.
이를 이용하면 된다.

#### PathFinder.cs
athFinder 클래스는 NavMeshAgent 컴포넌트를 경로를 구하기 위해서만 사용하고, 직접 이동시키는 부분만 열려 있게 설계한다. 경로를 직접 이동시키는 부분은 이벤트로 만들어 외부로부터 받도록 한다.
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;
using UnityEngine.Events;
using UnityEngine.UI;
using System;

public class Pathfinder : MonoBehaviour
{
	[SerializeField]
	private NavMeshAgent _agent;

	public Action<Vector3> onMove;
	public Action onStop;

	private NavMeshPath _path = null;

	private Vector3 _currentDest = new Vector3();

	private int _cornerIdx = 0;

	public void SetDestination(Vector3 dest)
	{
		Clear();
		_agent.CalculatePath(dest, _path);
	}

	private void Awake()
	{
		_path = new NavMeshPath();
	}
	
    void Update()
    {
		if (_path.corners.Length == _cornerIdx)
		{
			Clear();
			if (onStop != null)
			{
				onStop();
			}
		}

		if (_path.status == NavMeshPathStatus.PathComplete)
		{
			_currentDest = _path.corners[_cornerIdx];

			if (HasArriveDest())
			{
				_cornerIdx++;
			}
			else
			{
				Move();
			}
		}
	}

	private bool HasArriveDest()
	{
		if ((_currentDest - transform.position).magnitude < 0.5f)
		{
			return true;
		}
		return false;
	}

	private void Move()
	{
		if (onMove != null)
		{
			Vector3 toTarget = (_currentDest - transform.position).normalized;
			onMove(toTarget);
		}
	}

	private void Clear()
	{
		_cornerIdx = 0;
		_path.ClearCorners();
	}
}
```

#### PlayerCharacter.cs
Pathfinder 클래스를 직접 사용해주는 부분이다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class PlayerCharacter : MonoBehaviour
{
	[SerializeField]
	private NavMeshAgent _agent;

	[SerializeField]
	private MyAnimController _myAnimController;

	[SerializeField]
	private Pathfinder _pathfinder;

	void Awake()
	{
		_pathfinder.onMove = _myAnimController.Move;
		_pathfinder.onStop = _myAnimController.StopMove;
	}

	private void Update()
	{
		if (Input.GetMouseButtonDown(0))
		{
			RaycastHit hit;
			Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
			if (Physics.Raycast(ray, out hit))
			{
				Vector3 pos = hit.point;
				_pathfinder.SetDestination(pos);
			}
		}
	}
	
}
```


#### MyAnimController.cs
애니메이션을 통해 이동시키는 부분이 구현되어 있다.
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MyAnimController : MonoBehaviour
{
	[SerializeField]
	protected Animator _animator;


	[SerializeField]
	protected float _roateSpeed = 10;

	protected Dictionary<string, float> _clipTimes = new Dictionary<string, float>();

	protected void Start()
	{
		SaveAllClipTimes();
	}

	protected void Hit() {}

	protected void FootL() {}

	protected void FootR() {}

	protected void Attack()
	{
		_animator.SetTrigger("AttackTrigger");
		Debug.Log("Attacked!!");
	}

	public void Move(Vector3 toTarget)
	{
		_animator.SetBool("Moving", true);
		Quaternion targetRotation = Quaternion.LookRotation(toTarget);
		transform.rotation =
			Quaternion.Slerp(transform.rotation, targetRotation, Time.deltaTime * _roateSpeed);
	}

	public void StopMove()
	{
		_animator.SetBool("Moving", false);
	}


	protected void SaveAllClipTimes()
	{
		RuntimeAnimatorController ac = _animator.runtimeAnimatorController;    //Get Animator controller
		for (int i = 0; i < ac.animationClips.Length; i++)                 //For all animations
		{
			AnimationClip clip = ac.animationClips[i];
			Debug.Log(clip.name + "  " + clip.length);

			_clipTimes.Add(clip.name, clip.length);
		}
	}
}
```

#### Animation Controller
컨트롤러는 대충 이렇게 생겼다.
![image-center](/assets/images/unity-editor-customize-script-position.png){: .align-left}


#### 결과 화면
잘 걷는다.
![image-center](/assets/images/unity-editor-customize-script-position.png){: .align-left}


