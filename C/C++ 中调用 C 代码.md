在 `C` 的头文件中定义好函数，在 `cpp` 中调用时，需要

```c++
extern "C" {
#include "testc.h"
}
```

这样引入头文件。

