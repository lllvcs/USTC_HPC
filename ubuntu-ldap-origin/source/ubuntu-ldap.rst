简介
****

基本概念
========

LDAP（Lightweight Directory Access Protocol - 轻量级目录控制协议）是一种运行在TCP/IP上用于查询和修改基于X.500的目录服务的协议，可用于用户身份认证。当前的LDAP版本为LDAPv3（RFC4510定义），Ubuntu上的实现为OpenLDAP。

LDAP协议控制目录，常见的误解为称一个目录为一个LDAP目录或LDAP数据库，但由于这实在是太常见，而且我们也知道谈论的是什么，因此这是可以的。

以下为几个主要的关键概念和术语：

- 目录 **directory** 是数据条目 **entry** 树，且分级存放，被称为目录信息树(Directory Information Tree, DIT)；
- 每个条目含有一属性集 **attributes** ；
- 属性都有一个名字/描述 **key** 和一个或多个值 **value** ；
- 每个属性都被定义在至少一个对象类 **objectClass** 内；
- 属性和对象类在模式 **schemas** （对象类实际被认为一种特殊的属性）中定义；
- 每个条目都有唯一的标识符：它的专有名字(Distinguished Name, **DN** 或 **dn** )，这又含有一个被父条目 **DN** 跟随的相对专有名字(Relative Distinguished Name， **RDN** )；
- 条目 **DN** 不是一种属性，它不被视为条目自身的一部分。

================== ======================================================
objectClass	       含义
================== ======================================================
olcGlobal	       全局配置文件类型， 主要是 *cn=config.ldif* 中的配置项
top	               顶层的对象
organization       组织，比如公司名称，顶层的对象
organizationalUnit 重要，一个目录节点，通常是 *group* ，或者部门这样的含义
inetOrgPerson	   重要，真正的用户节点类型， *person* 类型， 叶子节点
groupOfNames	   重要，分组的group类型，标记一个 *group* 节点
olcModuleList	   配置模块的对象
================== ======================================================

常用关键字列表
==============

======  =======================   ==================================================================================================
关键字  英文全称	              含义
======  =======================   ==================================================================================================
dc      Domain Component          域名的部分，其格式是将完整的域名分成几部分（以 **“,”** 分隔，且加 **“dc=”** ），如域名为“example.com”，则变成“dc=example,dc=com”
uid	    User Id	                  用户ID，如“tom”
ou      Organization Unit         组织单位，类似于Linux文件系统中的子目录，它是一个容器对象，组织单位可以包含其他各种对象（包括其他组织单元），如“market” ；最多可以有四级，每级最长32个字符；可以为中文
cn	    Common Name	              公共名称，如“Thomas Johansson”，最长可以到80个字符，可以为中文
sn	    Surname	                  姓，如“Johansson”
dn      Distinguished Name        专有名称，类似于Linux文件系统中的绝对路径，每个对象都有一个惟一的名称，如“uid= tom,ou=market,dc=example,dc=com”，在一个目录树中DN总是惟一的
rdn	    Relative dn	              相对名称，类似于文件系统中的相对路径，它是与目录树结构无关的部分，如“uid=tom”或“cn= Thomas Johansson”
c	    Country	                  国家，如“CN”或“US”等，为2个字符长
o	    Organization	          组织名，如“Example, Inc.”，可以3—64个字符长
======  =======================   ==================================================================================================

如：

- DN：cn=hmli,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
- RDN：cn=hmli
- 父DN：dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

为了在命令行上执行命令时无需输入 **dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn** 这一长串字符（既简单，又避免错误），可以定义一个变量方便后面使用，如在bash下执行命令：
::

    export DC="dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"

之后在命令行（配置文件中不可以）即可用 **$DC** 来引用，如执行命令：
::

    ldapadd -x -D cn=Maneger,$DC -W -f add_content.ldif

LDIF(LDAP Data Interchange Format)格式
======================================

**LDIF** 是一种基于文本的格式，用于描述目录服务实体及其属性。使用 **LDIF** 格式，可以通过 *ldapadd* 和 *ldapmodify* 等命令将信息从一个目录移到另一个目录。下面是用户信息服务的 **LDIF** 格式的示例。

::

    dn: cn=hmli,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    cn: hmli
    givenName: Huimin
    sn: LI
    telephoneNumber: 188 551 xxxxx
    telephoneNumber: 188 551 yyyyy
    mail: hmli@ustc.edu.cn
    manager: cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: inetOrgPerson
    objectClass: organizationalPerson
    objectClass: person
    objectClass: top

.. note::
    **:** 前为变量， **:** 后为值。

节点分类
=========

按照功能不同可以分为以下三类：

- 服务端、服务节点：提供LDAP认证服务，如主管理节点
- 复制端、复制节点、冗余节点：提供LDAP服务端的复制，实现冗余功能，如从管理节点
- 客户端：需要发起LDAP认证服务，如登录节点、计算节点等

针对本文档约定：

- 服务端节点名：admin22-01
- 复制端节点名：admin22-02
- DNS域名后缀：hanhai.scc.ustc.edu.cn

服务端设置
**********

.. _TLS:

生成TLS加密证书
===============

当与OpenLDAP服务端进行认证时，最好采用加密会话，这可以利用TLS（Transport Layer Security，传输层安全性)协议实现。

在这里，我们将成为我们自己的证书颁发机构（ **Certificate Authority** ），然后创建并签署我们的LDAP服务器证书作为该 **CA** 。本指南将使用 *certtool* 实用程序来完成这些任务。为简单起见，这是在OpenLDAP服务器本身上完成的，但您真正的内部 **CA** 应该在其他地方。

.. note::
   以后除非特殊说明，否则都是以 *root* 用户执行。

安装所需要的包，执行命令：
::

    apt install gnutls-bin ssl-cert

生成Certificate Authority的私钥，执行命令：

::

    certtool --generate-privkey --bits 4096 --outfile /etc/ssl/private/mycakey.pem

生成定义该 **CA** 的模板文件：

::

    cn = USTC SCC
    ca
    cert_signing_key
    expiration_days = 3650

生成自签名 **CA** 证书存为文件 */usr/local/share/ca-certificates/mycacert.crt* ，执行命令：
::

    certtool --generate-self-signed \
        --load-privkey /etc/ssl/private/mycakey.pem \
        --template /etc/ssl/ca.info \
        --outfile /usr/local/share/ca-certificates/mycacert.crt

.. note::
   --outfile 是正确的，我们在目录 */usr/local/share/ca-certificates/* 下生成 **CA** 。这里是命令 *update-ca-certificates* 将拾起可信任的本地CA的目录。为了从目录 */usr/share/ca-certificates/* （不含有 *local* ）获取 **CA** ，需要执行命令 *dpkg-reconfigure ca-certificates* 。

执行命令 *update-ca-certificates* 将新的CA认证加入可信任的CA列表中（注意执行下述命令显示加入一个CA）：
::

    update-ca-certificates

屏幕显示：
::

    Updating certificates in /etc/ssl/certs...
    1 added, 0 removed; done.
    Running hooks in /etc/ca-certificates/update.d...
    done.

上述还会生成软链接 */etc/ssl/certs/mycacert.pem* ，该软链接指向实际的目录 */usr/local/share/ca-certificates/* 中的文件。以后会经常使用该软链接 */etc/ssl/certs/mycacert.pem* 。
 
设置服务端的私有证书，执行命令：

::

    certtool --generate-privkey --bits 2048 --outfile /etc/ldap/admin22_01_slapd_key.pem

.. note::
   用你实际的服务端节点名代替上述的 **admin22-01**  。为将要使用它们的主机和服务命名证书和密钥将有助于保持清晰。

生成包含下面信息的文件 *admin22-01.info* ：
::

    organization = USTC SCC
    cn = admin22-01
    scc.edu.cn
    tls_www_server
    encryption_key
    signing_key
    expiration_days = 3650

上述证书有效期为10年，仅对 *admin22-01* 节点有效。

生成服务端认证证书，执行命令：
::

    certtool --generate-certificate \
        --load-privkey /etc/ldap/admin22-01_slapd_key.pem \
        --load-ca-certificate /etc/ssl/certs/mycacert.pem \
        --load-ca-privkey /etc/ssl/private/mycakey.pem \
        --template /etc/ssl/ldap01.info \
        --outfile /etc/ldap/ldap01_slapd_cert.pem

调整权限与所有者，执行命令：
::

    chgrp openldap /etc/ldap/admin22-01_slapd_key.pem
    chmod 0640 /etc/ldap/admin22-01_slapd_key.pem

.. _installpkg:

安装所需LDAP包
==============

安装服务端所需包，执行命令：
::

    apt install slapd ldap-utils

如需要修改目录信息树 **DIT** 后缀，当前是一个很好的时机（因为修改会取消当前存在的配置信息）。如需修改，请运行下面命令：
::

    dpkg-reconfigure slapd

修改文件 */etc/ldap/ldap.conf* 内容为：
::

    #
    # LDAP Defaults
    #

    # See ldap.conf(5) for details
    # This file should be world readable but not world writable.

    BASE	dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    URI	ldap://localhost #ldap://ldap-provider.example.com:666

    #SIZELIMIT	12
    #TIMELIMIT	15
    #DEREF		never

    # TLS certificates (needed for GnuTLS)
    TLS_CACERT	/etc/ssl/certs/ca-certificates.crt
    TLS_REQCERT demand

默认DIF
=======

**slapd** 软件包被设计为在服务本身内配置，为此目的专门设置了一个独立的DIT。这样将允许动态配置 **slapd** 时无需重启服务或编辑配置文件。配置数据库含有一系列存储在目录 */etc/ldap/slapd.d/* 下基于文本的LDIF文件（但最好永远不要直接编辑这些文件）。这种工作方式有几个名称：slapd-config方法、RTC方法（实时配置）或cn=config方法。也可以采用传统的平面文件（flat-file，包含没有相对关系结构的记录的文件）方法(slapd.conf)，但这里不涉及。

完成安装后，将得到两个数据库或后缀：一个为基于您的主机域名(hanhai.scc.ustc.edu.cn)的数据；另一个为配置文件，其根位于cn=config。为了修改每个数据，需要不同的认证和访问方法：

- dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn：针对该后缀的管理账户账户为 *cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn* ，密码是在安装 **slapd** 包时设定的。
- cn=config：保存该后缀下slapd自身的配置。可以利用特定的 *DN gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth* 改为这。这为当采用SASL外部身份验证时（使用 *ldapi:///* 传输经由*/run/slapd/ldapi* unix套接字），使得本地系统的 *root* 用户(uid=0/gid=0)如何被目录看见。本质上，这意味着只有本地 *root* 用户才能更新cn=config数据库。

下面为利用LDAP协议的命令 *ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn* 看到的slapd-config DIT信息看起来的格式（#及之后内容为解释，实际显示不出）：

::
dn: cn=config #全局配置

    dn: cn=module{0},cn=config #动态加载模块

    dn: cn=schema,cn=config #含硬编码（Hard-coded，写死）的系统级模式￼

    dn: cn={0}core,cn=schema,cn=config #含硬编码核心模式

    dn: cn={1}cosine,cn=schema,cn=config #RFC 1274 COSINE模式

    dn: cn={2}nis,cn=schema,cn=config #NIS模式

    dn: cn={3}inetorgperson,cn=schema,cn=config #InetOrgPerson模式，存储真正的用户节点类型，person类型， 叶子节点

    dn: olcDatabase={-1}frontend,cn=config #前端数据库，存储针对其他数据库的默认设置

    dn: olcDatabase={0}config,cn=config #存储Slapd配置数据库信息（cn=config）

    dn: olcDatabase={1}mdb,cn=config #存储你自己的数据库实体（dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn）

.. note::
    + -Q：启用SASL的静默模式，即不提示。
    + -LLL：设置以LDIF格式显示搜索结果：

     - 单个 *-L* 设定以LDIFv1模式显示；
     - 第二个 *-L* 禁止显示注释；
     - 第三个 *-L* 禁止打印LDIF版本。

执行下面命令进行设置：
::

    dpkg-reconfigure slapd

当提示设置DNS domain name（设置DNS域名）时，输入：
::
    
    hanhai.scc.ustc.edu.cn

当提示Organization name（设置组织名）时，输入：
::
    
    USTC SCC

当提示输入Administrator password（设置管理账户密码）时，输入用户 *cn=Maneger* 的密码（千万别忘了该密码）：
::

    ldapsearch -x -LLL -H ldap:/// -b dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn dn

输出为：
::

    dn: dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

执行命令验证是否成功：
::

    ldapwhoami -x -D cn=Maneger,$DC -W

输出为下方输入密码提示：
::

    Enter LDAP Password: 

输入密码后，输出如下，则表示成功：
::

    dn:cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

两种不同的认证机制：

- -x：简单绑定，基础的文本认证。因为没有提供binddn（ *-D* 选项），这将为匿名绑定。未使用 *-x* 选项时，默认采用SASL绑定。
- -Y EXTERNAL：采用SASL绑定（无 *-x* 选项），采用EXTERNAL类型。与 *-H ldapi:///* 一起使用，采用本地UNIX socket连接。

这两种情形，我们都将只会获得服务端ACL基于我们的身份允许我们看到的信息。用于验证认证非常方便的命令为 *ldapwhoami* ，执行命令：
::

    ldapwhoami -x

显示如下，则表示成功：
::

    anonymous

执行命令：
::

    ldapwhoami -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W

在输入密码后显示如下，则表示成功：
::

    dn:cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

当采用简单绑定 *-x* 选项且用 *-D* 选项指定一个 **binddn** 作为认证 **dn** 时，服务端将寻找该条目的 **userPassword** 属性用于认证。在上面的特殊情况下，我们采用的是数据库中的 **rootDN** 条目，即实际管理账户，这是一种特殊情况，其密码是在安装包时在配置中设置的。

.. note::
    没有采用传输安全协议的简单绑定是明文传输的，非常不安全，应该尽快在OpenLDAP服务端中增加TLS支持（不要再用即将被淘汰的LDAPS）。

下面为采用SASL（Simple Authentication and Security Layer，简单认证与安全层）的例子（sudo为 *root* 用户权限，因此UidNumber与GidNumber为0）：
普通用户执行命令：
::

    ldapwhoami -Y EXTERNAL -H ldapi:/// -Q

显示：
::

    dn:gidNumber=1000+uidNumber=1000,cn=peercred,cn=external,cn=auth

切换到 *root* 用户或采用 *sudo* 执行命令： 
::

    ldapwhoami -Y EXTERNAL -H ldapi:/// -Q

显示：
::
    
   dn:gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth

当采用通过 *ldapi:///* 传输使用 *SASL EXTERNAL* 时， **binddn** 变为连接中用户的 *uid* 和 *gid* 的组合，且后面跟随后缀 *cn=peercred,cn=external,cn=auth* 。服务端的ACL知道此规则，可以保证本地 *root* 用户利用SASL机制完成针对cn=config的写访问。

填充目录
========

- People节点（存储用户）
- Groups节点（存储用户组）
- scc用户组
- Huimin LI用户

生成 *add_content.ldif* 文件（为方便检查修改等，可以自己建立个目录 */etc/ldap/slapd.d/ldif/* ，放在其下），其内容为：
::

    dn: ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: organizationalUnit
    ou: People

    dn: ou=Groups,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: organizationalUnit
    ou: Groups

    dn: cn=scc,ou=Groups,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: posixGroup
    cn: miners
    gidNumber: 10000

    dn: uid=hmli,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: inetOrgPerson
    objectClass: posixAccount
    objectClass: shadowAccount
    uid: hmli
    sn: LI
    givenName: Huimin
    cn: Huimin LI
    displayName: 李会民
    uidNumber: 10000
    gidNumber: 10000
    userPassword: {CRYPT}x
    gecos: Huimin LI
    loginShell: /bin/bash
    homeDirectory: /home/scc/hmli

执行命令：
::

    ldapadd -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -f add_content.ldif

根据"Enter LDAP Password:"提示输入密码后显示： 
::

    adding new entry "ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"

    adding new entry "ou=Groups,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"

    adding new entry "cn=scc,ou=Groups,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"

    adding new entry "uid=hmli,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"

其他配置等也可以采用命令 *ldapadd* 来增加。

设定上面生成的用户 *hmli* 的密码，执行命令：
::

    ldappasswd -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -S uid=hmli,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

.. note::
    *cn=Maneger* 后无 *ou=People* ，而 *cn=hmli* 后有 *ou=People* 

按照提示先输入两次用户 *hmli* 的密码，最后输入管理账户 *Maneger* 的密码，屏幕输出显示如下：
::

    New password:
    Re-enter new password:
    Enter LDAP Password:

现在可以利用命令 *ldapsearch* 来检查上面信息，执行：
::

    ldapsearch -x -LLL -b dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn (uid=hmli) cn gidNumber 

显示：
::
    
    dn: uid=hmli,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    cn: huimin li
    gidNumber: 10000

修改配置
========

slapd-config DIT可以被查询及修改。

增加索引
--------

如将 **Index** 增加到 *{1}mdb,cn=config* 数据库定义中，可以生成文件 *uid_index.ldif* ，内容为：
::

    dn: olcDatabase={1}mdb,cn=config
    add: olcDbIndex
    olcDbIndex: mail eq,sub  

执行命令提交：
::

    ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f uid_index.ldif

成功后，为查看确认，可以执行命令： 
::

    ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config '(olcDatabase={1}mdb)' olcDbIndex

如需要修改上面信息，可以利用命令 *ldapmodify* ，如生成 *mod_content.ldif* 文件，内容为：
::

    dn: cn=scc,ou=Groups,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    changetype: modify
    replace: gidNumber
    gidNumber: 10001

    dn: uid=hmli,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    changetype: modify
    replace: displayName
    displayName: Huimin LI
    -
    replace: gecos
    gecos: Huimin LI

.. note::

    - 上面两个 *dn* 间用空行分隔，在同一个条目下的两个动作时需要一个且只含有一个 *-* 的行区分。
    - changetype: modify 指示了是修改，其他还有add，delete等操作值
    - replace: gidNumber 指示了修改gidNumber属性
    - gidNumber: 10001 指示了修该后的gidNumber属性值

执行命令提交：
::

    ldapmodify -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -f mod_content.ldif

将显示：
::

    modifying entry "cn=scc,ou=Groups,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"

    modifying entry "uid=hmli,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"


也可以直接如下运行命令（注意最后的 **-** ，表示从屏幕输入），然后直接在屏幕上输入上述文件 *mod_content.ldif* 中的信息：
::

    ldapmodify -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -

修改 *rootDN* 密码
------------------

如忘记了LDAP管理账户密码，则需要由 *root* 用户或sudo权限的用户在LDAP系统的服务器上访问重置。

执行命令：
::

    slappasswd

按提示输入新密码获取新密码的哈希(hash)值，屏幕显示：
::

    New password: 
    Re-enter new password: 
    {SSHA}VKrYMxlSKhONGRpC6rnASKNmXG2xHXFo

生成文件 *changerootpw.ldif* ，内容为（注意olcRootPW值为上述生成的{SSHA}前缀字符串）：
::

    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    replace: olcRootPW
    olcRootPW: {SSHA}VKrYMxlSKhONGRpC6rnASKNmXG2xHXFo

执行命令进行修改：
::

    ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f changerootpw.ldif


仍旧有实际的 *cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn dn* 在 *dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn* 数据库中，因此也进行修改。由于这是数据库中的一个条目，因此可以使用命令 *ldappasswd* 。

执行命令：
::

    ldappasswd -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -S

显示：
::

    New password:
    Re-enter new password:
    Enter LDAP Password:

增加模式
--------

只有LDIF格式的模式schemas才能增加到 *cn=config* 。如不是，则需要先进行转化（如利用命令 *schema2ldif* ）。可以在目录 */etc/ldap/schema/* 中找到很多未转化的schemas。

.. note::
    务必要慎重，如果不是太麻烦，建议先在测试系统中测试，成功后再在生产系统中测试。

下面为增加密码策略(ppolicy)模式而创建一个文件名为 *ppolicy.ldif* 的文件（针对Ubuntu 22.04系统，虽然官方手册说有现成文件 */etc/ldap/schema/ppolicy.ldif* ，但实际并没有，需要生成），其内容为：
::

    # $OpenLDAP$
    ## This work is part of OpenLDAP Software <http://www.openldap.org/>.
    ##
    ## Copyright 2004-2021 The OpenLDAP Foundation.
    ## All rights reserved.
    ##
    ## Redistribution and use in source and binary forms, with or without
    ## modification, are permitted only as authorized by the OpenLDAP
    ## Public License.
    ##
    ## A copy of this license is available in the file LICENSE in the
    ## top-level directory of the distribution or, alternatively, at
    ## <http://www.OpenLDAP.org/license.html>.
    #
    ## Portions Copyright (C) The Internet Society (2004).
    ## Please see full copyright statement below.
    #
    # Definitions from Draft behera-ldap-password-policy-07 (a work in progress)
    #       Password Policy for LDAP Directories
    # With extensions from Hewlett-Packard:
    #       pwdCheckModule etc.
    #
    # Contents of this file are subject to change (including deletion)
    # without notice.
    #
    # Not recommended for production use!
    # Use with extreme caution!
    #
    # This file was automatically generated from ppolicy.schema; see that file
    # for complete references.
    #
    dn: cn=ppolicy,cn=schema,cn=config
    objectClass: olcSchemaConfig
    cn: ppolicy
    olcAttributeTypes: {0}( 1.3.6.1.4.1.42.2.27.8.1.1 NAME 'pwdAttribute' EQUALITY
      objectIdentifierMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.38 )
    olcAttributeTypes: {1}( 1.3.6.1.4.1.42.2.27.8.1.2 NAME 'pwdMinAge' EQUALITY in
     tegerMatch ORDERING integerOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
      SINGLE-VALUE )
    olcAttributeTypes: {2}( 1.3.6.1.4.1.42.2.27.8.1.3 NAME 'pwdMaxAge' EQUALITY in
     tegerMatch ORDERING integerOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
      SINGLE-VALUE )
    olcAttributeTypes: {3}( 1.3.6.1.4.1.42.2.27.8.1.4 NAME 'pwdInHistory' EQUALITY
      integerMatch ORDERING integerOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1
     .27 SINGLE-VALUE )
    olcAttributeTypes: {4}( 1.3.6.1.4.1.42.2.27.8.1.5 NAME 'pwdCheckQuality' EQUAL
     ITY integerMatch ORDERING integerOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.12
     1.1.27 SINGLE-VALUE )
    olcAttributeTypes: {5}( 1.3.6.1.4.1.42.2.27.8.1.6 NAME 'pwdMinLength' EQUALITY
      integerMatch ORDERING integerOrderingMatch  SYNTAX 1.3.6.1.4.1.1466.115.121.
     1.27 SINGLE-VALUE )
    olcAttributeTypes: {6}( 1.3.6.1.4.1.42.2.27.8.1.7 NAME 'pwdExpireWarning' EQUA
     LITY integerMatch ORDERING integerOrderingMatch  SYNTAX 1.3.6.1.4.1.1466.115.
     121.1.27 SINGLE-VALUE )
    olcAttributeTypes: {7}( 1.3.6.1.4.1.42.2.27.8.1.8 NAME 'pwdGraceAuthNLimit' EQ
     UALITY integerMatch ORDERING integerOrderingMatch  SYNTAX 1.3.6.1.4.1.1466.11
     5.121.1.27 SINGLE-VALUE )
    olcAttributeTypes: {8}( 1.3.6.1.4.1.42.2.27.8.1.9 NAME 'pwdLockout' EQUALITY b
     ooleanMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE )
    olcAttributeTypes: {9}( 1.3.6.1.4.1.42.2.27.8.1.10 NAME 'pwdLockoutDuration' E
     QUALITY integerMatch ORDERING integerOrderingMatch  SYNTAX 1.3.6.1.4.1.1466.1
     15.121.1.27 SINGLE-VALUE )
    olcAttributeTypes: {10}( 1.3.6.1.4.1.42.2.27.8.1.11 NAME 'pwdMaxFailure' EQUAL
     ITY integerMatch ORDERING integerOrderingMatch  SYNTAX 1.3.6.1.4.1.1466.115.1
     21.1.27 SINGLE-VALUE )
    olcAttributeTypes: {11}( 1.3.6.1.4.1.42.2.27.8.1.12 NAME 'pwdFailureCountInter
     val' EQUALITY integerMatch ORDERING integerOrderingMatch  SYNTAX 1.3.6.1.4.1.
     1466.115.121.1.27 SINGLE-VALUE )
    olcAttributeTypes: {12}( 1.3.6.1.4.1.42.2.27.8.1.13 NAME 'pwdMustChange' EQUAL
     ITY booleanMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE )
    olcAttributeTypes: {13}( 1.3.6.1.4.1.42.2.27.8.1.14 NAME 'pwdAllowUserChange'
     EQUALITY booleanMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE )
    olcAttributeTypes: {14}( 1.3.6.1.4.1.42.2.27.8.1.15 NAME 'pwdSafeModify' EQUAL
     ITY booleanMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE )
    olcAttributeTypes: {14}( 1.3.6.1.4.1.42.2.27.8.1.15 NAME 'pwdSafeModify' EQUAL
     ITY booleanMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE )
    olcAttributeTypes: {15}( 1.3.6.1.4.1.4754.1.99.1 NAME 'pwdCheckModule' DESC 'L
     oadable module that instantiates "check_password() function' EQUALITY caseExa
     ctIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 SINGLE-VALUE )
    olcAttributeTypes: {16}( 1.3.6.1.4.1.42.2.27.8.1.30 NAME 'pwdMaxRecordedFailur
     e' EQUALITY integerMatch ORDERING integerOrderingMatch  SYNTAX 1.3.6.1.4.1.
     1466.115.121.1.27 SINGLE-VALUE )
    olcObjectClasses: {0}( 1.3.6.1.4.1.4754.2.99.1 NAME 'pwdPolicyChecker' SUP top
      AUXILIARY MAY pwdCheckModule )
    olcObjectClasses: {1}( 1.3.6.1.4.1.42.2.27.8.2.1 NAME 'pwdPolicy' SUP top AUXI
     LIARY MUST pwdAttribute MAY ( pwdMinAge $ pwdMaxAge $ pwdInHistory $ pwdCheck
     Quality $ pwdMinLength $ pwdExpireWarning $ pwdGraceAuthNLimit $ pwdLockout $
      pwdLockoutDuration $ pwdMaxFailure $ pwdFailureCountInterval $ pwdMustChange
      $ pwdAllowUserChange $ pwdSafeModify $ pwdMaxRecordedFailure ) )

执行命令增加到配置中：
::

    ldapadd -Q -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/ppolicy.ldif

记录日志
--------

服务进程 *slapd* 的活动日志记录在实施基于OpenLDAP的解决方案时非常有用，但必须在软件安装后手动启用。否则，日志中只会出现基本消息。与任何其他此类配置一样，日志记录是通过slapd-config数据库启用的。

OpenLDAP含有多个日志级别，每个级别都包含（附加）较低的级别的信息。一个很好的尝试级别是 *stats* （记录统计数据）。slapd-config手册页有更多关于不同子系统的内容。

准备文件 *logging.ldif* ，其内容为：
::
    
    dn: cn=config
    changetype: modify
    replace: olcLogLevel
    olcLogLevel: stats

执行命令提交：
::

    ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f logging.ldif

这将产生大量的日志记录，一旦系统投入生产，将希望将其限制到不太详细的级别。在这种详细模式下，主机的系统日志引擎(rsyslog)可能难以跟上并可能丢弃消息，显示：
::

    rsyslogd-2177: imuxsock lost 228 messages from pid 2547 due to rate-limiting

可以修改rsyslog服务配置文件 */etc/rsyslog.conf* ，增加如下配置：
::

    # Disable rate limiting
    # (default is 200 messages in 5 seconds; below we make the 5 become 0)
    $SystemLogRateLimitInterval 0

然后重启rsyslog服务，执行命令：
::

    systemctl restart rsyslog.service

访问控制
========

**slapd** 包安装后就具有多种ACL访问控制策略。

查看现有mdb的ACL权限，执行命令：
::

    ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config '(olcDatabase={1}mdb)' olcAccess

显示：
::

    dn: olcDatabase={1}mdb,cn=config
    olcAccess: {0}to attrs=userPassword by self write by anonymous auth by * none
    olcAccess: {1}to attrs=shadowLastChange by self write by * read
    olcAccess: {2}to * by * read

查看现有frontend的ACL权限，执行命令：
::

    ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b  cn=config '(olcDatabase={-1}frontend)' olcAccess

显示：
::

    dn: olcDatabase={-1}frontend,cn=config
    olcAccess: {0}to * by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external
     ,cn=auth manage by * break
    olcAccess: {1}to dn.exact="" by * read
    olcAccess: {2}to dn.base="cn=Subschema" by * read

.. note::
     - **rootDN** 总是具有访问数据库的全部权限，无需包含在任何ACL中
     - **dn.exact** 表示约束这个特定DN的访问

最前面两个ACL至关重要：
::

    olcAccess: {0}to attrs=userPassword by self write by anonymous auth by * none
    olcAccess: {1}to attrs=shadowLastChange by self write by * read

变为易读模式更直观便于理解：
::

    to attrs=userPassword
        by self write
        by anonymous auth
        by * none

    to attrs=shadowLastChange
        by self write
        by * read

ACL强制：

- 为 *userPassword* 属性提供匿名“身份验证”访问权限，以便用户可以进行身份验证或绑定。也许与直觉相反，即使不需要匿名访问DIT，也需要“by anonymous auth"允许通过匿名身份验证，否则这将是鸡和蛋的问题：在身份验证之前，所有用户都是匿名的。
- *by self write* 规则，ACL将 *userPassword* 属性的写入权限授予以属性所在的 *dn* 进行身份验证的用户，也就是将用户可以更新自己的密码。
- *userPassword* 属性对其他用户都是不可访问的，只有 *rootDN* 用户除外，该用户总是具有权限也无需明确声明。
- 为了用户能利用 *passwd* 等其它工具修改自己的密码，用户自己的 *shadowLastChange* 属性需要可写。所有其他目录用户都可以读取此属性的内容。

该DIT可以被匿名用户查询，就在于该ACL中含有 *'to \* by \* read'* ，授予任何人（包括匿名）对其他所有内容的读取权限。
::

    to *
        by * read


如前所述，没有为 *slapd-config* 数据库创建管理帐户（“rootDN”）。但是，有一个 *SASL* 身份被授予对它的完全访问权限。它代表本机 *localhost* 的超级用户 (root/sudo)：
::

    dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth 

显示 *slapd-config* 数据库中的ACL，执行命令：
::

    ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config '(olcDatabase={0}config)' olcAccess

输出：
::

    dn: olcDatabase={0}config,cn=config
    olcAccess: {0}to * by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external
     ,cn=auth manage by * break

由于这是一个SASL身份，因此我们在调用相关LDAP实用程序时需使用SASL机制。它是外部机制，有关示例，请参见前面的命令。

.. note::
    - 需要 *root* 或您必须使用 *sudo* 成为根身份才能使ACL匹配。
    - EXTERNAL机制通过IPC（UNIX 域套接字）工作。这意味着您必须使用 *ldapi* URI格式。

获取所有ACL的简洁方法为执行命令：
::

    ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config '(olcAccess=*)' olcAccess olcSuffix


.. _getreadonly:

生成客户端等获取用户信息的LDAP账户 *readonly*
=============================================

生成LDAP账户 *readonly*
-----------------------

为了安全，采用客户端通过只读账户获取服务端LDAP用户信息进行认证，为此在LDAP服务端设置所需要的LDAP账户 *readonly* ，生成配置文件 *add_readonly.ldif* ：
::

    dn: cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: simpleSecurityObject
    objectClass: organizationalRole
    cn: readonly
    description: for sssd, readonly user
    userPassword: {CRYPT}x

.. note::
    上述账户 *readonly* 的配置与普通系统账户不同，不含有 **dc=People** 等， **objectClass** 也不同等。

执行命令提交修改：
::

    ldapadd -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -f add_readonly.ldif

设定账户 *readonly* 密码，执行命令：
::
    
    ldappasswd -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -S cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

按照提示，输入两次同样的 *readonly* 密码及一次LDAP管理账户 *Maneger* 密码。

.. _setacl:

设置账户 *readonly* ACL只读权限
-------------------------------

建立文件 *readonly-user_access.ldif* ，其内容为：
::

    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    replace: olcAccess
    olcAccess: {0}to attrs=userPassword,shadowLastChange
      by dn="cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" write
      by dn="cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" read
      by self write
      by anonymous auth
      by * none
    olcAccess: {1}to dn.base="" by * read
    olcAccess: {2}to *
      by dn="cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" write
      by dn="cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" read
      by self write
      by anonymous auth
      by * none

启用TLS认证
=========== 

创建认证信息文件 *certinfo.ldif* ，其内容为：
::

    dn: cn=config
    add: olcTLSCACertificateFile
    olcTLSCACertificateFile: /etc/ssl/certs/mycacert.pem
    -
    add: olcTLSCertificateFile
    olcTLSCertificateFile: /etc/ldap/admin22-01_slapd_cert.pem
    -
    add: olcTLSCertificateKeyFile
    olcTLSCertificateKeyFile: /etc/ldap/admin22-01_slapd_key.pem

将上述信息加入LDAP配置，执行命令：
::

    ldapmodify -Y EXTERNAL -H ldapi:/// -f certinfo.ldif

测试，执行命令：
::

    ldapwhoami -x -ZZ -H ldap://admin22-01.hanhai.scc.ustc.edu.cn

如输出如下，则表示正常：
::

    anonymous

高可用性设置：OpenLDAP复制（可选）
=======================================

随着越来越多的网络系统开始依赖LDAP，LDAP服务变得越来越重要。在这样的环境中，标准做法是在LDAP中构建冗余（高可用性），以防止LDAP服务器无响应时造成破坏。这是通过LDAP复制（Replication）完成的。

复制是通过 *Syncrepl* 引擎实现的。这允许使用消费者（Consumer，从服务端、备份服务端、副本服务端） - 提供者（Provider，主服务端）模型同步更改。可以在OpenLDAP管理账户指南及其定义RFC 4533中找到此复制机制的详细说明。

有两种方法可以使用此复制：

- 标准复制：更改的条目被完整地发送给消费者。例如，如果 **uid=hmli,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn** 条目的 **userPassword** 属性更改，则整个条目将发送给消费者。
- 增量复制：仅发送实际更改，而不是整个条目。

增量复制发送更少的数据，但是设置更加复杂。对于两种复制模式，实际数据量都不大，对于多数应用，可以忽略标准复制多发的数据。因此，本文档仅介绍标准复制方式，不介绍增量复制方式。

在设置之前，需要已设置好TLS验证。

为副本节点生成证书
------------------

在主节点上为复制者(replica)（消费者consumer） **admin22-01** 节点建立一个用于存储传输到 **admin22-02** 节点的文件夹 *admin22-02-ssl* ，并生成认证文件 *admin22-02_slapd_key.pem* ，执行命令：
::

    mkdir admin22-02-ssl
    cd admin22-02-ssl
    certtool --generate-privkey  --bits 2048 --outfile admin22-02_slapd_key.pem

生成存储副本节点信息的文件 *admin22-02.info* ，其内容为：
::

    organization = USTC SCC
    cn = admin22-02.hanhai.scc.ustc.edu.cn
    tls_www_server
    encryption_key
    signing_key
    expiration_days = 3650

生成副本节点所需的证书 *admin22-02_slapd_cert.pem* ，执行命令：
::

    certtool --generate-certificate \
    --load-privkey admin22-02_slapd_key.pem \
    --load-ca-certificate /etc/ssl/certs/mycacert.pem \
    --load-ca-privkey /etc/ssl/private/mycakey.pem \
    --template admin22-02.info \
    --outfile admin22-02_slapd_cert.pem

获取CA证书副本存到该目录，执行命令：
::

    cp /etc/ssl/certs/mycacert.pem .

将上述目录 *admin22-02-ssl* 传输到 **admin22-02** 副本节点，执行命令：
::

    cd ..
    scp -r admin22-02-ssl admin22-02:

在 **admin22-02** 副本节点执行下述命令，复制相关文件到对应目录：
::

    cd /root/admin22-02-ssl
    cp admin22-02_slapd_cert.pem admin22-02_slapd_key.pem /etc/ldap/
    chgrp openldap /etc/ldap/admin22-02_slapd_key.pem
    chmod 0640 /etc/ldap/admin22-02_slapd_key.pem
    cp mycacert.pem /usr/local/share/ca-certificates/mycacert.crt
    update-ca-certificates

提供者配置 - 设置用于复制LDAP信息的用户
---------------------------------------

两种复制策略都需要一个复制用户以及更新ACL和有关此用户的限制。

要创建复制用户，请将以下内容保存到文件 *replicator.ldif* 中：
::

    dn: cn=replicator,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: simpleSecurityObject
    objectClass: organizationalRole
    cn: replicator
    description: Replication user
    userPassword: {CRYPT}x

添加上述内容到LDAP，执行命令：
::

    ldapadd -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -f replicator.ldif

设定密码，执行命令（要记住设定的账户 *replicator* 的密码，这里假设为 *replicatorPassword* ）：
::

    ldappasswd -x -D cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn -W -S cn=replicator,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

查看原有的ACL权限，执行命令：
::

    ldapsearch -Q -Y EXTERNAL -H ldapi:/// -LLL -b cn=config '(olcSuffix=dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn)' olcAccess

显示如下：
::

    dn: olcDatabase={1}mdb,cn=config
    olcAccess: {0}to attrs=userPassword,shadowLastChange by dn="cn=Maneger,dc=hanhai
     22,dc=scc,dc=ustc,dc=edu,dc=cn" write by dn="cn=readonly,dc=hanhai,dc=scc,dc=us
     tc,dc=edu,dc=cn" read by self write by anonymous auth by * none
    olcAccess: {1}to dn.base="" by * read
    olcAccess: {2}to * by dn="cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" wr
     ite by dn="cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" read by self 
     write by anonymous auth by * none

生成针对账户 *replicator* 的授权文件 *replicator-acl-limits.ldif* ，其内容为：
::

    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    replace: olcAccess
    olcAccess: {0}to attrs=userPassword,shadowLastChange
      by dn="cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" write
      by dn="cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" read
      by self write
      by anonymous auth
      by * none
    olcAccess: {1}to dn.base="" by * read
    olcAccess: {2}to *
      by dn="cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" write
      by dn="cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" read
      by dn.exact="cn=replicator,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" read
      by self write
      by anonymous auth
      by * break
    -
    add: olcLimits
    olcLimits: dn.exact="cn=replicator,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"
      time.soft=unlimited time.hard=unlimited
      size.soft=unlimited size.hard=unlimited

将配置提交到服务端配置，执行命令：
::

    ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f replicator-acl-limits.ldif

标准复制
--------

提供者节点配置
^^^^^^^^^^^^^^

为了采用 **syncprov** 实现复制，建立文件 *provider_simple_sync.ldif* ，内容为：
::

    # Add indexes to the frontend db.
    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: entryCSN eq
    -
    add: olcDbIndex
    olcDbIndex: entryUUID eq

    # Load the syncprov module.
    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: syncprov

    # syncrepl Provider for primary db
    dn: olcOverlay=syncprov,olcDatabase={1}mdb,cn=config
    changetype: add
    objectClass: olcOverlayConfig
    objectClass: olcSyncProvConfig
    olcOverlay: syncprov
    olcSpCheckpoint: 100 10
    olcSpSessionLog: 100


定制化日志。上面的LDIF有一些参数，应该在将这些参数部署到目录中的生产环境之前查看这些参数。尤其是：

- **olcSpCheckpoint** 、 **olcSpSessionLog** ：请参阅 *slapo-syncprov(5)* 联机帮助页。通常 **olcSpSessionLog** 应等于或最好大于目录中的条目数。

将上述文件内容加入LDAP配置，执行命令：
::

    ldapadd -Q -Y EXTERNAL -H ldapi:/// -f provider_simple_sync.ldif

副本节点配置
^^^^^^^^^^^^

以下在副本节点 *admin22-02* 上操作。

参考 :ref:`installpkg` 安装所需要的软件，配置基础LDAP环境。

启用TLS支持，创建认证信息文件 *certinfo.ldif* ，其内容为：
::

    dn: cn=config
    add: olcTLSCACertificateFile
    olcTLSCACertificateFile: /etc/ssl/certs/mycacert.pem
    -
    add: olcTLSCertificateFile
    olcTLSCertificateFile: /etc/ldap/admin22-02_slapd_cert.pem
    -
    add: olcTLSCertificateKeyFile
    olcTLSCertificateKeyFile: /etc/ldap/admin22-02_slapd_key.pem

将上述信息加入LDAP配置，执行命令：
::

    ldapmodify -Y EXTERNAL -H ldapi:/// -f certinfo.ldif

测试，执行命令：
::

    ldapwhoami -x -ZZ -H ldap://admin22-02.hanhai.scc.ustc.edu.cn

如输出如下，则表示正常：
::

    anonymous

增加其他节点访问数据的用户 *readonly* ，参见 :ref:`getreadonly` 与 :ref:`setacl` 。

.. note::
    增加用户 *readonly* 的步骤务必在下面设置复制前完成，否则将无法在下面配置启用时设置（复制模式不允许）。

生成配置文件 *consumer_simple_sync.ldif* ，其内容为：
::

    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: syncprov

    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: entryUUID eq
    -
    add: olcSyncrepl
    olcSyncrepl: rid=0
      provider=ldap://admin22-01.hanhai.scc.ustc.edu.cn
      bindmethod=simple
      binddn="cn=replicator,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn" 
      credentials="replicatorPassword"
      searchbase="dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn"
      schemachecking=on
      type=refreshAndPersist retry="60 +"
      starttls=critical tls_reqcert=demand
    -
    add: olcUpdateRef
    olcUpdateRef: ldap://admin22-01.hanhai.scc.ustc.edu.cn

确保下面选项具有正确的值：

- provider：提供者服务节点的主机名（在本例中为 *admin22-01.hanhai.scc.ustc.edu.cn* 或IP地址），它必须与提供者的SSL证书中显示的内容相匹配
- binddn：复制节点用户的绑定DN
- credentials：凭据，为复制器用户选择的密码
- searchbase：正在使用的数据库后缀，即要复制的内容
- olcUpdateRef：提供者服务器的主机名或IP地址，如果客户端尝试写入此消费者，则提供给客户端
- rid：Replica ID，一个唯一的3位数，用于标识副本SSSD或nslcd等）

将上述文件内容加入LDAP配置，执行命令：

::

    ldapadd -Q -Y EXTERNAL -H ldapi:/// -f consumer_simple_sync.ldif

可以利用下面命令查看：
::

    ldapsearch -Q -Y EXTERNAL -H ldapi:/// -LLL -b cn=config '(cn=module{0})'
    ldapsearch -Q -Y EXTERNAL -H ldapi:/// -LLL -b cn=config '(olcDatabase={1}mdb)'

用户及组管理
************

默认的LDAP用户、组管理等，需要详细了解LDAP的DIF格式及其命令，对于不熟悉者非常不友好。用户可以自己封装原始的LDAP命令或者采用现成的，常用的有命令行版 **ldapscripts** 及WEB版 **LDAP Account Manager** 等。

简单命令行版：ldapscripts
=========================

安装所需要的包，执行命令：
::

    apt install ldapscripts

编辑文件 */etc/ldapscripts/ldapscripts.conf* ，其需要修改的对应部分内容为：
::

    SERVER=ldap://localhost
    LDAPBINOPTS="-ZZ"
    BINDDN='cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn'
    BINDPWDFILE="/etc/ldapscripts/ldapscripts.passwd"
    SUFFIX='dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn'
    GSUFFIX='ou=Groups'
    USUFFIX='ou=People'

.. note::
    - 调整 **SERVER** 及 **SUFFIX** 选项与您的目录结构对应
    - 强制这里使用 **START_TLS** 方式（ *-ZZ* 参数），必须配置LDAP支持TLS

将账户 *cn=Maneger* 的密码存储到文件 */etc/ldapscripts/ldapscripts.passwd* 。务必注意这里不能包含换行符，可以执行下面命令设置：
::

    echo -n "Yourcn=ManegerPassword" >/etc/ldapscripts/ldapscripts.passwd

上述命令 *echo* 中的 *-n* 参数确保了该行后面没换行符。

设置普通用户不能读取密码文件，执行命令：
::

    chmod 400 /etc/ldapscripts/ldapscripts.passwd

现在即可设置账户，如：

- 创建用户组 *george* ：

::

    ldapaddgroup george

- 创建用户 *george* ：

::

    ldapadduser george george

- 修改用户 *george* 密码：

::

    ldapsetpasswd george

- 删除用户 *george* ：

::

    ldapdeleteuser george

- 删除用户组 *george* ：

::

    ldapdeletegroup george

- 将用户加入另外组 *qa* ：

::

    ldapaddusertogroup george qa

- 将用户 *george* 从组 *qa* 中删除：

::

    ldapdeleteuserfromgroup george qa

- *ldapmodifyuser* 脚本允许增加、删除、修改用户属性，采用的是与命令 *ldapmodify* 相同的语法，执行命令：

::

    ldapmodifyuser george

根据屏幕输出输入下面语法：
::

    # About to modify the following entry :
    dn: uid=george,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    objectClass: account
    objectClass: posixAccount
    cn: george
    uid: george
    uidNumber: 10001
    gidNumber: 10001
    homeDirectory: /home/george
    loginShell: /bin/bash
    gecos: george
    description: User account
    userPassword:: e1NTSEF9eXFsxxxxxxxxxxF1eGUybVdFWHZKRzJVMjFTSG9vcHk=

    # Enter your modifications here, end with CTRL-D.
    dn: uid=george,ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    replace: gecos
    gecos: George Carlin

- *ldapscripts* 一个非常人性化的特性是可以利用模板来定制化用户、用户组等默认属性，为此需要设定使用的模板路径，如在文件 */etc/ldapscripts/ldapscripts.conf* 中设定：

::

    UTEMPLATE="/etc/ldapscripts/ldapadduser.template"

模板可以参考目录 */usr/share/doc/ldapscripts/examples/* 下的，如执行下述命令后再修改：
::

    cp /usr/share/doc/ldapscripts/examples/ldapadduser.template.sample /etc/ldapscripts/ldapadduser.template

如：

::

    dn: uid=<user>,<usuffix>,<suffix>
    objectClass: inetOrgPerson
    objectClass: posixAccount
    cn: <user>
    sn: <ask>
    uid: <user>
    uidNumber: <uid>
    gidNumber: <gid>
    homeDirectory: <home>
    loginShell: <shell>
    gecos: <user>
    description: User account
    title: Employee

其中 **sn** 选项的值采用 *<ask>* ，这将使命令 *ldapadduser* 提示您手动输入该值。

更加易用的WEB版：LDAP Account Manager
=====================================

如果普通用户不想花费时间学习命令行管理LDAP用户，那么可以采用WEB界面等，如 `LDAP Account Manager(LAM) <http://www.ldap-account-manager.org/>`_  或 `phpLDAPadmin(PLA) <http://phpldapadmin.sourceforge.net/>`_ 。个人感觉LAM更加方便，因此该处采用LAM。

在主服务节点上安装所需包，执行命令：

::

    apt install ldap-account-manager

修改 */var/lib/ldap-account-manager/config/lam.conf* ，部分内容修改为如下：
::

    Admins: cn=Maneger,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    Passwd: MyWebPassw0rd #WEB界面右上角修改配置登录密码，非Maneger密码
    tools: treeViewSuffix: dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    defaultLanguage: zh_CN.utf8
    types: suffix_user: ou=People,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    types: suffix_group: ou=groups,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn

WEB访问地址： http://admin主机IP/lam/

- 编辑通用设置：LDAP用户密码复杂度等，对应配置文件 */etc/ldap-account-manager/config.cfg* 等
- 编辑服务器设置：服务器本身设置，访问LDAP服务等设置，对应配置文件 */var/lib/ldap-account-manager/config/lam.conf* 等
- 配置导入导出：导入导出配置文件

客户端设置
**********

SSSD简介
========
如不想每次都访问LDAP服务器获取用户信息，允许本地主机缓存用户信息提高认证速度，降低主LDAP服务器压力，可以配置系统安全服务进程SSSD(System Security Services Daemon)。

普通认证客户节点设置
====================

.. note::
    对于只需要从LDAP服务端获取用户信息进行认证的节点，只需要执行本节操作，其余配置都是针对LDAP服务端的。

安装所需要的包，执行命令：
::

    apt install sssd libpam-sss libnss-sss

设置SSSD的配置文件 */etc/sssd/sssd.conf* ，其内容为：
::

    [sssd]
    services = nss, pam, autofs
    config_file_version = 2
    domains = default

    [nss]

    [pam]
    offline_credentials_expiration = 60

    [domain/default]
    ldap_id_use_start_tls = True
    cache_credentials = True
    ldap_search_base = dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    id_provider = ldap
    auth_provider = ldap
    chpass_provider = ldap
    #access_provider = ldap
    ldap_uri = ldap://admin22-01.hanhai.scc.ustc.edu.cn,ldap://admin22-02.hanhai.scc.ustc.edu.cn
    #ldap_uri = ldap://admin22-01.hanhai.scc.ustc.edu.cn #如果上面只有一个主节点，应采用此行
    ldap_default_bind_dn = cn=readonly,dc=hanhai,dc=scc,dc=ustc,dc=edu,dc=cn
    ldap_default_authtok_type = password
    ldap_default_authtok = readonlypassword
    ldap_tls_reqcert = demand
    ldap_tls_cacert = /etc/ssl/certs/mycacert.pem
    ldap_tls_cacertdir = /etc/ssl/certs
    ldap_search_timeout = 50
    ldap_network_timeout = 60
    ldap_access_order = filter
    #ldap_access_filter = (objectClass=posixAccount)

复制LDAP主节点上的文件 */etc/ssl/certs/mycacert.pem* 到本地文件 */etc/ssl/certs/mycacert.pem* ，也可以类似主节点实际文件为 */usr/local/share/ca-certificates/mycacert.crt* ，并生成软链接： */etc/ssl/certs/mycacert.pem* ，执行命令：
::

    scp admin22-01:/usr/local/share/ca-certificates/mycacert.crt /usr/local/share/ca-certificates/mycacert.crt
    update-ca-certificates

.. note::
    上面需要在文件 */etc/hosts* 中设定主机 *admin22-01* 对应的LDAP服务端的IP地址。

为使得用户登录时自动生成用户主目录，设置PAM认证文件 */etc/pam.d/common-session* ，在 **session optional  pam_sss.so** 下一行添加：
::

    session required    pam_mkhomedir.so skel=/etc/skel/ umask=0077

设置文件 */etc/sssd/sssd.conf* 权限（不允许普通账户查看），执行：
::

    chmod 600 /etc/sssd/sssd.conf

重启SSSD服务，执行命令：
::

    systemctl restart sssd

利用 *id* 等命令可以测试账户是否成功，如装了sssd-tools工具，可以利用命令 *sss_cache -E* 清除sssd用户信息缓存。

参考资料
********

- Ubuntu 22.04 Server：https://ubuntu.com/server/docs/service-ldap-introduction
- Configure SSSD for LDAP Authentication on Ubuntu 22.04：https://kifarunix.com/configure-sssd-for-ldap-authentication-on-ubuntu-22-04/
- openldap介绍和使用：https://www.cnblogs.com/woshimrf/p/ldap.html
