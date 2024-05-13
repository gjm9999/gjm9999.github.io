# auto_assert说明文档

## 前言

不想写不会写断言，又想借助断言检查及早检查隐藏的问题加速定位。

对于这种既要又要的需求...是时候搬出auto_assert脚本了。

有了他，写断言不用一句话，各种检查哗啦啦的来呀！

## 工程路径

[src/auto_assert.py · 尼德兰的喵/rtl_note_script - Gitee.com](https://gitee.com/gjm9999/rtl_note_script/blob/master/src/auto_assert.py)

## 更新记录

| 时间  | 更新  | 说明  |
| --- | --- | --- |
|     |     |     |

## 功能列表

1. 仿真过程中的不定态检查；
2. 仿真过程中条件使能时的不定态检查；
3. 仿真过程中的取值范围检查；
4. 仿真过程中的条件使能时的取值范围检查；
5. 仿真结束后的信号值检查；

## 使用说明

脚本工作于linux环境，建议在gvim中使用，在.vimrc的尾部增加：：

```
command! AS :execute '%!  {script_path}/auto_assert.py %'
```

之后在gvim中键入:AS即可刷新文件。

脚本支持的语法适用范围：input/output/wire/reg，基本语法如下：

**1.仿真过程中的不定态检查**

适用信号：valid/ready/配置信息

```
input ddr_mst_link_valid;//ASSERT: chk_xz
```

**2.仿真过程中条件使能时的不定态检查**

适用信号：随路信息

```
input  [CHANNEL_NUM -1:0]ddr_mst_link_strb ;//ASSERT: chk_xz while ddr_mst_link_valid
```

**3.仿真过程中的取值范围检查**

适用信号：配置信息/状态机/计数器等

```
input  [CHANNEL_NUM -1:0]ddr_mst_tid_exist ;//ASSERT: chk_xz while ddr_mst_link_valid; == [0~10]
input  [CHANNEL_NUM -1:0]ddr_mst_tid_repeat;//ASSERT:  == [5~(ddr_mst_tid_exist-2)]
```

**4.仿真过程中的条件使能时的取值范围检查**

适用信号：随路信息

```
input  [CHANNEL_NUM -1:0]ddr_mst_tid_repeat;//ASSERT:  == [5~(ddr_mst_tid_exist-2)] while ddr_mst_link_valid
```

**5.仿真结束后的信号值检查**

适用信号：valid/ready/计数器/状态机等

```
input  [CHANNEL_NUM -1:0]ddr_mst_tid_exist ;//ASSERT: chk_xz while ddr_mst_link_valid; == [0~10]; END = 10
```

## 使用示例

以如下代码为例（仅为举例，不是实际检查约束）： 

```
input ddr_mst_link_valid;//ASSERT: chk_xz
output  mst_ddr_link_ready;//ASSERT: chk_xz
input [CHANNEL_NUM -1:0]ddr_mst_link_strb ;//ASSERT: chk_xz while ddr_mst_link_valid
input [TID_WIDTH*CHANNEL_NUM   -1:0]ddr_mst_tid       ;//ASSERT: chk_xz while ddr_mst_link_valid
input  [INDEX_WIDTH*CHANNEL_NUM -1:0]ddr_mst_tid_index ;//ASSERT: chk_xz while ddr_mst_link_valid
input [CHANNEL_NUM -1:0]ddr_mst_tid_exist ;//ASSERT: chk_xz while ddr_mst_link_valid; == [0~10]; END = 10
input [CHANNEL_NUM -1:0]ddr_mst_tid_repeat;//ASSERT: chk_xz while ddr_mst_link_valid
input ddr_mst_link_last ;//ASSERT: chk_xz while ddr_mst_link_valid
```

如果.vimrc配置完成了，则在gvim打开的文件内键入:AS后，会在endmodule上方填充如下代码：

```
`ifdef AUTO_ASSERT_ON // AUTO ADD

wire assert_clk   = clk;
wire assert_rst_n = rst_n;

reg after_rst = 1'b0;
initial begin
    wait(assert_rst_n == 1'b0);
    wait(assert_rst_n == 1'b1);
    repeat(10) @(posedge assert_clk);
    after_rst = 1'b1;
end

always @(posedge assert_clk)
    if(assert_rst_n == 1'b0) after_rst <= 1'b1;

property chk_xz(info);
    @(posedge assert_clk) disable iff(~after_rst)
    ~$isunknown(info);
endproperty

property chk_xz_valid(info, valid);
    @(posedge assert_clk) disable iff(~after_rst)
    valid |-> ~$isunknown(info);
endproperty


//CHACK SIGNAL X and Z
assert property (chk_xz_valid(ddr_mst_link_strb, ddr_mst_link_valid)) else $error("ddr_mst_link_strb has xz error");
assert property (chk_xz_valid(ddr_mst_tid, ddr_mst_link_valid)) else $error("ddr_mst_tid has xz error");
assert property (chk_xz_valid(ddr_mst_tid_exist, ddr_mst_link_valid)) else $error("ddr_mst_tid_exist has xz error");
assert property (chk_xz_valid(ddr_mst_link_last, ddr_mst_link_valid)) else $error("ddr_mst_link_last has xz error");
assert property (chk_xz(ddr_mst_link_valid)) else $error("ddr_mst_link_valid has xz error");
assert property (chk_xz(mst_ddr_link_ready)) else $error("mst_ddr_link_ready has xz error");
assert property (chk_xz_valid(ddr_mst_tid_repeat, ddr_mst_link_valid)) else $error("ddr_mst_tid_repeat has xz error");
assert property (chk_xz_valid(ddr_mst_tid_index, ddr_mst_link_valid)) else $error("ddr_mst_tid_index has xz error");

//CHACK SIGNAL VALUE
property chk_ddr_mst_tid_exist_value(ddr_mst_tid_exist);
    @(posedge assert_clk) disable iff(~after_rst)
    ((ddr_mst_tid_exist >= 0) && (ddr_mst_tid_exist <= 10));
endproperty
chk_ddr_mst_tid_exist_value_assert: assert property (chk_ddr_mst_tid_exist_value(ddr_mst_tid_exist)) else $assertoff(0, chk_ddr_mst_tid_exist_value_assert);



//CHACK SIGNAL END VALUE
property chk_ddr_mst_tid_exist_end(ddr_mst_tid_exist);
    @(posedge assert_clk) disable iff(~after_rst | harness.end_of_check == 1'b0)
    ddr_mst_tid_exist == 10;
endproperty
chk_ddr_mst_tid_exist_end_assert: assert property (chk_ddr_mst_tid_exist_end(ddr_mst_tid_exist)) else $assertoff(0, chk_ddr_mst_tid_exist_end_assert);


`endif //AUTO ADD AUTO_ASSERT_ON
```

在Makefile或用例配置中增加compile宏定义：

+define+AUTO_ASSERT_ON

如果涉及到end_of_check，请在harness中补充end_of_check信号，并在main_phase之后由环境主动将harness.end_of_check置1。

之后正常跑用例即可，发生断言问题时会自动报错。
