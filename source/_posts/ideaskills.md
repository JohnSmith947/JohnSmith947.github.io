---

title: IDEA Skills

date: 2022-10-30 15:07:54

tags:

---
# IDEA Skills

前言：

> 记录在 IDEA 开发过程中一些快速编码的小技巧。



## 一、postfix

**nn**

非空判断

```java
id.nn
->
if(id != null) {
  
}
```

**try**

自动生成 `try...catch` 代码块，且可以自动推断类型

```java
inputStream.close();.try
->
try {
  inputStream.close();
} catch (IOException e) {
  throw new RuntimeException(e);
}
```

**cast**

快速实现类型强转

```java
obj.cast
->
String newObj = (String) obj;
```

**if**

快速生成 `if` 代码块

```java
a.if
->
if (a) {
    ...
}
```

**throw**

快速抛异常

```java
new Exception().throw
->
throw new Exception();
```

**for**

快速生成迭代器

```java
List<String> list = new ArrayList<>();
list.for
->
for (String s : list) {

}
```

**fori**

快速生成普通 `for` 循环

```java
List<String> list = new ArrayList<>();
list.fori
->
for (int i = 0; i < list.size(); i++) {

}
```

**sout/soutv**

快速实现（不带参数/带参数）的控制台输出

```java
String name = "zhangsan";

name.sout
->
System.out.println(name);

name.soutv
->
System.out.println("name = " + name);
```

**return**

快速返回值

```java
String name = "zhangsan";
name.return
->
return name;
```

**format**

快速实现字符串格式化

```java
"%s".format
->
String.format("%s", var);
```



## 二、手动设置postfix

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afa6bac3780341cea072c14dc7881131~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c8d3066dd55412684ead6f537416661~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

效果：

```java
Object a  = new Object();

a.in
->
if (a == null) {

}
```



## 三、shortcuts

|                 Mac                 |             Windows             |    功能    |
| :---------------------------------: | :-----------------------------: | :------: |
|             Control + G             |             Alt + J             | 快速选择重复元素 |
|        Option + Shift + ↑/↓         |        Ctrl+ Shift + ↑/↓        | 调整整行的位置  |
|            Command + F6             |            Ctrl+ F6             |  修改方法签名  |
| Command + . & Command + Shift + +/- | Ctrl + +/- & Ctrl + Shift + +/- | 展开 / 收起  |
|         Command + Shift + V         |        Ctrl + Shift + V         | 查看历史剪切板  |
|            按住Command拖动鼠标            |         按住Alt或鼠标中键拖动鼠标          |  多行同时编辑  |



## 四、代码调试

**代码植入**

>​	在断点处右击或者按住 Shift 单击，可以在弹出的对话框中的 condition 输入框中输入要植入的代码，最好在最后加上 `return false` ，然后在 debug 模式下运行代码，就会执行植入的代码了。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0be94e65fcc448f0a603f4e85fcb4849~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2bc5561f0a114c1bbd01622c9c1fcbd5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)



## 五、基本设置

**设置主题**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e92c14b1a8d4f949613160fed0f1c58~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**字体大小**

可以通过按住Ctrl滑动鼠标滚轮调整字体大小

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4c63e5fdde14c688d14187fd8dacaba~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**自动导包**

勾选两个

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca77c3535a9a4b0d9356f26755ffcb04~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**显示方法分隔符**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6371e741aab46538b44723400d70286~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**忽略大小写提示**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff4815d1534c4f5fa52bd43110301d45~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**设置文件头**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52620d857ea24369832e65f195b03987~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**类序列号**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eaca31af1dc54b55a0586c37ad418b4a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**类显示数量**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fecfb66b5e6444afb6b2611d7ee917a1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1483594b7934540a78b5f50c3805e0b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**注释颜色**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8868e36ecba940c285f5e20542aa654c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**项目编码**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dbf42a4ac3148f7864ee0b540bdfbbd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**字体大小**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34a4ea6c34ca431dbf796e2bbb70868f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**已修改文件带\***

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d531aee936434f20a682c489c9c4df67~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**方法形参提示**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e0a7ea3616a4043884eb6b883859959~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**取消 import\***

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7523d0900fe4e0184ab27f3bbb3cc05~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)