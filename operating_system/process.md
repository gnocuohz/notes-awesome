### Linux 二进制程序格式 ELF （Executeable and Linkable Format，可执行与可链接格式）  
#### 编译过程
![](resources/operating_system/compile.jpeg)
#### 文件类型
- .o 文件（可重定位文件）
- 可执行文件
- .so 文件（共享对象）
#### 进程树
- ps -ef: 用户进程不带中括号, 内核进程带中括号
- 用户进程祖先(1号进程, systemd); 内核进程祖先(2号进程, kthreadd)
- tty ? 一般表示后台服务
