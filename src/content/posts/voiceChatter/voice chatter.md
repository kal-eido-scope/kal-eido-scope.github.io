---
title: Voice Chatter 下载及使用说明
published: 2025-11-18
description: '互联网拾遗，实用工具下载'
image: 'pic\voiceChatter-icon-for-blog.png'
tags: [下载,工具,语音]
category: '资源下载'
draft: false 
lang: 'zh-cn'
---
# 下载说明

项目原网页见[http://voicechatter.net](http://voicechatter.net)，目前已失效，可通过[webArchieve](https://web.archive.org/web/20121124022625/http://voicechatter.org/downloads.php)访问历史版本。

备份、源码及下载链接可见[此链接](https://sourceforge.net/projects/voicechatter/)。

### Voice Chatter (Client)

voiceChatter客户端下载[点此](https://sourceforge.net/projects/voicechatter/files/VoiceChatter%20Client/1.5.0/VoiceChatter-win32-1.5.0.exe/download)，备用链接1[点此](dl/VoiceChatter-win32-1.5.0.exe)，备用链接2[点此](https://web.archive.org/web/20121124022625/http://voicechatter.net/files/VoiceChatter-win32-1.5.0.exe)。

最后更新：2011年11月19日

### Voice Chatter (Server)

voiceChatter服务器端下载[点此](https://sourceforge.net/projects/voicechatter/files/VoiceChatter%20Server/1.5.0/VoiceChatterServer-win32-1.5.0.zip)，备用链接1[点此](dl/VoiceChatterServer-win32-1.5.0.exe)，备用链接2[点此](https://web.archive.org/web/20121124022625/http://voicechatter.net/files/VoiceChatterServer-win32-1.5.0.zip)。

最后更新：2011年11月19日

<br>

# 使用方法

## 服务器端

### VoiceChatterServer软件设置

服务器端安装并打开VoiceChatterServer.exe

1. 指定通信使用的端口，直接回车即使用默认使用UDP通信，默认使用7878端口`<span id="port">`

```
Config file path is ./. file is vchatserver.conf
This appears to be a new server. Some information is needed in order to configure the servers.
Answer the following questions:
Enter the UDP port that will be used for this server.
Leaving this blank will default the port to 7878:
```

2. 输入管理员密码，最长32位

```
Enter an admin password (max length is 32)
: yourPassword1
```

3. 为服务器指定密码（访客密码），最长32位，直接回车即不设置密码`<span id="GuestPassword">`

```
Enter a password for the server, or leave blank for no password (max length is 32)
: yourPaaword2
```

4. 为服务器命名

```
Enter a name for the server
: yourServerName
```

5. 指定服务器允许链接的最大客户端数量

```
Enter a limit to the number of clients that can be in the server at the same time,
or leave this blank or enter 0 to have no limit
: yourLimit
```

6. 输入所需的语音编解码器频段，从1到3语音质量从差到好

```
Enter the desired voice codec band (value between 1 and 3)
  (1) Narrow Band - 8KHz (worst quality)
  (2) Wide Band - 16KHz (good quality)
  (3) Ultra-wide Band - 32KHz (best quality)
: yourChoice
```

7. 输入所需的语音编解码器质量，从0到10语音质量从差到好，语音质量越好，用的带宽越多

```
Enter the desired voice codec quality. This is a value between 0 and 10,
0 being the worst quality, 10 being the best quality.
Note that the better the quality the more bandwidth it will use.
: yourChoice 
```

8. 是否启用远程管理？（似乎即使这里关闭了，在客户端依然可以通过输入管理员密码以启用）
   如果启用了则后会使用到TCP。`<span id="TCP">`

```
Should the remote admin interface be enabled? 
Enabling this will bind a TCP socket to the same port as the port entered above
that will allow the use of Voicechatter's remote admin interface.
If you do not know what this means, it is perfectly safe to answer 'no'
(y/n): yourChoice
```

9. 是否启用IPv6？（一般不开就行）

```
Should IPv6 functionality be enabled? 
Turning this on may make the server unreachable if your os is not configured for proper IPv6 support.
If you don't know what any of this means, or if you are not sure that your OS or network will support IPv6,
answer 'no' to this question.
(y/n): yourChoice
```

10. 服务器端设置完成，出现如下字样

```
Configuration is now complete, starting server...
YYYY/MM/DD HH:MM:SS: Starting server version 1.5.0...
Starting server
Checking for inactive clients
READ: 0 bytes/10 sec(0.00 kB/s，0.00 kb/s)
SEND: 0 bytes/10 sec(0.00 KB/s，0.00 kb/s)
```

<br>

### 防火墙设置

1. 按下Win，搜索防火墙，或按照如下顺序点击 控制面板 - 系统和安全 - Windows Defender 防火墙 - 高级设置。
   打开 **高级安全 Windows防火墙**，在入站和出站规则中新建允许VoiceChatterServer.exe的规则

* 高级防火墙界面
  ![防火墙1](pic\fwDefender1.png "界面展示")
* 右键新建规则
  ![防火墙2](pic\fwDefender2.png "新建规则")
* 选择规则类型
  ![防火墙3](pic\fwDefender3.png "规则类型")
* 选择VoiceChatter程序所在路径
  ![防火墙4](pic\fwDefender4.png "程序路径")
* 指定连接匹配时的操作
  ![防火墙5](pic\fwDefender5.png "连接操作")
* 指定规则运用场景
  ![防火墙6](pic\fwDefender6.png "配置文件")
* 命名并附注
  ![防火墙7](pic\fwDefender7.png "名称备注")

2. 前往云服务器的控制台为防火墙添加放行规则。
   规则选择UDP，如果在[第8步](#TCP)选择了启用，请额外添加TCP规则
   端口默认填7878，如果在[第1步](#port)自定义了端口，则填写你输入的端口

## 客户端

客户端下载后打开VoiceChatter.exe

1. Self - Connect - New Server

   其中Password输入[第3步](#GuestPassword)设定的服务器密码，如未设置则留空即可。
   ![Sample](pic\sample1.png "示例图1")

   填完信息点击connect，听到"connection established"提示音
   ![Sample](pic\sample2.png "示例图2")
2. Self - Settings
   选择你使用的麦克风、扬声器，
   Talk Key指定按住说话的按键
   ![Sample](pic\sample3.png "示例图3")
