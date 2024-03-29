# TIM定时中断

定时器共四个部分，分为八个小节笔记。本小节为第一部分第一节。

在第一部分，是定时器的基本定时的功能：定时中断功能、内外时钟源选择

在第二部分，是定时器的输出比较功能，最常见的用途是产生PWM波形，用于驱动电机等设备

在第三部分，是定时器的输入捕获功能和主从触发模式，来实现测量方波频率

在第四部分，是定时器的编码器接口，能够更加方便读取正交编码器的输出波形，编码电机测速

## TIM简介
TIM（Timer）定时器，定时触发中断

定时器本质上就是一个计数器

定时器可以对输入的时钟进行计数（在stm32中定时器的基准时钟一般是主频72MHz，如果对72MHz记72个数，那就是1MHz也就是1us的时间（72MHz就是1秒记72M个数，可以理解为对72个数计数1M次，记72个数的频率就是1MHz，用时1us）），如果记72000个数，那就是1KHz也就是1ms的时间，并在计数值达到设定值时触发中断

stm32的定时器拥有16位（2的16次方是65536）的计数器（计数器就是用来执行计数定时的寄存器，每来一个时钟，计数器加1）、预分频器（可以对计数器的时钟进行分频，让计数更加灵活）、自动重装寄存器（是计数的目标值，计多少个时钟申请中断）的时基单元，在72MHz计数时钟下可以实现最大59.65s的定时。

预分频值（PSC）、自动重装载值（ARR）。

定时器的计数频率 = 时钟频率 / （PSC + 1）

最大定时时间 = （ARR + 1）/ 定时器的计数频率

即，最大定时时间 = (ARR + 1) * (PSC + 1) / 时钟频率

不仅具备基本的定时中断功能，而且还包含内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等多种功能

根据复杂度和应用场景分为了高级定时器、通用定时器、基本定时器三种类型
|类型|编号|总线|功能|
|:-:|:-:|:-:|:-:|
|高级定时器|TIM1、TIM8|APB2|拥有通用定时器全部功能，并额外具有重复计数器、死区生成、互补输出、刹车输入等功能|
|通用定时器|TIM2、TIM3、TIM4、TIM5|APB1|拥有基本定时器全部功能，并额外具有内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等功能|
|基本定时器|TIM6、TIM7|APB1|拥有定时中断、主模式触发DAC的功能|

除了TIM1-8，在库函数中还出现了TIM9、10、11等（这些一般都用不到）

高级定时器额外具有的重复计数器、死区生成、互补输出、刹车输入等，这些功能主要是为了三相无刷电机的驱动设计的

STM32F103C8T6定时器资源：TIM1、TIM2、TIM3、TIM4；不同的型号，定时器的数量是不同的

## 基本定时器
理解时基单元的工作流程（定时器产生中断的全部流程）、主模式触发DAC的功能，如下内容：
### 基本定时器时基单元

![](https://pic.xhcheats.cn/assets/2024/01/15/023530.png)

下面这三个构成了最基本的计数计时电路，所以这一块电路就叫做时基单元

时基单元：预分配器（PSC）、自动重装载寄存器（ARR）、计数器（CNT）

![](https://pic.xhcheats.cn/assets/2024/01/15/023603.png)

### 时基单元的工作流程
内部时钟的来源是RCC_TIMxCLK，频率值是系统的主频72MHz，所以通向时基单元的计数基准频率就是72MHz

进入时基单元首先是预分频器（PSC），它可以对72MHz的计数时钟进行预分频（比如，预分频器写0就是不分频输出72MHz，写1是进行二分频输出36MHz，写2是三分频输出24MHz  ..，所以预分频的值和实际的分频系数相差1，即，实际分频系数=预分频器的值+1），预分频器是16位的，最大值可以写65535，也就是最大65536分频。

然后是计数器，对预分频后的计数时钟进行计数，计数时钟每来一个上升沿，计数器的值加1，这个计数器的值也是16位的，值可以从0一直加到65535，如果再加的话，计数器就会回到0重新开始。所以计数器的值在计时过程中会不断地自增运行，当自增运行到目标值时，产生中断，那就完成了定时的任务，所以还需一个存储目标值的寄存器，那就是自动重装载寄存器了

自动重装寄存器也是16位的，它存的是我们写入的计数目标，在运行的过程中，计数值不断自增，自动重装载是固定的目标，当计数值等于自动重装值时，也就是计时时间到了，那它就会产生一个中断信号，并且清零计数器，计数器自动开始下一次的计数计时。像这种计数值等于自动重装值产生的中断，叫做“更新中断”，这个更新中断之后就会通向NVIC，我们再配置好NVIC的定时器通道，那定时器的更新中断就能够得到CPU的响应。

总结定时器产生中断的全部流程：从基准时钟到预分频器再到计数器，计数器计数自增，同时不断地与自动重装寄存器进行比较，值相等时，即计时时间到，这时就会产生一个更新中断和更新事件，CPU响应更新中断，就完成了我们定时中断的任务了。

下图红圈，是一个向上的折线箭头，就代表这里会产生中断信号，像这种计数值等于自动重装值产生的中断，叫做“更新中断”。

![](https://pic.xhcheats.cn/assets/2024/01/15/023644.png)

下图红圈，是一个向下的折线箭头，代表的是产生一个事件，这里对应的事件就叫做“更新事件”，更新事件不会触发中断，但可以触发内部其它电路的工作。

![](https://pic.xhcheats.cn/assets/2024/01/15/023701.png)

以上就是定时中断和时基单元的工作流程。

### 主模式触发DAC的功能

下面，简单介绍一下（后续讲），主模式触发DAC的功能，stm32定时器的一大特色就是主从触发模式（主从触发模式能让内部的硬件在不受程序的控制下实现自动运行），如果能把主从触发模式掌握好，那在某些情景下将会极大地减轻CPU的负担。

主模式触发DAC的作用就是，在我们使用DAC的时候，可能会用DAC输出一段波形，那就需要每隔一段时间来触发一次DAC，让它输出下一个电压点。如果用正常的思路来实现的话，就是先设置一个定时器产生中断，每隔一段时间在中断程序中调用代码手动触发一次DAC转换，然后DAC输出，这样会使主程序处于频繁被中断的状态，这会影响主程序的运行和其他中断的响应，所以定时器就设计了一个主模式，使用这个主模式可以把定时器的更新事件映射到触发输出TRGO（Trigger Out）的位置，然后TRGO直接接到DAC的触发转换引脚上，这样，定时器的更新就不需要再通过中断来触发DAC转换了，仅需要把更新事件通过主模式映射到TRGO，然后TRGO就会直接区触发DAC，整个过程不需要软件的参与，实现了硬件自动化，这就是主模式的作用，当然除了主模式外，还有更多硬件自动化的设计（后续讲）

![](https://pic.xhcheats.cn/assets/2024/01/15/023734.png)

 以上，就是基本定时器的内容

 ## 通用定时器

 ### 通用定时器与基本定时器异同

 首先，中间最核心的部分，还是时基单元，如下，这部分结构和工作流程和基本定时器是一样的，不过对于通用定时器而言，计数器的计数模式就不止向上计数一种了（向上自增），通用定时器和高级定时器支持向上计数模式、向下计数模式和中央对齐模式。（基本定时器仅支持向上计数模式）。最常用的还是向上计数模式。

向下计数模式就是从重装值开始，向下自减，减到0之后，回到重装值同时申请中断，然后继续下一轮，依次循环

中央对齐模式就是从0开始，先向上自增，计到重装值，申请中断，然后再向下自减，减到0，再申请中断，然后继续下一轮，依次循环

![](https://pic.xhcheats.cn/assets/2024/01/15/023815.png)

### 内外时钟源选择功能

如下，是内外时钟源选择和主从触发模式的结构。

![](https://pic.xhcheats.cn/assets/2024/01/15/023836.png)

内外时钟源选择：对于基本定时器，定时只能选择内部时钟，也就是系统频率72MHz；对于通用定时器，时钟源可以选择内部时钟或者外部时钟

外部时钟的选择有如下四种：

第一个外部时钟就是来自TIMx_ETR引脚上的外部时钟，可以在TIM2的ETR引脚也就是PA0上接一个外部方波时钟，然后配置一下内部的极性选择、边沿检测和预分频器电路，再配置一下输入滤波电路，这两块电路可以对外部时钟进行一定的整形（因为是外部时钟，所以难免会有毛刺，这些电路就可以对输入的波形进行滤波），同时也可以选择一下极性和预分频器，最后滤波后的信号，兵分两路，上面一路ETRF进入触发控制器，紧跟着就可以选择作为时基单元的时钟了，在stm32中，这一路也叫做‘外部时钟模式2’（如图中红线）；另一路与其他信号通过一个数据选择器输出TRGI（Trigger In，触发输入），当这个TRGI当作外部时钟来使用时，这一路就称为 外部时钟模式1（如图中黄线所示）。后者从名字上看，它主要是作为触发输入来使用的，这个触发输入可以触发定时器的从模式。关于从模式的内容之后再涉及，本节主要考量把这个触发输入当作外部时钟来考虑的情况。

![](https://pic.xhcheats.cn/assets/2024/01/15/023902.png)

TIMx_ETR引脚的位置可以参考引脚定义表中关于默认复用功能和重定义功能的定义，如下图所示。可以看到TIM2的CH1和ETR都复用在了引脚PA0上。其他定时器的引脚也可以在表中找到。

![](https://pic.xhcheats.cn/assets/2024/01/15/023920.png)

第二个外部时钟可以是来自其他定时器的信号ITR

![](https://pic.xhcheats.cn/assets/2024/01/15/024001.png)

主模式的输出TRGO可以通向其他定时器，实际上通向的就是ITR引脚，通过这一路就可以实现定时器级联的功能。如上如黄线所示，ITR0到ITR3分别来自其他4个定时器的TRGO输出，具体的连接方式如下表所示，这就是ITR和定时器的连接关系。实现定时器级联功能例如，可以先初始化TIM3，然后使用主模式把它的更新事件映射到TRGO上，接着再初始化TIM2，选择ITR2对应的就是TIM3的TRGO，然后后面再选择时钟为外部时钟模式1，这样TIM3的更新事件就可以驱动TIM2的时基单元，也就是实现了定时器的级联

![](https://pic.xhcheats.cn/assets/2024/01/15/024028.png)

第三个外部时钟可来自TIMx_CH1的TI1_ED，CH1引脚的边沿，即从CH1引脚连接的输入捕获模块获得时钟，ED意为Edge，意为通过这一路的时钟，上升沿和下降沿均有效。

第四个外部时钟可来自TIMx_CH1的TI1FP1和来自TIMx_CH2的TI2FP2

> 总结一下，外部时钟模式1的输入可以是ETR引脚、其他定时器、CH1引脚的边沿、CH1引脚和CH2引脚；外部时钟模式2的输入只能是ETR引脚。如果要使用外部时钟，首选ETR引脚外部时钟模式2的输入，这一路最简单最直接

以上就是有关时钟输入的部分

### 编码器接口功能

图中的编码器接口，它可以读取正交编码器的输出模型，后续的课程也会讲到。

![](https://pic.xhcheats.cn/assets/2024/01/15/024133.png)

### 主从触发模式功能
图中的TRGO与基本定时器类似，它可以将定时器内部的一些事件映射到其他电路，从而完成其他电路的功能。

![](https://pic.xhcheats.cn/assets/2024/01/15/024145.png)

### 输出比较功能

通用定时器结构图的右下角即为定时器的输出比较功能的结构，如下图所示。有四个输出通道，分别对应CH1到CH4的引脚，可以用来输出PWM波形，驱动电机。

![](https://pic.xhcheats.cn/assets/2024/01/15/024221.png)

### 输入捕获电路

通用定时器的左下角即为输入捕获电路的结构图，它同输出比较功能一样有四个通道，对应CH1到CH4。可以用于测量输入方波的频率。因为输入捕获和输出比较不能同时使用，故中间的捕获/比较寄存器是输入捕获和输出比较电路共用的，CH1到CH4的引脚也是共用的。

![](https://pic.xhcheats.cn/assets/2024/01/15/024243.png)

## 高级定时器
高级定时器拥有通用定时器全部功能，并额外具有重复计数器、死区生成、互补输出、刹车输入等功能。高级定时器结构框图如下图所示：

![](https://pic.xhcheats.cn/assets/2024/01/15/024304.png)

![](https://pic.xhcheats.cn/assets/2024/01/15/024314.png)

高级定时器的大部分结构和通用定时器相同，只在部分作了功能拓展。相比于通用定时器，拓展了框图右边红圈的内容。

![](https://pic.xhcheats.cn/assets/2024/01/15/024329.png)

### 重复次数计数器

在申请中断的的信号输出处，增加了一个重复次数计数器，它的作用是：可以实现每隔几个计数周期，才发生一次更新事件和中断。原来的结构是每个计数周期完成后就都会发生更新，现在这个计数器实现每隔几个周期再更新一次，相当于对输出的更新信号又作了一次分频。（对于高级定时器，我们之前计算的最大定时时间59秒多，在这里就还需要再乘一个65536，也就是提升了很多的定时时间）

![](https://pic.xhcheats.cn/assets/2024/01/15/024753.png)

下面部分，是高级定时器对输出比较模块的升级了，暂时了解即可

### 死区生成电路与三相无刷电机

图中的DTG和DTG寄存器组成死区生成电路，右侧的引脚TIMx_CH1/CH2/CH3由原来的每路一个变成了两个互补的输出引脚（TIMx_CH1/CH2/CH3和TIMx_CH1N/CH2N/CH3N），可以输出一对互补的PWM波。这些电路是为了驱动三相无刷电机设计的。在四轴飞行器、电动车后轮、电钻中都可以发现三相无刷电机。三相无刷电机的驱动电路需要三个桥臂，每个桥臂需要2个大功率开关管来控制，总共需要6个大功率开关管控制。所以输出的PWM引脚的前三路就变为了互补的输出引脚，而第四路TIMx_CH4没有变化。三相电机只需要三路。
  为了防止互补输出的PWM驱动桥臂时，在开关切换的瞬间，由于器件的不理想，造成短暂的直通现象，故添加了死区生成电路。在开关切换的瞬间，产生一定时长的死区，让桥臂的上下管全部关断，防止出现直通现象。

### 刹车输入

刹车输入的主要作用是给电机驱动提供安全保障。如果外部引脚BKIN（Break In）产生了刹车信号，或者内部时钟失效，产生了故障，控制电路就会自动切断电机的输出，防止意外的发生。

## 定时中断基本结构

定时中断的基本结构如下图所示：

![](https://pic.xhcheats.cn/assets/2024/01/15/024431.png)

在定时器中最核心的部分是时基单元。图中的“运行控制”就是控制寄存器的一些位，用来启动或停止计数器，配置向上向下计数方式等，操作这些寄存器就能控制时基单元的运行了。时基单元左边关于时钟源选择的部分，在上文都有详细的叙述，这里不再赘述。
  计时时间到后，产生的中断信号会先在状态寄存器中置一个中断标志位，这个标志位会通过中断输出控制，到NVIC申请中断。这个中断输出控制存在的原因是：定时器模块有很多地方都要申请中断，如果需要该中断，就允许输出；如果不需要这个中断，就禁止输出。简单来说，中断输出控制就是中断输出的允许位。

## 时基单元运行时序举例

STM32中，关于时序运行的内容很多，具体请见手册的详细讨论，这里仅举一些时基单元的例子作简要分析。

### 缓冲（影子）寄存器

结构图中如下红圈中，带黑色阴影的寄存器，都是有影子寄存器这样的缓冲机制，包括预分频器，自动重装载寄存器和捕获比较寄存器；这个缓冲寄存器是用还是不用，是可以自己设置的。

STM32在设计之初，为了保证能适用于多种多样的情况，故对时序运行过程中突然手动更改寄存器对时序的影响作了严谨的设计。这里引入缓冲（影子）寄存器，主要目的就是同步，即可以让寄存器设定的某些目标值的变化和更新事件同时发生，防止在运行途中更改造成错误。在定时器结构图中，有些寄存器的画法采用了方框下加阴影的方式，就说明该寄存器不是只有一个寄存器，而是有两个寄存器来形成缓冲机制。实际上，真正使时序电路状态发生更改的都是影子寄存器。

![](https://pic.xhcheats.cn/assets/2024/01/15/024856.png)

### 预分频器时序分析

下图描述了当预分频器的分频系数从1变为2时，计数器的时序图。第一行是CK_PSC是预分频器的输入时钟，这个时钟在不断运行；下面的CNT_EN是计数器使能，高电平计数器正常运行，低电平计数器停止，再下面是CK_CNT是计数器时钟既是预分频器的时钟输出也是计数器的时钟输入。开始时，计数器未使能，计数器时钟不运行；然后使能后，前半段，当计数器使能信号CNT_EN变为高电平后的下一个CK_PSC的高电平，定时器时钟CK_CNT接收CK_PSC。且此时预分频器的分频系数为1，PSC = 0，预分频器完成一分频，计数器时钟等于预分频前的时钟，即，CK_PSC = CK_CNT；后半段，预分频系数变为2，计数器时钟变为预分频前时钟的一半。

在计数器时钟的驱动下，下面的计数器寄存器也跟随时钟的上升沿不断自增；当计数器寄存器的值依次递增达到0xFC后立即跳变为0x00，说明重装载寄存器ARR设计的目标计数值就是0xFC，此时电路产生一个更新事件脉冲信号UEV，并产生中断信号，计数值清0。这就是一个计数周期的工作流程。

然后是最下面的三行时序，描述的是预分频寄存器的一种缓冲机制，也就是这个预分频寄存器实际上是有两个：一个是倒数第三行的预分频控制寄存器，供读写用并不直接决定分频系数；另一个是倒数第二行的预分频缓冲寄存器（影子寄存器），才是真正起作用的寄存器，

在更新事件信号之前在TIMx_PSC中写入新数值，将预分频器的分频系数从1改为2，但是由于缓冲寄存器的存在，CK_CNT不会立即变为CK_PSC / 2，而是在下一次更新中断产生的同时，由预分频缓冲器（影子寄存器）修改分频系数为2，PSC = 1。

由预分频计数器时序可以看到，预分频的分频功能实际上也是通过计数器来实现的。当分频系数变为2后，预分频计数器按0、1、0、1依次计数，每当预分频计数器回到0时，预分频器输出信号，CN_CNT输出一个脉冲。

![](https://pic.xhcheats.cn/assets/2024/01/15/024938.png)

> 计数器计数频率：CK_CNT = CK_PSC / (PSC + 1)

### 计数器时序分析

1 计数器工作时序图如下

![](https://pic.xhcheats.cn/assets/2024/01/15/025011.png)

内部分频因子为2，就是分频系数为2。第一行是内部时钟72MHz，第二行是时钟使能，高电平启动，第三行是计数器时钟，因为分频系数为2，所以这个频率是上面CK_INT除2，然后计数器在这个时钟每个上升沿自增，当增到0036时发生溢出，之后再来一个上升沿，计数器清零，产生一个更新事件脉冲，另外还会置一个更新中断标志位UIF，标志位UIF只要置1就会去申请中断，然后中断响应后，需要在中断程序中手动清零，以上就是计数器的工作流程。

> 计数器溢出频率：CK_CNT_OV = CK_CNT / (ARR + 1) = CK_PSC / (PSC + 1) / (ARR + 1)

> 计数器溢出时间：1/计数器溢出频率

2 计数器无预装时序图（缓冲机制失效 APRE = 0）

计数器无预装时序就是没有缓冲寄存器的情况。

在计数器正在进行自增计数，突然更改了自动加载寄存器，就是自动重装载寄存器由FF改成了36，即计数值的目标值就由FF变成了36，所以计数器寄存器计到36之后，就直接更新，开始下一轮计数。

![](https://pic.xhcheats.cn/assets/2024/01/15/025110.png)

3.计数器有预装时序（缓冲机制有效 APRE = 1）

计数器有预装时序就是有缓冲寄存器的情况。

通过设置ARPE位，就可以选择是否使用预装功能。

在有预装的情况下，在计数中途，若突然将自动加载寄存器计数目标由F5改成了36，下面影子寄存器才是真正起作用的，它还是F5，所以现在计数的目标还是计到F5，产生更新事件，同时，要更改的36才被传递到影子寄存器，在下一个计数周期这个更改的36才有效（类似10086，本月更改，下月生效），所以引入影子寄存器的目的实际上是为了同步，就是让值的变化和更新事件同步发生，防止在运行途中更改造成错误。

在上面这个例子中，若不用影子寄存器的话，更改TIMx_ARR寄存器的值有一种不严谨情况：当F5改到36立即生效，但此时计数器已经到了F1，已经超过36了，F1只能增加，但它的目标值却是36比F1小，此时计数器寄存器的值只能递增，故该寄存器会一直递增到最大值0xFFFF之后回到0x0000，再依次递增，再加到36，才能产生更新。这里就可以看出，如果不使用缓冲机制，可能会给电路时序的工作造成一些问题。

![](https://pic.xhcheats.cn/assets/2024/01/15/025142.png)

### RCC时钟树简介

RCC时钟树：在STM32中用来产生和配置时钟，并且把配置好的各个外设都发射到各个外设的系统。 时钟是所有外设运行的基础，所以时钟是最先配置的东西。在程序执行时，在执行主程序之前还会执行一个SystemInit函数，这个函数的作用就是配置RCC时钟树。

RCC时钟树可以分为左右两部分：时钟产生电路（左）和时钟分配电路（右）。中间的SYSCLK就是系统时钟72MHz。

![](https://pic.xhcheats.cn/assets/2024/01/15/025213.png)

RCC时钟树可以分为左右两部分：时钟产生电路（左）和时钟分配电路（右）。

1.时钟产生电路

 在时钟产生电路，有四个振荡源，分别是内部的8MHz高速RC振荡器、外部的4-16MHz高速晶振振荡器（也就是晶振，一般都外接8MHz）、外部的32.768kHz低速晶振振荡器（一般给RTC提供时钟）、内部的40kHz低速RC振荡器（给看门狗WDG提供时钟）。上面的连个该高速晶振是用来提供系统时钟的，AHB\APB2\APB1的时钟都是来源于这两个高速晶振。内部和外部都有一个8MHz的晶振，只不过外部的石英振荡器比内部的RC振荡器更加稳定，所以一般都用外部晶振。如果系统非常简单，且不需要过于精确的时钟，就可以使用内部的RC振荡器，这样可以省下外部的晶振电路。

  在SystemInit函数中是这样来配置时钟的：首先会启动内部的8MHz高速RC振荡器产生时钟，选择该时钟为系统时钟，暂时以8MHz的内部时钟运行；然后再启动外部时钟，配置外部时钟信号流经如下图所示的电路：

外部晶振信号进入PLLMUL锁相环进行倍频，8MHz倍频9倍，得到72MHz，待锁相环输出稳定后，选择锁相环输出为系统时钟。这样就把系统时钟从8MHz切换为了72MHz，以上就是配置的过程。

![](https://pic.xhcheats.cn/assets/2024/01/15/025246.png)

这样分析可以解决一个问题：如果外部晶振出问题，可能会出现程序时钟慢大概10倍的现象。如果外部时钟的硬件电路有问题（晶振短路或连接错误等），系统的时钟就无法切换到72MHz，会保持内部的8MHz运行。8M相比于72M大概就慢了10倍。

  图中的CSS称为时钟安全系统，它同样负责切换时钟。CSS可以检测时钟的运行状态，一旦外部时钟失效，它就会自动把外部时钟切换为内部时钟，从而保证程序可以正常运行，不会卡死造成事故。另外在高级定时器的刹车输入功能中，一旦CSS检测到外部时钟失效，通过或门就会立刻反应到输出控制器，让输出控制的电机立刻停止，防止意外。（即切断输出控制引脚，切断电机输出，防止发生意外。）

2.时钟分配电路

  首先系统时钟72MHz进入AHB总线，在AHB总线上有一个预分频器，在SystemInit函数配置的默认分频系数为1，所以AHB总线的时钟自然是72MHz。

  之后信号进入APB1总线，APB1上同样有预分频器，这里SystemInit默认配置的分频系数为2，输出为36MHz，所以APB1总线的时钟为36MHz。通用定时器和基本定时器是接在APB1上的，但是APB1（APB2同理）连接定时器还有如图所示的以下结构：

![](https://pic.xhcheats.cn/assets/2024/01/15/025318.png)

通用定时器和基本定时器通过图中APB1下方的支路与APB1连接。由于APB1的预分频系数默认为2，则输出到定时器的时钟频率×2。APB2的预分频器的分频系数默认配置为1，其他流程与APB1同理。所以基本定时器，通用定时器，高级定时器的内部基准时钟都是72MHz，这样设计为我们使用定时器带来了方便，不用考虑不同定时器时钟不同的问题了（前提是不乱修改SystemInit函数中的默认配置）。

  在时钟输出端口，都有一个与门进行时钟输出控制。控制端外部时钟使能就是程序中 RCC_APB2/1 PeriphClockCmd 函数作用的地方。打开时钟就是写1进行按位与。

注意：（实际）预分频系数=预分频器的值+1

## 参考手册

![](https://pic.xhcheats.cn/assets/2024/01/15/025348.png)