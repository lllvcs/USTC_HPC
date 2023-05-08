用户登录与文件传输
==================

本超算系统的操作系统为64位 **CentOS 7.x** 和 **Ubuntu Server 22.04 LTS Linux** ，用户需以 **SSH** 方式（在MS Windows下可利用PuTTY、Xshell等支持 **SSH** 协议的客户端软件 [1]_，Windows的命令行也支持ssh及sftp命令）登录到用户登录节点后进行编译、提交作业等操作。用户数据可以利用 **SFTP** （不支持FTP）协议进行数据传输。

本超算系统禁止SSH密钥登录，并且采用google authenticator二次验证，用法参见：\ http://scc.ustc.edu.cn/2018/0926/c409a339006/page.htm\ 。

用户若10分钟内5次密码错误登录，那么登录时所使用IP将被自动封锁10分钟禁止登录，之后自动解封，可以等待10分钟后再尝试，或换个IP登录，或联系超算中心老师解封。

本超算系统可从校内IP登录，校外一般无法直接访问。如果需要从校外等登录，可以使用\ `学校的VPN <http://openvpn.ustc.edu.cn>`__\ （教师的\ `网络通 <http://wlt.ustc.edu.cn>`__\ 带有此功能，学生的不带），或者申请校超算中心VPN（\ http://scc.ustc.edu.cn/vpn/\ ）。

用户修改登录密码及shell，只能通过\ http://scc.ustc.edu.cn/user/chpasswd.php\ 修改密码，而不能通过\ ``passwd``\ 和\ ``chsh``\ 修改。请不要设置简单密码和向无关人员泄漏密码，以免给用户造成损失。如果忘记密码，请邮件（\ sccadmin@ustc.edu.cn\ ）联系中心老师申请重置，并需提供帐号名、所在超算系统名称等必要信息。

用户登录进来的默认语言环境为zh_CN.UTF-8中文 [2]_，以方便查看登录后的中文提示。如希望使用英文或GBK中文，可以在自己的中添加\ ``export LC_ALL=C``\ 或 ``export LC_ALL=zh_CN.GBK``\ 。

登录进来后请注意登录后的中文提示，或运行\ ``cat /etc/motd``\ 查看登录提示，也可以运行\ ``faq``\ 命令查看常见问题的回答。

您可运行\ ``du -hs`` 可以查看目录占用的空间。请及时清除不需要的文件，以释放空间。如需更大存储空间，请与超算中心老师联系，并说明充分理由及所需大小。

超算中心不提供数据备份服务，数据一但丢失或误删将无法恢复，**请务必及时下载保存自己的数据**。

`CentOS <http://www.centos.org/>`__\ (Community ENTerprise Operating System)是Linux主流发行版之一，它来自于Red Hat Enterprise Linux依照开放源代码规定释出的源代码所编译而成；Ubuntu基于Debian开发而成。一般来说可以用\ ``man 命令``\ 或命令加\ ``-h``\ 或\ ``–help``\ 等选项来查看该命令的详细用法，详细信息可参考CentOS、Red Hat Enterprise Linux、Ubuntu、Debian手册或通用Linux手册。

.. [1]
   客户端下载：\ \ http://scc.ustc.edu.cn/411/list.htm

.. [2]
   SSH Secure Shell Client不支持UTF-8中文，不建议使用
