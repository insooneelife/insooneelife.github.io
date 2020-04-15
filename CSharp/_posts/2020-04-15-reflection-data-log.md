---
title: "[C#] Reflection Data Logging"
categories:
  - Reflection
---

Reflection의 대한 얘기를 하기 전에 왜 reflection이 필요하게 되었는지에 대해 이해할 필요가 있다.

다음과 같은 상황을 생각해보자.
파일로부터 data를 읽어서 CharacterLoadData라는 클래스의 인스턴스를 생성하는 코드를 작성하려 한다.

```c++
public class CharacterLoadData
{
    private string _name;
    private int _x;
    private int _y;
    private List<string> _items;

    public string Name { get { return _name; } }
    public int X { get { return _x; } }
    public int Y { get { return _y; } }
    public List<string> Items { get { return _items; } }

    public CharacterLoadData(string name, int x, int y, List<string> items)
    {
      _name = name;
      _x = x;
      _y = Y;
      _items = items;
    }

    // for create instance from data
    public static CharacterLoadData Create(string[] result)
    {
      string name = result[0];
      int x = Int32.Parse(result[1]);
      int y = Int32.Parse(result[2]);

      string item1 = result[3];
      string item2 = result[4];
      string item3 = result[5];

      CharacterLoadData data = new CharacterLoadData(name, x, y, new List<string>() { item1, item2, item3 });
      return data;
    }
}
```

다음과 같이 작성할 수 있다.
하지만 개발을 하다 보니 이 로딩과정에서 데이터가 정확하게 들어갔는지 확인하고 싶어졌다.
그래서 create 함수에 다음과 같은 로깅 코드를 추가한다.

```c++
// for create instance from data
public static CharacterLoadData Create(string[] result)
{
  string name = result[0];
  int x = Int32.Parse(result[1]);
  int y = Int32.Parse(result[2]);

  string item1 = result[3];
  string item2 = result[4];
  string item3 = result[5];

  CharacterLoadData data = new CharacterLoadData(name, x, y, new List<string>() { item1, item2, item3 });

  // for logging
  string logName = String.Format("{0, -16}", name);
  string logX = String.Format("{0, -8}", x);
  string logY = String.Format("{0, -8}", y);
  string logItem1 = String.Format("{0, -12}", item1);
  string logItem2 = String.Format("{0, -12}", item2);
  string logItem3 = String.Format("{0, -12}", item3);

  Logger.Log($"[CharacterLoadData] {logName} {logX} {logY} {logItem1} {logItem2} {logItem3}");

  return data;
}
    
```

데이터도 예쁘게 출력되었다.
```c++
[CharacterLoadData] insooneelife     50       100      Gun          Sword        Spear       
[CharacterLoadData] enemy1           -50      0        Spear        Spear        Spear       
[CharacterLoadData] enemy2           0        -50      Sword        Sword        Sword 
```

여기까지만 보면 딱히 문제가 없는 것 같다.
하지만 저런 형태의 data class 종류가 많아지게 된다면 어떻게 될까??

```c++
public class ImageLoadData
...
public class ItemLoadData
...
public class MapLoadData
...

```
새로운 클래스가 추가될 때마다 로깅을 위한 로직보다도 많은 양의 코드를 반복적으로 작성해야한다.

그냥 정의만 하면 "알아서" 로깅을 할수는 없는것인가??
여기서부터 reflection이 필요하다.
