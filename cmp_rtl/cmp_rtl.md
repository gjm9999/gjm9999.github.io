# cmp_rtl说明文档

## 前言

cmp_rtl顾名思义就是用来快速编译rtl代码的脚本。

## 工程路径

[cmp_rtl · 尼德兰的喵/myscript_python - Gitee.com](https://gitee.com/gjm9999/myscript_python/blob/master/cmp_rtl)

## 更新记录

| 时间       | 更新                                             | 说明  |
| -------- | ---------------------------------------------- | --- |
| 2024/2/1 | 1.将-t作为可选传参<br>2.增加-sv为可选传参<br>3.增加-verdi为可选传参 |     |

## 功能列表

1.根据filelist对RTL代码进行编译；

2.编译后通过verdi打开代码层次；

## 使用说明

```
usage: cmp_rtl [-h] [-o] [-sv] [-verdi] [-t T] [-f F]

argparse info

optional arguments:
  -h, --help  show this help message and exit
  -o          open this script
  -sv         sv mode
  -verdi      open verdi
  -t T        top_module
  -f F        filelist
```

## 使用示例

cmp_rtl -f filslist：编译RTL代码

cmp_rtl -f filslist -sv：以systemverilog模式编译

cmp_rtl -f filslist -t top：指定顶层

cmp_rtl -f filslist -verdi：编译完成后打开verdi查看代码

cmp_rtl -f filslist -t top -sv -verdi：指定顶层，以systemverilog模式编译RTL代码，编译完成后打开verdi
