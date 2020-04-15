---
title: "[C#] Reflection Data Logging"
categories:
  - Reflection
---

## Reflection
Reflection의 대한 얘기를 하기 전에 왜 reflection이 필요하게 되었는지에 대해 이해할 필요가 있다.

#### 문제상황
다음과 같은 상황을 생각해보자.
파일로부터 data를 읽어서 CharacterLoadData라는 클래스의 인스턴스를 생성하는 코드를 작성하려 한다.

```c#
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
      _y = y;
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

```c#
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
```
[CharacterLoadData] insooneelife     50       100      Gun          Sword        Spear       
[CharacterLoadData] enemy1           -50      0        Spear        Spear        Spear       
[CharacterLoadData] enemy2           0        -50      Sword        Sword        Sword 
```

여기까지만 보면 딱히 문제가 없는 것 같다.
하지만 저런 형태의 data class 종류가 많아지게 된다면 어떻게 될까??

```c#
public class ImageLoadData
...
public class ItemLoadData
...
public class MapLoadData
...

```
새로운 클래스가 추가될 때마다 로깅을 위한 로직보다도 많은 양의 코드를 반복적으로 작성해야한다.

#### 해결
그냥 정의만 하면 "알아서" 로깅을 할수는 없는것인가??
여기서부터 reflection의 영역이다.

이 작업을 하려면 먼저 클래스에서 어떤 멤버변수나 프로퍼티 등을 가지고 있는지 알아야 한다.
즉 현재까지 우리가 type으로만 사용하던 class를 표현하는 meta data에 접근할 수 있어야 한다.

```c#
using System.Reflection;
```

```c#
public static string LogReflection(object data)
{
  Type type = data.GetType();
  PropertyInfo[] properties = type.GetProperties();
  string logText = "";

  foreach (var p in properties)
  {
    object val = p.GetValue(data);
    logText += $"{String.Format("{0, -13}", val.ToString())} ";
  }
  return logText;
}
```

```c#
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
  Logger.Log($"[CharacterLoadData] {Utils.LogReflection(data)}");

  return data;
}
```

```
[CharacterLoadData] insooneelife  50            100             System.Collections.Generic.List`1[System.String] 
[CharacterLoadData] enemy1        -50           0             System.Collections.Generic.List`1[System.String] 
[CharacterLoadData] enemy2        0             -50             System.Collections.Generic.List`1[System.String]  
```

하지만 여전히 문제는 남아있다.
1. 모든 프로퍼티를 참조하기 때문에, 원하지 않는 프로퍼티라도 로깅의 대상이 될 수 있다.
2. 출력 string의 format이 "{0, -13}"으로 고정되어있다.
3. Collection에 대한 처리가 수행되지 않았다.

세 가지 문제를 해결하기 위해 다음 것들이 필요하다.
1. 로깅 대상이 될 프로퍼티를 특정할 수 있어야 한다.
2. 로깅 대상이 될 프로퍼티에는 Format에 대한 추가 정보가 있어야 한다.
3. 로깅 대상이 컬렉션인 경우 로깅 방법을 바꿔야 한다.

로깅 대상이 될 프로퍼티에 특별한 추가 정보를 주기 위해서,
c#에는 Attribute라는 녀석을 사용한다.

```c#
public class LogDataAttribute : System.Attribute
{
  private string _format;
  public string Format { get { return _format; } }
  public LogDataAttribute(int paddingStart = 0, int paddingEnd = -20)
  {
    _format = $"{{{paddingStart}, {paddingEnd}}}";
  }
}
```

```c#
public class CharacterLoadData
{
  private string _name;
  private int _x;
  private int _y;
  private List<string> _items;

  // Logging의 padding을 위한 meta data를 Attribute를 통해 추가 
  [LogDataAttribute(0, -16)]
  public string Name { get { return _name; } }

  [LogDataAttribute(0, -8)]
  public int X { get { return _x; } }

  [LogDataAttribute(0, -8)]
  public int Y { get { return _y; } }

  [LogDataAttribute(0, -12)]
  public List<string> Items { get { return _items; } }
  ...
}
```

```c#
public static string LogReflection(object data)
{
  Type type = data.GetType();
  PropertyInfo[] properties = type.GetProperties();
  string logText = "";

  foreach (var p in properties)
  {
    if (Attribute.IsDefined(p, typeof(LogDataAttribute)))
    {
      LogDataAttribute logAtt = 
        (LogDataAttribute)p.GetCustomAttribute(typeof(LogDataAttribute), false);

      object val = p.GetValue(data);
      // LogDataAttribute를 통해 format의 padding의 대한 데이터를 가져온다.
      logText += $"{String.Format(logAtt.Format, val.ToString())} ";
    }
  }
  return logText;
}
```

다시 예쁘게 출력되었다.
```
[CharacterLoadData] insooneelife     50       100        System.Collections.Generic.List`1[System.String] 
[CharacterLoadData] enemy1           -50      0        System.Collections.Generic.List`1[System.String] 
[CharacterLoadData] enemy2           0        -50        System.Collections.Generic.List`1[System.String] 
```

이제 컬렉션을 처리해보자.
```c#
public static string LogReflection(object data)
{
  Type type = data.GetType();
  PropertyInfo[] properties = type.GetProperties();
  string logText = "";

  foreach (var p in properties)
  {
    if (Attribute.IsDefined(p, typeof(LogDataAttribute)))
    {
      LogDataAttribute logAtt = 
        (LogDataAttribute)p.GetCustomAttribute(typeof(LogDataAttribute), false);
      // list인 경우 예외처리
      bool isList = typeof(System.Collections.IList).IsAssignableFrom(p.PropertyType);

      if (isList)
      {
        object val = p.GetValue(data);
        var list = val as System.Collections.IList;
        foreach (var d in list)
        {
          logText += $"{String.Format(logAtt.Format, d.ToString())} ";
        }
      }
      else
      {
        object val = p.GetValue(data);
        // LogDataAttribute를 통해 format의 padding의 대한 데이터를 가져온다.
        logText += $"{String.Format(logAtt.Format, val.ToString())} ";
      }

    }
  }
  return logText;
}
```

```
[CharacterLoadData] insooneelife     50       100      Gun          Sword        Spear        
[CharacterLoadData] enemy1           -50      0        Spear        Spear        Spear        
[CharacterLoadData] enemy2           0        -50      Sword        Sword        Sword      
```

List인 경우 예외적으로 로깅하도록 하였다.
Dictionary인 경우는 비슷한 방법으로 따로 처리해야 할 것이다.

## 평가
Reflection을 활용하면 위와 같은 생각지도 못한 부분들조차 자동화시킬 수 있다.
실제로 Create함수의 파일로부터 데이터를 읽어오는 부분도 비슷한 방법으로 일반화시킬 수 있고,
또한 테스팅, 전처리, 코드를 통한 코드 생성까지도 가능하다.
엄청난 개념이기 때문에 알아두면 두고두고 유용하게 사용될 것이다.


## 심화된 문제상황
모든 문제가 해결된 것 같지만 아직 문제가 더 남아있다.
위와 같은 여러 종류의 LoadData들의 공통적으로 사용되는 부분을 재사용 하기 위해, 
LoadData라는 base class를 만들어 상속받도록 하는 경우를 생각해보자.

```c#
public class LoadData
{
  private string _name;
  [LogDataAttribute(0, -16)]
  public string Name { get { return _name; } }
  public LoadData(string name) { _name = name; }
}

public class CharacterLoadData : LoadData
{
  private int _x;
  private int _y;
  private List<string> _items;

  [LogDataAttribute(0, -8)]
  public int X { get { return _x; } }

  [LogDataAttribute(0, -8)]
  public int Y { get { return _y; } }

  [LogDataAttribute(0, -12)]
  public List<string> Items { get { return _items; } }

  public CharacterLoadData(string name, int x, int y, List<string> items)
    :base(name)
  {
    _x = x;
    _y = y;
    _items = items;
  }

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
    Logger.Log($"[CharacterLoadData] {Utils.LogReflection(data)}");

    return data;
  }
}
```

```
[CharacterLoadData] 50       100      Gun          Sword        Spear        insooneelife     
[CharacterLoadData] -50      0        Spear        Spear        Spear        enemy1           
[CharacterLoadData] 0        -50      Sword        Sword        Sword        enemy2     
```

이 경우 Name이 뒤에 출력되게 된다.
