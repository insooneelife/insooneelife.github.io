---
title: "[C#] Reflection Data Logging"
categories:
  - Reflection
---

Reflection의 대한 얘기를 하기 전에 왜 reflection이 필요하게 되었는지에 대해 이해할 필요가 있다.

다음과 같은 상황을 생각해보자.

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
}
```
