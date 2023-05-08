前言
****
Ubuntu 22.04下的PXE自动无人值守安装配置服务设置，原文内容比较简单，基本为翻译的 `Ubuntu 22.04官方手册 <https://ubuntu.com/server/docs>`_ 。

- 服务端提供PXE自动安装服务的信息：
   * 提供DHCPD服务的网卡：enp2s0
   * IP：192.168.22.254
   * 掩码：255.255.255.0
- 客户端（目标对象）网络启动引导的信息：
   * 安装系统名：Cleint1
   * 采用DHCP协议引导的网卡：enp1s0
   * MAC地址：08:00:20:0A:0C:01
   * 自身IP：192.168.22.1
   * 网关IP：192.168.22.254
   * 掩码：255.255.255.0
   * DNS：202.38.64.7

下述带有 **1.、2.** 编号的才是实际执行的步骤，其他都是介绍。

AMD64(x86_64)架构网络安装
*************************

AMD64（也称为x86_64）架构的系统启动既可以采用 **UEFI** 也可以采用传统 **legacy** (“BIOS”) 模式（很多系统可以被配置为采用其中任一模式启动）。确切的细节取决于系统固件，但两种模式都支持 **PXE(“Preboot eXecution Environment”)** 规范，使得可以通过网络来提供启动器引导加载程序启动主机。

两种模式的网络启动实时服务器安装程序的过程相似，如下所示：

- 待安装机器启动，并定向到网络启动。
- **DHCP/bootp** 服务器告诉机器它的网络配置以及从哪里获取引导加载程序。
- 机器的固件通过 **tftp** 协议下载引导加载程序并执行它。
- 引导加载程序通过 **tftp** 协议下载配置，告诉它在哪里下载内核(**kernel**)、 **ramdisk** 和内核命令行以使用。
- **ramdisk** 查看内核命令行以了解如何配置网络以及从何处下载服务器 **ISO** 操作系统安装镜像文件。
- **ramdisk** 下载操作系统 **ISO** 并将其安装为循环设备。
- 从此时起，安装遵循与操作系统 **ISO** 在本地块设备上相同的路径。

**UEFI** 和传统模式之间的区别在于：在 **UEFI** 模式下，引导加载程序是 **EFI** 可执行文件，经过签名以便安全引导 **SecureBoot** 接受，而在传统模式下，它是 **PXELINUX** 。大多数 **DHCP/bootp** 服务器可以配置为为特定机器提供正确的引导加载程序。

设置DHPC/bootp和tftp服务
========================

有几种可用的 **DHCP/bootp** 和 **tftp** 协议实现。本文档将简要介绍如何配置 **dnsmasq** 服务（相比配置 **isc-dhcp-server** 与 **tftpd** 简单的多）以执行这两个角色。

#. 创建存放为客户端提供 **tftp** 服务所需要的文件的目录 */srv/tftp* ： 
    ::

        mkdir /srv/tftp

#. 安装 **dnsmasq** 包，执行命令：
    ::

        apt install dnsmasq

#. 设置文件 */etc/dnsmasq.conf.d/pxe.conf* ，其内容：
    ::

        #设置本服务器提供DHCP服务的网卡
        interface=enp2s0,lo
        bind-interfaces
        #提供DHCP服务的网卡,开始IP,结束IP
        dhcp-range=enp2s0,192.168.22.1,192.168.22.254
        dhcp-boot=pxelinux.0
        #设置采用UEFI模式
        dhcp-match=set:efi-x86_64,option:client-arch,7
        dhcp-boot=tag:efi-x86_64,bootx64.efi
        #启用tftp服务
        enable-tftp
        #设置tftp服务目录
        tftp-root=/srv/tftp
        #设定将要网络安装的MAC地址为08:00:20:0A:0C:01的主机名为Client1，IP为192.168.22.1
        dhcp-host=08:00:20:0A:0C:01,Client1,192.168.22.1
        #设定将要网络安装的MAC地址为08:00:20:0A:0C:02的主机名为Client2，IP为192.168.22.2
        dhcp-host=08:00:20:0A:0C:02,Client2,192.168.22.2

    上面 **enp2s0** 为对客户端安装服务时使用的网卡名（与客户端启动网络安装服务的网卡在同一网络）。

#. 重启 **dnsmasq** 服务，执行命令：
    ::

        systemctl restart dnsmasq

提供引导加载程序和配置
======================

设置所需要的包
--------------

#. 安装对应包
    ::

        apt install cd-boot-images-amd64
#. 生成软链接 */srv/tftp/boot-amd64* 指向 */usr/share/cd-boot-images-amd64* ：

    ::

        ln -s /usr/share/cd-boot-images-amd64 /srv/tftp/boot-amd64

安装WEB服务
-----------

安装时，需要通过 **HTTP** 协议下载所需文件，为此需要配置 **WEB** 服务。

#. 安装所需要包，并重新启动服务：
    ::

        apt install apache2
        systemctl restart apache2

或者不用上述 **apache** 服务，如python3支持http模块，则可直接执行下述命令启动简易 **WEB** 服务：
    ::
        
        python3 -m http.server 80

模式独立设置
------------

#. 针对该次使用 **Ubuntu 22.04 server** 版（代号： **jammy** ），建议有些配置文件放置在目录 */var/www/html/jammy* 下，为此先生成该目录：
    ::

        mkdir /var/www/html/jammy

#. 下载所需要的操作系统 **Live ISO** 镜像到目录 */var/www/html* ：

    ::

        cd /var/www/html/jammy
        wget http://mirrors.ustc.edu.cn/ubuntu-releases/jammy/ubuntu-22.04.1-live-server-amd64.iso

#. 挂载该ISO文件：
      ::

        mount -o loop /var/www/html/jammy/ubuntu-22.04.1-live-server-amd64.iso /mnt

#. 将内核和 **initrd** 从其中复制到 **dnsmasq** 中设置的 **tftp** 的位置：
    ::

        cp /mnt/casper/{vmlinuz,initrd} /srv/tftp/

为 **UEFI** 引导设置文件
------------------------

#. 将签名的 shim 二进制文件复制到位：
    ::

        apt download shim-signed
        dpkg-deb --fsys-tarfile shim-signed*deb | tar x ./usr/lib/shim/shimx64.efi.signed -O > /srv/tftp/bootx64.efi

#. 将签名的 grub 二进制文件复制到位：
    ::

        apt download grub-efi-amd64-signed
        dpkg-deb --fsys-tarfile grub-efi-amd64-signed*deb | tar x ./usr/lib/grub/x86_64-efi-signed/grubnetx64.efi.signed -O > /srv/tftp/grubx64.efi

#. Grub 还需要在 tftp 上可用的字体：
    ::

        apt download grub-common
        dpkg-deb --fsys-tarfile grub-common*deb | tar x ./usr/share/grub/unicode.pf2 -O > /srv/tftp/unicode.pf2

#. 生成文件 */srv/tftp/grub/grub.cfg* ，内容为：
    ::

        set default="0"
        set timeout=0

        if loadfont unicode ; then
          set gfxmode=auto
          set locale_dir=$prefix/locale
          set lang=en_US
        fi
        terminal_output gfxterm

        set menu_color_normal=white/black
        set menu_color_highlight=black/light-gray
        if background_color 44,0,30; then
          clear
        fi

        function gfxmode {
                set gfxpayload="${1}"
                if [ "${1}" = "keep" ]; then
                        set vt_handoff=vt.handoff=7
                else
                        set vt_handoff=
                fi
        }

        set linux_gfx_mode=keep

        export linux_gfx_mode

        menuentry 'Ubuntu 22.04.1' {
                gfxmode $linux_gfx_mode
                linux /vmlinuz $vt_handoff ip=dhcp url=http://192.168.22.254/jammy/ubuntu-22.04.1-live-server-amd64.iso autoinstall ds=nocloud-net\;s=http://192.168.22.254/jammy/ ---
                #注意上面最后面有三个-
                initrd /initrd
        }

    .. note::
       - linux /vmlinuz 行的 *ds=nocloud-net\;s=http* 部分的“;”前面有个转义符“\”，最后有三个“-”
       - 上述配置非常简单。PXELINUX有很多选项，可以在 https://wiki.syslinux.org/wiki/index.php?title=PXELINUX 查阅其文档以获取更多信息。

自动化服务器安装
****************
介绍
====
从 **20.04**  版起，服务器安装程序支持一种新的操作模式：自动安装（简称 **autoinstall** ），此功能也被称为无人值守、甩手或预置安装。

自动安装使得用户可以通过提前配置自动安装配置文件回答所有这些安装配置问题，并让安装过程无需任何交互即可自行运行，适合批量安装多台主机。

与 **debian-installer** 预置安装的区别
======================================
**preseeds** 是基于 **debian-installer** （又名 **d-i** ）自动化安装程序的方法。

Ubuntu新 **autoinstall** 方式的自动安装主要在以下方面不同于 **preseeds** ：

- 格式完全不同（ **cloud-init config** 格式，通常是 **yaml** 格式，vs **debconf-set-selections** 格式）
- 当一个问题的答案不存在于 **preseeds** 中时， **d-i**  停止执行并要求用户输入。 **autoinstalls** 自动安装不是这样的：默认情况下，如果有任何自动安装配置，安装程序会为任何未回答的问题采用默认设置（如没有默认设置则失败）。
    - 用户可以将配置中的特定部分指定为“交互式”，这意味着安装程序仍会停止并询问这些部分。

提供自动安装配置
================

自动安装配置是通过 **cloud-init** 配置提供的，几乎无限灵活。在大多数情况下，最简单的方法是通过 **nocloud** 数据源提供用户数据。

自动安装配置应在配置中的关键字 **autoinstall** 下提供，如：
::

    #cloud-config
    autoinstall:
      version: 1
      ...

运行真正的自动安装
==================

即使找到完全非交互式的自动安装配置，服务器安装程序也会在写入磁盘之前要求确认，除非内核命令行上存在自动安装关键字 **autoinstall** 。这是为了避免意外创建一个U盘，结果该U盘会在主机启动时重新格式化它插入的机器。许多自动安装将通过 **netboot** 完成，其中内核命令行由 **netboot** 配置控制 —— 只需记住将自动安装放在那里！

快速开始
========

如只是想试试看，可以参看页面 https://ubuntu.com/server/docs/install/autoinstall-quickstart 。

创建自动安装配置
================

当服务器安装好Ubuntu操作系统时，会在安装好后的服务器上创建一个可用于重复安装的自动安装文件 */var/log/installer/autoinstall-user-data* ，这可用于做后续需要的文件 *user-data* 的模板。

转换现有 **preseed** 文件
=========================

如已经有了Debian系统的 **preseed** 文件，可利用自动安装生成器 `autoinstall-generator <https://snapcraft.io/autoinstall-generator?_ga=2.223897308.698614161.1664162991-1711338095.1646124381>`_  **snap** 包（snap 是一种包管理器，可用于安装远程snap包）可以帮助将该预置数据转换为自动安装文件。

安装 **autoinstall-generator** 包，执行：
::

    snap install autoinstall-generator

将 **preseed** 文件转换为自动安装格式的基本用法为：
::

    autoinstall-generator my-preseed.txt my-cloud-config.yaml --cloud

有关详细信息，请 `参阅 <https://discourse.ubuntu.com/t/autoinstall-generator-tool-to-help-with-creation-of-autoinstall-files-based-on-preseed/21334?_ga=2.35077545.964202620.1664022245-197937644.1660466546>`_ 。

自动安装配置文件的结构
======================

自动安装配置的完整说明参见：https://ubuntu.com/server/docs/install/autoinstall-reference 。

从技术上讲，虽然配置未定义为文本格式，但 **cloud-init** 配置通常以 **YAML** 形式提供，这是本文档使用的语法。

一个最小的配置是：
::

    version: 1
    identity:
        hostname: hostname
        username: username
        password: $crypted_pass

含有更多特性的配置为：
::

    version: 1
    reporting:
        hook:
            type: webhook
            endpoint: http://example.com/endpoint/path
    early-commands:
        - ping -c1 198.162.1.1
    locale: en_US
    keyboard:
        layout: gb
        variant: dvorak
    network:
        network:
            version: 2
            ethernets:
                enp0s25:
                   dhcp4: yes
                enp3s0: {}
                enp4s0: {}
            bonds:
                bond0:
                    dhcp4: yes
                    interfaces:
                        - enp3s0
                        - enp4s0
                    parameters:
                        mode: active-backup
                        primary: enp3s0
    proxy: http://squid.internal:3128/
    apt:
        primary:
            - arches: [default]
              uri: http://repo.internal/
        sources:
            my-ppa.list:
                source: "deb http://ppa.launchpad.net/curtin-dev/test-archive/ubuntu $RELEASE main"
                keyid: B59D 5F15 97A5 04B7 E230  6DCA 0620 BBCF 0368 3F77
    storage:
        layout:
            name: lvm
    identity:
        hostname: hostname
        username: username
        password: $crypted_pass
    ssh:
        install-server: yes
        authorized-keys:
          - $key
        allow-pw: no
    snaps:
        - name: go
          channel: 1.14/stable
          classic: true
    debconf-selections: |
        bind9      bind9/run-resolvconf    boolean false
    packages:
        - libreoffice
        - dns-server^
    user-data:
        disable_root: false
    late-commands:
        - sed -ie 's/GRUB_TIMEOUT=.*/GRUB_TIMEOUT=30/' /target/etc/default/grub
    error-commands:
        - tar c /var/log/installer | nc 192.168.0.1 1000

许多关键字和值一般直接对应于安装程序提出的问题（例如键盘选择），请参阅相关资料。

错误处理
========

安装过程中的进度及出错信息，可以通过报告 `reporting <https://ubuntu.com/server/docs/install/autoinstall-reference#reporting>`_ 关键字设定的系统获取。此外，当发生致命错误时，将执行错误命令 `error-commands <https://ubuntu.com/server/docs/install/autoinstall-reference#error-commands>`_ 并将回溯打印到控制台，然后服务器等待人为交互式干预。

未来可能的方向
==============

可能希望扩展磁盘的 **match specs** （匹配规范）以涵盖选择磁盘的其他方式。

自动安装快速入门
****************
**cloud-init** 使用以下三种数据并对其进行操作。

- 用户数据 **user-data** ：包含执行无人值守自动安装所需的指令，如要安装的软件包、分区布局、网络配置等。
- 元数据 **meta-data** ：包括与特定数据源关联的数据，如可以包括服务器名称和实例ID等。
- 供应商数据 **vendor-data** ：由组织（如，云提供商）提供的，包括可以自定义镜像以更好地适应镜像运行环境的信息，该种数据为可选。

#. 设置用户数据文件 */var/www/html/jammy/user-data* （可以采用文件 */var/log/installer/autoinstall-user-data* 为模板修改），其内容为：

::

    #cloud-config
    autoinstall:
      version: 1
      late-commands:
        - mkdir /target/root/.ssh && echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZUS2FXGm2yK09Tv0biWIwLszp0TPGzB2u1PD/QXCGPimLOmEP2xAFJbgwzeXlgfnj01LP7BD65lidSPd68WwaUsNHdlikfwf9iRdlSXxNg+PVueY2KPHRzgQ5omtAzJFUUnHKIOSH/Ozo9tmwrs4tZc4CNpx44InvgQY3yeKfhqvVn2O+WazirNlGsFRcpMcjLJjZ+U2OfMGM4l0soSDJKJTKoaHmV0XpIOA2iqfQmOgFDZExqmjc8Vj/1hTBvTzyhgMPlzkDz28UsuBT3U+RZiZJokPtXks5mnVkVYzQvOkln3r0+bmOX0Q4qDg2HhinSpGXAZGln1PKH3AFU7StMIg6e1Lb8AMNHNxWBR61uXIwJJw3dcP6Sy3N4Gq3WW+hkVsHJgZRD+dPQNRwMHIY1CIfBtB9TySKpwNwSAeXiBPFoYgAi0RlbrdHLpjDl9LxK/vyZ1gvGSumtU9IM01EZWfXfCKoMX68ljIaVsHHhHbqIXSS7AjaEsJO+ss8nQU= root@ubuntu22-server.hanhai22.ustc.edu.cn" >/target/root/.ssh/authorized_keys
      apt:
        disable_components: []
        geoip: true
        preserve_sources_list: false
        primary:
        - arches:
          - amd64
          - i386
          uri: http://mirrors.ustc.edu.cn/ubuntu
        - arches:
          - default
          uri: http://ports.ubuntu.com/ubuntu-ports
      drivers:
        install: false
      identity:
        hostname: Client1.hanhai.scc.ustc.edu.cn
        password: $6$dIUrqz$MJN/0cc47EFl8OJBsbxm/o37ScqlZtXb8r63rGa0JLnbwpYQSLrCq7qgEcOAJc6RlkNbIicK6VlPPK402FD7wMQv.sHqNIl1
        realname: hmli
        username: hmli
      kernel:
        package: linux-generic
      keyboard:
        layout: us
        toggle: null
        variant: ''
      locale: en_US.UTF-8
      network:
        ethernets:
          enp1s0:
            addresses:
            - 192.168.22.1/24
            gateway4: 192.168.22.254
            nameservers:
              addresses:
              - 202.38.64.7
              search:
              - ustc.edu.cn
        version: 2
      ssh:
        allow-pw: true
        authorized-keys: []
        install-server: true
      storage:
        config:
        - ptable: gpt
    #      serial: c9d3b07e01624ab69df6
          path: /dev/vda
          wipe: superblock
          preserve: false
          name: ''
          grub_device: false
          type: disk
          id: disk-vda
        - device: disk-vda
          size: 999292928
          wipe: superblock
          flag: boot
          number: 1
          preserve: false
          grub_device: true
          type: partition
          id: partition-0
        - fstype: fat32
          volume: partition-0
          preserve: false
          type: format
          id: format-0
        - device: disk-vda
          size: 1902116864
          wipe: superblock
          flag: ''
          number: 2
          preserve: false
          grub_device: false
          type: partition
          id: partition-1
        - fstype: ext4
          volume: partition-1
          preserve: false
          type: format
          id: format-1
        - device: disk-vda
          size: 18571329536
          wipe: superblock
          flag: ''
          number: 3
          preserve: false
          grub_device: false
          type: partition
          id: partition-2
        - name: ubuntu-vg
          devices:
          - partition-2
          preserve: false
          type: lvm_volgroup
          id: lvm_volgroup-0
        - name: ubuntu-lv
          volgroup: lvm_volgroup-0
          size: 10737418240B
          wipe: superblock
          preserve: false
          type: lvm_partition
          id: lvm_partition-0
        - fstype: ext4
          volume: lvm_partition-0
          preserve: false
          type: format
          id: format-2
        - path: /
          device: format-2
          type: mount
          id: mount-2
        - path: /boot
          device: format-1
          type: mount
          id: mount-1
        - path: /boot/efi
          device: format-0
          type: mount
          id: mount-0
      updates: security

.. note::
     late-commands 等中的字符串在传递给执行安装的主机时，有些特殊字符串需要特殊处理。

2. 生成所需要的文件 *meta-data* （空文件即可），执行命令：

::

    touch /var/www/html/jammy/meta-data

3. 供应商数据文件 *vendor-data* 非必需，可不管。

4. 确保目录 */var/www/html/jammy/* 下的文件对所有人可读：

::

    chmod -R a+r /var/www/html/jammy/

上述各项含义参见： :ref:`reference` 。

.. note::
    做好上述工作后，同一子网内的客户机启动采用PXE引导时，即可自动安装配置所需要的系统。

.. _reference:

自动服务器安装配置文件参考
**************************

整体格式(Overall format)
========================

自动安装 **autoinstall** 文件是 **YAML** 格式的。在顶层，它必须是包含本文档中描述的关键字 **key** 的映射。无法识别的关键字将被忽略。

概要(Schema)
============

自动安装配置在使用前会根据 **JSON** 模式进行转换并进行验证。

命令列表(Command lists)
=======================

几个配置关键字是要执行的命令列表。每个命令可以是字符串（在这种情况下通过 ``sh -c`` 执行）或列表，在这种情况下直接执行。任何以非零返回码退出的命令都被视为错误并中止安装（错误命令除外，它被忽略）。

顶级关键字(Top-level keys)
==========================

- version
    * 名称：版本
    * 类型：整型
    * 默认值：无

    为将来不同版本准备的版本信息，目前只能为 **1** 。

- interactive-sections
    * 名称：交互式部分
    * 类型：字符串列表
    * 默认值：[]

    仍然显示在用户界面 **UI** 中的配置键列表，如：
        ::

            version: 1
            interactive-sections:
             - network
            identity:
             username: ubuntu
             password: $crypted_pass

    交互式操作，将在网络屏幕上停止并允许用户更改默认值。如果为交互式部分提供了值，则将其用作默认值。

    可以使用特殊的块名称 **“*”** 来指示安装程序应该询问所有常见问题——在这种情况下，文件 *autoinstall.yaml* 根本不是真正的“自动安装”文件，而只是一种用于更改用户界面中的默认值的文件。

    并非所有配置关键字都对应于用户界面中的屏幕。该文档指示给定块是否可以交互。

    如果有任何交互块，则忽略报告关键字 **reporting** 。

- early-commands
    * 名称：早期命令
    * 默认值：无
    * 是否可交互：不可

    安装程序启动后立即调用的shell命令列表，特别是在执行探测块和网络设备操作之前执行。自动安装配置在 */autoinstall.yaml* 中可用（不管它是如何被提供），并且在早期命令 **early-commands** 运行后将重新读取该文件，以允许它们在必要时更改配置。

- locale
    * 名称：语言环境
    * 类型：字符串
    * 默认值：en_US.UTF-8
    * 是否可交互：可，对于任何块中是的话，总是可以交互

    为已安装系统配置的语言环境。

- refresh-installer
    * 名称：更新安装
    * 类型：映射
    * 默认值：参见下面
    * 是否可交互：可

    控制安装程序在继续之前是否更新到给定频道 **channel** 中可用的新版本。

    映射包含关键字：

    - update
        * 名称：更新
        * 类型：布尔型
        * 默认值：无
        
        控制是否执行系统更新。

    - channel
        * 名称：频道、通道
        * 类型：字符串
        * 默认值：stable/ubuntu-$REL

        用于检查系统更新的频道。

- keyboard
    * 名称：键盘
    * 类型：映射，参见下面
    * 默认值：US English keyboard
    * 是否可交互：可
    
    任何附属键盘的布局。通常自动安装的系统根本没有键盘，在这种情况下，此处使用的值无关紧要。

    映射的关键字对应于 */etc/default/keyboard* 配置文件中的设置。有关更多详细信息，请参阅其手册页。

    映射包含关键字：

    - layout
        * 名称：布局
        * 类型：字符串
        * 默认值：us

        对应于键盘 **XKBLAYOUT** 设置。

    - variant
        * 名称：变种
        * 类型：字符串
        * 默认值：""

        对应于键盘 **XKBVARIANT** 设置。

    - toggle
        * 名称：切换
        * 类型：字符串或null
        * 默认值：null

        对应于 **grp** 值: 来自键盘 **XKBOPTIONS** 设置的选项的值。可接受的值是（但请注意，安装程序不会验证这些）：caps_toggle、toggle、rctrl_toggle、rshift_toggle、rwin_toggle、menu_toggle、alt_shift_toggle、ctrl_shift_toggle、ctrl_alt_toggle、alt_caps_toggle、lctrl_lshift_toggle、lalt_toggle、lctrl_toggle、lwinshift_toggle

    与20.04 GA一起发布的 **subiquity** 版本由于一个bug不接受该字段为null。

- network
    * 名称：网络
    * 类型：netplan-format映射，参看下面
    * 默认值：用于DHCP协议的名字为eth*en*的网卡
    * 是否可交互：可

    `netplan <https://netplan.io/reference?_ga=2.114730432.698614161.1664162991-1711338095.1646124381>`_ 格式的网络配置。将在安装期间以及已安装的系统中应用。默认是解释安装媒介的配置，它在名称匹配 **“eth*”** 或 **“en*”** 的任何网卡上运行 **DHCPv4** 请求，并随后禁用任何未获取到IP地址的网卡。

    例如，要在特定网卡 **enp0s31f6** 上运行 **dhcp6** 请求：
    ::

        network:
          version: 2
          ethernets:
            enp0s31f6:
              dhcp6: yes

    请注意，由于一个错误，随 **20.04 GA** 发布的 **subiquity** 版本强制您使用额外的 **“network:”** 关键字编写此代码，如下所示：
    ::

        network:
        #多了上一层network:
          network:
            version: 2
            ethernets:
              enp0s31f6:
                dhcp6: yes

    更高版本也支持此语法（以实现兼容性），但如果可以确定采用修复后的新版本，则应使用前者（无需使用额外的 **“network:”** 关键字）。

- proxy
    * 名称：代理
    * 类型：URL或null
    * 默认值：null
    * 是否可交互：可

    在安装期间以及在目标系统中为 **apt** 和 **snapd** 配置的代理，以便访问网络。

- apt
    * 名称：APT高级包管理工具
    * 类型：映射
    * 默认值：参看下面
    * 是否可交互：可

    APT配置，在安装期间和引导到目标系统后都使用。

    这使用与 https://curtin.readthedocs.io/en/latest/topics/apt_source.html 中描述的 **curtin** (the curt installer) 安装格式相同的格式，但有一个扩展：关键字 **geoip** 控制是否完成地理IP(geoip)查找。

    默认值为：
    ::

        apt:
            preserve_sources_list: false
            primary:
                - arches: [i386, amd64]
                  uri: "http://archive.ubuntu.com/ubuntu"
                - arches: [default]
                  uri: "http://ports.ubuntu.com/ubuntu-ports"
            geoip: true

    如关键字 **geoip** 为 **true** 并且要使用的镜像源是默认值（ **http://archive.ubuntu.com/ubuntu** 或 **http://ports.ubuntu.com/ubuntu-ports** 等），则向 **https://geoip.ubuntu.com/lookup** 发出请求，并且将使用的镜像URI更改为 **http://CC.archive.ubuntu.com/ubuntu**  , 其中 **CC** 是查找返回的国家代码（或类似的端口，对于中国为 **CN** ），这将使用所在国家区域的源，提升网络更新速度。如果此部分不是交互式的，则请求会在10秒后超时。

    任何提供的配置都会与默认配置合并，而不是替换它。

    如果您只想设置镜像源，请使用如下配置：
    ::

        apt:
            primary:
                - arches: [default]
                  uri: YOUR_MIRROR_GOES_HERE

    增加一个 **PPA**  源：
    ::

        apt:
            sources:
                curtin-ppa:
                    source: ppa:curtin-dev/test-archive

- storage
    * 名称：存储
    * 类型：映射，参看下面
    * 默认值：对于单块硬盘为 **lvm** ，对于多块硬盘则无默认值
    * 是否可交互：可

    存储配置是一个复杂的话题，自动安装文件中所需配置的描述也可能很复杂。安装程序支持“布局layouts”，即表达常见配置的简单方法。

    - 支持的布局

        目前支持的布局就两种，分别是逻辑卷模式 **lvm** 和直通模式 **direct** 。
        ::

            storage:
              layout:
                name: lvm
            storage:
              layout:
                name: direct

        默认情况下，这些将安装到系统中容量最大的磁盘，但您可以提供匹配规范（“match: {}”，见下文）来指示要使用的磁盘：
        ::

            storage:
              layout:
                name: lvm
                match:
                  serial: CT*
            storage:
              layout:
                name: disk
                match:
                  ssd: yes

        （可以用“match: {}”来匹配任意磁盘）

        默认采用 **lvm** 。

    - 基于动作的配置

        为了获得完全的灵活性，安装程序允许使用一种语法完成存储配置，该语法是 **curtin** 支持的语法的超集，在 https://curtin.readthedocs.io/en/latest/topics/storage.html 中进行了描述。

        如果使用 **layout** 功能配置磁盘，则不会使用 **config** 部分。

        除了将操作列表放在关键字 **config** 下之外， `grub <https://curtin.readthedocs.io/en/latest/topics/config.html#grub>`_ 和 `swap <https://curtin.readthedocs.io/en/latest/topics/config.html#swap>`_ curtin配置项也可以放在此处。因此存储部分可能如下所示：
        ::

            storage:
                swap:
                    size: 0
                config:
                    - type: disk
                      id: disk0
                      serial: ADATA_SX8200PNP_XXXXXXXXXXX
                    - type: partition
                      ...

        Curtin语法的扩展围绕磁盘选择和分区/逻辑卷大小调整。

    - 磁盘选择扩展

        Curtin支持通过串行（如 *Crucial_CT512MX100SSD1_14250C57FECE* ）或路径（如 */dev/sdc* ）识别磁盘，服务器安装程序也支持这一点。安装程序还支持磁盘操作上的 **match spec** ，支持更灵活的匹配。

        存储配置中的操作按照它们在自动安装文件中的顺序进行处理。任何磁盘操作都会被分配一个匹配的磁盘——如果有多个磁盘，则从一组未分配的磁盘中任意选择，如果没有未分配的匹配磁盘，则会导致安装失败。

        匹配规范支持以下关键字：
        - **model: foo** ：匹配 **udev** 中 **ID_VENDOR=foo** 的磁盘，支持通配符
        - **path: foo** ：匹配 **udev** 中 **DEVPATH=foo** 的磁盘，支持通配符（通配符支持将此与直接在磁盘操作中指定 **path: foo** 区分开来）
        - **serial: foo** ：匹配 **udev** 中 **ID_SERIAL=foo** 的磁盘，支持通配符（通配符支持将此与直接在磁盘操作中指定 **serial: foo** 区分开来）
        - **ssd: yes|no** ：匹配是或不是 **SSD** 的磁盘（相对于机械硬盘）
        - **size: largest|smallest** ：如果有多个匹配项，则取最大或最小的磁盘而不是任意一个（在 20.06.1 版本中添加了对最小 **smallest** 的支持）

        因此，例如，要匹配任意磁盘，只需：
        ::

            - type: disk
              id: disk0

        匹配容量最大的SSD硬盘：
        ::

            - type: disk
              id: big-fast-disk
              match:
                ssd: yes
                size: largest

        匹配希捷Seagate硬盘：
        ::

         - type: disk
           id: data-disk
           match:
             model: Seagate

    - 分区/逻辑卷扩展

       curtin中的分区或逻辑卷的大小指定为字节数。自动安装配置更加灵活：
           - 您可以使用安装程序用户界面中支持的“1G”、“512M”语法指定大小
           - 您可以将大小指定为包含磁盘（或RAID）的百分比，例如“50%”
           - 对于为特定设备指定的最后一个分区，您可以将大小指定为“-1”以指示该分区应填充剩余空间。

       ::

         - type: partition
           id: boot-partition
           device: root-disk
           size: 10%
         - type: partition
           id: root-partition
           size: 20G
         - type: partition
           id: data-partition
           device: root-disk
           size: -1

- identity
    * 名称：身份
    * 类型：映射，参看下面
    * 默认值：无
    * 是否可交互：可

    配置系统的初始用户。这是唯一必须存在的配置关键字（除非存在用户数据部分，在这种情况下它是可选的）。

    可以包含关键字的映射，所有关键字都采用字符串值：
     - realname：实际名
     - username：用户名
     - hostname： 主机名
     - password：密码，加密的。这是与 sudo 一起使用时所必需的，即使配置了 SSH 访问。

- ssh
    * 名称： **SSH** 服务
    * 类型：映射，参看下面
    * 默认值：参看下面
    * 是否可交互：可

    为已安装的系统配置 **SSH** 服务。可以包含关键字的映射：

    - install-server
        * 名称：安装 **SSH** 服务
        * 类型：布尔型
        * 默认值：false，不安装

        是否安装 **OpenSSH** 服务。

    - authorized-keys
        * 名称： **SSH** 认证公钥
        * 类型：字符串列表
        * 默认值：[]

        要安装在初始用户帐户中的 SSH 公钥列表，方便其他主机采用密钥通过 **SSH** 访问该主机。

    - allow-pw
        * 名称：是否允许密码
        * 类型：布尔型
        * 默认值：当 **authorized_keys** 为空时为真 **true** ，否则为否 **false**

- snaps
    * 名称：snap包
    * 类型：列表
    * 默认值：不安装其他snap
    * 是否可交互：可

    要安装的snap包列表。每个snap都表示为具有必需的关键字 **name** 和可选的关键字 **chanel** （默认为 **stable** ）和 **classic** （经典默认为 **false** ）的映射。如：
    ::

        snaps:
            - name: etcd
              channel: edge
              classic: false

- debconf-selections
    * 名称：采用 **debconf** 设置包选择
    * 类型：字符串
    * 默认值：无
    * 是否可交互：否

    安装程序将使用 **debconf** 设置选择值更新目标。用户需要熟悉软件包 **debconf** 选项。

- packages
    * 名称：软件包
    * 类型：列表
    * 默认值：无软件包
    * 是否可交互：否

    要安装到目标系统中的软件包列表。更准确地说，是传递给命令 ``apt-get install`` 的字符串列表，因此这包括任务选择（ **dns-server^** ）和安装特定版本的包（ **my-package=1-1** ）。

- late-commands
    * 名称：后期命令
    * 类型：命令列表
    * 默认值：无命令
    * 是否可交互：否

    在安装成功完成并安装任何更新和软件包之后运行的 Shell 命令，就在系统重新启动之前。它们在安装程序环境中运行，已安装的系统安装在目录 */target* 。您可以运行 ``curtin in-target -- $shell_command`` （使用20.04 GA发布的subiquity安装程序版本，采用 **curtin**  格式，您需要将其指定为 ``curtin in-target --target=/target -- $shell_command`` ）以在目标系统中运行（类似了解如何在 **d-i preseed/late_command** 中使用简单的目标内）。

- error-commands
    * 名称：出错处理命令
    * 类型：命令列表
    * 默认值：无命令
    * 是否可交互：否
    
    安装失败后运行的Shell命令。它们在安装程序环境中运行，并且目标系统（或安装程序设法配置的尽可能多的系统）将安装在目录 */target* 。日志将在实时会话中的目录  */var/log/installer* 中可用。

- reporting
    * 名称：报告
    * 类型：映射
    * 默认值： **type: print** ，导致tty1和任何已配置的串行控制台上的输出
    * 是否可交互：否

    安装程序支持向各种目的地报告进度。请注意，如果有任何交互部分，则忽略此部分；它仅适用于全自动安装。

    配置，实际上实现，与\ `curtin使用 <https://curtin.readthedocs.io/en/latest/topics/reporting.html>`_\ 的90%相同。

    配置中报告映射中的每个键都定义了一个目标，其中子关键字 **type** 是以下之一：

    （原文在该处有 **The rsyslog reporter does not yet exist** ，没理解为什么在这个位置说这个，也不清楚什么含义）

        - **print** ：在tty1和任何配置的串行控制台上打印进度信息。没有其他配置。
        - **rsyslog** ：通过rsyslog报告进度。目标键指定将输出发送到何处。
        - **webhook** ：通过将JSON报告发布到 URL 来报告进度。接受与curtin相同的配置。
        - **none** ：不报告进度。仅对禁止默认输出有用。

    例子：

    默认配置：
    ::

        reporting:
         builtin:
          type: print

    输出到 **rsyslog** ：
    ::

        reporting:
         central:
          type: rsyslog
          destination: @192.168.0.1

    抑制默认输出：
    ::

        reporting:
         builtin:
          type: none

    输出到 **curtin** 样式的 **webhook** ：
    ::

        reporting:
         hook:
          type: webhook
          endpoint: http://example.com/endpoint/path
          consumer_key: "ck_foo"
          consumer_secret: "cs_foo"
          token_key: "tk_foo"
          token_secret: "tk_secret"
          level: INFO

- user-data
    * 名称：用户数据
    * 类型：映射
    * 默认值：{}
    * 是否可交互：否

    提供 **cloud-init** 用户数据，它将与安装程序生成的用户数据 **user-data** 合并。如果您提供此信息，则无需提供身份 ** identity** 部分（但您有责任确保您可以登录到已安装的系统！）。

参考
****

- 网络启动： https://ubuntu.com/server/docs/install/netboot-amd64
- 自动安装： https://ubuntu.com/server/docs/install/autoinstall
