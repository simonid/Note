
Linux数据流由三种状态码，可以在/usr/include/unistd.h中找到宏定义：
```
#define STDIN_FILENO    0   
#define STDOUT_FILENO   1   
#define STDERR_FILENO   2  
```
2代表stderr,输出的错误信息<br>
1代表sdtout,输出信息<br>
`&>` 则表示把符号左边的内容以符号右边的形式输出
<br>
所以`2&>1`就是把stderr做为stdout输出<br>

### >/dev/null 2>&1丢弃信息
`/dev/null`是一个黑洞文件，代表一个空设备，就输出的内容输入到里面相当于丢弃输出。<br>
结合上述可以得出：<br>
`2&>1`定义了把`stderr`做为标准的`stdout`流输出，然后`stdout`的内容全部写入`/dev/null`，也就是说被舍弃掉<br>

