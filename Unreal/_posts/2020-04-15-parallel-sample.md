---
title: "[Unreal] Parallel"
categories:
  - Markup
  - Data
---

### Parallel

#### parallel for
Unreal engine에서 병렬계산이 필요한 경우 ParallelFor를 이용하자.

```c++
#include "Runtime/Core/Public/Async/ParallelFor.h"

...
ParallelFor(Array.Num(), [&](int32 Idx) {
    ++Array[Idx];
});
...

```

이론적으로는 코어의 개수에 비례해서 속도가 빨라지길 기대하겠지만, 
실제로는 언리얼 엔진 수준에서 이미 많은 자원을 사용하고 있기 때문에,
원하는 성능향상을 기대하기는 어렵다.
그러므로 반드시 성능 모니터링을 해보고 적용하도록 하자.
