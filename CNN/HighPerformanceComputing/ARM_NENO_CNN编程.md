# ARM_NENO_CNN编程

> 术语： 

System-on-Chip(SOC) 片上系统：核心、内存控制器、片上内存、外围设备、总线互连和其他逻辑（可能包括模拟或射频组件），以便产生系统。 SOC通常指集成度较高的设备，包括单个设备中系统的许多部分，可能包括模拟、混合信号或射频电路。

专用集成电路Application Specific Integrated Circuit(ASIC) :包含ARM内核、内存和其他组件。显然，ASIC和SOC之间有很大的重叠。

嵌入式系统 Embedded systems，
内存消耗 Memory Footprint(memory usage),
SIMD Single Instruction, Multiple Data.单指令多数据流，
MMU Memory Management Unit.内存管理单元，

[参考1 ARM NEON 编程系列](http://hongbomin.com/2016/05/13/arm_neon_introduction/)

[arm官方数据手册](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.subset.swdev.sdt/index.html)

[Cortex-A Series Programmer’s Guide Version: 4.0](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.subset.swdev.sdt/index.html)

ARM CPU最开始只有普通的寄存器，可以进行基本数据类型的基本运算。
自ARMv5开始引入了VFP（Vector Floating Point）指令，该指令用于向量化加速浮点运算。
自ARMv7开始正式引入NEON指令，NEON性能远超VFP，因此VFP指令被废弃。

SIMD即单指令多数据指令，目前在x86平台下有MMX/SSE/AVX系列指令，arm平台下有NEON指令。
一般SIMD指令通过intrinsics(内部库C函数接口的函数) 或者 汇编 实现。

Intrinsics是使用C语言的方式对NEON寄存器进行操作，因为相比于传统的使用纯汇编语言，具有可读性强，开发速度快等优势。如果需要在代码中调用NEON Intrinsics函数，需要加入头文件"arm_neon.h"。

![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/simd.PNG)

![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/simd_add-op.PNG)

![](https://upload-images.jianshu.io/upload_images/3270173-633789004154255f.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/634/format/webp)

在这里，一条SIMD加法指令可以同时得到8个加法结果。就计算步骤本身而言，比单独使用8条加法指令能够获得8倍的加速比。从该示例也可以看出，随着寄存器长度的变长，单指令能够处理的数据量也越来越大，从而获得更高的加速性能。
在Intel最新的AVX2指令集中，寄存器最大长度已经达到512位。

类似于Intel CPU下的MMX/SSE/AVX/FMA指令，ARM CPU的NEON指令同样是通过向量化计算来进行速度优化，通常应用于图像处理、音视频处理等等需要大量计算的场景。

> NEON支持的数据类型：

* 32bit  single precision floatingpoint  ， 32bit 单精度浮点数；
* 8, 16, 32 and 64bit unsigned and signed integers ，  8, 16, 32 and 64bit 无符号/有符号 整型；
* 8 and 16bit polynomials 8 and 16bit 多项式。


	B字节Byte：      8 bits.
	H半字Halfword：  16 bits.   半精度浮点16位
	S字Word：        32 bits.   单精度浮点32位
	D双字Doubleword：64 bits.   双精度浮点64位
	Q四字Quadword：  128 bits.

> 浮点数取整:

向负无穷取整(向左取整) Round towards Minus Infinity (RM) roundTowardsNegative

向正无穷取整(向右取整) Round towards Plus Infinity (RP) roundTowardsPositive

向零取整(向中间取整)Round towards Zero (RZ) roundTowardZero

就近取整 Round to Nearest (RN) roundTiesToEven

随机取整

>NEON数据类型说明符：

* Unsigned integer  无符号整形 U8 U16 U32 U64
* Signed integer    有符号整形 S8 S16 S32 S64
* Integer of unspecified type  未指定类型的整数  I8 I16 I32 I64
Floating point number F16 F32  浮点数 16位浮点数(半精度) 32位浮点数(全精度)
Polynomial over {0,1} P8       多项式

注：F16不适用于数据处理运算，只用于数据转换，仅用于实现半精度体系结构扩展的系统。

多项式算术在实现某些加密、数据完整性算法中非常有用。

寄存器 ARMV7架构包含：

16个通用寄存器（32bit），R0-R15

16个NEON寄存器（128bit），Q0-Q15（同时也可以被视为32个64bit的寄存器，D0-D31）

16个VFP寄存器（32bit），S0-S15

NEON和VFP的区别在于VFP是加速浮点计算的硬件不具备数据并行能力，同时VFP更尽兴双精度浮点数（double）的计算，NEON只有单精度浮点计算能力。

16个通用寄存器

![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/register.PNG)

寄存器 r0 到 r7 称为低位寄存器。 寄存器 r8 到r15 称为高位寄存器。

下列寄存器名称是预先声明的：

* r0-r15 和 R0-R15
* a1-a4（自变量、结果或暂存寄存器，r0 到 r3 的同义词）
* v1-v8（变量寄存器，r4 到 r11）
* sb 和 SB（静态基址，r9）
* ip 和 IP（内部程序调用暂存寄存器，r12）
* sp 和 SP（堆栈指针，r13）
* lr 和 LR（链接寄存器，r14）
* pc 和 PC（程序计数器，r15）。

> NEON寄存器有几种形式：

* 16×128bit寄存器(Q0-Q15)；  16个128位的寄存器
* 或32×64bit寄存器(D0-D31)   32个64位的寄存器
* 或上述寄存器的组合。

以下扩展寄存器名称是预先声明的：

* q0-q15 和 Q0-Q15（NEON™ 四字寄存器）
* d0-d31 和 D0-D31（NEON 双字寄存器，VFP 双精度寄存器）
* s0-s31 和 S0-S31（VFP 单精度寄存器）。

![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/neon.PNG)

![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/neon-regest.PNG)

一个D寄存器64位是双字宽度，一个Q寄存器是128位是四字宽度。

注：每一个Q0-Q15寄存器映射到 一对D寄存器。

> 寄存器之间的映射关系：

* D<2n> 偶数 映射到 Q 的最低有效半部；
* D<2n+1> 奇数 映射到 Q 的最高有效半部；
* S<2n> 映射到 D<n> 的最低有效半部
* S<2n+1> 映射到 D<n> 的最高有效半部
	
例如，通过引用 D12 可以访问 Q6 中向量元素的最低有效半部，通过引用 D13 可以访问这些元素的最高有效半部。
	
![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/extern-regest.PNG)	
	
## 指令集概述
所有 ARM 指令的长度都是 32 位。 这些指令是按字对齐方式存储的，因此在ARM 状态下，指令地址的两个最低有效位始终为零。

> **跳转指令**，此类指令用于：

* 1.向后跳转以构成循环
* 2.在条件结构中向前跳转
* 3.跳转到子例程
* 4.在 ARM 状态和 Thumb 状态之间转换处理器状态。

> **寄存器加载和存储指令**

此类指令用于从内存加载单个寄存器的值，或者在内存中存储单个寄存器的值。它们可加载或存储 32 位字、16 位半字或 8 位无符号字节。 可以用符号或零扩展字节和半字加载以填充 32 位寄存器。此外，还定义了几个可将 64 位双字值加载或存储到两个 32 位寄存器的指令。

> **数据处理指令**

此类指令用于对通用寄存器执行运算。 它们可对两个寄存器的内容执行加法、减法或按位逻辑等运算，并将结果存放到第三个寄存器中。 此外，它们还可以对单个寄存器中的值执行运算，或者对寄存器中的值与指令中提供的常数（立即值）执行运算。

> NEON 数据处理指令可分为：

* 1. 正常指令 Normal instructions 结果 同 操作数 同大小同类型。

     生成大小相同且类型通常与操作数向量相同到结果向量。
     
     正常指令可对上述任意向量类型执行运算，并生成大小相同且类型通常与操作数向量相同的结果向量。

     **通过将 Q 附加到指令助记符，可以指定正常指令的操作数和结果必须全部为四字。** 

     这样指定后，如果操作数或结果不是四字，则汇编程序会生成错误。


* 2. 长指令   Long instructions   操作双字vectors，生成四倍长字vectors 结果的宽度一般比操作数加倍，同类型。

     在指令中加L
     
     长指令对双字向量操作数执行运算，并生成四字向量结果。 所生成的元素通常是操作数元素宽度的两倍，并属于同一类型。通过将 L 追加到指令助记符来指定长指令。
     
     对双字向量操作数执行运算，生成四字向量到结果。所生成的元素一般是操作数元素宽度到两倍，并属于同一类型。L标记，如VMOVL。
     
![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/long.PNG)
     
* 3. 宽指令   Wide instructions   操作 双字+四倍长字，生成四倍长字，结果和第一个操作数都是第二个操作数的两倍宽度。

     在指令中加W
     
     一个双字向量操作数和一个四字向量操作数执行运算，生成四字向量结果。W标记，如VADDW。
     
     宽指令对一个双字向量操作数和一个四字向量操作数执行运算。 此类指令生成四字向量结果。 所生成的元素和第一个操作数的元素是第二个操作数元素宽度的两倍。
     
     通过将 W 追加到指令助记符来指定宽指令。

![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/wide.PNG)
     
* 4. 窄指令   Narrow instructions 操作四倍长字，生成双字 结果宽度一般是操作数的一半
     
     在指令中加N
     
     四字向量操作数执行运算，并生成双字向量结果，所生成的元素一般是操作数元素宽度的一半。N标记，如VMOVN。
     
     窄指令对四字向量操作数执行运算，并生成双字向量结果。 所生成的元素通常是操作数元素宽度的一半。
     
     通过将 N 追加到指令助记符来指定窄指令。
     
![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/narrow.PNG)
     
* 5. 饱和指令 Saturating variants

        通过在 V 和指令助记符之间使用 Q 前缀来指定饱和指令。
	
	对于有符号饱和运算，如果结果小于 –2^n，则返回的结果将为 –2^n；
	 
	对于无符号饱和运算，如果整个结果将是负值，那么返回的结果是 0；如果结果大于 2^n–1，则返回的结果将为 2^n–1；
	
	NEON中的饱和算法：通过在V和指令助记符之间使用Q前缀可以指定饱和指令，原理与上述内容相同。
        
	饱和指令：当超过数据类型指定到范围则自动限制在该范围内。Q标记，如VQSHRUN

![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/data-range.PNG)

> **NEON指令集（重点）ARMv7/AArch32指令格式**

所有的支持NEON指令都有一个助记符V，下面以32位指令为例，说明指令的一般格式：

V{<mod模式>}<op操作>{<shape指令类型>}{<cond条件>}{.<dt数据类型>}{<dest目标地址>}, src1, src2
	
> <mod模式> 可选：

	Q: 饱和效果The instruction uses saturating arithmetic, so that the result is saturated within the range of the specified data type, such as VQABS, VQSHLetc.

	H: 结果右移动移位，相当于得到结构后在除以2 The instruction will halve the result. It does this by shifting right by one place (effectively a divide by two with truncation), such as VHADD,VHSUB.
	
	D: 双倍结果 The instruction doubles the result, such as VQDMULL, VQDMLAL, VQDMLSL and VQ{R}DMULH.
	R: 取整 The instruction will perform rounding on the result, equivalent to adding 0.5 to the result before truncating, such as VRHADD, VRSHR.
	
> <op操作>：  必须

the operation (for example, ADD加, SUB减, MUL乘).	

NEON指令按照作用可以分为：加载数据、存储数据、加减乘除运算、逻辑AND/OR/XOR运算、比较大小运算

> <shape> shape指令类型 可选：
	
即前文中的Long (L), Wide (W), Narrow (N).

> <cond条件> Condition 可选,
	
	used with IT instruction.
> <.dt> Datatype 可选 数据类型  .数据类型  前面有点
 
	such as .s8, .u8, .f32 , .I16, .S16 etc.
	
![](https://github.com/Ewenwan/MVision/blob/master/CNN/HighPerformanceComputing/img/dtypr.PNG)	
	
	
> <dest> Destination. 可选  目标操作数地址

> <src1> Source operand 1. 必须 源操作数地址
> <src2> Source operand 2. 必须 源操作数地址


注: {} 表示可选的参数。

比如：

VADD.I16 D0, D1, D2   @ 16位整数 加法

VMLAL.S16 Q2, D8, D9  @ 有符号16位整数 乘加

> 使用NEON主要有四种方法：

* 1. NEON优化库(Optimized libraries)
* 2. 向量化编译器(Vectorizing compilers)
* 3. NEON intrinsics
* 4. NEON assembly

根据优化程度需求不同，第4种最为底层，若熟练掌握效果最佳，一般也会配合第3种一起使用。

1. 优化库 Libraries：直接在程序中调用优化库

  OpenMax DL：支持加速视频编解码、信号处理、色彩空间转换等；
  
  Ne10：一个ARM的开源项目，提供数学运算、图像处理、FFT函数等。
  
2. 向量化编译 Vectorizing compilers：GCC编译器的向量优化选项

在GCC选项中加入向量化表示能有助于C代码生成NEON代码，如‐ftree‐vectorize。


3. NEON intrinsics：提供了一个连接NEON操作的C函数接口，编译器会自动生成相关的NEON指令，支持ARMv7或ARMv8平台。

[所有的intrinsics函数都在GNU官方说明文档 ](https://gcc.gnu.org/onlinedocs/gcc-4.7.4/gcc/ARM-NEON-Intrinsics.html#ARM-NEON-Intrinsics)

## 3. NEON Instrinsic函数

NEON Instrinsic是编译器支持的一种buildin类型和函数的集合，基本涵盖NEON的所有指令，通常这些Instrinsic包含在arm_neon.h头文件中。

[ARM-NEON-Intrinsics](https://gcc.gnu.org/onlinedocs/gcc-4.6.1/gcc/ARM-NEON-Intrinsics.html)

[使用ARM NEON Intrinsics加速Video Codec 参考](https://www.jianshu.com/p/70601b36540f)

### 数据类型

NEON 向量数据类型是根据以下模式命名的：<type类型><size大小宽度>x<number of lanes通道数量>_t

例如，int16x4_t 是一个包含四条向量线的向量，每条向量线包含一个有符号 16位整数。

NEON Intrinsics内置的整数数据类型主要包括以下几种:

* (u)int8x8_t;
* (u)int8x16_t;
* (u)int16x4_t;
* (u)int16x8_t;
* (u)int32x2_t;
* (u)int32x4_t;
* (u)int64x1_t;

其中，第一个数字代表的是数据类型宽度为8/16/32/64位，第二个数字代表的是一个寄存器中该类型数据的数量。如int16x8_t代表16位有符号数，寄存器中共有8个数据。

某些内在函数使用以下格式的向量类型数组：

<type><size>x<number of lanes>x<length of array>_t
	
这些类型被视为包含名为 val 的单个元素的普通 C 结构。

以下是一个结构定义示例：
```c
struct int16x4x2_t
{
int16x4_t val[2];
};
```

### 内在函数 inline function
每个内在函数的格式如下：

<opname><flags>_<type>
	
另外提供 q 标记来指定内在函数对 128 位向量进行运算。

例如：

* vmul_s16，表示两个有符号 16 位值的向量相乘multiply。
这编译为 VMUL.I16 d2, d0, d1。

* vaddl_u8，l为long长指令标识，是指两个包含无符号 8 位值的 64 位向量按长型相加，结果为无符号 16 位值的 128 位向量。
这编译为 VADDL.U8 q1, d0, d1。

	
〉**示例函数指令分析**
```c
int16x8_t vqaddq_s16 (int16x8_t, int16x8_t)
int16x4_t vqadd_s16 (int16x4_t, int16x4_t)
```

* 第一个字母'v'指明是vector向量指令，也就是NEON指令；
* 第二个字母'q'指明是饱和指令，即后续的加法结果会自动饱和；
* 第三个字段'add'指明是加法指令；
* 第四个字段'q'指明操作寄存器宽度，为'q'时操作QWORD, 为128位；未指明时操作寄存器为DWORD，为64位；
* 第五个字段's16'指明操作的基本单元为有符号16位整数，其最大表示范围为-32768 ~ 32767；
* 第六个字段为空，普通指令，形参和返回值类型约定与C语言一致。

其它可能用到的助记符包括:

* l 长指令，数据扩展，双字运算得到四字结果
* w 宽指令，数据对齐，双字和四字运算得到四字结果
* n 窄指令, 数据压缩，四字运算得到双字结果

> 示例2
```c
uint8x8_t vld1_u8 (const uint8_t *)
```
* 第一个字母'v'指明是vector向量指令，也就是NEON指令；
* 第二个字段'ld'表示加载指令 load
* 第三个字段'1'(注意是1，不是l)表示顺次加载。如果需要处理图像的RGB分量，可能会用到vld3间隔3个单元加载。


NEON指令按照作用可以分为：加载数据、存储数据、加减乘除运算、逻辑AND/OR/XOR运算、比较大小运算

> **初始化寄存器**
```c
// 寄存器的每个lane（通道）都赋值为一个值N
Result_t vcreate_type(Scalar_t N)   // type需要换成具体类型 s8, u8, f32, I16, S16
Result_t vdup_type(Scalar_t N)      // vcreate_s8  vdup_s8   vmov_s8
Result_t vmov_type(Scalar_t N)
```
> **加载load 内存数据进寄存器**
```c
// 间隔为x，加载数据进NEON寄存器, 间隔：交叉存取，是ARM NEON特有的指令
Result_t vld[x]_type(Scalar_t* N)  // 
Result_t vld[x]q_type(Scalar_t* N) // vld1q_s32 间隔1 即连续内存访问， 

// **通过将 Q 附加到指令助记符，可以指定正常指令的操作数和结果必须全部为四字。** 


float32x4x3_t = vld3q_f32(float32_t* ptr)
// 此处间隔为3，即交叉读取12个float32进3个NEON寄存器中。
// 3个寄存器的值分别为：
// {ptr[0],ptr[3],ptr[6],ptr[9]}，   // 128为Q寄存器
// {ptr[1],ptr[4],ptr[7],ptr[10]}，
// {ptr[2],ptr[5],ptr[8],ptr[11]}。
```

> **存储set 寄存器数据到内存   间隔为x，存储NEON寄存器的数据到内存中**
```cpp
void vst[x]_type(Scalar_t* N)
void vst[x]q_type(Scalar_t* N)
```

> **算数运算指令**

[普通指令]  普通加法运算 res = M+N
```c
Result_t vadd_type(Vector_t M,Vector_t N)
Result_t vaddq_type(Vector_t M,Vector_t N)

```
[长指令 long] 变长加法运算 res = M+N

为了防止溢出，一种做法是使用如下指令，加法结果存储到长度x2的寄存器中，

如：
```c

Result_t vaddl_type(Vector_t M,Vector_t N)

vuint16x8_t res = vaddl_u8(uint8x8_t M,uint8x8_t N)
```

[宽指令] 加法运算 res = M+N，第一个参数M宽度大于第二个参数N。
```c
Result_t vaddw_type(Vector_t M,Vector_t N)
```

[普通指令] 减法运算 res = M-N
```c
Result_t vsub_type(Vector_t M,Vector_t N)
```

[普通指令] 乘法运算 res = M*N
```c
Result_t vmul_type(Vector_t M,Vector_t N)
Result_t vmulq_type(Vector_t M,Vector_t N)
```

[普通指令] 乘&加法运算 res = M + N*P
```c
Result_t vmla_type(Vector_t M,Vector_t N,Vector_t P)
Result_t vmlaq_type(Vector_t M,Vector_t N,Vector_t P)
```

乘&减法运算 res = M-N*P
```c
Result_t vmls_type(Vector_t M,Vector_t N,Vector_t P)
Result_t vmlsq_type(Vector_t M,Vector_t N,Vector_t P)
```

> **数据处理指令**

[普通指令] 计算绝对值 res=abs(M)
```c
Result_t vabs_type(Vector_t M)
```
[普通指令] 计算负值 res=-M   negative
```c
Result_t vneg_type(Vector_t M)
```
[普通指令] 计算最大值 res=max(M,N)   maxmum
```c
Result_t vmax_type(Vector_t M,Vector_t N)
```
[普通指令] 计算最小值 res=min(M,N)
```c
Result_t vmin_type(Vector_t M,Vector_t N)
```

> **比较指令**

[普通指令] 比较是否相等 res=mask(M == N)  compare equal
```c
Result_t vceg_type(Vector_t M,Vector_t N)
```
[普通指令] 比较是否大于或等于 res=mask(M >= N)  compare greate and  equal
```c
Result_t vcge_type(Vector_t M,Vector_t N)
```
[普通指令] 比较是否大于 res=mask(M > N)
```c
Result_t vcgt_type(Vector_t M,Vector_t N)
```
[普通指令] 比较是否小于或等于 res=mask(M <= N)  compare little  and equal
```c
Result_t vcle_type(Vector_t M,Vector_t N)
```
[普通指令] 比较是否小于 res=mask(M < N)        compare little 
```c
Result_t vclt_type(Vector_t M,Vector_t N)
```
####  向量加法：

> **正常向量加法 vadd -> Vr[i]:=Va[i]+Vb[i]**
Vr、Va、Vb 具有相等的向量线大小。
```c
//64位==
int8x8_t vadd_s8(int8x8_t a, int8x8_t b); // VADD.I8 d0,d0,d0
int16x4_t vadd_s16(int16x4_t a, int16x4_t b); // VADD.I16 d0,d0,d0
int32x2_t vadd_s32(int32x2_t a, int32x2_t b); // VADD.I32 d0,d0,d0
int64x1_t vadd_s64(int64x1_t a, int64x1_t b); // VADD.I64 d0,d0,d0
float32x2_t vadd_f32(float32x2_t a, float32x2_t b); // VADD.F32 d0,d0,d0
uint8x8_t vadd_u8(uint8x8_t a, uint8x8_t b); // VADD.I8 d0,d0,d0
uint16x4_t vadd_u16(uint16x4_t a, uint16x4_t b); // VADD.I16 d0,d0,d0
uint32x2_t vadd_u32(uint32x2_t a, uint32x2_t b); // VADD.I32 d0,d0,d0
uint64x1_t vadd_u64(uint64x1_t a, uint64x1_t b); // VADD.I64 d0,d0,d0
//128位==
int8x16_t vaddq_s8(int8x16_t a, int8x16_t b); // VADD.I8 q0,q0,q0
int16x8_t vaddq_s16(int16x8_t a, int16x8_t b); // VADD.I16 q0,q0,q0
int32x4_t vaddq_s32(int32x4_t a, int32x4_t b); // VADD.I32 q0,q0,q0
int64x2_t vaddq_s64(int64x2_t a, int64x2_t b); // VADD.I64 q0,q0,q0
float32x4_t vaddq_f32(float32x4_t a, float32x4_t b); // VADD.F32 q0,q0,q0
uint8x16_t vaddq_u8(uint8x16_t a, uint8x16_t b); // VADD.I8 q0,q0,q0
uint16x8_t vaddq_u16(uint16x8_t a, uint16x8_t b); // VADD.I16 q0,q0,q0
uint32x4_t vaddq_u32(uint32x4_t a, uint32x4_t b); // VADD.I32 q0,q0,q0
uint64x2_t vaddq_u64(uint64x2_t a, uint64x2_t b); // VADD.I64 q0,q0,q0
```

> **向量长型加法：vaddl -> Vr[i]:=Va[i]+Vb[i]**

Va、Vb 具有相等的向量线大小，结果为向量线宽度变成两倍的 128 位向量。
```c
int16x8_t vaddl_s8(int8x8_t a, int8x8_t b); // VADDL.S8 q0,d0,d0
int32x4_t vaddl_s16(int16x4_t a, int16x4_t b); // VADDL.S16 q0,d0,d0
int64x2_t vaddl_s32(int32x2_t a, int32x2_t b); // VADDL.S32 q0,d0,d0
uint16x8_t vaddl_u8(uint8x8_t a, uint8x8_t b); // VADDL.U8 q0,d0,d0
uint32x4_t vaddl_u16(uint16x4_t a, uint16x4_t b); // VADDL.U16 q0,d0,d0
uint64x2_t vaddl_u32(uint32x2_t a, uint32x2_t b); // VADDL.U32 q0,d0,d0

```
> **向量宽型加法：vaddw -> Vr[i]:=Va[i]+Vb[i] 64位与128位运算得到128位**
```c
int16x8_t vaddw_s8(int16x8_t a, int8x8_t b); // VADDW.S8 q0,q0,d0
int32x4_t vaddw_s16(int32x4_t a, int16x4_t b); // VADDW.S16 q0,q0,d0
int64x2_t vaddw_s32(int64x2_t a, int32x2_t b); // VADDW.S32 q0,q0,d0
uint16x8_t vaddw_u8(uint16x8_t a, uint8x8_t b); // VADDW.U8 q0,q0,d0
uint32x4_t vaddw_u16(uint32x4_t a, uint16x4_t b); // VADDW.U16 q0,q0,d0
uint64x2_t vaddw_u32(uint64x2_t a, uint32x2_t b); // VADDW.U32 q0,q0,d0
```
> **向量半加：vhadd -> Vr[i]:=(Va[i]+Vb[i])>>1 求和后除以2**
```c
//64位
int8x8_t vhadd_s8(int8x8_t a, int8x8_t b); // VHADD.S8 d0,d0,d0
int16x4_t vhadd_s16(int16x4_t a, int16x4_t b); // VHADD.S16 d0,d0,d0
int32x2_t vhadd_s32(int32x2_t a, int32x2_t b); // VHADD.S32 d0,d0,d0
uint8x8_t vhadd_u8(uint8x8_t a, uint8x8_t b); // VHADD.U8 d0,d0,d0
uint16x4_t vhadd_u16(uint16x4_t a, uint16x4_t b); // VHADD.U16 d0,d0,d0
uint32x2_t vhadd_u32(uint32x2_t a, uint32x2_t b); // VHADD.U32 d0,d0,d0
// 128位
int8x16_t vhaddq_s8(int8x16_t a, int8x16_t b); // VHADD.S8 q0,q0,q0
int16x8_t vhaddq_s16(int16x8_t a, int16x8_t b); // VHADD.S16 q0,q0,q0
int32x4_t vhaddq_s32(int32x4_t a, int32x4_t b); // VHADD.S32 q0,q0,q0
uint8x16_t vhaddq_u8(uint8x16_t a, uint8x16_t b); // VHADD.U8 q0,q0,q0
uint16x8_t vhaddq_u16(uint16x8_t a, uint16x8_t b); // VHADD.U16 q0,q0,q0
uint32x4_t vhaddq_u32(uint32x4_t a, uint32x4_t b); // VHADD.U32 q0,q0,q0
```

> **向量舍入半加：vrhadd -> Vr[i]:=(Va[i]+Vb[i]+1)>>1 求和再加1后除以2**

```c
//64位
int8x8_t vrhadd_s8(int8x8_t a, int8x8_t b); // VRHADD.S8 d0,d0,d0
int16x4_t vrhadd_s16(int16x4_t a, int16x4_t b); // VRHADD.S16 d0,d0,d0
int32x2_t vrhadd_s32(int32x2_t a, int32x2_t b); // VRHADD.S32 d0,d0,d0
uint8x8_t vrhadd_u8(uint8x8_t a, uint8x8_t b); // VRHADD.U8 d0,d0,d0
uint16x4_t vrhadd_u16(uint16x4_t a, uint16x4_t b); // VRHADD.U16 d0,d0,d0
uint32x2_t vrhadd_u32(uint32x2_t a, uint32x2_t b); // VRHADD.U32 d0,d0,d0
//128位
int8x16_t vrhaddq_s8(int8x16_t a, int8x16_t b); // VRHADD.S8 q0,q0,q0
int16x8_t vrhaddq_s16(int16x8_t a, int16x8_t b); // VRHADD.S16 q0,q0,q0
int32x4_t vrhaddq_s32(int32x4_t a, int32x4_t b); // VRHADD.S32 q0,q0,q0
uint8x16_t vrhaddq_u8(uint8x16_t a, uint8x16_t b); // VRHADD.U8 q0,q0,q0
uint16x8_t vrhaddq_u16(uint16x8_t a, uint16x8_t b); // VRHADD.U16 q0,q0,q0
uint32x4_t vrhaddq_u32(uint32x4_t a, uint32x4_t b); // VRHADD.U32 q0,q0,q0
```
> **向量饱和加法：vqadd -> Vr[i]:=sat<size>(Va[i]+Vb[i])**

```c
//64位	
int8x8_t vqadd_s8(int8x8_t a, int8x8_t b); // VQADD.S8 d0,d0,d0
int16x4_t vqadd_s16(int16x4_t a, int16x4_t b); // VQADD.S16 d0,d0,d0
int32x2_t vqadd_s32(int32x2_t a, int32x2_t b); // VQADD.S32 d0,d0,d0
int64x1_t vqadd_s64(int64x1_t a, int64x1_t b); // VQADD.S64 d0,d0,d0
uint8x8_t vqadd_u8(uint8x8_t a, uint8x8_t b); // VQADD.U8 d0,d0,d0
uint16x4_t vqadd_u16(uint16x4_t a, uint16x4_t b); // VQADD.U16 d0,d0,d0
uint32x2_t vqadd_u32(uint32x2_t a, uint32x2_t b); // VQADD.U32 d0,d0,d0
uint64x1_t vqadd_u64(uint64x1_t a, uint64x1_t b); // VQADD.U64 d0,d0,d0
//128位  前面的q表示饱和运算，后面的q表示q寄存器，128位寄存器操作数
int8x16_t vqaddq_s8(int8x16_t a, int8x16_t b); // VQADD.S8 q0,q0,q0
int16x8_t vqaddq_s16(int16x8_t a, int16x8_t b); // VQADD.S16 q0,q0,q0
int32x4_t vqaddq_s32(int32x4_t a, int32x4_t b); // VQADD.S32 q0,q0,q0
int64x2_t vqaddq_s64(int64x2_t a, int64x2_t b); // VQADD.S64 q0,q0,q0
uint8x16_t vqaddq_u8(uint8x16_t a, uint8x16_t b); // VQADD.U8 q0,q0,q0
uint16x8_t vqaddq_u16(uint16x8_t a, uint16x8_t b); // VQADD.U16 q0,q0,q0
uint32x4_t vqaddq_u32(uint32x4_t a, uint32x4_t b); // VQADD.U32 q0,q0,q0
uint64x2_t vqaddq_u64(uint64x2_t a, uint64x2_t b); // VQADD.U64 q0,q0,q0
```
> **高位半部分向量加法：- > Vr[i]:=Va[i]+Vb[i]**
```c
int8x8_t vaddhn_s16(int16x8_t a, int16x8_t b); // VADDHN.I16 d0,q0,q0
int16x4_t vaddhn_s32(int32x4_t a, int32x4_t b); // VADDHN.I32 d0,q0,q0
int32x2_t vaddhn_s64(int64x2_t a, int64x2_t b); // VADDHN.I64 d0,q0,q0
uint8x8_t vaddhn_u16(uint16x8_t a, uint16x8_t b); // VADDHN.I16 d0,q0,q0
uint16x4_t vaddhn_u32(uint32x4_t a, uint32x4_t b); // VADDHN.I32 d0,q0,q0
uint32x2_t vaddhn_u64(uint64x2_t a, uint64x2_t b); // VADDHN.I64 d0,q0,q0
```
> **高位半部分向量舍入加法**
```c
int8x8_t vraddhn_s16(int16x8_t a, int16x8_t b); // VRADDHN.I16 d0,q0,q0
int16x4_t vraddhn_s32(int32x4_t a, int32x4_t b); // VRADDHN.I32 d0,q0,q0
int32x2_t vraddhn_s64(int64x2_t a, int64x2_t b); // VRADDHN.I64 d0,q0,q0
uint8x8_t vraddhn_u16(uint16x8_t a, uint16x8_t b); // VRADDHN.I16 d0,q0,q0
uint16x4_t vraddhn_u32(uint32x4_t a, uint32x4_t b); // VRADDHN.I32 d0,q0,q0
uint32x2_t vraddhn_u64(uint64x2_t a, uint64x2_t b); // VRADDHN.I64 d0,q0,q0
```

#### 向量减法

>**正常向量减法 vsub -> Vr[i]:=Va[i]-Vb[i]**
```c
//64bits
int8x8_t vsub_s8(int8x8_t a, int8x8_t b); // VSUB.I8 d0,d0,d0
int16x4_t vsub_s16(int16x4_t a, int16x4_t b); // VSUB.I16 d0,d0,d0
int32x2_t vsub_s32(int32x2_t a, int32x2_t b); // VSUB.I32 d0,d0,d0
int64x1_t vsub_s64(int64x1_t a, int64x1_t b); // VSUB.I64 d0,d0,d0
float32x2_t vsub_f32(float32x2_t a, float32x2_t b); // VSUB.F32 d0,d0,d0
uint8x8_t vsub_u8(uint8x8_t a, uint8x8_t b);        // VSUB.I8 d0,d0,d0
uint16x4_t vsub_u16(uint16x4_t a, uint16x4_t b);    // VSUB.I16 d0,d0,d0
uint32x2_t vsub_u32(uint32x2_t a, uint32x2_t b);    // VSUB.I32 d0,d0,d0
uint64x1_t vsub_u64(uint64x1_t a, uint64x1_t b);    // VSUB.I64 d0,d0,d0
//128bits
int8x16_t vsubq_s8(int8x16_t a, int8x16_t b); // VSUB.I8 q0,q0,q0
int16x8_t vsubq_s16(int16x8_t a, int16x8_t b); // VSUB.I16 q0,q0,q0
int32x4_t vsubq_s32(int32x4_t a, int32x4_t b); // VSUB.I32 q0,q0,q0
int64x2_t vsubq_s64(int64x2_t a, int64x2_t b); // VSUB.I64 q0,q0,q0
float32x4_t vsubq_f32(float32x4_t a, float32x4_t b); // VSUB.F32 q0,q0,q0
uint8x16_t vsubq_u8(uint8x16_t a, uint8x16_t b); // VSUB.I8 q0,q0,q0
uint16x8_t vsubq_u16(uint16x8_t a, uint16x8_t b); // VSUB.I16 q0,q0,q0
uint32x4_t vsubq_u32(uint32x4_t a, uint32x4_t b); // VSUB.I32 q0,q0,q0
uint64x2_t vsubq_u64(uint64x2_t a, uint64x2_t b); // VSUB.I64 q0,q0,q0
```


>**向量长型减法：vsubl -> Vr[i]:=Va[i]-Vb[i]**
```c
int16x8_t vsubl_s8(int8x8_t a, int8x8_t b); // VSUBL.S8 q0,d0,d0
int32x4_t vsubl_s16(int16x4_t a, int16x4_t b); // VSUBL.S16 q0,d0,d0
int64x2_t vsubl_s32(int32x2_t a, int32x2_t b); // VSUBL.S32 q0,d0,d0
uint16x8_t vsubl_u8(uint8x8_t a, uint8x8_t b); // VSUBL.U8 q0,d0,d0
uint32x4_t vsubl_u16(uint16x4_t a, uint16x4_t b); // VSUBL.U16 q0,d0,d0
uint64x2_t vsubl_u32(uint32x2_t a, uint32x2_t b); // VSUBL.U32 q0,d0,d0
```

>**向量宽型减法：vsubw -> Vr[i]:=Va[i]+Vb[i]**
```c
int16x8_t vsubw_s8(int16x8_t a, int8x8_t b); // VSUBW.S8 q0,q0,d0
int32x4_t vsubw_s16(int32x4_t a, int16x4_t b); // VSUBW.S16 q0,q0,d0
int64x2_t vsubw_s32(int64x2_t a, int32x2_t b); // VSUBW.S32 q0,q0,d0
uint16x8_t vsubw_u8(uint16x8_t a, uint8x8_t b); // VSUBW.U8 q0,q0,d0
uint32x4_t vsubw_u16(uint32x4_t a, uint16x4_t b); // VSUBW.U16 q0,q0,d0
uint64x2_t vsubw_u32(uint64x2_t a, uint32x2_t b); // VSUBW.U32 q0,q0,d0
```

>**向量饱和减法 vqsub-> Vr[i]:=sat<size>(Va[i]-Vb[i])**
	
```c
//64bits
int8x8_t vqsub_s8(int8x8_t a, int8x8_t b); // VQSUB.S8 d0,d0,d0
int16x4_t vqsub_s16(int16x4_t a, int16x4_t b); // VQSUB.S16 d0,d0,d0
int32x2_t vqsub_s32(int32x2_t a, int32x2_t b); // VQSUB.S32 d0,d0,d0
int64x1_t vqsub_s64(int64x1_t a, int64x1_t b); // VQSUB.S64 d0,d0,d0
uint8x8_t vqsub_u8(uint8x8_t a, uint8x8_t b); // VQSUB.U8 d0,d0,d0
uint16x4_t vqsub_u16(uint16x4_t a, uint16x4_t b); // VQSUB.U16 d0,d0,d0
uint32x2_t vqsub_u32(uint32x2_t a, uint32x2_t b); // VQSUB.U32 d0,d0,d0
uint64x1_t vqsub_u64(uint64x1_t a, uint64x1_t b); // VQSUB.U64 d0,d0,d0
//128bits
int8x16_t vqsubq_s8(int8x16_t a, int8x16_t b); // VQSUB.S8 q0,q0,q0
int16x8_t vqsubq_s16(int16x8_t a, int16x8_t b); // VQSUB.S16 q0,q0,q0
int32x4_t vqsubq_s32(int32x4_t a, int32x4_t b); // VQSUB.S32 q0,q0,q0
int64x2_t vqsubq_s64(int64x2_t a, int64x2_t b); // VQSUB.S64 q0,q0,q0
uint8x16_t vqsubq_u8(uint8x16_t a, uint8x16_t b); // VQSUB.U8 q0,q0,q0
uint16x8_t vqsubq_u16(uint16x8_t a, uint16x8_t b); // VQSUB.U16 q0,q0,q0
uint32x4_t vqsubq_u32(uint32x4_t a, uint32x4_t b); // VQSUB.U32 q0,q0,q0
uint64x2_t vqsubq_u64(uint64x2_t a, uint64x2_t b); // VQSUB.U64 q0,q0,q0
```

>**向量半减Vr[i]:=(Va[i]-Vb[i])>>1**
```c
int8x8_t vhsub_s8(int8x8_t a, int8x8_t b); // VHSUB.S8 d0,d0,d0
int16x4_t vhsub_s16(int16x4_t a, int16x4_t b); // VHSUB.S16 d0,d0,d0
int32x2_t vhsub_s32(int32x2_t a, int32x2_t b); // VHSUB.S32 d0,d0,d0
uint8x8_t vhsub_u8(uint8x8_t a, uint8x8_t b); // VHSUB.U8 d0,d0,d0
uint16x4_t vhsub_u16(uint16x4_t a, uint16x4_t b); // VHSUB.U16 d0,d0,d0
uint32x2_t vhsub_u32(uint32x2_t a, uint32x2_t b); // VHSUB.U32 d0,d0,d0
int8x16_t vhsubq_s8(int8x16_t a, int8x16_t b); // VHSUB.S8 q0,q0,q0
int16x8_t vhsubq_s16(int16x8_t a, int16x8_t b); // VHSUB.S16 q0,q0,q0
int32x4_t vhsubq_s32(int32x4_t a, int32x4_t b); // VHSUB.S32 q0,q0,q0
uint8x16_t vhsubq_u8(uint8x16_t a, uint8x16_t b); // VHSUB.U8 q0,q0,q0
uint16x8_t vhsubq_u16(uint16x8_t a, uint16x8_t b); // VHSUB.U16 q0,q0,q0
uint32x4_t vhsubq_u32(uint32x4_t a, uint32x4_t b); // VHSUB.U32 q0,q0,q0
```

#### 乘法

>**向量乘法：vmul -> Vr[i] := Va[i] * Vb[i]**
```c
//64bits===
int8x8_t vmul_s8(int8x8_t a, int8x8_t b); // VMUL.I8 d0,d0,d0
int16x4_t vmul_s16(int16x4_t a, int16x4_t b); // VMUL.I16 d0,d0,d0
int32x2_t vmul_s32(int32x2_t a, int32x2_t b); // VMUL.I32 d0,d0,d0
float32x2_t vmul_f32(float32x2_t a, float32x2_t b); // VMUL.F32 d0,d0,d0
uint8x8_t vmul_u8(uint8x8_t a, uint8x8_t b); // VMUL.I8 d0,d0,d0
uint16x4_t vmul_u16(uint16x4_t a, uint16x4_t b); // VMUL.I16 d0,d0,d0
uint32x2_t vmul_u32(uint32x2_t a, uint32x2_t b); // VMUL.I32 d0,d0,d0
poly8x8_t vmul_p8(poly8x8_t a, poly8x8_t b); // VMUL.P8 d0,d0,d0
//128bits==
int8x16_t vmulq_s8(int8x16_t a, int8x16_t b); // VMUL.I8 q0,q0,q0
int16x8_t vmulq_s16(int16x8_t a, int16x8_t b); // VMUL.I16 q0,q0,q0
int32x4_t vmulq_s32(int32x4_t a, int32x4_t b); // VMUL.I32 q0,q0,q0
float32x4_t vmulq_f32(float32x4_t a, float32x4_t b); // VMUL.F32 q0,q0,q0
uint8x16_t vmulq_u8(uint8x16_t a, uint8x16_t b); // VMUL.I8 q0,q0,q0
uint16x8_t vmulq_u16(uint16x8_t a, uint16x8_t b); // VMUL.I16 q0,q0,q0
uint32x4_t vmulq_u32(uint32x4_t a, uint32x4_t b); // VMUL.I32 q0,q0,q0
poly8x16_t vmulq_p8(poly8x16_t a, poly8x16_t b); // VMUL.P8 q0,q0,q0

```
>**向量长型乘法：vmull -> Vr[i] := Va[i] * Vb[i]**
```c
int16x8_t vmull_s8(int8x8_t a, int8x8_t b); // VMULL.S8 q0,d0,d0
int32x4_t vmull_s16(int16x4_t a, int16x4_t b); // VMULL.S16 q0,d0,d0
int64x2_t vmull_s32(int32x2_t a, int32x2_t b); // VMULL.S32 q0,d0,d0
uint16x8_t vmull_u8(uint8x8_t a, uint8x8_t b); // VMULL.U8 q0,d0,d0
```

>**向量乘加：vmla -> Vr[i] := Va[i] + Vb[i] * Vc[i]**
```c
//64bits===
int8x8_t vmla_s8(int8x8_t a, int8x8_t b, int8x8_t c); // VMLA.I8 d0,d0,d0
int16x4_t vmla_s16(int16x4_t a, int16x4_t b, int16x4_t c); // VMLA.I16 d0,d0,d0
int32x2_t vmla_s32(int32x2_t a, int32x2_t b, int32x2_t c); // VMLA.I32 d0,d0,d0
float32x2_t vmla_f32(float32x2_t a, float32x2_t b, float32x2_t c); // VMLA.F32 d0,d0,d0
uint8x8_t vmla_u8(uint8x8_t a, uint8x8_t b, uint8x8_t c); // VMLA.I8 d0,d0,d0
uint16x4_t vmla_u16(uint16x4_t a, uint16x4_t b, uint16x4_t c); // VMLA.I16 d0,d0,d0
uint32x2_t vmla_u32(uint32x2_t a, uint32x2_t b, uint32x2_t c); // VMLA.I32 d0,d0,d0
//128bits==
int8x16_t vmlaq_s8(int8x16_t a, int8x16_t b, int8x16_t c); // VMLA.I8 q0,q0,q0
int16x8_t vmlaq_s16(int16x8_t a, int16x8_t b, int16x8_t c); // VMLA.I16 q0,q0,q0
int32x4_t vmlaq_s32(int32x4_t a, int32x4_t b, int32x4_t c); // VMLA.I32 q0,q0,q0
float32x4_t vmlaq_f32(float32x4_t a, float32x4_t b, float32x4_t c); // VMLA.F32 q0,q0,q0
uint8x16_t vmlaq_u8(uint8x16_t a, uint8x16_t b, uint8x16_t c); // VMLA.I8 q0,q0,q0
uint16x8_t vmlaq_u16(uint16x8_t a, uint16x8_t b, uint16x8_t c); // VMLA.I16 q0,q0,q0
uint32x4_t vmlaq_u32(uint32x4_t a, uint32x4_t b, uint32x4_t c); // VMLA.I32 q0,q0,q0
```


>**向量长型乘加：vmlal -> Vr[i] := Va[i] + Vb[i] * Vc[i]**
```c
int16x8_t vmlal_s8(int16x8_t a, int8x8_t b, int8x8_t c); // VMLAL.S8 q0,d0,d0
int32x4_t vmlal_s16(int32x4_t a, int16x4_t b, int16x4_t c); // VMLAL.S16 q0,d0,d0
int64x2_t vmlal_s32(int64x2_t a, int32x2_t b, int32x2_t c); // VMLAL.S32 q0,d0,d0
uint16x8_t vmlal_u8(uint16x8_t a, uint8x8_t b, uint8x8_t c); // VMLAL.U8 q0,d0,d0
uint32x4_t vmlal_u16(uint32x4_t a, uint16x4_t b, uint16x4_t c); // VMLAL.U16 q0,d0,d0
uint64x2_t vmlal_u32(uint64x2_t a, uint32x2_t b, uint32x2_t c); // VMLAL.U32 q0,d0,d0
```

>**向量乘减：vmls -> Vr[i] := Va[i] - Vb[i] * Vc[i]**
```c
//64bits==
int8x8_t vmls_s8(int8x8_t a, int8x8_t b, int8x8_t c); // VMLS.I8 d0,d0,d0
int16x4_t vmls_s16(int16x4_t a, int16x4_t b, int16x4_t c); // VMLS.I16 d0,d0,d0
int32x2_t vmls_s32(int32x2_t a, int32x2_t b, int32x2_t c); // VMLS.I32 d0,d0,d0
float32x2_t vmls_f32(float32x2_t a, float32x2_t b, float32x2_t c); // VMLS.F32 d0,d0,d0
uint8x8_t vmls_u8(uint8x8_t a, uint8x8_t b, uint8x8_t c); // VMLS.I8 d0,d0,d0
uint16x4_t vmls_u16(uint16x4_t a, uint16x4_t b, uint16x4_t c); // VMLS.I16 d0,d0,d0
uint32x2_t vmls_u32(uint32x2_t a, uint32x2_t b, uint32x2_t c); // VMLS.I32 d0,d0,d0
//128bits==
int8x16_t vmlsq_s8(int8x16_t a, int8x16_t b, int8x16_t c); // VMLS.I8 q0,q0,q0
int16x8_t vmlsq_s16(int16x8_t a, int16x8_t b, int16x8_t c); // VMLS.I16 q0,q0,q0
int32x4_t vmlsq_s32(int32x4_t a, int32x4_t b, int32x4_t c); // VMLS.I32 q0,q0,q0
float32x4_t vmlsq_f32(float32x4_t a, float32x4_t b, float32x4_t c); // VMLS.F32 q0,q0,q0
uint8x16_t vmlsq_u8(uint8x16_t a, uint8x16_t b, uint8x16_t c); // VMLS.I8 q0,q0,q0
uint16x8_t vmlsq_u16(uint16x8_t a, uint16x8_t b, uint16x8_t c); // VMLS.I16 q0,q0,q0
uint32x4_t vmlsq_u32(uint32x4_t a, uint32x4_t b, uint32x4_t c); // VMLS.I32 q0,q0,q0
```

>**向量长型乘减 vmlsl -> Vr[i] := Va[i] - Vb[i] * Vc[i]**
```c
int16x8_t vmlsl_s8(int16x8_t a, int8x8_t b, int8x8_t c); // VMLSL.S8 q0,d0,d0
int32x4_t vmlsl_s16(int32x4_t a, int16x4_t b, int16x4_t c); // VMLSL.S16 q0,d0,d0
int64x2_t vmlsl_s32(int64x2_t a, int32x2_t b, int32x2_t c); // VMLSL.S32 q0,d0,d0
uint16x8_t vmlsl_u8(uint16x8_t a, uint8x8_t b, uint8x8_t c); // VMLSL.U8 q0,d0,d0
uint32x4_t vmlsl_u16(uint32x4_t a, uint16x4_t b, uint16x4_t c); // VMLSL.U16 q0,d0,d0
uint64x2_t vmlsl_u32(uint64x2_t a, uint32x2_t b, uint32x2_t c); // VMLSL.U32 q0,d0,d0
```

#### 比较compare
提供一系列比较内在函数。如果对于一条向量线比较结果为 true，则该向量线的结果为将所有位设置为一。如果对于一条向量线比较结果为 false，则将所有位设置为零。返回类型是无符号整数类型。这意味着可以将比较结果用作 vbsl内在函数的第一个参数。


>**向量比较 等于否 vceq_type vceqq_type  compare equal**
```c
// 64位
uint8x8_t vceq_s8(int8x8_t a, int8x8_t b); // VCEQ.I8 d0, d0, d0
uint16x4_t vceq_s16(int16x4_t a, int16x4_t b); // VCEQ.I16 d0, d0, d0
uint32x2_t vceq_s32(int32x2_t a, int32x2_t b); // VCEQ.I32 d0, d0, d0
uint32x2_t vceq_f32(float32x2_t a, float32x2_t b); // VCEQ.F32 d0, d0, d0
uint8x8_t vceq_u8(uint8x8_t a, uint8x8_t b); // VCEQ.I8 d0, d0, d0
uint16x4_t vceq_u16(uint16x4_t a, uint16x4_t b); // VCEQ.I16 d0, d0, d0
uint32x2_t vceq_u32(uint32x2_t a, uint32x2_t b); // VCEQ.I32 d0, d0, d0
uint8x8_t vceq_p8(poly8x8_t a, poly8x8_t b); // VCEQ.I8 d0, d0, d0
// 128位
uint8x16_t vceqq_s8(int8x16_t a, int8x16_t b); // VCEQ.I8 q0, q0, q0
uint16x8_t vceqq_s16(int16x8_t a, int16x8_t b); // VCEQ.I16 q0, q0, q0
uint32x4_t vceqq_s32(int32x4_t a, int32x4_t b); // VCEQ.I32 q0, q0, q0
uint32x4_t vceqq_f32(float32x4_t a, float32x4_t b); // VCEQ.F32 q0, q0, q0
uint8x16_t vceqq_u8(uint8x16_t a, uint8x16_t b); // VCEQ.I8 q0, q0, q0
uint16x8_t vceqq_u16(uint16x8_t a, uint16x8_t b); // VCEQ.I16 q0, q0, q0
uint32x4_t vceqq_u32(uint32x4_t a, uint32x4_t b); // VCEQ.I32 q0, q0, q0
uint8x16_t vceqq_p8(poly8x16_t a, poly8x16_t b); // VCEQ.I8 q0, q0, q0
```

>**向量比较大于或等于 vcge vcgeq : compare greate or equal**
```c
// 64位
uint8x8_t vcge_s8(int8x8_t a, int8x8_t b); // VCGE.S8 d0, d0, d0
uint16x4_t vcge_s16(int16x4_t a, int16x4_t b); // VCGE.S16 d0, d0, d0
uint32x2_t vcge_s32(int32x2_t a, int32x2_t b); // VCGE.S32 d0, d0, d0
uint32x2_t vcge_f32(float32x2_t a, float32x2_t b); // VCGE.F32 d0, d0, d0
uint8x8_t vcge_u8(uint8x8_t a, uint8x8_t b); // VCGE.U8 d0, d0, d0
uint16x4_t vcge_u16(uint16x4_t a, uint16x4_t b); // VCGE.U16 d0, d0, d0
uint32x2_t vcge_u32(uint32x2_t a, uint32x2_t b); // VCGE.U32 d0, d0, d0

// 128位
uint8x16_t vcgeq_s8(int8x16_t a, int8x16_t b); // VCGE.S8 q0, q0, q0
uint16x8_t vcgeq_s16(int16x8_t a, int16x8_t b); // VCGE.S16 q0, q0, q0
uint32x4_t vcgeq_s32(int32x4_t a, int32x4_t b); // VCGE.S32 q0, q0, q0
uint32x4_t vcgeq_f32(float32x4_t a, float32x4_t b); // VCGE.F32 q0, q0, q0
uint8x16_t vcgeq_u8(uint8x16_t a, uint8x16_t b); // VCGE.U8 q0, q0, q0
uint16x8_t vcgeq_u16(uint16x8_t a, uint16x8_t b); // VCGE.U16 q0, q0, q0
uint32x4_t vcgeq_u32(uint32x4_t a, uint32x4_t b); // VCGE.U32 q0, q0, q0

```

>**向量比较小于或等于 vcle vcleq : compare little or equal**
```c
//64bits
uint8x8_t vcle_s8(int8x8_t a, int8x8_t b); // VCGE.S8 d0, d0, d0
uint16x4_t vcle_s16(int16x4_t a, int16x4_t b); // VCGE.S16 d0, d0, d0
uint32x2_t vcle_s32(int32x2_t a, int32x2_t b); // VCGE.S32 d0, d0, d0
uint32x2_t vcle_f32(float32x2_t a, float32x2_t b); // VCGE.F32 d0, d0, d0
uint8x8_t vcle_u8(uint8x8_t a, uint8x8_t b); // VCGE.U8 d0, d0, d0
uint16x4_t vcle_u16(uint16x4_t a, uint16x4_t b); // VCGE.U16 d0, d0, d0
uint32x2_t vcle_u32(uint32x2_t a, uint32x2_t b); // VCGE.U32 d0, d0, d0
// 128bits
uint8x16_t vcleq_s8(int8x16_t a, int8x16_t b); // VCGE.S8 q0, q0, q0
uint16x8_t vcleq_s16(int16x8_t a, int16x8_t b); // VCGE.S16 q0, q0, q0
uint32x4_t vcleq_s32(int32x4_t a, int32x4_t b); // VCGE.S32 q0, q0, q0
uint32x4_t vcleq_f32(float32x4_t a, float32x4_t b); // VCGE.F32 q0, q0, q0
uint8x16_t vcleq_u8(uint8x16_t a, uint8x16_t b); // VCGE.U8 q0, q0, q0
uint16x8_t vcleq_u16(uint16x8_t a, uint16x8_t b); // VCGE.U16 q0, q0, q0
uint32x4_t vcleq_u32(uint32x4_t a, uint32x4_t b); // VCGE.U32 q0, q0, q0
```

>**向量比较大于 vcgt vcgtq compare great **
```c
// 64bits
uint8x8_t vcgt_s8(int8x8_t a, int8x8_t b); // VCGT.S8 d0, d0, d0
uint16x4_t vcgt_s16(int16x4_t a, int16x4_t b); // VCGT.S16 d0, d0, d0
uint32x2_t vcgt_s32(int32x2_t a, int32x2_t b); // VCGT.S32 d0, d0, d0
uint32x2_t vcgt_f32(float32x2_t a, float32x2_t b); // VCGT.F32 d0, d0, d0
uint8x8_t vcgt_u8(uint8x8_t a, uint8x8_t b); // VCGT.U8 d0, d0, d0
uint16x4_t vcgt_u16(uint16x4_t a, uint16x4_t b); // VCGT.U16 d0, d0, d0
uint32x2_t vcgt_u32(uint32x2_t a, uint32x2_t b); // VCGT.U32 d0, d0, d0
// 128bits
uint8x16_t vcgtq_s8(int8x16_t a, int8x16_t b); // VCGT.S8 q0, q0, q0
uint16x8_t vcgtq_s16(int16x8_t a, int16x8_t b); // VCGT.S16 q0, q0, q0
uint32x4_t vcgtq_s32(int32x4_t a, int32x4_t b); // VCGT.S32 q0, q0, q0
uint32x4_t vcgtq_f32(float32x4_t a, float32x4_t b); // VCGT.F32 q0, q0, q0
uint8x16_t vcgtq_u8(uint8x16_t a, uint8x16_t b); // VCGT.U8 q0, q0, q0
uint16x8_t vcgtq_u16(uint16x8_t a, uint16x8_t b); // VCGT.U16 q0, q0, q0
uint32x4_t vcgtq_u32(uint32x4_t a, uint32x4_t b); // VCGT.U32 q0, q0, q0
```

>**向量比较小于 vclt vcltq : compare little **
```c
//64bits==
uint8x8_t vclt_s8(int8x8_t a, int8x8_t b); // VCGT.S8 d0, d0, d0
uint16x4_t vclt_s16(int16x4_t a, int16x4_t b); // VCGT.S16 d0, d0, d0
uint32x2_t vclt_s32(int32x2_t a, int32x2_t b); // VCGT.S32 d0, d0, d0
uint32x2_t vclt_f32(float32x2_t a, float32x2_t b); // VCGT.F32 d0, d0, d0
uint8x8_t vclt_u8(uint8x8_t a, uint8x8_t b); // VCGT.U8 d0, d0, d0
uint16x4_t vclt_u16(uint16x4_t a, uint16x4_t b); // VCGT.U16 d0, d0, d0
uint32x2_t vclt_u32(uint32x2_t a, uint32x2_t b); // VCGT.U32 d0, d0, d0
// 128bits===
uint8x16_t vcltq_s8(int8x16_t a, int8x16_t b); // VCGT.S8 q0, q0, q0
uint16x8_t vcltq_s16(int16x8_t a, int16x8_t b); // VCGT.S16 q0, q0, q0
uint32x4_t vcltq_s32(int32x4_t a, int32x4_t b); // VCGT.S32 q0, q0, q0
uint32x4_t vcltq_f32(float32x4_t a, float32x4_t b); // VCGT.F32 q0, q0, q0
uint8x16_t vcltq_u8(uint8x16_t a, uint8x16_t b); // VCGT.U8 q0, q0, q0
uint16x8_t vcltq_u16(uint16x8_t a, uint16x8_t b); // VCGT.U16 q0, q0, q0
uint32x4_t vcltq_u32(uint32x4_t a, uint32x4_t b); // VCGT.U32 q0, q0, q0
```

>**向量绝对值比较大于或等于 vcage vcageq: compare abs great equal**
```c

uint32x2_t vcage_f32(float32x2_t a, float32x2_t b); // VACGE.F32 d0, d0, d0
uint32x4_t vcageq_f32(float32x4_t a, float32x4_t b); // VACGE.F32 q0, q0, q0
```

>**向量绝对值比较小于或等于 vcale vcaleq: compare abs little equal **
```c
uint32x2_t vcale_f32(float32x2_t a, float32x2_t b); // VACGE.F32 d0, d0, d0
uint32x4_t vcaleq_f32(float32x4_t a, float32x4_t b); // VACGE.F32 q0, q0, q0
```

>**向量绝对值比较大于 vcagt vcagtq: compare abs great**
```c
uint32x2_t vcagt_f32(float32x2_t a, float32x2_t b); // VACGT.F32 d0, d0, d0
uint32x4_t vcagtq_f32(float32x4_t a, float32x4_t b); // VACGT.F32 q0, q0, q0
```

>**向量绝对值比较小于 vcalt vcaltq:compare abs little**
```c
uint32x2_t vcalt_f32(float32x2_t a, float32x2_t b); // VACGT.F32 d0, d0, d0
uint32x4_t vcaltq_f32(float32x4_t a, float32x4_t b); // VACGT.F32 q0, q0, q0
```

>**向量测试位 test**
```c
uint8x8_t vtst_s8(int8x8_t a, int8x8_t b); // VTST.8 d0, d0, d0
uint16x4_t vtst_s16(int16x4_t a, int16x4_t b); // VTST.16 d0, d0, d0
uint32x2_t vtst_s32(int32x2_t a, int32x2_t b); // VTST.32 d0, d0, d0
uint8x8_t vtst_u8(uint8x8_t a, uint8x8_t b); // VTST.8 d0, d0, d0
uint16x4_t vtst_u16(uint16x4_t a, uint16x4_t b); // VTST.16 d0, d0, d0
uint32x2_t vtst_u32(uint32x2_t a, uint32x2_t b); // VTST.32 d0, d0, d0
uint8x8_t vtst_p8(poly8x8_t a, poly8x8_t b); // VTST.8 d0, d0, d0

uint8x16_t vtstq_s8(int8x16_t a, int8x16_t b); // VTST.8 q0, q0, q0
uint16x8_t vtstq_s16(int16x8_t a, int16x8_t b); // VTST.16 q0, q0, q0
uint32x4_t vtstq_s32(int32x4_t a, int32x4_t b); // VTST.32 q0, q0, q0
uint8x16_t vtstq_u8(uint8x16_t a, uint8x16_t b); // VTST.8 q0, q0, q0
uint16x8_t vtstq_u16(uint16x8_t a, uint16x8_t b); // VTST.16 q0, q0, q0
uint32x4_t vtstq_u32(uint32x4_t a, uint32x4_t b); // VTST.32 q0, q0, q0
uint8x16_t vtstq_p8(poly8x16_t a, poly8x16_t b); // VTST.8 q0, q0, q0

```
#### 差值绝对值
>**参数间的差值绝对值：Vr[i] = | Va[i] - Vb[i] |  vabd: abs difference**
```c
int8x8_t vabd_s8(int8x8_t a, int8x8_t b); // VABD.S8 d0,d0,d0
int16x4_t vabd_s16(int16x4_t a, int16x4_t b); // VABD.S16 d0,d0,d0
int32x2_t vabd_s32(int32x2_t a, int32x2_t b); // VABD.S32 d0,d0,d0
uint8x8_t vabd_u8(uint8x8_t a, uint8x8_t b); // VABD.U8 d0,d0,d0
uint16x4_t vabd_u16(uint16x4_t a, uint16x4_t b); // VABD.U16 d0,d0,d0
uint32x2_t vabd_u32(uint32x2_t a, uint32x2_t b); // VABD.U32 d0,d0,d0
float32x2_t vabd_f32(float32x2_t a, float32x2_t b); // VABD.F32 d0,d0,d0
// 128bits
int8x16_t vabdq_s8(int8x16_t a, int8x16_t b); // VABD.S8 q0,q0,q0
int16x8_t vabdq_s16(int16x8_t a, int16x8_t b); // VABD.S16 q0,q0,q0
int32x4_t vabdq_s32(int32x4_t a, int32x4_t b); // VABD.S32 q0,q0,q0
uint8x16_t vabdq_u8(uint8x16_t a, uint8x16_t b); // VABD.U8 q0,q0,q0
uint16x8_t vabdq_u16(uint16x8_t a, uint16x8_t b); // VABD.U16 q0,q0,q0
uint32x4_t vabdq_u32(uint32x4_t a, uint32x4_t b); // VABD.U32 q0,q0,q0
float32x4_t vabdq_f32(float32x4_t a, float32x4_t b); // VABD.F32 q0,q0,q0
```

>**差值绝对值 - 长型 **
```c
int16x8_t vabdl_s8(int8x8_t a, int8x8_t b); // VABDL.S8 q0,d0,d0
int32x4_t vabdl_s16(int16x4_t a, int16x4_t b); // VABDL.S16 q0,d0,d0
int64x2_t vabdl_s32(int32x2_t a, int32x2_t b); // VABDL.S32 q0,d0,d0
uint16x8_t vabdl_u8(uint8x8_t a, uint8x8_t b); // VABDL.U8 q0,d0,d0
uint32x4_t vabdl_u16(uint16x4_t a, uint16x4_t b); // VABDL.U16 q0,d0,d0
uint64x2_t vabdl_u32(uint32x2_t a, uint32x2_t b); // VABDL.U32 q0,d0,d0
```
#### 加载存储指令
>**加载并存储单个向量 加载并存储某类型的单个向量。vld1q_type**
```c

```

>** **
```c

```

>** **
```c

```


### 示例1：向量加法**
```c
// 假设 count 是4的倍数
#include<arm_neon.h>

// C version
void add_int_c(int* dst, int* src1, int* src2, int count)
{
	int i;
	for (i = 0; i < count; i++)
		{
		    dst[i] = src1[i] + src2[i];
		}
}

// NEON version
void add_float_neon1(int* dst, 
                     int* src1, 
		     int* src2, // 传入三个数据单元的指针（地址）
		     int count) // 数据量 假设为4的倍数
{
	int i;
	for (i = 0; i < count; i += 4) // 寄存器操作每次 进行4个数据的运输（单指令多数据SIMD）
	{
		int32x4_t in1, in2, out;
		
		// 1. 从内存 载入 数据 到寄存器
		in1 = vld1q_s32(src1);// intrinsics传入的为内存数据指针
		src1 += 4;// 数据 指针 递增+4 
		
		in2 = vld1q_s32(src2);
		src2 += 4;
		
		// 2. 在寄存器中进行数据运算 加法add
		out = vaddq_s32(in1, in2);
		
		// 3. 将寄存器中的结果 保存到 内存地址中
		vst1q_s32(dst, out);
		dst += 4;// 
	}
	// 实际情况，需要做最后不够4个的数的运输，使用普通c函数部分进行
	// 可参考下面的代码进行改进
}


``` 

代码中的 vld1q_s32 会被编译器转换成 vld1.32 {d0, d1}, [r0] 指令，

同理 vaddq_s32 被转换成 vadd.i32 q0, q0, q0，

 vst1q_s32 被转换成 vst1.32 {d0,d1}, [r0]。



### 示例2：向量乘法 

```neon
//NRON优化的vector相乘
static void neon_vector_mul(
  const std::vector<float>& vec_a, // 向量a 常量引用
  const std::vector<float>& vec_b, // 向量b 常量引用 
  std::vector<float>& vec_result)  // 结果向量 引用
{
	assert(vec_a.size() == vec_b.size());
	assert(vec_a.size() == vec_result.size());
	int i = 0;// 向量索引 从0开始
  
	//neon process
	for (; i < (int)vec_result.size() - 3 ; i+=4)// 每一步会并行执行四个数(单指令多数据simd) 注意每次增加4
	{// 不够 4的部分留在后面用 普通 c代码运算
               // 从内存载入数据到寄存器
		const auto data_a = vld1q_f32(&vec_a[i]);// 函数传入的是 地址（指针）
		const auto data_b = vld1q_f32(&vec_b[i]);
    
		float* dst_ptr = &vec_result[i];// 结果向量的地址(内存中)
    
                // 在寄存器中进行运算，乘法 mulp 运算
		const auto data_res = vmulq_f32(data_a, data_b);
    
                // 将处于寄存器中的结果 保存传输到 内存中国
		vst1q_f32(dst_ptr, data_res);
	}
  
	// normal process 普通C代码 数据相乘= 剩余不够4个数的部分===可能为 1,2,3个数
	for (; i < (int)vec_result.size(); i++)
	{
		vec_result[i] = vec_a[i] * vec_b[i];
	}
}

```

## 4. NEON assembly

NEON可以有两种写法：
* 1. Assembly文件： 纯汇编文件，后缀为”.S”或”.s”。注意对寄存器数据的保存。
* 2. inline assembly内联汇编

### 1.纯汇编 Assembly

#### 数据加载保存移动

> **扩展 寄存器 加载和存储 指令**

语法：
```asm
VLDR{cond}{.size} Fd, [Rn{, #offset}]   # load加载，从内存中加载一个扩展寄存器。
VSTR{cond}{.size} Fd, [Rn{, #offset}]   # set保存，将一个扩展寄存器的内容保存到内存中。
VLDR{cond}{.size} Fd, label
VSTR{cond}{.size} Fd, label
```
cond: 是一个可选的条件代码，EQ等于\NE不等于\HI无符号大于\LS无符号小于等于\GE有符号大于等于\LT有符号小于\GT有符号大于\LE有符号小于等于

size：是一个可选的数据大小说明符。 如果 Fd 是单精度 VFP 寄存器，则必须为 32，传送一个字；否则必须为 64，传送两个字。

Fd：是要加载或保存的扩展寄存器。 对于 NEON 指令，它必须为 Dd。 对于 VFP 指令，它可以为 Dd 或 Sd。

Rn：是存放要传送的基址的 ARM 寄存器。

offset：是一个可选的数值表达式。 在汇编时，该表达式的值必须为一个数字常数。 该值必须是 4 的倍数，并在 -1020 到 +1020 的范围内。 该值被加到基址上以构成用于传送的地址。

label：是一个程序相对的表达式。必须位于当前指令的 ±1KB 范围之内。

> **扩展寄存器加载多个、存储多个、从堆栈弹出、推入堆栈**

语法:
```asm
VLDMmode{cond} Rn,{!} Registers # 加载多个
VSTMmode{cond} Rn,{!} Registers # 存储多个
VPOP{cond} Registers            # 从堆栈弹出 VPOP Registers 等效于 VLDM sp!,Registers
VPUSH{cond} Registers           # 推入堆栈   VPUSH Registers 等效于 VSTMDB sp!,Registers
```
mode 必须是下列值之一：

	IA 表示在每次传送后递增地址。IA 是缺省值，可以省略。 increase
	DB 表示在每次传送前递减地址。 decrease
	EA 表示空的升序堆栈操作。 对于加载操作，该值与 DB 相同；对于保存操作，该值与 IA 相同。
	FD 表示满的降序堆栈操作。 对于加载操作，该值与 IA 相同；对于保存操作，该值与 DB 相同。

! 是可选的。! 指定必须将更新后的基址写回到 Rn 中。 如果未指定!，则 mode 必须为 IA。

Registers 是一个用大括号 { 和 } 括起的连续扩展寄存器的列表。 该列表可用逗号分隔，也可以采用范围格式。 列表中必须至少有一个寄存器。可指定 S、D 或 Q 寄存器，但一定不能混用这些寄存器。 D 寄存器的数目不得超过 16 个，Q 寄存器的数目不得超过 8 个。 如果指定 Q 寄存器，则在反汇编时它们将显示为 D 寄存器。

> **VMOV（在两个 ARM 寄存器和一个扩展寄存器之间传送内容）**

在两个 ARM 寄存器与一个 64 位扩展寄存器或两个连续的 32 位 VFP 寄存器之间传送内容。

语法:
```asm
VMOV{cond} Dm, Rd, Rn # 将 Rd 的内容传送到 Dm 的低半部分，并将 Rn 的内容传送到 Dm 的高半部分
VMOV{cond} Rd, Rn, Dm # 将 Dm 的低半部分的内容传送到 Rd，并将 Dm 的高半部分的内容传送到 Rn
VMOV{cond} {Sm, Sm1}, Rd, Rn # 将 Sm 的内容传送到 Rd，并将 Sm1 的内容传送到
VMOV{cond} Rd, Rn, {Sm, Sm1} # 将 Rd 的内容传送到 Sm，并将 Rn 的内容传送到 Sm1
```

	Dm 是一个 64 位扩展寄存器。
	Sm 是一个 VFP 32 位寄存器。
	Sm1 是 Sm 之后的下一个 VFP 32 位寄存器。
	Rd、Rn 是 ARM 寄存器。 不要使用 r15。
> **VMOV（在一个 ARM 寄存器R 和一个 NEON 标量之间）**

在一个 ARM 寄存器和一个 NEON 标量之间传送内容。

语法
VMOV{cond}{.size} Dn[x], Rd     # 将 Rd 的最低有效字节、半字或字的内容传送到 Sn。
VMOV{cond}{.datatype} Rd, Dn[x] # 将 Dn[x] 的内容传送到 Rd 的最低有效字节、半字或字。

size 是数据大小。 可以为 8、16 或 32。 如果省略，则 size 为 32。

datatype 是数据类型。 可以为 U8、S8、U16、S16 或 32。 如果省略，则 datatype为 32。

Dn[x] 是 NEON 标量,16 位标量限定为寄存器 D0-D7，其中 x 位于范围 0-3 内,32 位标量限定为寄存器 D0-D15，其中 x 为 0 或 1。

Rd 是 ARM 寄存器。Rd 不得为 R15。

#### NEON 逻辑运算和比较运算
> **VAND、VBIC、VEOR、VORN 和 VORR（寄存器）**

VAND（按位与）、VBIC（位清除）、VEOR（按位异或）、VORN（按位或非）和 VORR（按位或）指令在两个寄存器之间执行按位逻辑运算，并将结果存放到目标寄存器中。

语法:
```asm
Vop{cond}.{datatype} {Qd}, Qn, Qm
Vop{cond}.{datatype} {Dd}, Dn, Dm
```

op 必须是下列值之一：
AND 逻辑“与”\ORR 逻辑“或”\EOR 逻辑异或\BIC 逻辑“与”求补\ORN 逻辑“或”求补。

Qd、Qn、Qm 为四字运算指定目标寄存器、第一个操作数寄存器和第二个操作数寄存器。

Dd、Dn、Dm 为双字运算指定目标寄存器、第一个操作数寄存器和第二个操作数寄存器。

> **VBIC 和 VORR（立即数）**

VBIC（位清除（立即数））获取目标向量的每个元素，对其与一个立即数执行按位与求补运算，并将结果返回到目标向量。

VORR（按位或（立即数））获取目标向量的每个元素，对其与一个立即数执行按位或运算，并将结果返回到目标向量。


语法:
```asm
Vop{cond}.datatype Qd, #imm
Vop{cond}.datatype Dd, #imm
```
op 必须为 BIC 或 ORR。

datatype 必须为 I16 或 I32。

Qd 或 Dd 是用于存放源和结果的 NEON 寄存器。

imm 是立即数。

立即数 

如果 datatype 为 I16，则立即数必须采用下列格式之一：
• 0x00XY
• 0xXY00。

如果 datatype 为 I32，则立即数必须采用下列格式之一：
• 0x000000XY
• 0x0000XY00
• 0x00XY0000
• 0xXY000000。

〉**VBIF、VBIT 和 VBSL**

VBIT（为 True 时按位插入）：如果第二个操作数的对应位为 1，则该指令将第一个操作数中的每一位插入目标中；否则将目标位保持不变。

VBIF（为 False 时按位插入）：如果第二个操作数的对应位为 0，则该指令将第一个操作数中的每一位插入目标中；否则将目标位保持不变。

VBSL（按位选择）：如果目标的对应位为 1，则该指令从第一个操作数中选择目标的每一位；如果目标的对应位为 0，则从第二个操作数中选择目标的每一位。

语法：
```asm
Vop{cond}{.datatype} {Qd}, Qn, Qm
Vop{cond}{.datatype} {Dd}, Dn, Dm

```

> **VMOV、VMVN（寄存器）**

VMOV向量移动（寄存器）将源寄存器中的值复制到目标寄存器中。

VMVN向量求反移动（寄存器）对源寄存器中每一位的值执行求反运算，并将结果存放到目标寄存器中。


语法:
```asm
VMOV{cond}{.datatype} Qd, Qm
VMOV{cond}{.datatype} Dd, Qm
VMVN{cond}{.datatype} Qd, Qm
VMVN{cond}{.datatype} Dd, Qm
```

#### NEON 乘法指令

VMUL（向量乘法））将两个向量中的相应元素相乘，并将结果存放到目标向量中。
VMLA（向量乘加）将两个向量中的相应元素相乘，并将结果累加到目标向量的元素中。
VMLS（向量乘减）将两个向量中的相应元素相乘，从目标向量的相应元素中减去相乘的结果，并将最终结果放入目标向量中。
语法:
```asm
Vop{cond}.datatype {Qd}, Qn, Qm
Vop{cond}.datatype {Dd}, Dn, Dm
VopL{cond}.datatype Qd, Dn, Dm
```



### 2.内联汇编 inline assembly
[ARM GCC Inline Assembler Cookbook](http://www.ethernut.de/en/documents/arm-inline-asm.html)

优点：在C代码中嵌入汇编，调用简单，无需手动存储寄存器；
缺点：有较为复杂的格式需要事先学习，不好移植到其他语言环境。


比如上述intrinsics代码产生的汇编代码为：
```c
// ARMv7‐A/AArch32
void add_float_neon2(int* dst, int* src1, int* src2, int count)
{
	asm volatile (
		"1: \n"
		"vld1.32 {q0}, [%[src1]]! \n"
		"vld1.32 {q1}, [%[src2]]! \n"
		"vadd.f32 q0, q0, q1 \n"
		"subs %[count], %[count], #4 \n"
		"vst1.32 {q0}, [%[dst]]! \n"
		"bgt 1b \n"
		: [dst] "+r" (dst)
		: [src1] "r" (src1), [src2] "r" (src2), [count] "r" (count)
		: "memory", "q0", "q1"
	);
}

```


## 建议的NEON调优步骤

* 1. 理清所需的寄存器、指令。 建议根据要实现的任务，画出数据变换流程，和每步所需的具体指令，尽可能找到最优的实现流程。这一步非常关键，如果思路出错或是不够优化，则会影响使用NEON的效果，并且对程序修改带来麻烦，一定要找到最优的实现算法哦~

* 2. 先实现intrinsics（可选）。 初学者先实现intrinsics是有好处的，字面理解性更强，且有助于理解NEON指令。建议随时打印关键步骤的数据，以检查程序的正误。

* 3. 写成汇编进一步优化。 将intrinsics生成的汇编代码进行优化调整。一般来说，有以下几点值得注意【干货】：

* 只要intrinsics运算指令足够精简，运算类的汇编指令就不用大修；
* 大部分的问题会出在存取、移动指令的滥用、混乱使用上；
* 优化时要尽量减少指令间的相关性，包括结构相关、数据相关控制相关，保证流水线执行效率更高；
* 大概估算所有程序指令取指、执行、写回的总理论时间，以此估算本程序可以优化的空间；
* 熟练对每条指令准备发射、写回时间有一定的认识，有助于对指令的优化排序；
* 一定要多测试不同指令的处理时间！！原因是你所想跟实际有出入，且不同的编译器优化的效果可能也有些不同；
* 一定要有一定的计算机体系结构基础，对存储结构、流水线有一定的体会！！

总结一下NEON优化就是：
* 第一优化算法实现流程；
* 第二优化程序存取；
* 第三优化程序执行；
* 第四哪儿能优化，就优化哪儿


## 内联汇编使用心得
[ARM GCC Inline Assembler Cookbook](http://www.ethernut.de/en/documents/arm-inline-asm.html)

inline assembly下面的三个冒号一定要注意
output/input registers的写法一定要写对，clobber list也一定要写完全，否则会造成令你头疼的问题 (TT)

这个问题在给出的cookbook中也有介绍，但是并不全面，有些问题只有自己碰到了再去解决。 笔者就曾经被虐了很久，从生成的汇编发现编译器将寄存器乱用，导致指针操作完全混乱，毫无头绪…


一般情况下建议的写法举例：
```asm
asm volatile (
	... /* assembly code */
	: "+r"(arg0) // %0
	  "+r"(arg1) // %1 // 输入寄存器 Output Registers
	: "r"(arg2)  // %2 // 输入寄存器 Input Registers
	: "cc", "memory", r0, r1  // 寄存器变化
);
```


