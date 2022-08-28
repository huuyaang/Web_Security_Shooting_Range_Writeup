

## 声明

​		由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。如欲转载或传播此文章，必须保证此文章的完整性，未经众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

## 靶场配置

​		靶场地址：https://github.com/zhuifengshaonianhanlu/pikachu

​		本文章靶场环境 `Ubuntu 22.04 LTS` + `Apache 2.4.39` + `MySQL 5.7.27` + `PHP 5.5.38`；

​		操作系统推荐使用Linux，Windows平台部分页面会出现中文乱码；

​		PHP推荐使用`PHP 5.X`版本，高版本PHP会出现错误。

## 一.暴力破解

​		事先准备好字典文件，字典文件不用太过复杂

![image-20220515225333350](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515225333350.png)

### 1.1 基于表单的暴力破解

随便输入`username`和`password`进行尝试，使用burp抓包发现是POST方式提交数据，并且没有验证码等防御措施

![image-20220515225637232](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515225637232.png)

![](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515225057808.png)

最简单的暴力破解，使用事先准备好的字典文件进行破解

设置变量位置

![image-20220515230019620](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515230019620.png)

设置`payload 1`，`payload 2`

![image-20220515230044005](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515230044005.png)

![image-20220515230109709](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515230109709.png)

查看爆破结果，得到其中一对`username`为`admin`，`password`为`123456`

![image-20220515231144729](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515231144729.png)

### 1.2 验证码绕过（on server）

服务器端验证码常见问题：

1. 验证码不存在有限时间，同一验证码一直有效
2. 

随便输入`username`、`password`和`错误的验证码`，发现提示`验证码错误`

![image-20220515231618461](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515231618461.png)

输入正确的验证码并进行抓包分析，尝试更改用户名或者密码进行重放，发现验证码长期有效，可以在username和password处设置变量进行爆破

![image-20220515232643032](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515232643032.png)

![image-20220515232706236](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515232706236.png)

![image-20220515232818827](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515232818827.png)

![image-20220515232840172](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515232840172.png)

![image-20220515232857481](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515232857481.png)

爆破成功

![image-20220515232938456](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515232938456.png)

### 1.3 验证码绕过（on client）

验证码使用前端js验证，可以通过剔除js代码来进行爆破

![image-20220515233114218](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515233114218.png)

使用burp移除js代码进行爆破

![image-20220515233243134](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515233243134.png)

爆破成功

![image-20220515233403618](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515233403618.png)

### 1.4 token防爆破？

输入用户名和密码抓包分析，发现表单中带有token参数，在暴力破解过程中自动抓取token填入表单即可。

![image-20220613213714793](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220613213714793.png)

审查网页，发现带有隐藏标签

![image-20220515234157163](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515234157163.png)

![image-20220515235251092](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220515235251092.png)

![image-20220516001255168](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516001255168.png)

![image-20220516001327250](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516001327250.png)![image-20220516001415294](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516001415294.png)

![image-20220516001444695](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516001444695.png)

![image-20220516001502284](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516001502284.png)

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516001514926.png" alt="image-20220516001514926" style="zoom:67%;" />

## 二. Cross-Site Scripting

### 2.1 反射型xss（get）

输入<>等符号发现原样输出，直接注入，发现输入长度受限，审查元素修改长度后再次注入

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220613215556560.png" alt="image-20220613215556560"  />

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220613215718636.png" alt="image-20220613215718636"  />

### 2.2 反射型xss（post）

输入<>等符号发现原样输出，直接注入

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220613231637400.png"  />

### 2.3 存储型xss

存储型xss插入后，刷新页面仍出现xss弹窗。

![image-20220613232406761](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220613232406761.png)

### 2.4 DOM型xss

闭合`<a>`标签，payload： `'><img src="#" onmouseover="alert('xss')">`，鼠标移到图片位置便会出发弹窗。

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220622143756237.png"  />

### 2.5 DOM型xss-x

闭合`<a>`标签，payload：`'><img src="#" onmouseover="alert('xss')">`

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220622144802078.png" alt="image-20220622144802078"  />

### 2.6 xss之盲打

XSS盲打是一种攻击场景，我们输出的payload不会在前端进行输出，当管理员查看时就会遭到xss攻击，登录后台查看盲打，后台地址`/xssblind/admin_login.php`

![image-20220622145038804](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220622145038804.png)

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220622145145533.png" alt="image-20220622145145533"  />

### 2.7 xss之过滤

输入`'"<script>`发现被过滤，尝试大小写绕过过滤，payload：`<SCRIPT>alert(/xss/)</sCRIpt>`

![image-20220623093744342](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220623093744342.png)

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220623093917532.png" alt="image-20220623093917532"  />

### 2.8 xss之htmlspecialchars

可以使用单引号构造payload：`\#' onclick='alert(/xss/)`

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220623095554844.png" alt="image-20220623095554844"  />

### 2.9 xss之href输出

在a标签的href属性里面,可以使用javascript协议来执行js，可以尝试使用伪协议绕过。

payload:`javascript:alert(/xss/)`

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220623100806007.png" alt="image-20220623100806007"  />

### 2.10 xss之js输出

payload:`</script><script>alert(/xss/)</script>`

<img src="https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220623105335832.png" alt="image-20220623105335832"  />

## 三. CSRF

### 3.1 CSRF(get)

get 型 csrf，构造虚假get请求诱导受害者点击即可，先登录vince账户，构造get请求，随后诱导受害者在登录账户的情况下点击链接。

http://127.0.0.1/pikachu-master/vul/csrf/csrfget/csrf_get_edit.phpsex=boy&phonenum=18626545453&add=chain&email=vince8888%40pikachu.com&submit=submit

![image-20220516150155847](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516150155847.png)

![image-20220516150558928](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516150558928.png)

![image-20220516150641296](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516150641296.png)

![image-20220516160214751](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516160214751.png)

allen个人信息已被更改

### 3.2 CSRF(post)

post'型csrf需要构造表单诱导用户提交，使用burp构造站点诱导用户提交表单

![image-20220516162038227](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516162038227.png)

使用burp（专业版可以使用次功能）

![image-20220516162120180](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516162120180.png)

![](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516162437519.png)

![image-20220516162527456](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516162527456.png)

### 3.3 CSRF Token

抓包分析，url中带有token，无法伪造，token可以防范csrf

![image-20220516162738072](https://blog-1304194110.cos.ap-nanjing.myqcloud.com/image-20220516162738072.png)

## 四. SQL-Inject

## 五. RCE

如果靶场搭建在Windows平台会出现中文乱码，解决方法如下

方法一：使用Linux平台搭建。

方法二：将cmd编码方式从`gbk`改为`utf-8`，cmd默认编码方式为gbk，服务器为UTF-8，所以出现乱码。

方法三：将rce_ping.php文件中的`//header("Content-type:text/html; charset=gbk");`改为`header("Content-type:text/html; charset=gbk");`，rce中文乱码问题解决，但别的部分出现乱码。

### 5.1 exec(ping)

### 5.2 exec(evel)

## 六. File Inclusion

### 6.1 File Inclusion(local)

### 6.2 File Inclusion(remote)

## 七. Unsafe Filedownload

### 7.1 Unsafe Filedownload

## 八. Unsafe Fileupload

### 8.1 client check

### 8.2 MIME type

### 8.3 getimagesize

## 九. Over Permission

### 9.1 水平越权

### 9.2 垂直越权

## 十. .../.../

### 10.1目录遍历

## 十一. 敏感信息泄露

### 11.1 IcanseeyourABC

## 十二. PHP反序列化

### 12.1 PHP反序列化漏洞

## 十三. XXE

### 13.1 XXE漏洞

### 十四. URL重定向

### 14.1 不安全的URL跳转

### 十五. SSRF

### 15.1 SSRF(curl)

### 15.2 SSRF(file_get_vontent)



## 参考文章

1. [pikachu靶场-暴力破解 - 陈子硕 - 博客园 (cnblogs.com)](https://www.cnblogs.com/c1047509362/p/12622650.html)
2. [Pikachu漏洞练习平台实验——CSRF（三） - 那少年和狗 - 博客园 (cnblogs.com)](https://www.cnblogs.com/dogecheng/p/11583412.html)