# 单用户模式

记录一些遇到需要进入单用户模式解决问题的场景

- 案例一

```go
//在CentOS 7.6上不小心卸载了Fish shell(fish)并且现在无法通过SSH登录到您的系统，因为默认shell被设置为Fish,可通过下面方法恢复；
chsh -s /bin/bash your_username
exec /sbin/init
```

<img src="./assets/image-20260810121426647.png" alt="image-20260810121426647" style="zoom:50%;" />

