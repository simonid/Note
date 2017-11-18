# 初接触Ajax
## jQuery基础内容
### 使用load()方法异步请求数据
格式：`load(url,[data],[callback])`<br>
参数说明：
<ul>
<li>url：服务器地址</li>
<li>data：请求时发送的数据</li>
<li>callback：请求成功后的回调</li>
</ul>
demo：

```javascript
$(function () {
    $("#btnShow").bind("click", function () {
        var $this = $(this);
        $("ul")
        .html("<img src='Images/Loading.gif' alt=''/>")
        .load('http://www.imooc.com/data/fruit_part.html',function(){
            $this.attr("disabled", "true");
        });
    })
});
```
### 使用getJSON()异步加载JSON数据
通过Ajax异步请求获取json文件并解析<br>
格式：`jQuery.getJSON(url,data,[callback])或$.getJSON(url,data,[callback])`<br>
参数内容和load()一致<br>
demo:

```html
<body>
    <div id="divtest">
        <div class="title">
            <span class="fl">我最喜欢的一项运动</span> 
            <span class="fr">
                <input id="btnShow" type="button" value="加载" />
            </span>
        </div>
        <ul></ul>
    </div>
    
    <script type="text/javascript">
        $(function () {
            $("#btnShow").bind("click", function () {
                var $this = $(this);
                $.getJSON("http://www.imooc.com/data/sport.json",function(data){
                    $this.attr("disabled", "true");
                    $.each(data, function (index, sport) {
                        if(index==3)
                        $("ul").append("<li>" + sport["name"] + "</li>");
                    });

                });
            });
        });
    </script>
</body>
```
### 使用getScript()方法异步加载并执行js文件
异步请求并执行服务器中的JavaScript格式的文件<br>
格式：`jQuery.getScript(url,[callback])或$.getScript(url,[callback])`<br>

demo:

```html
<body>
    <div id="divtest">
        <div class="title">
            <span class="fl">我最喜欢的运动</span> 
            <span class="fr">
                <input id="btnShow" type="button" value="加载" />
            </span>
        </div>
        <ul></ul>
    </div>
    
    <script type="text/javascript">
        $(function () {
            $("#btnShow").bind("click", function () {
                var $this = $(this);
                $.getScript('http://www.imooc.com/data/sport_f.js',function() {
                    $this.attr("disabled", "true");
                });
            })
        });
    </script>
</body>
```
### 使用get()以GET方式获取
采用GET方式向服务器请求数据，并通过方法中回调函数的参数返回请求的数据<br>
格式：`$.get(url,data,success(response,status,xhr),dataType)`<br>
回调函数中的参数说明：
<ul>
<li>data - 包含来自请求的结果数据</li>
<li>status - 包含请求的状态（"success"、"notmodified"、"error"、"timeout"、"parsererror"）
</li>
<li>xhr - 包含 XMLHttpRequest 对象</li>
</ul>

`dataType`: 返回的数据格式 	可选。规定预期的服务器响应的数据类型默认地，jQuery 会智能判断<br>

### 使用post()方法以POST方式从服务器发送数据
格式：`jQuery.post(url,data,success(data, textStatus, jqXHR),dataType)`<br>
demo：

```html
<body>
    <div id="divtest">
        <div class="title">
            <span class="fl">我的个人资料</span> 
            <span class="fr">
                <input id="btnShow" type="button" value="加载" />
            </span>
        </div>
        <ul></ul>
    </div>
    
    <script type="text/javascript">
        $(function () {
            $("#btnShow").bind("click", function () {
                var $this = $(this);
                $.get('http://www.imooc.com/data/info_f.php',function(data){
                    $this.attr("disabled", "true");
                    $("ul").append("<li>我的名字叫：" + data.name + "</li>");
                    $("ul").append("<li>男朋友对我说：" + data.say + "</li>");
                });
            })
        });
    </script>
</body>
```
```javascript
$(function () {
    $("#btnCheck").bind("click", function () {
        $.post('http://www.imooc.com/data/check_f.php',{
            num=$("#txtNumber").val();
        },
        function (data) {
            $("ul").append("<li>你输入的<b>  "
            + $("#txtNumber").val() + " </b>是<b> "
            + data + " </b></li>");
        });
    })
});
```
### 使用serialize()方法序列化表单元素值
将表单中有name属性的元素值进行序列化，生成标准URL编码文本字符串，直接可用于ajax请求<br>
格式：`$(selector).serialize()`<br>
其中selector参数是一个或多个表单中的元素或表单元素本身<br>
demo：

```html
<body>
    <div id="divtest">
        <div class="title">
            <span class="fl">我的个人资料</span> 
            <span class="fr">
                <input id="btnAction" type="button" value="序列化" />
            </span>
        </div>
        <form action="">
        <ul>
            <li>姓名：<input name="Text1" type="text" size="12" /></li>
            <li>
                <select name="Select1">
                    <option value="0">男</option>
                    <option value="1">女</option>
                </select>
            </li>
            <li><input name="Checkbox1" type="checkbox" />资料是否可见 </li>
            <li id="litest"></li>
        </ul>
        </form>
    </div>
    
    <script type="text/javascript">
        $(function () {
            $("#btnAction").bind("click", function () {
                $("#litest").html($("form").serialize());
            })
        })
    </script>
    </body>
```
注意：<br>
<ul>
<li>能被序列化的是含有name属性的表单元素；</li>
<li>input[type="submit"]、button[type="submit"]、input[type="file"]不会被序列化；</li>
<li>input[type="checkbox"]和input[type="radio"]只有被选中时才会序列化；</li>
<li>没有value属性的表单元素，其值被序列化为空字符串。</li>
</ul>

### 使用ajax()方法加载服务器数据
格式：`$.ajax([settings])`<br>
其中，`setting`包含的内容：
```
url:"",
    data:{"参数名":""}，
    datatype:"post",
    success:function(data){}
```
参数settings为发送ajax请求时的配置对象，在该对象中，url表示服务器请求的路径，data为请求时传递的数据，dataType为服务器返回的数据类型，success为请求成功的执行的回调函数，type为发送数据请求的方式，默认为get。<br>

```html
<body>
<div id="divtest">
    <div class="title">
        <span class="fl">检测数字的奇偶性</span> 
        <span class="fr">
            <input id="btnCheck" type="button" value="检测" />
        </span>
    </div>
    <ul>
        <li>请求输入一个数字 
            <input id="txtNumber" type="text" size="12" />
        </li>
    </ul>
</div>

<script type="text/javascript">
    $(function () {
        $("#btnCheck").bind("click", function () {
            $.ajax({
                url:"http://www.imooc.com/data/check.php",
                data: { num: $("#txtNumber").val() },
                type:"post",
                success: function (data) {
                    $("ul").append("<li>你输入的<b>  "
                    + $("#txtNumber").val() + " </b>是<b> "
                    + data + " </b></li>");
                }
            });
        })
    });
</script>
</body>
```

### 使用ajaxSetup()方法设置全局Ajax默认选项
可以设置Ajax请求的一些全局性选项值，设置完成后，后面的Ajax请求将不需要再添加这些选项值<br>
格式：`$.ajaxSetup([options])`<br>
其中，`option`选项中包含：
```
  dataType:"text",
  type:"POST",
  success:function
```
demo：

```html
<body>
    <div id="divtest">
        <div class="title">
            <span class="fl">奇偶性和是否大于0</span> 
            <span class="fr">
                <input id="btnShow_1" type="button" value="验证1" />
                <input id="btnShow_2" type="button" value="验证2" />
            </span>
        </div>
        <ul>
            <li>请求输入一个数字 
                <input id="txtNumber" type="text" size="12" />
            </li>
        </ul>
    </div>
    
    <script type="text/javascript">
        $(function () {
            $.ajaxSetup({
            type:"POST",
            dataType:"text",
            success:function(data){
                    $("ul").append("<li>你输入的<b>  "
                        + $("#txtNumber").val() + " </b>是<b> "
                        + data + " </b></li>");
                }
            });
            $("#btnShow_1").bind("click", function () {
                $.ajax({
                    data: { num: $("#txtNumber").val() },
                    url: "http://www.imooc.com/data/check.php"
                });
            })
            $("#btnShow_2").bind("click", function () {
                $.ajax({
                    data: { num: $("#txtNumber").val() },
                    url: "http://www.imooc.com/data/check_f.php"
                });
            })
        });
    </script>
</body>
```

### 使用ajaxStart()和ajaxStop()方法
ajaxStart()和ajaxStop()方法是绑定Ajax事件。ajaxStart()方法用于在Ajax请求发出前触发函数，ajaxStop()方法用于在Ajax请求完成后触发函数<br>
格式：`$(selector).ajaxStart(function())和$(selector).ajaxStop(function())`<br>
demo：

```html
<body>
    <div id="divtest">
        <div class="title">
            <span class="fl">加载一段文字</span> 
            <span class="fr">
                <input id="btnShow" type="button" value="加载" />
            </span>
        </div>
        <ul>
            <li id="divload"></li>
        </ul>
    </div>
    
    <script type="text/javascript">
        $(function () {
            $("#divload").ajaxStart(function() {
                $(this).html("正在请求数据...");
            });
            $("#divload").ajaxStop(function() {
                $(this).html("数据请求完成！");
            });
            $("#btnShow").bind("click", function () {
                var $this = $(this);
                $.ajax({
                    url: "http://www.imooc.com/data/info_f.php",
                    dataType: "json",
                    success: function (data) {
                        $this.attr("disabled", "true");
                    $("ul").append("<li>我的名字叫：" + data.name + "</li>");
                    $("ul").append("<li>男朋友对我说：" + data.say + "</li>");
                    }
                });
            })
        });
    </script>
</body>
```

编程练习：<br>
定义一个json对象，用于保存学生的相关资料

```html
<body>
    <form action="#" method="get">
        <label>
            <input id="txt" class="txt" type="text" name="username" placeholder="input your name">
        </label>
        <label>
            <input id="grade" class="grade" type="text" name="grade" placeholder="input your grade">
        </label>
        <input id="save-btn" type="button" value="save">
        <input id="show-btn" type="button" value="show">
    </form>
    <ul></ul>
    
    <script>
        var txt = $('#txt');
        var grade = $("#grade");
        var json = [];
        $('#save-btn').on('click',function(){
            json.push({
                name : txt.val(),
                grade : grade.val()
            });
            txt.val('');
            grade.val('');
        });
        $('#show-btn').on('click',function(){
            $.each(json,function(i){
                $('ul').append('<li>' + 'name:' + json[i].name + '  ' +'grade:' + json[i].grade + '</li>');
            });
        });
    </script>
</body>
```

上述的方法基本就是jQuery Ajax中的方法，在实际开发中，可以通过一些插件来简化开发难度<br>


### XMLHttpRequest对象
```javascript
var request;
if(window.XMLHttpRequest){
    request = new XMLHttpRequest(); //IE7+
}else{
    request = new ActiveXObject("Microsoft.XMLHTTP");
}
```

### HTTP的概念
#### GET和POST请求的区别
`GET`:<br>
      一般用于信息获取<br>
      使用URL传递参数<br>
      对所发送信息数量有限制，一般在2000个字符<br>
`POST`:<br>
       一般用于修改服务器上的资源<br>
       对所发送信息数量无限制<br>
