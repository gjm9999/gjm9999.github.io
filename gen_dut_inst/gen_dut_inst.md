# gen_dut_inst说明文档

## 前言

在快速验证的需求场景下，我一般通过之前做的三个脚本来搭环境，即auto_testbench做unit level test，gen_uvm_agent+gen_uvm_tb做block level test。而相较于auto_testbench和gen_uvm_agent的生成即可用模式，gen_uvm_tb做的是比较粗糙的，只能简单的搭起一个框架需要手动补充很多的东西。而即使搭起的这个简单框架里，也还缺少一个比较重要的组成部分，就是dut的例化和连线。在目前的gen_uvm_tb脚本中，生成的代码借助了verilog-mode来进行dut的例化，而与agent之间的连线则需要手动来处理：

```
// ----------------------------------------------------------------
// AUTO declare and inst
// ----------------------------------------------------------------
/*AUTOOUTPUT*/
/*AUTOINPUT*/
/*AUTOLOGIC*/

rr_dispatch #(/*AUTOINSTPARAM*/)
u_dut(/*AUTOINST*/);

// ----------------------------------------------------------------
// assign
// ----------------------------------------------------------------
```

因此很长时间我都有补充dut例化和连线代码的想法，不过连线本身是没有一定规则，和agent的对应关系也极大地依赖编写者的习惯，故而就迟迟没有动手。而这次因为想到了一个还算可以的方案，因此独立做了gen_dut_inst脚本来作为gen_uvm_tb的补充，以完善该功能同时避免影响主脚本的代码生成。

## 工程路径

[gen_dut_inst · 尼德兰的喵 - Gitee.com](https://gitee.com/gjm9999/gen_uvm_tb/blob/master/src/gen_dut_inst)

## 更新记录

| 时间         | 更新               | 说明  |
| ---------- | ---------------- | --- |
| 2024/11/30 | 补充gen_dut_inst脚本 |     |



## 功能列表

1. 根据tb.cfg和sim.list生成dut inst文件。

## 使用说明

脚本工作于linux环境，下载工程后执行命令：

```
${script_path}/gen_uvm_tb ${filelist_path}/sim.list gen_tb.cfg outp_link.sv
```

sim.list即编译的filelist，脚本会基于${filelist_path}进行自动遍历和解析；

gen_tb.cfg就是gen_uvm_tb使用的cfg文件，其中有agetn和rtl的类名和例化名；

outp_link.sv是输出文件路径，执行后生成的dut inst会写入此文件

## 使用示例
