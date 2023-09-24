# 零基础FPGA图像处理——RGB LCD 彩条显示
# 0 致读者

此篇为专栏 **《FPGA零基础教程》** 的第（六）讲，区别与市面上先讲解“FPGA是什么、点灯”等教程，此专栏力求最快速让读者学会如何开发FPGA，即**使用FPGA做可以落地的项目**，大家可以先去此专栏置顶 **《FPGA零基础入门学习路线》** 来做最基础的扫盲。

此篇同时为专栏 **《FPGA图像处理》** 的第一篇博客，将详细的带大家**零基础入门 FPGA 图像处理**，因此概念和思路讲解会比较多，已经熟悉的读者可以选择性的阅读，，**诚挚**地欢迎各位读者在评论区或者私信我交流！


本文的工程文件**开源地址**如下（基于ZYNQ7020，大家 **clone** 到本地就可以直接跑仿真，如果要上板请根据自己的开发板更改约束即可）：

> [https://github.com/ChinaRyan666/FPGA-LCD-Colorbar](https://github.com/ChinaRyan666/FPGA-LCD-Colorbar)
# 1 实验任务

本文的实验任务是使用 **ZYNQ 7020** 开发板上的 **RGB TFT-LCD** 接口，驱动 **RGB LCD** 液晶屏（支持目前推出的所有 **RGB LCD** 屏），并显示出彩条。






# 2 RGB TFT-LCD 简介

**TFT-LCD** 的全称是 **Thin Film Transistor-Liquid Crystal Display**，即薄膜晶体管液晶显示屏，它显示的每个像素点都是由集成在液晶后面的薄膜晶体管独立驱动， 因此 **TFT-LCD** 具有较高的响应速度以及较好的图像质量。 市面上的 **RGB TFT-LCD** 液晶屏种类较多， 7 寸 **RGB TFT-LCD** 屏的实物图如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/0e9d9c4b6a304f71bb94ac7879159d33.png)

液晶显示器是现在最常用到的显示器，手机、电脑、各种人机交互设备等基本都用到了 **LCD**， 在我们日常生活中比较常见的就是手机和电脑的显示器了。

> **LCD** 的构造是在两片平行的玻璃基板当中放置液晶盒，下基板玻璃上设置 **TFT（薄膜晶体管）**，上基板玻璃上设置彩色滤光片，通过 TFT上的信号与电压改变来控制液晶分子的转动方向，从而达到控制每个像素点偏振光出射与否而达到显示目的。我们现在要在 **FPGA** 开发板上使用 **LCD**，所以不需要去研究 LCD 的具体实现原理，我们只需要从使用的角度去关注 **LCD** 的几个重要点。


## 2.1 分辨率

提起 **LCD** 显示器，我们都会听到 720P、 1080P、 2K 或 4K 这样的字眼，这个就是 **LCD** 显示器**分辨率**。LCD 显示器都是由一个一个的像素点组成，像素点就类似一个灯 (在 **OLED** 显示器中，像素点就是一个小灯) ，这个小灯是 **RGB** 灯，也就是由 R(红色)、 G(绿色)和 B(蓝色)这三种颜色组成的，而 RGB 就是光的三原色。 **1080P** 的意思就是一个 **LCD** 屏幕上的像素数量是 **1920*1080** 个，也就是这个屏幕一列 1080 个像素点，一共 1920 列，如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/9c2af87443db43d2adef5eba9924ab26.png)


上图就是 **1080P** 显示器的像素示意图， X 轴就是 LCD 显示器的横轴， Y 轴就是显示器的竖轴。图中的小方块就是像素点，一共有 **1920*1080=2073600** 个像素点。左上角的 A 点是第一个像素点，右下角的 C 点就是最后一个像素点。2K 就是 2560* 1440 个像素点， 4K 是 3840* 2160 个像素点。

> 很明显，在 **LCD** 尺寸不变的情况下，分辨率越高越清晰。同样的，分辨率不变的情况下， LCD 尺寸越小越清晰。比如我们常用的 24 寸显示器基本都是 1080P 的，而我们现在使用的 5 寸的手机基本也是 1080P 的，但是手机显示细腻程度就要比 24 寸的显示器要好很多。由此可见， **LCD** 显示器的分辨率是一个很重要的参数，但是并不是分辨率越高的 **LCD** 就越好。衡量一款 **LCD** 的好坏，分辨率只是其中的一个参数，还有色彩还原程度、色彩偏离、亮度、可视角度、屏幕刷新率等其他参数。

## 2.2 像素格式

上面讲了一个像素点就相当于一个 **RGB** 小灯，通过控制 R、 G、 B 这三种颜色的亮度就可以显示出各种各样的色彩。那该如何控制 R、 G、 B 这三种颜色的显示亮度呢？一般一个 R、 G、 B 这三部分分别使用 8bit 的数据，那么**一个像素点就是 8bit*3=24bit**，也就是说**一个像素点 3 个字节**，这种像素格式称为 **RGB888**。当然常用的像素点格式还有 RGB565，只需要两个字节，但在色彩鲜艳度上较差一些。

市面上常见开发板上的 **RGB TFT-LCD** 接口大多采用的 **RGB888** 的像素格式，共需要 **24** 位，每一位对应 **RGB** 的颜色分量如下图所示：


![在这里插入图片描述](https://img-blog.csdnimg.cn/b00ddb956fb84487a4586f8126e95a08.png)


>在上图中，一个像素点占用 3 个字节，其中 bit23~ bit16 是 **RED** 通道， bit15~ bit8 是 **GREEN** 通道，bit7~ bit0 是 **BLUE** 通道。所以红色对应的值就是 24’hFF0000，绿色对应的值就是 24’h00FF00，蓝色对应的值为 24’h0000FF。通过调节 R、 G、 B 的比例可以产生其它的颜色，比如 24’hFFFF00 就是黄色， 24’h000000就是黑色， 24’hFFFFFF 就是白色。大家可以打开电脑的 “画图” 工具，在里面使用调色板即可获取到想要的颜色对应的数值，如图所示：


![在这里插入图片描述](https://img-blog.csdnimg.cn/90ebfbaa25814878a83ab93d900a3a92.png)


## 2.3 LCD 屏幕接口

**LCD** 屏幕或者说显示器有很多种接口，比如在显示器上常见的 **VGA、 HDMI、 DP** 等等。本文我们介绍的是 **RGB LCD** 接口，下一篇博客我将会详细讲解 HDMI 接口。 **RGB LCD** 接口的信号线如下图所示：


![在这里插入图片描述](https://img-blog.csdnimg.cn/4a54dc89ebb148fe941eae6c6c7169f7.png)

上图就是 **RGB LCD** 的信号线， **R [7:0]、 G [7:0]和 B [7:0]** 是 24 位数据， **DE、 VSYNC、 HSYNC 和 PCLK** 是四个控制信号。

本文将介绍五款市面上常见的 **RGB LCD** 屏幕（以正点原子的产品为例来讲解），型号分别为 ATK-4342（4.3 寸， **480*272**）、 ATK-4384（4.3 寸，**800*480**）、 ATK-7084（7 寸，**800*480**）、 ATK-7016（7 寸， **1024*600**）和 ATK-1018（10.1 寸， **1280*800**）。这里以 ATK-7016 这款屏幕为例讲解， ATK-7016 的屏幕接口原理图如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/bda7ed14c241422d8170639b54436c1d.png)

>图中 J1 就是对外接口，是一个 40PIN 的 FPC 座（0.5mm 间距），通过 FPC 线，可以连接到开发板上面，从而实现和开发板的连接。该接口十分完善，采用 RGB888 格式，并支持触摸屏和背光控制。右侧的几个电阻，并不是都焊接的，而是可以用户自己选择。默认情况， R1 和 R6 焊接，设置 LCD_LR 和 LCD_UD，控制 LCD 的扫描方向，是从左到右，从上到下（横屏看）。而 LCD_R7/G7/B7 则用来设置 LCD 的 ID，由于 RGB LCD 没有读写寄存器，也就没有所谓的 ID，这里我们通过在模块上面，控制 R7/G7/B7 的上/下拉，来**自定义 LCD 模块的 ID**，帮助大家判断当前 LCD 面板的分辨率和相关参数，以提高程序兼容性。这几个位的设置关系如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/829f9a149b0c482fb4be58c5634edd5e.png)


ATK-7016 模块，就设置 M2:M0 = 010 即可。这样，我们在程序里面，读取 LCD_R7/G7/B7，得到 M0~M2 的值，从而**判断 RGB LCD 模块的型号，并执行不同的配置，即可实现不同 LCD 模块的兼容。**

## 2.4 LCD 时间参数


如果将 **LCD** 显示一帧图像的过程想象成绘画，那么在显示的过程中就是用一根“笔”在不同的像素点画上不同的颜色。这根笔按照从左至右、从上到下的顺序扫描每个像素点，并且在像素画上对应的颜色，当画到最后一个像素点的时候一幅图像就绘制好了。假如一个 LCD 的分辨率为 **1024*600**，那么其扫描如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/0857238ee8c14edcbaba0ab221255ca2.png)

结合上图我们来看一下 **LCD** 是怎么扫描显示一帧图像的。一帧图像也是由一行一行组成的。**HSYNC** 是水平同步信号，也叫做**行同步信号**，当产生此信号的话就表示开始显示新的一行了，所以此信号都是在上图的最左边。 **VSYNC** 信号是垂直同步信号，也叫做**帧同步信号**，当产生此信号的话就表示开始显示新的一帧图像了，所以此信号在上图的左上角。

>在上图可以看到有一圈 “黑边” ，真正有效的显示区域是中间的白色部分。那这一圈 “黑边” 是什么东西呢？这就要从显示器的 “祖先” CRT 显示器开始说起了， CRT 显示器就是以前很常见的那种大屁股显示器， 现在市场上应该没有了。 CRT 显示器屁股后面是个电子枪，这个电子枪就是我们上面说的 “画笔” ，电子枪打出的电子撞击到屏幕上的荧光物质使其发光。只要控制电子枪从左到右扫完一行(也就是扫描一行)，然后从上到下扫描完所有行，这样一帧图像就显示出来了。也就是说，显示一帧图像电子枪是**按照 ‘Z’ 形在运动**，当扫描速度很快的时候看起来就是一幅完成的画面了。

当显示完一行以后会发出 **HSYNC 信号**，此时电子枪就会关闭，然后迅速的移动到屏幕的左边，当 **HSYNC 信号**结束以后就可以显示新的一行数据了，电子枪就会重新打开。在 **HSYNC 信号**结束到电子枪重新打开之间会插入一段延时，这段延时就是上图中的 **HBP**。

当显示完一行以后就会关闭电子枪等待 **HSYNC 信号**产生，关闭电子枪到 **HSYNC 信号**产生之间会插入一段延时，这段延时就是上图中的 HFP 信号。同理，当显示完一帧图像以后电子枪也会关闭，然后等到 **VSYNC 信号**产生，期间也会加入一段延时，这段延时就是上图中的 **VFP**。 **VSYNC 信号**产生，电子枪移动到左上角，当 **VSYNC 信号**结束以后电子枪重新打开，中间也会加入一段延时，这段延时就是上图中的 **VBP**。

>HBP、 HFP、 VBP 和 VFP 就是导致上图中黑边的原因，但是这是 CRT 显示器存在黑边的原因，现在是 **LCD** 显示器，不需要电子枪了，那么为何还会有黑边呢？这是因为 **RGB LCD** 屏幕内部是有一个 IC 的，发送一行或者一帧数据给 IC， IC 是需要反应时间的。通过这段反应时间可以让 IC 识别到一行数据扫描完了，要换行了，或者一帧图像扫描完了，要开始下一帧图像显示了。因此，**在 LCD 屏幕中继续存在 HBP、HFP、 VPB 和 VFP 这四个参数的主要目的是为了锁定有效的像素数据**。

## 2.5 RGB LCD 屏幕时序

上面介绍了 **LCD** 的时间参数，我们接下来看一下**行显示**对应的**时序图**，如下图所示。

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b05c5ba09b1464e9f42ffb87e95c913.png)

上图就是 **RGB LCD** 的**行显示时序**，我们来分析一下其中重要的几个参数：

>**HSYNC：** 行同步信号，当此信号有效的时候就表示开始显示新的一行数据，查阅所使用的 LCD 数据手册可以知道此信号是低电平有效还是高电平有效， 上图为低电平有效。

>**HSPW：** 行同步信号宽度，也就是 HSYNC 信号持续时间。 HSYNC 信号不是一个脉冲，而是需要持续一段时间才是有效的，单位为 CLK。

>**HBP：** 行显示后沿（或后肩），单位是 CLK。

>**HOZVAL：** 行有效显示区域，即显示一行数据所需的时间，假如屏幕分辨率为 **1024*600**，那么 HOZVAL 就是 **1024**，单位为 CLK。

>**HFP：** 行显示前沿（或前肩），单位是 CLK。

当 **HSYNC 信号**发出以后，需要等待 HSPW+HBP 个 CLK 时间才会接收到真正有效的像素数据。当显示完一行数据以后需要等待 HFP 个 CLK 时间才能发出下一个 HSYNC 信号，所以显示一行所需要的时间就是： **HSPW + HBP + HOZVAL + HFP**。

一帧图像就是由很多个行组成的， **RGB LCD** 的**帧显示时序**如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/6d1ef6c5fda941ae9b8170e86812e056.png)

上图就是 **RGB LCD** 的**帧显示时序**，我们来分析一下其中重要的几个参数：

>**VSYNC：** 帧（场）同步信号，当此信号有效的时候就表示开始显示新的一帧数据，查阅所使用的 LCD数据手册可以知道此信号是低电平有效还是高电平有效，上图为低电平有效。

>**VSPW：** 帧同步信号宽度，也就是 VSYNC 信号持续时间，单位为 1 行的时间。

>**VBP：** 帧显示后沿（或后肩），单位为 1 行的时间。

>**LINE：** 帧有效显示区域，即显示一帧数据所需的时间，假如屏幕分辨率为 **1024*600**，那么 LINE 就是 **600** 行的时间。

>**VFP：** 帧显示前沿（或前肩），单位为 1 行的时间。

显示一帧所需要的时间就是： **VSPW+VBP+LINE+VFP** 个行时间，最终的计算公式：

>**T = (VSPW+VBP+LINE+VFP) * (HSPW + HBP + HOZVAL + HFP)**

因此我们在配置一款 **RGB LCD** 屏的时候需要知道这几个参数： **HSPW**（行同步）、 **HBP**（行显示后沿）、**HOZVAL**（行有效显示区域)、 **HFP**（行显示前沿）、 **VSPW**（场同步）、 **VBP**（场显示后沿）、 **LINE**（场有效显示区域）和 **VFP**（场显示后沿）。

**RGB LCD** 液晶屏一般有两种数据同步方式，一种是**行场同步模式（ HV Mode）**，另一种是**数据使能同步模式（DE Mode）**。

>当选择**行场同步模式**时， LCD 接口的时序与 VGA 接口的时序图非常相似，只是参数不同。 如上图中的行同步信号（HSYNC）和场同步信号（VSYNC）作为数据的同步信号，此时数据使能信号（ DE）必须为低电平。

>当选择 **DE 同步模式**时， LCD 的 DE 信号作为数据的有效信号，如上图的 DE 信号所示。只有同时扫描到帧有效显示区域和行有效显示区域时， DE 信号才有效（高电平）。当选择 DE 同步模式时，此时行场同步信号 VS 和 HS 必须为高电平。

由于 **RGB LCD** 液晶屏一般都支持 DE 模式，因此本文我们采用 **DE 同步**的方式驱动 **LCD** 液晶屏。


## 2.6 像素时钟

像素时钟就是 **RGB LCD** 的时钟信号，以 ATK7016 这款屏幕为例，显示一帧图像所需要的时钟数就是：

>**N（CLK） = (VSPW+VBP+LINE+VFP) * (HSPW + HBP + HOZVAL + HFP) = (3 + 20 + 600 + 12) * (20 + 140 + 1024 + 160) = 635 * 1344 = 853440**

显示一帧图像需要 853440 个时钟数，那么显示 60 帧就是：

>**853440 * 60 = 51206400≈51.2M，所以像素时钟就是 51.2MHz。**

当然我们在为 **RGB-LCD** 屏提供驱动时钟的时候，也可以不用严格按照 60 帧来进行计算。为了方便操作，我们可以给 ATK7016 模块输出一个 **50MHz** 的时钟，其**刷新率是接近于 60Hz** 的，同时也非常方便我们来编写代码。

为了方便大家查找 **LCD** 屏的时序参数，这里整理了不同分辨率的时序参数，如下图所示：


![在这里插入图片描述](https://img-blog.csdnimg.cn/417da6f3f15c41c5a89c32a067a49048.png)


# 3 程序设计

## 3.1 总体模块设计

**RGB TFT-LCD** 输入时序包含三个要素： **像素时钟**、**同步信号**、 以及**图像数据**， 由此我们可以大致规划出系统结构如下图所示。 

>其中， **读取 ID 模块**用于获取 LCD 屏的 ID；由于不同分辨率的屏幕需要不同的驱动时钟，因此**时钟分频模块**根据 LCD ID 来输出不同频率的像素时钟； **LCD 显示模块**负责产生液晶屏上显示的数据，即彩条数据； **LCD 驱动模块**根据 LCD 屏的 ID，输出不同参数的时序，来驱动 LCD 屏，并将输入的彩条数据显示到 LCD 屏上。

![在这里插入图片描述](https://img-blog.csdnimg.cn/8527e1c893c54d2f91b4601f15443e19.png)

由系统框图可知， **FPGA** 部分包括五个模块， **顶层模块（lcd_rgb_colorbar）**、**读取 ID 模块（rd_id）**、**时钟分频模块（clk_div）**、 **LCD 显示模块（lcd_display）** 以及 **LCD 驱动模块（lcd_driver）**， 在顶层模块中完成对其余模块的例化。

>**读取 ID 模块（rd_id）**在上电时将 RGB 双向数据总线设置为**输入**，来读取 LCD 屏的 ID；**时钟分频模块（clk_div）** 根据读取的 ID 来配置 LCD 的像素时钟； **LCD 驱动模块（lcd_driver）** 在像素时钟的驱动下输出数据使能信号用于数据同步，同时还输出像素点的纵横坐标， 供 **LCD 显示模块（lcd_display）** 调用，以绘制彩条图案。

## 3.2 读 ID模块设计

下面我们先来设计一下 **RGB-LCD** 读 ID 模块，读 ID 模块的输入信号有 **clk（时钟信号）**、 **rst_n（复位信号）**以及 **lcd_rgb（RGB-LCD 像素数据）**，输出为 **lcd_id（RGB-LCD 屏 ID）**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/52f95bda233045389abf3355adc9d64d.png)


**读取 ID 模块**根据输入的 **lcd_rgb** 值来寄存 **LCD** 屏的 ID， **lcd_rgb[7] （B7）**、 **lcd_rgb[15] （G7）** 和 **lcd_rgb[23]（R7）** 分别对应 M2、 M1 和 M0。

>除此之外，为了方便将 LCD 的 ID 和分辨率对应起来，这里对 M2、 M1 和 M0 的值做了一个译码。 如 3’b000 译码成 16’h4342，表示当前连接的是 4.3 寸屏，分辨率为 480*272。 其它 ID 的译码请参考下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/9a2b9067f9214125b45d7e730ccbafda.png)

### 3.2.1 绘制波形图

根据设计的**读 ID 模块**端口的介绍以及 **ID 译码真值表**，我们可以绘制读 ID 模块的的波形图。在硬件设计中， 我们介绍双向引脚 **lcd_rgb** 会根据 **lcd_de** 信号的高低电平来频繁的切换方向，但本模块的设计只在上电后获取了一次 ID， 我们可以定义一个寄存器 **rd_flag**， 通过 **rd_flag** 作为读 ID 的标志。

>当 **rd_flag** 等于 0 时，获取一次 ID，并将 **rd_flag** 赋值为 1；而当 **rd_flag** 等于 1 时，不再获取 LCD 屏的 ID。

本次的波形图以分辨率为 **800*480** 的 4.3 寸 **RGB-LCD** 屏幕来绘制的，波形图如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/649792d74adb42a8ab33d99dc5018caf.png)

### 3.2.2 编写代码

通过绘制的波形图，我们可以进行**读 ID 模块**代码的编写，读 ID 代码如下所示：


```
module rd_id(
    input                   clk    ,    //时钟
    input                   rst_n  ,    //复位，低电平有效
    input           [23:0]  lcd_rgb,    //RGB LCD像素数据,用于读取ID
    output   reg    [15:0]  lcd_id     //LCD屏ID
    );

//reg define
reg            rd_flag;  //读ID标志

//*****************************************************
//**                    main code
//*****************************************************

//获取LCD ID   M2:B7  M1:G7  M0:R7
always @(posedge clk or negedge rst_n) begin
    if(!rst_n) begin
        rd_flag <= 1'b0;
        lcd_id <= 16'd0;
    end    
    else begin
        if(rd_flag == 1'b0) begin
            rd_flag <= 1'b1; 
            case({lcd_rgb[7],lcd_rgb[15],lcd_rgb[23]})
                3'b000 : lcd_id <= 16'h4342;    //4.3' RGB LCD  RES:480x272
                3'b001 : lcd_id <= 16'h7084;    //7'   RGB LCD  RES:800x480
                3'b010 : lcd_id <= 16'h7016;    //7'   RGB LCD  RES:1024x600
                3'b100 : lcd_id <= 16'h4384;    //4.3' RGB LCD  RES:800x480
                3'b101 : lcd_id <= 16'h1018;    //10'  RGB LCD  RES:1280x800
                default : lcd_id <= 16'd0;
            endcase    
        end
    end    
end

endmodule
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6b4eba6743e442da176a3406f562f79.png)

>当**复位信号**拉高之后，下一个上升沿进行**读 ID** 的操作，从仿真波形中可以看出，仿真波形读出的 ID 为 **4384** 与我们编写的代码以及绘制的波形图是一致的。


## 3.3 时钟分频模块设计


时钟分频模块根据输入的 **LCD ID** 对 **50Mhz** 时钟进行分频， 由于**不同分辨率的 LCD 屏需要的像素时钟频率不一样**，因此分频模块根据输入的 LCD ID，来输出不同频率的像素时钟 **lcd_pclk**。 时钟分频模块如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/98bc441cfa324d039cf10f7d57a56f21.png)

基于 2.6 像素时钟部分对于各个 **RGB-LCD** 屏的频率介绍，我们可以画出 **RGB-LCD** 屏幕时钟的真值表：

![在这里插入图片描述](https://img-blog.csdnimg.cn/58af56b53cb94a718da1551896c99b50.png)

>表格里记录了不同分辨率的屏幕所需的像素时钟频率，为了兼容所有的 lcd 屏，我们这里没有严格按照表格里所要求的时钟频率进行输出，而是输出接近于表格所要求的时钟频率。例如 10.1 寸屏，分辨率 **1280*800**，如果刷新率为 **60Hz** 的话，需要输出 **70Mhz** 的像素时钟，这个时钟频率是无法通过编写代码的方式来得到，而是必须例化时钟模块 MMCM/PLL IP 核来得到。因此，对于分辨率为 **1280*800** 的 10.1寸屏幕来说，我们输出的是 **50Mhz** 的像素时钟，当然大家使用的是 10.1 寸屏幕，也可以通过例化时钟模块的方式，来输出一个 **70Mhz** 的像素时钟。

### 3.3.1 绘制波形图

通过 **RGB-LCD** 时钟分配真值表可知，输出的像素时钟频率为 **12.5Mhz**、 **25Mhz** 以及 **50Mhz**。 50Mhz 的像素时钟，可以直接使用系统时钟， 12.5Mhz 和 25Mhz 的像素时钟可以通过对系统时钟分频得到。

下面我们就根据这个思路进行 **4.3 寸屏幕波形图**的绘制：

![在这里插入图片描述](https://img-blog.csdnimg.cn/417ef81d9e2a4b81a4bc4bf50e67cffc.png)

根据绘制的波形图，我们可知分频模块需要进行**2分频**和**4分频**，得到一个25Mhz的时钟和一个12.5Mhz的时钟。再结合 **RGB-LCD** 时钟分配的真值表，**根据读取的 ID 进行像素时钟的选择**。

### 3.3.2 编写代码

根据绘制的波形可以**编写分频模块的代码**， 分频模块的代码如下：

```
module clk_div(
    input               clk,          //50Mhz
    input               rst_n,
    input       [15:0]  lcd_id,
    output  reg         lcd_pclk
    );

//reg define
reg          clk_25m;
reg          clk_12_5m;
reg          div_4_cnt;

//*****************************************************
//**                    main code
//*****************************************************

//时钟2分频 输出25MHz时钟 
always @(posedge clk or negedge rst_n) begin
    if(!rst_n)
        clk_25m <= 1'b0;
    else 
        clk_25m <= ~clk_25m;
end

//时钟4分频 输出12.5MHz时钟 
always @(posedge clk or negedge rst_n) begin
    if(!rst_n) begin
        div_4_cnt <= 1'b0;
        clk_12_5m <= 1'b0;
    end    
    else begin
        div_4_cnt <= div_4_cnt + 1'b1;
        if(div_4_cnt == 1'b1)
            clk_12_5m <= ~clk_12_5m;
		else
			clk_12_5m <= clk_12_5m;
    end        
end

always @(*) begin
    case(lcd_id)
        16'h4342 : lcd_pclk = clk_12_5m;
        16'h7084 : lcd_pclk = clk_25m;       
        16'h7016 : lcd_pclk = clk;
        16'h4384 : lcd_pclk = clk_25m;
        16'h1018 : lcd_pclk = clk;
        default :  lcd_pclk = 1'b0;
    endcase      
end

endmodule
```

时钟分频模块波形：

![在这里插入图片描述](https://img-blog.csdnimg.cn/54b9134df9744e419697bda7e45bd8a5.png)

>根据输入的 ID 为 **16’h4384** 可知， **lcd_pclk**（像素时钟）为 **25Mhz**。从仿真波形图中可以看出 lcd_pclk 的频率，与我们绘制的波形图以及代码是一致的，时钟分频模块的波形是没有问题的。

## 3.4 LCD 驱动模块设计


**LCD** 驱动模块主要是作用是驱动 **RGB-LCD** 显示屏， 将输入模块的彩条图形像素点信息，按照前面介绍的 **RGB-LCD** 的 **DE** 时序扫描显示到 **LCD** 显示器上。 **LCD** 驱动模块框图如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/68b95d4d758e440a8dc8cbccb7e0e9f3.png)

由 LCD 驱动模块框图，我们可以知道 **RGB-LCD** 包含 4 路输入以及 11 路输出信号。

输入信号中，时钟信号 **lcd_clk**，频率为 **25MHz**（由识别 ID 后的分频时钟所得） ，为 **RGB-LCD** 显示屏的工作时钟；复位信号 **rst_n** 为系统的复位信号输入，低电平有效； **pixel_data** 为彩条图像像素点色彩信息数据，由图像数据生成模块产生并传入，在 **RGB-LCD** 有效图像显示区域赋值给信号 **RGB** 图像色彩信息。

>其中 **lcd_hs** 为 LCD 行同步信号， **lcd_vs** 为 LCD 场同步信号， **lcd_bl** 为 LCD 背光控制信号， **lcd_rst** 为 LCD 复位信号。因为 **RGB LCD** 采用 **DE** 模式时，行场同步信号需要拉高，所以可以直接赋值为 1。

>同理 **lcd_bl**（LCD 背光信号）以及 **lcd_rst**（LCD 复位信号）我们这里也直接赋值为 1，不做过多的配置。

而 **h_disp** 和 **v_disp** 为 LCD 屏水平分辨率和 LCD 屏垂直分辨率，它的分辨率带下是根据识别的 **LCD** 屏幕来进行分辨率的赋值。本次波形图的设计均采用尺寸为 4.3 寸，分辨率为 **800*480** 的屏幕进行的设计。在绘制波形图之前，我们还是需要看一下 4384 屏幕的时序参数真值表。

![在这里插入图片描述](https://img-blog.csdnimg.cn/13f12a40a42d4ad49d7f7f726495a5cb.png)

### 3.4.1 绘制波形图


通过模块框图部分，我们对 **LCD** 时序控制模块的具体功能做了说明，对输入输出信号做了简单介绍，那么如何利用模块输入信号实现模块功能，输出我们想要得到的数据信号呢？在波形图绘制部分，我会通过绘制波形图，对各信号做详细讲解，带领您学习掌握模块功能的实现方法。

**LCD** 驱动模块波形图，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/794cd2a9a6764738aea608b6a5cbe525.png)

这是我们最终生成得 **RGB-LCD** 时序驱动模块波形图，下面我们结合 **RGB-LCD** 时序部分，讲解一下绘制波形图的具体思路。

>**第一部分：** 行计数器（**h_cnt**）、场计数器（**v_cnt**）以及 LCD 数据使能（**lcd_de**）信号波形的绘制。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3dd830064f204b4784c4f534d8d1c626.png)

本次绘制的波形图是以 4.3 寸 **RGB-LCD** 显示屏为例，结合**时序参数真值表**（见上图）我们可以知道 **h_sync**（行同步信号）数值为 **128**， **h_back**（行后沿）为 **88**， **h_disp** 为 **800**（行有效数据）以及 **h_total** 为 **1056**（行扫描周期）；**v_sync**（场同步信号）数值为 **2**， **v_back**（场后沿）为 **33**， **v_disp** 为 **480**（场有效数据）以及 **v_total** 为 **525**（场扫描周期）。

**lcd_de** 使能的范围应该等于 **h_disp** 的数值，所以 **lcd_de** 拉高的条件应该为 **h_cnt**（行计数器）大于等于 **h_sync+h_back**（也就是 128+88），且 **h_cnt**（行计数器）小于 **h_sync+h_back+h_disp(128+88+800)** ；并且 **v_cnt**（场计数器）大于等于 **v_sync+v_back(2+33)** ，且 **v_cnt**（场计数器）小于 **v_sync+v_back+v+disp(2+33+480)** 。

>**第二部分：** 像素数据有效信号、像素时钟坐标以及数据请求信号标志位波形的绘制。

![在这里插入图片描述](https://img-blog.csdnimg.cn/99675461454b4e0d84f7fcbca042fea3.png)

第二部分的波形图主要表达在 **lcd_de** 拉高的范围之内， **lcd_rgb**（像素数据）传输有效的像素数据。**pixel_xpos** 和 **pixel_ypos** 为当前像素点的横纵坐标值。另外需要说明一下 **data_req（比 DE 信号提前一拍，当作数据请求信号使用）** 为 **LCD** 驱动模块预留的接口，它的时序要求要比 **lcd_de** 提前一个时钟周期，这个会在我的后续博客中为大家做一个详细的介绍，本实验暂时用不上。


### 3.4.2 编写代码

根据 **LCD** 驱动模块绘制波形图部分的分析，我们可以编写驱动模块的代码，**驱动模块代码**如下：

```
module lcd_driver(
    input                lcd_pclk,    //时钟
    input                rst_n,       //复位，低电平有效
    input        [15:0]  lcd_id,      //LCD屏ID
    input        [23:0]  pixel_data,  //像素数据
    output  reg  [10:0]  pixel_xpos,  //当前像素点横坐标
    output  reg  [10:0]  pixel_ypos,  //当前像素点纵坐标   
    output  reg  [10:0]  h_disp,      //LCD屏水平分辨率
    output  reg  [10:0]  v_disp,      //LCD屏垂直分辨率 
	output  reg          data_req,    //数据请求信号
    //RGB LCD接口
    output  reg          lcd_de,      //LCD 数据使能信号
    output               lcd_hs,      //LCD 行同步信号
    output               lcd_vs,      //LCD 场同步信号
    output               lcd_bl,      //LCD 背光控制信号
    output               lcd_clk,     //LCD 像素时钟
    output               lcd_rst,     //LCD复位
    output       [23:0]  lcd_rgb      //LCD RGB888颜色数据
    );

//parameter define  
// 4.3' 480*272
parameter  H_SYNC_4342   =  11'd41;     //行同步
parameter  H_BACK_4342   =  11'd2;      //行显示后沿
parameter  H_DISP_4342   =  11'd480;    //行有效数据
parameter  H_FRONT_4342  =  11'd2;      //行显示前沿
parameter  H_TOTAL_4342  =  11'd525;    //行扫描周期
   
parameter  V_SYNC_4342   =  11'd10;     //场同步
parameter  V_BACK_4342   =  11'd2;      //场显示后沿
parameter  V_DISP_4342   =  11'd272;    //场有效数据
parameter  V_FRONT_4342  =  11'd2;      //场显示前沿
parameter  V_TOTAL_4342  =  11'd286;    //场扫描周期
   
// 7' 800*480   
parameter  H_SYNC_7084   =  11'd128;    //行同步
parameter  H_BACK_7084   =  11'd88;     //行显示后沿
parameter  H_DISP_7084   =  11'd800;    //行有效数据
parameter  H_FRONT_7084  =  11'd40;     //行显示前沿
parameter  H_TOTAL_7084  =  11'd1056;   //行扫描周期
   
parameter  V_SYNC_7084   =  11'd2;      //场同步
parameter  V_BACK_7084   =  11'd33;     //场显示后沿
parameter  V_DISP_7084   =  11'd480;    //场有效数据
parameter  V_FRONT_7084  =  11'd10;     //场显示前沿
parameter  V_TOTAL_7084  =  11'd525;    //场扫描周期       
   
// 7' 1024*600   
parameter  H_SYNC_7016   =  11'd20;     //行同步
parameter  H_BACK_7016   =  11'd140;    //行显示后沿
parameter  H_DISP_7016   =  11'd1024;   //行有效数据
parameter  H_FRONT_7016  =  11'd160;    //行显示前沿
parameter  H_TOTAL_7016  =  11'd1344;   //行扫描周期
   
parameter  V_SYNC_7016   =  11'd3;      //场同步
parameter  V_BACK_7016   =  11'd20;     //场显示后沿
parameter  V_DISP_7016   =  11'd600;    //场有效数据
parameter  V_FRONT_7016  =  11'd12;     //场显示前沿
parameter  V_TOTAL_7016  =  11'd635;    //场扫描周期
   
// 10.1' 1280*800   
parameter  H_SYNC_1018   =  11'd10;     //行同步
parameter  H_BACK_1018   =  11'd80;     //行显示后沿
parameter  H_DISP_1018   =  11'd1280;   //行有效数据
parameter  H_FRONT_1018  =  11'd70;     //行显示前沿
parameter  H_TOTAL_1018  =  11'd1440;   //行扫描周期
   
parameter  V_SYNC_1018   =  11'd3;      //场同步
parameter  V_BACK_1018   =  11'd10;     //场显示后沿
parameter  V_DISP_1018   =  11'd800;    //场有效数据
parameter  V_FRONT_1018  =  11'd10;     //场显示前沿
parameter  V_TOTAL_1018  =  11'd823;    //场扫描周期

// 4.3' 800*480   
parameter  H_SYNC_4384   =  11'd128;    //行同步
parameter  H_BACK_4384   =  11'd88;     //行显示后沿
parameter  H_DISP_4384   =  11'd800;    //行有效数据
parameter  H_FRONT_4384  =  11'd40;     //行显示前沿
parameter  H_TOTAL_4384  =  11'd1056;   //行扫描周期
   
parameter  V_SYNC_4384   =  11'd2;      //场同步
parameter  V_BACK_4384   =  11'd33;     //场显示后沿
parameter  V_DISP_4384   =  11'd480;    //场有效数据
parameter  V_FRONT_4384  =  11'd10;     //场显示前沿
parameter  V_TOTAL_4384  =  11'd525;    //场扫描周期    

//reg define
reg  [10:0] h_sync ;
reg  [10:0] h_back ;
reg  [10:0] h_total;
reg  [10:0] v_sync ;
reg  [10:0] v_back ;
reg  [10:0] v_total;
reg  [10:0] h_cnt  ;
reg  [10:0] v_cnt  ;

//*****************************************************
//**                    main code
//*****************************************************

//RGB LCD 采用DE模式时，行场同步信号需要拉高
assign  lcd_hs = 1'b1;        //LCD行同步信号
assign  lcd_vs = 1'b1;        //LCD场同步信号

assign  lcd_bl = 1'b1;        //LCD背光控制信号  
assign  lcd_clk = lcd_pclk;   //LCD像素时钟
assign  lcd_rst= 1'b1;        //LCD复位

//RGB888数据输出
assign lcd_rgb = lcd_de ? pixel_data : 24'd0;

//像素点x坐标
always@ (posedge lcd_pclk or negedge rst_n) begin
    if(!rst_n)
        pixel_xpos <= 11'd0;
    else if(data_req)
        pixel_xpos <= h_cnt + 2'd2 - h_sync - h_back ;
    else 
        pixel_xpos <= 11'd0;
end
   
//像素点y坐标   
always@ (posedge lcd_pclk or negedge rst_n) begin
    if(!rst_n)
        pixel_ypos <= 11'd0;
    else if(v_cnt >= (v_sync + v_back)&&v_cnt < (v_sync + v_back + v_disp))
        pixel_ypos <= v_cnt + 1'b1 - (v_sync + v_back) ;
    else 
        pixel_ypos <= 11'd0;
end

//行场时序参数
always @(*) begin
    case(lcd_id)
        16'h4342 : begin
            h_sync  = H_SYNC_4342; 
            h_back  = H_BACK_4342; 
            h_disp  = H_DISP_4342; 
            h_total = H_TOTAL_4342;
            v_sync  = V_SYNC_4342; 
            v_back  = V_BACK_4342; 
            v_disp  = V_DISP_4342; 
            v_total = V_TOTAL_4342;            
        end
        16'h7084 : begin
            h_sync  = H_SYNC_7084; 
            h_back  = H_BACK_7084; 
            h_disp  = H_DISP_7084; 
            h_total = H_TOTAL_7084;
            v_sync  = V_SYNC_7084; 
            v_back  = V_BACK_7084; 
            v_disp  = V_DISP_7084; 
            v_total = V_TOTAL_7084;        
        end
        16'h7016 : begin
            h_sync  = H_SYNC_7016; 
            h_back  = H_BACK_7016; 
            h_disp  = H_DISP_7016; 
            h_total = H_TOTAL_7016;
            v_sync  = V_SYNC_7016; 
            v_back  = V_BACK_7016; 
            v_disp  = V_DISP_7016; 
            v_total = V_TOTAL_7016;            
        end
        16'h4384 : begin
            h_sync  = H_SYNC_4384; 
            h_back  = H_BACK_4384; 
            h_disp  = H_DISP_4384; 
            h_total = H_TOTAL_4384;
            v_sync  = V_SYNC_4384; 
            v_back  = V_BACK_4384; 
            v_disp  = V_DISP_4384; 
            v_total = V_TOTAL_4384;             
        end        
        16'h1018 : begin
            h_sync  = H_SYNC_1018; 
            h_back  = H_BACK_1018; 
            h_disp  = H_DISP_1018; 
            h_total = H_TOTAL_1018;
            v_sync  = V_SYNC_1018; 
            v_back  = V_BACK_1018; 
            v_disp  = V_DISP_1018; 
            v_total = V_TOTAL_1018;        
        end
        default : begin
            h_sync  = H_SYNC_4342; 
            h_back  = H_BACK_4342; 
            h_disp  = H_DISP_4342; 
            h_total = H_TOTAL_4342;
            v_sync  = V_SYNC_4342; 
            v_back  = V_BACK_4342; 
            v_disp  = V_DISP_4342; 
            v_total = V_TOTAL_4342;          
        end
    endcase
end
	
//数据使能信号		
always@ (posedge lcd_pclk or negedge rst_n) begin
    if(!rst_n)	
		lcd_de <= 1'b0;
	else
		lcd_de <= data_req;
end
				  
//请求像素点颜色数据输入  
always@ (posedge lcd_pclk or negedge rst_n) begin
    if(!rst_n)	
		data_req<=1'b0;
	else if((h_cnt >= h_sync + h_back - 2'd2) && (h_cnt < h_sync + h_back + h_disp - 2'd2)
             && (v_cnt >= v_sync + v_back) && (v_cnt < v_sync + v_back + v_disp))
		data_req <= 1'b1;
	else
		data_req <= 1'b0;
end
				  
//行计数器对像素时钟计数
always@ (posedge lcd_pclk or negedge rst_n) begin
    if(!rst_n) 
        h_cnt <= 11'd0;
    else begin
        if(h_cnt == h_total - 1'b1)
            h_cnt <= 11'd0;
        else
            h_cnt <= h_cnt + 1'b1;           
    end
end

//场计数器对行计数
always@ (posedge lcd_pclk or negedge rst_n) begin
    if(!rst_n) 
        v_cnt <= 11'd0;
    else begin
        if(h_cnt == h_total - 1'b1) begin
            if(v_cnt == v_total - 1'b1)
                v_cnt <= 11'd0;
            else
                v_cnt <= v_cnt + 1'b1;    
        end
    end    
end

endmodule
```

下图为 **LCD 驱动模块**部分波形：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f67a77f70d064bc0984a76ede499c915.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/30a3fa285df044eaa298476db53240d6.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/fd36075976724fa0abef410f62cdb6cc.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/65666db1792145bbad577455b2cad660.png)

>根据我们绘制的波形图以及编写的代码，可以看出 **lcd_de** 在 **v_cnt** 为 35~ 514 且 **h_cnt** 为 215~ 1015 的计算器范围内有效，同时 **data_req** 比 **lcd_de** 提前一个时钟周期也是符合我们设计的需求。

## 3.5 LCD 显示模块设计

**LCD** 显示模块的作用是将 **LCD** 驱动模块传入的图像有效显示区域的**横坐标参数（pixel_xpos）** 等分成 5 等份的彩条图像数据并回传给 LCD 驱动模块。

**LCD** 显示模块框图， 如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e10d48773d99447ca9f4455bd8c07201.png)


### 3.5.1 绘制波形图

根据模块设计的分析，可知**本模块是将输入的驱动模块的横坐标等分成 5 等份的彩条图像色彩信息**，再输出到驱动模块。根据这一设计需求，我们可以绘制 **LCD** 显示模块的波形图，波形图如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a624a8186f9241eaa8f5eecfe42ec1d6.png)

>根据输入像素点坐标 **(pixel_xpos， pixel_ypos)** ，在有效显示区域将 **pixel_xpos** 计数范围五等分，在不同的计数部分给 **pixel_data** 赋值对应的色彩信息，因为采用时序逻辑的赋值方式，所以 **pixel_data** 滞后 **pixel_xpos** 、**pixel_ypos** 信号一个时钟周期。

### 3.5.2 编写代码

绘制完波形图后，我们就可以根据波形图进行代码的编写， **LCD** 显示模块的代码如下：


```
module lcd_display(
    input                lcd_pclk,    //时钟
    input                rst_n,       //复位，低电平有效
    input        [10:0]  pixel_xpos,  //当前像素点横坐标
    input        [10:0]  pixel_ypos,  //当前像素点纵坐标  
    input        [10:0]  h_disp,      //LCD屏水平分辨率
    input        [10:0]  v_disp,      //LCD屏垂直分辨率       
    output  reg  [23:0]  pixel_data   //像素数据
    );

//parameter define  
parameter WHITE = 24'hFFFFFF;  //白色
parameter BLACK = 24'h000000;  //黑色
parameter RED   = 24'hFF0000;  //红色
parameter GREEN = 24'h00FF00;  //绿色
parameter BLUE  = 24'h0000FF;  //蓝色

//*****************************************************
//**                    main code
//*****************************************************

//根据当前像素点坐标指定当前像素点颜色数据，在屏幕上显示彩条
always @(posedge lcd_pclk or negedge rst_n) begin
    if(!rst_n)
        pixel_data <= BLACK;
    else begin
        if((pixel_xpos >= 11'd0) && (pixel_xpos < h_disp/5*1))
            pixel_data <= WHITE;
        else if((pixel_xpos >= h_disp/5*1) && (pixel_xpos < h_disp/5*2))    
            pixel_data <= BLACK;
        else if((pixel_xpos >= h_disp/5*2) && (pixel_xpos < h_disp/5*3))    
            pixel_data <= RED;   
        else if((pixel_xpos >= h_disp/5*3) && (pixel_xpos < h_disp/5*4))    
            pixel_data <= GREEN;                
        else
            pixel_data <= BLUE; 
    end    
end
  
endmodule
```

**LCD** 显示模块波形图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/05d1f7081b334f7c9654a46064517ded.png)

根据绘制波形图以及编写的代码，可以看出 **pixel_data** 输出的数据与绘制的波形及代码是一致的。

## 3.6 顶层模块编写

### 3.6.1 代码编写

实验工程的各子功能模块均已讲解完毕，在本小节对顶层模块做一下介绍。 **lcd_rgb_colorbar** 顶层模块主要是对各个子功能模块的实例化，以及对应信号的连接。代码编写较为容易，无需波形图的绘制。

由于 **lcd_rgb** 是 **24 位的双向引脚**，所以这里对**双向引脚**的方向做一个切换。

>当 **lcd_de** 信号为高电平时，此时输出的像素数据有效，将 **lcd_rgb** 的引脚方向切换成输出，并将 **LCD** 驱动模块输出的 **lcd_rgb_o**（像素数据）连接至 **lcd_rgb** 引脚；
>当 **lcd_de** 信号为低电平时，此时输出的像素数据无效，将 **lcd_rgb** 的引脚方向切换成输入。代码中将高阻状态 **“Z”** 赋值给 **lcd_rgb** 的引脚，表示此时 **lcd_rgb** 的引脚电平由外围电路决定，此时可以读取 **lcd_rgb** 的引脚电平，从而获取到 **LCD** 屏的 ID。

顶层模块 **lcd_rgb_colorbar** 的代码如下：

```
module lcd_rgb_colorbar(
    input                sys_clk,     //系统时钟
    input                sys_rst_n,   //系统复位

    //RGB LCD接口
    output               lcd_de,      //LCD 数据使能信号
    output               lcd_hs,      //LCD 行同步信号
    output               lcd_vs,      //LCD 场同步信号
    output               lcd_bl,      //LCD 背光控制信号
    output               lcd_clk,     //LCD 像素时钟
    output               lcd_rst,     //LCD 复位
    inout        [23:0]  lcd_rgb      //LCD RGB888颜色数据
    );                                                      
    
//wire define    
wire  [15:0]  lcd_id    ;    //LCD屏ID
wire          lcd_pclk  ;    //LCD像素时钟
              
wire  [10:0]  pixel_xpos;    //当前像素点横坐标
wire  [10:0]  pixel_ypos;    //当前像素点纵坐标
wire  [10:0]  h_disp    ;    //LCD屏水平分辨率
wire  [10:0]  v_disp    ;    //LCD屏垂直分辨率
wire  [23:0]  pixel_data;    //像素数据
wire  [23:0]  lcd_rgb_o ;    //输出的像素数据
wire  [23:0]  lcd_rgb_i ;    //输入的像素数据

//*****************************************************
//**                    main code
//*****************************************************

//像素数据方向切换
assign lcd_rgb = lcd_de ?  lcd_rgb_o :  {24{1'bz}};
assign lcd_rgb_i = lcd_rgb;

//读LCD ID模块
rd_id u_rd_id(
    .clk          (sys_clk  ),
    .rst_n        (sys_rst_n),
    .lcd_rgb      (lcd_rgb_i),
    .lcd_id       (lcd_id   )
    );    

//时钟分频模块    
clk_div u_clk_div(
    .clk           (sys_clk  ),
    .rst_n         (sys_rst_n),
    .lcd_id        (lcd_id   ),
    .lcd_pclk      (lcd_pclk )
    );    

//LCD显示模块    
lcd_display u_lcd_display(
    .lcd_pclk       (lcd_pclk  ),
    .rst_n          (sys_rst_n ),
    .pixel_xpos     (pixel_xpos),
    .pixel_ypos     (pixel_ypos),
    .h_disp         (h_disp    ),
    .v_disp         (v_disp    ),
    .pixel_data     (pixel_data)
    );    

//LCD驱动模块
lcd_driver u_lcd_driver(
    .lcd_pclk      (lcd_pclk  ),
    .rst_n         (sys_rst_n ),
    .lcd_id        (lcd_id    ),
    .pixel_data    (pixel_data),
    .pixel_xpos    (pixel_xpos),
    .pixel_ypos    (pixel_ypos),
    .h_disp        (h_disp    ),
    .v_disp        (v_disp    ),
	.data_req	   (		  ),
	
    .lcd_de        (lcd_de    ),
    .lcd_hs        (lcd_hs    ),
    .lcd_vs        (lcd_vs    ),
    .lcd_bl        (lcd_bl    ),
    .lcd_clk       (lcd_clk   ),
    .lcd_rst       (lcd_rst   ),
    .lcd_rgb       (lcd_rgb_o )
    );

endmodule
```









# 4 仿真验证

## 4.1 编写 TestBench

顶层模块参考代码介绍完毕，开始对顶层模块进行仿真，对顶层模块的仿真就是对实验工程的整体仿真。

在顶层模块中我们已经重点强调过 **lcd_rgb** 是 24 位的双向引脚了，如果对该引脚还有疑问可以再仔细阅读一下前面的介绍。

当 **lcd_de** 信号为低电平时， 此时进入读取 **lcd_rgb** 的引脚电平，本次仿真的 **RGB-LCD** 屏为 4.3 寸 **800*480** 的显示屏。根据硬件设计部分内容以及读 ID 模块设计中，我们可以知道 4384 的屏幕 ID 为 3’b001，也就是 24’h80。所以只需要在 lcd_de 为低电平时，给 **lcd_rgb** 赋值为 24’h80 就完成了输入端口读 ID 的激励了。**RGB-LCD** 彩条 TB 模块代码编写如下：


```
`timescale 1ns / 1ns

module tb_lcd_rgb_colorbar();

//reg define
reg     sys_clk;
reg     sys_rst_n;     

//wire define
wire          lcd_de ;
wire          lcd_hs ;
wire          lcd_vs ;
wire          lcd_bl ;
wire          lcd_clk;
wire  [23:0]  lcd_rgb;
        
always #10 sys_clk = ~sys_clk;
assign lcd_rgb = lcd_de ?  {24{1'bz}} :  24'h80;

initial begin
    sys_clk = 1'b0;
    sys_rst_n = 1'b0;
    #200
    sys_rst_n = 1'b1;
end

lcd_rgb_colorbar u_lcd_rgb_colorbar(
    .sys_clk          (sys_clk  ),
    .sys_rst_n        (sys_rst_n),

    .lcd_de           (lcd_de ),
    .lcd_hs           (lcd_hs ),
    .lcd_vs           (lcd_vs ),
    .lcd_bl           (lcd_bl ),
    .lcd_clk          (lcd_clk),
    .lcd_rgb          (lcd_rgb)
    );

endmodule
```


## 4.2 代码仿真

接下来打开 **Modelsim** 软件对代码进行仿真，仿真的波形如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/85be6ff534844783a2ac561fb229ee14.png)

从波形图中可以看出， **lcd_rgb** 在 **lcd_de** 拉高的范围内，将 **RGB888** 颜色分为五种颜色，分别是白（24’hffffff）、黑（24’h000000）、红（24’hff0000）、绿（24’h00ff00）和蓝（24’h0000ff）。与我们设计的代码是一致的，而具体模块的波形图，我们也结合前面的各个子功能模块进行了验证分析。至此本章节 **RGB LCD** 彩条实验的设计已经完成，接下来就进入下载验证部分了。











# 5 下载验证

## 5.1 引脚约束

本实验中，各端口信号的**管脚分配**（根据自己的开发板原理图为准）如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/27dfd3387ccc47c0a63b2f08caed1686.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9d83aa73d224106ba23c5c0d18dad8e.png)


## 5.2 上板验证

将显示屏与开发板连接，并连接下载器，将程序下载进开发板，验证 **RGB TFT-LCD** 彩条显示功能。下载完成后观察 **RGB LCD** 模块显示的图案如下图所示， 说明 **RGB TFT-LCD** 彩条显示程序下载验证成功！


![在这里插入图片描述](https://img-blog.csdnimg.cn/8b9c3cf5539147df82b6209540fb0c36.png)














# 6 总结

到这里，本博客关于 **RGB-LCD** 的讲解就完毕，通过实验，相信读者对于 **RGB-LCD** 显示的基本知识和概念，以及 **FPGA** 与 **RGB-LCD** 显示屏之间数据通信流程已经了解，对于 **RGB-LCD** 时序，务必要认真理解并掌握，此博客也是我们入门 **FPGA** 图像处理的第一篇博客，希望对您有所帮助，有兴趣的朋友可以进一步联系我交流。






微博：沂舟Ryan ([@沂舟Ryan 的个人主页 - 微博 ](https://weibo.com/u/7619968945))

GitHub：[ChinaRyan666](https://github.com/ChinaRyan666)

微信公众号：沂舟无限进步

如果对您有帮助的话请点赞支持下吧！



>**认识到有差距但不要低下头去，是你走出迷茫的第一步。**

