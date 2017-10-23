&nbsp;&nbsp;&nbsp;&nbsp;Google voice有什么用途呢？如果你是一个常年使用国内服务的用户，那么它不适合你。相反，如果你经常用国外的软件产品，那么就会知道，很多时候国内的手机号码发送了注册信息或者说国内手机号码接收验证码都是经常出错的。下面来说一下注册方法：<br>
&nbsp;&nbsp;&nbsp;&nbsp;网上有些说法是通过新版Google voice和textnow一起完成注册，然而我试了很久却不能注册到GV，还是要通过旧版的GV网页注册。<br>
&nbsp;&nbsp;&nbsp;&nbsp;在网页上选择心仪的号码，其实最好不要选热门地区的号码，找一个冷门地方比较合适。<br>
正常点击是会一直报错的，我们需要非常手段来实现。<br>
### 方法1：Bash脚本（推荐）
这个方法可以说最简单高效，不用用户操心。脚本基本格式：
```
contains() {
string="$1"
substring="$2"
if test "${string#*$substring}" != "$string"
then
return 0    # $substring is in $string
else
return 1    # $substring is not in $string
fi
}
while true; do
result=$(...)
contains "$result" "error" || exit 0
sleep 1
done
```
另外一个版本：
```
#!/bin/bash
#请注意使用UTF-8 without BOM格式，否则可能会出错
#脚本 by Sunbread
########## Setting ##########
threads=16 # 根据自身情况调整，若不会调整则按默认值即可
########## Code ##########
echo 伪多线程脚本
for((i=0;i<$threads;++i));do while :;do
    [curl ...] >/dev/null 2>/dev/null
    # 用刚才的命令替换中括号里的内容（包括两个中括号本身）
done &
done
echo 结束脚本请使用Ctrl+C，使用其它方法可能会出现无法预料的异常，谢谢合作
wait
```
用户需要做的，就是把`result=$(...)`内的感叹号替换下面提到的内容。这部分内容需要调出控制台的`Network`选项查看。调出控制台后，点击菜单中的continute，然后`Network`就会有个`POST`字样的内容出现，右键，选中`Copy as cURL (bash)`。然后将复制的内容粘贴到上述位置。<br>
将最终内容放到vps上，注意要赋予x权限。然后执行：<br>
```
nohup sh > /dev/null 2>&1 &
```

### 方法2：js脚本
将下列脚本直接在console中运行：
```
var sleep = delay => new Promise(resolve => setTimeout(resolve, delay));
var composeClick = function x(btn) {
var rect = btn.getBoundingClientRect();
var x = Math.floor(rect.clientX + rect.width * Math.random());
var y = Math.floor(rect.clientY + rect.height * Math.random());
var screenX = Math.floor(x + window.screen.availLeft);
var screenY = Math.floor(y + window.screen.availTop);
const mousedown = new MouseEvent("mousedown", {
screenX: screenX,
screenY: screenY,
clientX: x,
clientY: y,
});
const click = new MouseEvent("click", {
screenX: screenX,
screenY: screenY,
clientX: x,
clientY: y,
});
const mouseup = new MouseEvent("mouseup", {
screenX: screenX,
screenY: screenY,
clientX: x,
clientY: y,
});
btn.dispatchEvent(mousedown);
return sleep(150 + Math.random() * 30)
.then(() => {
btn.dispatchEvent(click);
return sleep(30 + Math.random() * 30);
}).then(() => {
btn.dispatchEvent(mouseup);
});
}
function task() {
var btn = document.querySelector(".continueButton");
if (!btn) {
alert("Finish");
return;
}
composeClick(btn)
.then(() => {
//在此调整点击时间间隔
return sleep(500 + Math.random() * 3000);
})
.then(() => {
task();
});
}
task(); 
```

### 方法3：模拟器
借用一个网友的模拟鼠标点击器：
```
链接: https://pan.baidu.com/s/1nvcFcbZ 密码: snwu
来自：https://www.vpsbuluo.com/ziyuan/27.html
```

Android和iOS都可以通过安装环聊（Android还需安装环聊拨号器）来使用这个美国号码。<br>

### Google voice回收政策
据说充值不会被回收，但一定不会被回收的条件就是半年内使用Google Voice的美国号码发短信或者打电话。
这点相信还是比较容易做到的，毕竟Google Voice的美国号码拨打美国、加拿大号码是免费的，发短信也是免费的。<br>
Google在回收前，会提前发邮件给你，如果你没做出应答，回收后的30天，你仍可以找回，超过的话，只能重新申请了。<br>
防止回收还有个办法，可以用[IFTTT详情](https://www.vpsbuluo.com/wp-content/themes/begin/inc/go.php?url=https://ifttt.com/applets/131839p-keep-google-voice-active)<br>
当然你也可以用Google Voice号码随便注册一个会经常给你发短信的网站。<br>

参考自：https://www.vpsbuluo.com/ziyuan/27.html
