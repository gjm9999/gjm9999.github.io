# auto_testbench说明文档

## 前言

在我年轻的时候，在例化了几个近百行接口的model，声明了还几百行的接口和wire后，精神就已经恍惚了，一直恍惚到今天。于是后来我尝试了各种办法来简化这个过程，包括生成简单的例化代码，通过VBA做例化文件等等，但是使用无法解决还是需要手动修改和连线的问题。

终于有一天我使用了verilog-mode工具完美的解决了这个问题，但是居安思危万一哪天这个工具用不了了呢（当时不知道这个是开源的），所以我就想着自己做一个类似的工具来尝试做同样的事。

## 工程路径

[src/gen_link.py · 尼德兰的喵/rtl_note_script - Gitee.com](https://gitee.com/gjm9999/rtl_note_script/blob/master/src/gen_link.py)

## 更新记录

| 时间  | 更新  | 说明  |
| --- | --- | --- |
|     |     |     |

## 功能列表

脚本只实现模块的互连以及信号声明功能。

## 使用说明

下载脚本后，在.vimrc中补充：

```
command! L :execute '%! 你的路径/gen_link -f %'
command! D :execute '%! /home/xiaotu/my_work/gen_link/gen_link -d -f %'
```

之后在vim打开的文件里键入:L进行模块互连，:D删除互连生成对的代码。

## 使用示例

以tinyrisc地高层的代码为例，展示如何使用脚本，顶层代码在如下目录：

[prj/rtl/tyrc_core.v · 尼德兰的喵/tinyrisc_demo - Gitee.com](https://gitee.com/gjm9999/tinyrisc_demo/blob/master/prj/rtl/tyrc_core.v)
