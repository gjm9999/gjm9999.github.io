# auto_testbench说明文档

## 前言

这个脚本最原始的需求来源，是对generate for进行循环展开。为什么会有这个需求呢，因为公司不推荐使用generate-for语法。为什么不推荐使用generate-for语法呢？

Verilog 的 generate-for 语法是一种用于在设计中生成重复结构的语法。虽然在某些情况下它可以提高代码的灵活性和可维护性，但也有一些原因解释为什么一些芯片设计公司可能不推荐或避免使用这种语法。

1. **可读性和可维护性：** 使用 generate-for 语法的代码可能会变得比较复杂，降低了代码的可读性和可维护性。尤其是当生成的代码嵌套层次较多时，很难理解和调试。
2. **综合工具支持：** 一些综合工具对 generate-for 语法的支持可能不如对传统结构化 Verilog 语法的支持好。这可能导致一些综合工具无法正确处理或优化使用 generate-for 语法的代码。
3. **性能和面积：** 生成的代码可能在性能和面积方面不如手动编写的代码优化。设计工程师可能更倾向于手动控制生成的结构，以便更好地满足设计的性能和面积要求。
4. **教育和经验：** 一些设计团队可能更习惯于传统的 Verilog 编写方式，而不太愿意在项目中引入新的语法，尤其是当设计团队的经验主要集中在传统的 Verilog 编码上时。
5. **工具和流程一致性：** 一些公司可能要求在整个设计流程中保持一致性，包括仿真、综合、布局和布线。如果某些工具在处理 generate-for 语法时存在不一致性，可能会影响整个设计流程的一致性。

总体而言，虽然 generate-for 语法可以提供一些灵活性，但在实际应用中需要谨慎使用，特别是在需要强调可读性、可维护性和工具兼容性的项目中。设计团队通常会根据具体的项目需求和团队经验来决定是否使用 generate-for 语法。

generate-for这个事其实完全是团队选择，做不做工具两可。

还有一些场景，比如这种代码：
    assign out_sig = (sel == 3'd0) ? in_sig0 :
                     (sel == 3'd1) ? in_sig1 :
                     ...
                     (sel == 3'd30) ? in_sig30 :
                                      in_sig31 ;

为了避免手改带来的麻烦，有一个工具支持下其实也非常的欢迎。

## 工程路径

[src/auto_unfold.py · 尼德兰的喵/rtl_note_script - Gitee.com](https://gitee.com/gjm9999/rtl_note_script/blob/master/src/auto_unfold.py)

## 更新记录

| 时间  | 更新  | 说明  |
| --- | --- | --- |
|     |     |     |

## 功能列表

1. 支持循环范围大小随意；
2. 支持多行生成；
3. 支持多循环变量嵌套；
4. 支持参数；
5. 支持循环变量运算；
6. 支持IF条件语句；

## 使用说明

将脚本下载到工作站后，在.vimrc中添加：

```
command! U  :execute '%! 你的路径/auto_unfold.py -f %'
command! UD :execute '%! 你的路径/auto_unfold.py -d -f %'
```

之后在vim打开的文件中，键入:U展开循环，:UD删除循环。

## 使用示例

脚本识别的典型注释是这样的：

```
/*AUTO_UNFOLD
#for i 1..4
wire [15:0] sig#i#;
END*/
```

/* AUTO_UNFOLD ~ END */之间是循环展开的主体，i为循环变量，1和4为上下界均为包含模式。在vim中键入:U后会展开为：

```
/*AUTO_UNFOLD
#for i 1..4
wire [15:0] sig#i#;
END*/
//AUTO_UNFOLD_START
wire [15:0] sig1;
wire [15:0] sig2;
wire [15:0] sig3;
wire [15:0] sig4;
//AUTO_UNFOLD_END
```

键入:UD后会删除所有//AUTO_UNFOLD_START ~ //AUTO_UNFOLD_END之间的生成内容，恢复原本的文本。注意在复制循环去修改的时候，千万别把//AUTO_UNFOLD_START也复制走，要不一展开就把后面的RTL都吃了。

上下界可以从大到小，从小到大，相等也没事（不像python事那么多）：

```
/* AUTO_UNFOLD
#for i 1..1
wire [15:0] sig#i#;
END */
//AUTO_UNFOLD_START
wire [15:0] sig1;
//AUTO_UNFOLD_END

/* AUTO_UNFOLD
#for i 5..1
wire [15:0] sig#i#;
END */
//AUTO_UNFOLD_START
wire [15:0] sig5;
wire [15:0] sig4;
wire [15:0] sig3;
wire [15:0] sig2;
wire [15:0] sig1;
//AUTO_UNFOLD_END
```

生成项可以有多行，每行都是独立的：

```
/* AUTO_UNFOLD
#for i 5..1
wire [15:0] sig#i#;
wire [15:0] reg#i#;

END */
//AUTO_UNFOLD_START
wire [15:0] sig5;
wire [15:0] reg5;

wire [15:0] sig4;
wire [15:0] reg4;

wire [15:0] sig3;
wire [15:0] reg3;

wire [15:0] sig2;
wire [15:0] reg2;

wire [15:0] sig1;
wire [15:0] reg1;

//AUTO_UNFOLD_END
```

还能支持多个循环变量嵌套，嵌套时先循环上方的变量，想想都可怕：

```
/* AUTO_UNFOLD
#for i 2..1
#for j 5..5
#for k 4..3
wire [15:0] k#k#_sig#j#_sub#i#;
END */
//AUTO_UNFOLD_START
wire [15:0] k4_sig5_sub2;
wire [15:0] k4_sig5_sub1;
wire [15:0] k3_sig5_sub2;
wire [15:0] k3_sig5_sub1;
//AUTO_UNFOLD_END
```

然后呢竟然还能支持参数，感动 o(TωT)o 。不过只能支持localparam，同时要求localparam的格式很固定。因为localparam是本地的参数，原则上不能被修改只能是一个固定的值。如果有运算式或者和parameter关联了，脚本无法解析了：

```
localparam LOOP_I_MIN = 3, LOOP_I_MAX = 5;
localparam LOOP_J_MIN = 2;

/* AUTO_UNFOLD
#for i LOOP_I_MIN..LOOP_I_MAX
#for j LOOP_J_MIN..5
wire [15:0] sig#j#_sub#i#;
END */
//AUTO_UNFOLD_START
wire [15:0] sig2_sub3;
wire [15:0] sig2_sub4;
wire [15:0] sig2_sub5;
wire [15:0] sig3_sub3;
wire [15:0] sig3_sub4;
wire [15:0] sig3_sub5;
wire [15:0] sig4_sub3;
wire [15:0] sig4_sub4;
wire [15:0] sig4_sub5;
wire [15:0] sig5_sub3;
wire [15:0] sig5_sub4;
wire [15:0] sig5_sub5;
//AUTO_UNFOLD_END
```

参数只能在出现在两个位置：循环里和一会提到的IF中，下面很快就会看到了。脚本还支持简单的运算，当然了运算只能是在几个变量间进行运算哈：

```
/* AUTO_UNFOLD
#for i LOOP_I_MIN..LOOP_I_MAX
#for j LOOP_J_MIN..5
wire [15:0] sig#i%j#_#j#_sub#i#;
END */
//AUTO_UNFOLD_START
wire [15:0] sig1_2_sub3;
wire [15:0] sig0_2_sub4;
wire [15:0] sig1_2_sub5;
wire [15:0] sig0_3_sub3;
wire [15:0] sig1_3_sub4;
wire [15:0] sig2_3_sub5;
wire [15:0] sig3_4_sub3;
wire [15:0] sig0_4_sub4;
wire [15:0] sig1_4_sub5;
wire [15:0] sig3_5_sub3;
wire [15:0] sig4_5_sub4;
wire [15:0] sig0_5_sub5;
//AUTO_UNFOLD_END
```

重头戏哈，脚本支持生成行里带有条件，但是只支持if一种，不支持elseif和else：

```
/* AUTO_UNFOLD
#for i 3..2
#for j 0..0
#for k 5..4
IF(#i==3 and k==4#)reg [15:0] sig_k#k#_j#j#_i#i#;
END */
//AUTO_UNFOLD_START
reg [15:0] sig_k4_j0_i3;
//AUTO_UNFOLD_END
```

IF中可以带有参数，不过还是像上面说的，这个参数必须是localparam直接指明的数值，比如下面的例子，SEL_NUM_SUB1不能是SEL_NUM-1，脚本识别不了参数里的运算：

```
localparam SEL_NUM = 32;
localparam SEL_NUM_SUB1 = 31;
/* AUTO_UNFOLD
#for i 0..SEL_NUM_SUB1
IF(#i==0#)assign out_sig = (sel == 3'd#i#) ? in_sig#i# :
IF(#i>0 and i<SEL_NUM_SUB1#)                 (sel == 3'd#i#) ? in_sig#i# :
IF(#i==SEL_NUM_SUB1#)                                  in_sig#i# ;
END */
//AUTO_UNFOLD_START
assign out_sig = (sel == 3'd0) ? in_sig0 :
                 (sel == 3'd1) ? in_sig1 :
                 (sel == 3'd2) ? in_sig2 :
                 (sel == 3'd3) ? in_sig3 :
                 (sel == 3'd4) ? in_sig4 :
                 (sel == 3'd5) ? in_sig5 :
                 (sel == 3'd6) ? in_sig6 :
                 (sel == 3'd7) ? in_sig7 :
                 (sel == 3'd8) ? in_sig8 :
                 (sel == 3'd9) ? in_sig9 :
                 (sel == 3'd10) ? in_sig10 :
                 (sel == 3'd11) ? in_sig11 :
                 (sel == 3'd12) ? in_sig12 :
                 (sel == 3'd13) ? in_sig13 :
                 (sel == 3'd14) ? in_sig14 :
                 (sel == 3'd15) ? in_sig15 :
                 (sel == 3'd16) ? in_sig16 :
                 (sel == 3'd17) ? in_sig17 :
                 (sel == 3'd18) ? in_sig18 :
                 (sel == 3'd19) ? in_sig19 :
                 (sel == 3'd20) ? in_sig20 :
                 (sel == 3'd21) ? in_sig21 :
                 (sel == 3'd22) ? in_sig22 :
                 (sel == 3'd23) ? in_sig23 :
                 (sel == 3'd24) ? in_sig24 :
                 (sel == 3'd25) ? in_sig25 :
                 (sel == 3'd26) ? in_sig26 :
                 (sel == 3'd27) ? in_sig27 :
                 (sel == 3'd28) ? in_sig28 :
                 (sel == 3'd29) ? in_sig29 :
                 (sel == 3'd30) ? in_sig30 :
                                  in_sig31 ;
//AUTO_UNFOLD_END
```

在生成时候，脚本会直接把IF(xxxx)给删除，所以对齐这个事得慢慢调调，靠缘分吧。

最后是一个实战的例子，生成寄存器堆的32个寄存器和选择输出：

```

```
