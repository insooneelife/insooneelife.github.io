
```c#

public static T CreateData<T>(string[] result) where T : class
{
  Type dataType = typeof(T);

  ConstructorInfo[] cons = dataType.GetConstructors();
  foreach (var c in cons)
  {
    if (c.IsDefined(typeof(MainConstructorAttribute)))
    {
      ParameterInfo[] parameters = c.GetParameters();
      object[] args = new object[parameters.Length];
      for (int i = 0; i < parameters.Length; ++i)
      {
        ParameterInfo p = parameters[i];
        string r = result[i];

        if (p.ParameterType.IsEnum)
        {
          args[i] = Enum.Parse(p.ParameterType, r);
        }
        else
        {
          args[i] = Convert.ChangeType(r, p.ParameterType);
        }
      }

      return (T)Activator.CreateInstance(dataType, args);
    }
  }

  return null;
}

```
