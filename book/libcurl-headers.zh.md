
## 头文件

应用程序需要包含的libcurl只有一个头:

```
#include <curl/curl.h>
```

该文件又包含了一些其他的公共头文件,但基本上可以假设它们不存在.(从历史角度来说,我们开始时略有不同,但随着时间的推移,我们已经稳定下来,只使用单一形式作为包含.)