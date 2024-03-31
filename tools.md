# Magisk 工具

Magisk 为开发人员提供了大量安装工具、守护程序和实用程序。本文档涵盖了4个二进制文件和所有包含的小程序。二进制文件和小程序如下所示：

``` txt
magiskboot                 /* binary */
magiskinit                 /* binary */
magiskpolicy               /* binary */
supolicy -> magiskpolicy
magisk                     /* binary */
resetprop -> magisk
su -> magisk
```

## magiskboot

一个用于解压缩/重新打包 boot 映像、解析/修补/解压缩 cpio 、修补 dtb 、十六进制修补二进制文件，以及使用多种算法压缩/解压缩文件的工具。

`magiskboot` 支持常见的压缩格式（这意味着它不依赖于外部工具），包括 `gzip`、`lz4`、`lz4_legacy`（[仅在LG上使用](https://events.static.linuxfound.org/sites/events/files/lcjpcojp13_klee.pdf))、`lzma`、`xz` 和 `bzip2`。

`magiskboot` 的概念是使 boot 映像修改更简单。对于解包，它解析标头并提取映像中的所有部分，如果在任何部分中检测到压缩，则会立即解压缩。对于重新打包，需要原始 boot 映像，以便可以使用原始标头，只需更改必要的内容，如节大小和校验和。如果需要，所有部分将被压缩回原始格式。该工具还支持许多 CPIO 和 DTB 操作。

``` bash
用法: ./magiskboot <操作> [参数...]

支持的操作:
  unpack [-n] [-h] <bootimg>
      将 <bootimg> 解压缩到其各个组件，每个组件到一个文件，并在当前目录中具有相
    应的文件名。
      支持的组件：kernel、kernel_dtb、ramdisk.cpio、second、dtb、extra 和
    recovery_dtbo。
      默认情况下，每个组件将在写入输出文件之前即时自动解压缩。
      如果提供“-n”，则将跳过所有解压缩操作;每个组件将保持不变，以原始格式转储。
      如果提供了“-h”，则启动映像标头信息将转储到文件“header”，该文件可用于
    在重新打包期间修改标头配置。

    返回值：
    0:valid    1:error    2:chromeos

  repack [-n] <原始bootimg> [输出bootimg]
      使用当前目录中的文件将启动映像组件重新打包到 [输出bootimg]，如果未指
    定，则为“new-boot.img”。
      <原始bootimg> 是用于解压缩组件的原始启动映像。
      默认情况下，每个组件将使用 <原始bootimg> 中检测到的相应格式自动压缩。如
    果当前目录中的组件文件已被压缩，则不会对该特定组件执行任何附加压缩。
      如果提供“-n”，则将跳过所有压缩操作。
    如果 env 变量 PATCHVBMETAFLAG 设置为 true，则将设置启动映像的 vbmeta
    header 中的所有禁用标志。

  hexpatch <文件> <hexpattern1> <hexpattern2>
    在 <文件> 中搜索 <hexpattern1>，并将其替换为 <hexpattern2>

  cpio <incpio> [命令...]
    对 <incpio> 执行 cpio 命令（修改已到位）
    每个命令都是一个参数，请为每个命令添加引号。
    支持的命令:
      exists ENTRY
        如果 ENTRY 存在，则返回 0，否则返回 1
      rm [-r] ENTRY
        删除 ENTRY，指定 [-r] 以递归删除
      mkdir MODE ENTRY
        在 MODE 权限下创建目录 ENTRY
      ln TARGET ENTRY
        使用名称 ENTRY 创建指向 TARGET 的符号链接
      mv SOURCE DEST
        将 SOURCE 移动到 DEST
      add MODE ENTRY INFILE
        在 MODE 权限中添加 INFILE 作为 ENTRY；替换 ENTRY（如果存在）
      extract [ENTRY OUT]
        将 ENTRY 解压到 OUT，或将所有条目解压到当前目录
      test
        测试 cpio 的状态
        返回值为0或或运算以下值：
        0x1:Magisk    0x2:unsupported    0x4:Sony
      patch
        应用 ramdisk 补丁
        用 env 变量进行配置：KEEPVERITY KEEPFORCEENCRYPT
      backup ORIG
        从 ORIG 创建 ramdisk 备份
      restore
        从 incpio 中存储的 ramdisk 备份恢复 ramdisk
      sha1
        如果以前已在 ramdisk 中备份，则输出原始引导SHA1

  dtb <文件> <操作> [参数...]
    对 <文件> 执行与 dtb 相关的操作
    支持的操作：
      print [-f]
        打印 dtb 的所有内容以进行调试
        指定 [-f] 以仅打印 fstab 节点
      patch
        搜索 fstab 并删除 verity/avb
        直接对文件进行修改
        使用 env 变量进行配置：KEEPVERITY
      test
        测试 fstab 的状态
        返回值:
        0:valid    1:error

  split <文件>
    将 image.*-dtb 拆分为 kernel + kernel_dtb

  sha1 <文件>
    出 <文件> 的 SHA1 校验和

  cleanup
    清理当前工作目录

  compress[=格式] <输入文件> [输出文件]
    使用 [格式] 将 <输入文件> 压缩为 [输出文件]。
    可以是“-”，以作为 STDIN/STDOUT。
    如果未指定 [格式]，则将使用 gzip。
    如果未指定 [输出文件]，则 <输入文件> 将被替换为
    后缀为匹配文件扩展名的另一个文件。
    支持格式: gzip zopfli xz lzma bzip2 lz4 lz4_legacy lz4_lg 

  decompress <输入文件> [输出文件]
    检测格式并将 <输入文件> 解压缩到 [输出文件]。
    <输入文件>/[输出文件] 可以是“-”，以作为 STDIN/STDOUT。
    如果未指定 [输出文件]，则 <输入文件> 将替换为另一个文件，
    删除其文件格式文件扩展名。
    支持格式: gzip zopfli xz lzma bzip2 lz4 lz4_legacy lz4_lg 
```

## magiskinit

这个二进制文件将替换 Magisk 补丁启动映像的 ramdisk 中的 `init`。它最初是为支持以 system-as-root 的设备而创建的，但该工具被扩展为支持所有设备，并成为 Magisk 的关键部分。更多详细信息可以在 [Magisk 启动过程](details.md#magisk-启动过程) 中的 **预初始化（Pre-Init）** 部分找到。

## magiskpolicy

（此工具别名为 `supolicy`，以与 SuperSU 的 sepolicy 工具兼容）

高级开发人员可以使用此工具修改 SELinux 策略。在像 Linux 服务器管理员这样的常见场景中，他们会直接修改 SELinux 策略源（`*.te`）并重新编译 `sepolicy` 二进制文件，但在 Android 上，我们直接修补二进制文件（或运行时策略）。

Magisk 守护进程派生的所有进程，包括 root shell 及其所有分支，都在上下文 `u:r:magisk:s0` 中运行。所有安装了 Magisk 的系统上使用的规则都可以被视为官方的 `sepolicy` 具有以下补丁：`magiskpolicy --magisk 'allow magisk * * *'`。

``` bash
用法: ./magiskpolicy [--选项...] [策略声明...]

选项:
   --help            显示 policy 语句的帮助消息
   --load FILE       从 FILE 加载 sepolicy
   --load-split      从预编译的 sepolicy 加载或编译拆分的 cil 策略
   --compile-split   编译拆分的cil策略
   --save FILE       将整体策略转储到 FILE 文件
   --live            立即将 sepolicy 加载到内核中
   --magisk          应用内置 Magisk sepolicy 规则
   --apply FILE      应用 FILE 中的规则，作为策略语句逐行读取和分析
                     (允许多重 --apply)

如果既没有指定 --load、--load-split，也没有指定 --compile-split，则它
将从当前活动策略（/sys/fs/selinux/policy）加载


一个策略声明应被视为一个参数，这意味着每个策略声明都应该用引号括起来。
可以在一个命令中提供多个策略语句。

语句的格式为“<rule_name> [args...]”。
标有 (^) 的参数可以接受一个或多个条目。多个条目由大括号 ({}) 中的空格分隔列表组成。
标有 (*) 的参数与 (^) 相同，但另外支持 match-all 运算符 (*)。

示例："allow { s1 s2 } { t1 t2 } class *"
将扩展到：

allow s1 t1 class { all-permissions-of-class }
allow s1 t2 class { all-permissions-of-class }
allow s2 t1 class { all-permissions-of-class }
allow s2 t2 class { all-permissions-of-class }

支持的策略声明：

"allow *source_type *target_type *class *perm_set"
"deny *source_type *target_type *class *perm_set"
"auditallow *source_type *target_type *class *perm_set"
"dontaudit *source_type *target_type *class *perm_set"

"allowxperm *source_type *target_type *class operation xperm_set"
"auditallowxperm *source_type *target_type *class operation xperm_set"
"dontauditxperm *source_type *target_type *class operation xperm_set"
- 唯一支持的操作是“ioctl”
- xperm_set 格式为 “low-high”、“value”或“*”。
  “*”将被视为“0x0000-0xFFFF”。
  所有值应以十六进制书写。

"permissive ^type"
"enforce ^type"

"typeattribute ^type ^attribute"

"type type_name ^(attribute)"
- 参数“attribute”是可选的，默认为“domain”

"attribute attribute_name"

"type_transition source_type target_type class default_type (object_name)"
- 参数“object_name”是可选的

"type_change source_type target_type class default_type"
"type_member source_type target_type class default_type"

"genfscon fs_name partial_path fs_context"
```

## magisk

当使用名称 `magisk` 调用 magisk 二进制文件时，它作为一个实用工具，具有许多助手函数和几个 Magisk 服务的入口点。

``` bash
用法: magisk [小程序 [参数]...]
   或: magisk [选项]...

选项：
   -c                        打印当前二进制版本
   -v                        打印正在运行的守护程序版本
   -V                        打印正在运行的守护程序版本号
   --list                    列出所有可用的小程序
   --remove-modules          移除所有模块并重新启动
   --install-module ZIP      安装模块 zip 文件

高级选项（内部 APIs）：
   --daemon                  手动启动 Magisk 守护进程
   --stop                    移除所有 Magisk 更改并停止守护程序
   --[init trigger]          初始化触发器上的回调。有效的触发器：
                             post-fs-data, service, boot-complete, zygote-restart
   --unlock-blocks           将所有块设备的 BLKROSET 标志设置为 OFF
   --restorecon              恢复 Magisk 文件上的 selinux 上下文
   --clone-attr SRC DEST     克隆权限、所有者和 selinux 上下文
   --clone SRC DEST          克隆 SRC 到 DEST
   --sqlite SQL              执行 SQL 命令到 Magisk 数据库
   --path                    打印 Magisk tmpfs 挂载路径
   --denylist ARGS           拒绝列表配置 CLI

可用的小程序:
    su, resetprop

用法: magisk --denylist [操作 [参数...] ]
操作:
   status          返回强制的状态
   enable          启用拒绝列表强制
   disable         禁用拒绝列表强制
   add PKG [PROC]  将新目标添加到拒绝列表
   rm PKG [PROC]   从拒绝列表中删除目标
   ls              打印当前拒绝列表
   exec CMDs...    在隔离的挂载命名空间中执行命令并执行
                   所有卸载操作
```

## su

MagiskSU 入口点 `magisk` 的小程序。不错的旧 `su` 命令。

``` bash
用法: su [选项] [-] [user [参数...]]

选项:
  -c, --command 命令         将命令传递给调用的 shell
  -h, --help                    显示此帮助消息并退出
  -, -l, --login                将 shell 伪装成一个登录 shell
  -m, -p,
  --preserve-environment        保护整个环境
  -s, --shell SHELL             使用 SHELL 而不是默认的 /system/bin/sh
  -v, --version                 显示版本名并退出
  -V                            显示版本号并退出
  -mm, -M,
  --mount-master                在全局装载命名空间中强制运行
```

::: tip 注意
尽管上面没有列出 `-Z, --context` 选项，但该选项仍然存在，以便与为 SuperSU 设计的应用程序进行 CLI 兼容。然而，该选项被默默地忽略，因为它不再相关。
:::

## resetprop

`magisk` 的小程序。高级系统属性操作实用程序。查看 [Resetprop 详细信息](details.md#重置属性-resetprop) 以了解更多背景信息。

``` bash
用法: resetprop [标志] [选项...]

选项:
   -h, --help        显示此消息
   (no arguments)    输出所有属性
   NAME              获取 NAME 属性
   NAME VALUE        使用 VALUE 设置 NAME 属性
   --file FILE       从 FILE 加载属性
   --delete NAME     删除 NAME 属性

标志:
   -v      将详细输出打印到 stderr
   -n      设置 props 而不经过 property_service
           (此标志仅影响 setprop)
   -p      从/向持久存储读取/写入属性
           (此标志仅影响 getprop 和 delprop)
```

## 参考链接

* [Magisk Tools](https://topjohnwu.github.io/Magisk/tools.html)（官方）
* [Magisk 工具](https://e7kmbb.github.io/Magisk/tools.html)
