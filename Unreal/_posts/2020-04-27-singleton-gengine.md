---
title: "[Unreal] Singleton GEngine"
categories:
  - Singleton
---

### Console Command
Unreal engine GEngine 싱글턴 객체를 통해 World를 참조하는 예제이다.


```c++
UWorld* World = GEngine->GetWorldContexts()[0].World();
```
