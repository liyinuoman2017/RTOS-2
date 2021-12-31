# RTOS-2
嵌入式实时操作系统2——任务调度子系统
**程序在处理器中如何运行**
要理解任务调度，就需要先理解任务切换。
要理解任务切换，就要先理解程序在处理器中如何运行。

**处理器内部结构**
要理解程序在处理器中如何运行，需要先了解一下处理器内部结构，处理器结构框图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/00c714c4ab254a35bede282e28d651e9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_15,color_FFFFFF,t_70,g_se,x_16)

处理器通常包括：寄存器堆，运算单元，控制单元，流水线结构，指令存储器，数据存储器。
通常情况下处理器执行指令有5个阶段：取指，译码，执行，访存，回写。
![在这里插入图片描述](https://img-blog.csdnimg.cn/278ce4ff2a334aa98fbfe76c26b80c51.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_16,color_FFFFFF,t_70,g_se,x_16)

取指：从指令存储器中读取指令
译码：指令译码，读取寄存器
执行：执行操作或计算地址
访存：从数据存储器中读取操作数
回写：将结果回写到寄存器中

**处理器简化模型**
 处理器可以简化为：寄存器堆，运算单元，指令存储器，数据存储器。简化模式如下：
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/f91e63c7168b4cf091c88e77553b4b54.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_15,color_FFFFFF,t_70,g_se,x_16)

处理器模型中会产生变化，并影响程序运行流程的部分为：
1、寄存器堆，包括PC，通用寄存器堆，状态寄存器。
2、指令存储器。
3、数据存储器。

**寄存器堆**
以ARM CM3内核为例，寄存器堆包含17个寄存器：

![在这里插入图片描述](https://img-blog.csdnimg.cn/97c8034e4b224472beda40b21b8e7e6e.png)

处理器是通过操作寄存器堆实现程序运行，通过几个汇编指令来熟悉一下处理器工作方式。

![在这里插入图片描述](https://img-blog.csdnimg.cn/106ef68c45634a2083eb84335a31a48b.png)

处理器操作寄存器是需要遵循一定规定的，如ARM架构的C编译器遵循AAPCS规范。其中有一项规定为：函数调用使用R0~R3作为参数输入（大于4个参数使用栈操作），这也是为什么很过C语言编程规范中规定函数参数要不超过4个，因为超过4个会有一个额外的操作栈的过程，影响函数效率。

**指令存储器**
在嵌入式领域，代码的二进制格式存放在指令存储器中，指令存储器使用只读存储器ROM。目前有许多指令存储器使用NOR-FLASH,使用NOR-FLASH的优势是：
1、可反复擦写
2、支持芯片内执行（XIP）

指令存储器内部不仅仅是保存代码数据，同时还保存静态区的非零数据。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4667fbb8648e444f972fea4535df71cf.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_15,color_FFFFFF,t_70,g_se,x_16)

非零静态变量区是从指令存储器中直接加载到RAM中完成初始化。如果使用的是ARM的内核芯片，系统将调用scatter-loading完成这个加载工作。
指令存储器内部的数据绝大数情况下是不会变的（有些特殊的应用使用处理的片内FLASH保存一些用户数据）。
汇编代码如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ca2aed8eb8d54141869e9ffd4c7ee91c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)

代码编译之后的MAP文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4b547d125b774e4797c43311a8e2336d.png)
根据MAP文件可知：代码段和静态数据段在指令存储空间中，RESET段地址为0X08000000（FLASH起始地址），代码段地址为0X08000010 ，静态数据段址为0X08000098。

**数据存储器**
在嵌入式领域，数据存储器通常是SRAM,数据存储器分为以下3个区域：
1、静态区，存放静态变量数据。
2、栈区，存放局部变量数据。
3、堆区，存放动态申请的数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/515d3ee678fe4a0c9418a5772f9bdfb5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_15,color_FFFFFF,t_70,g_se,x_16)
静态区中存放静态变量和全局变量。示例中红色框内变量为静态变量：
![在这里插入图片描述](https://img-blog.csdnimg.cn/0da2e8b85944443c9de1fa09324c8510.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_11,color_FFFFFF,t_70,g_se,x_16)
button_flag ， communication_flag ，senser_flag存放在静态区中的初始化不为0区，clk存放在静态区中的初始化为0区。
局部变量，函数参数，中断保存保存寄存器，都会操作栈空间。
![在这里插入图片描述](https://img-blog.csdnimg.cn/aaa3c00a666a4ac7ae5188a0600c2f3c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_13,color_FFFFFF,t_70,g_se,x_16)
**程序运行资源**
程序运行时，会用到如下资源：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ecfc5b1f96a454a8bc26ed911057a7b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_10,color_FFFFFF,t_70,g_se,x_16)
处理器模型中会产生变化，并影响程序运行的部分为：
1、寄存器堆的数据
2、栈空间的数据（空间大小会变，内容会变）
3、堆空间的数据（空间大小会变，内容会变）
4、静态空间的数据（内容会变）

**程序运行过程**
假设有一程序，程序内有一个无限循环，在循环内部有5个表达式，代码如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a75b91203934069942632724bfa7021.png)
程序运行后，会依次执行表达式1-》表达式2-》表达式3-》表达式4-》表达式5-》表达式1无限循环。**假设没有使用静态变量，没有使用堆空间，没有中断程序。**程序每执行一个表达式后的处理器状态如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/449dff4726cf43d2b674dda2015a4a0e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)
**程序循环周期执行，处理器的状态也循环周期变化。我们将在一个时钟周期内指令存储器，数据存储器，寄存器堆的所有数据称为：处理器总值。上图中处理器总值为P1,P2,P3,P4,P5 。**
假设现在有一种“神奇的力量”将处理的状态改变成P3，处理将如何运行？
处理器将会按照下图运行：![在这里插入图片描述](https://img-blog.csdnimg.cn/067bca6f56724843a2adc4f937700c85.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)
对应的程序运行状态：表达式3-》表达式4-》表达式5-》表达式1-》表达式2-》表达式3无限循环。
我们可以得出一个结论：**在不考虑外设，中断等因素，给处理器一个合理总值Pn,处理器的下一个总值必然为Pn+1 。**
**利用这个原理，给处理器任意一个合理总值Pn，使得程序从任意一个合理位置开始运行。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/92110dcb0048414d8c8cb7af1587469a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)
这种机制就像我们把游戏存档，接下来可以随时恢复一个存档继续进行，游戏运行效果不受是否存档影响。

我们一起来看看常见的几种使用改变总值来改变程序运行状态的例子。
**示例1：处理器复位。**
处理器复位就是强制给处理器一个总值P0,让程序从P0重新开始运行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/de7ea4eed45442cd8253ec54d17009bd.png)
**示例2：系统中断程序。**
用户程序在运行过程中，系统中断产生后，处理保存现场（保存“相对总值”），执行中断程序，恢复现场（恢复“相对总值”），返回用户程序，用户程序继续执行，对于用户程序认为它是在完整的运行，从未被中断过。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d560656d75f34d6fa6cef0bdbdace679.png)
**示例3：操作系统程序切换。**
任务A运行中，产生时钟节拍中断，保存现场（保存任务A“相对总值”），时钟节拍函数运行，更新就绪表，选择最高优先级任务B，恢复现场（恢复任务B“相对总值”），任务B运行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/feb355934ea244178034970e5f111d2d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_10,color_FFFFFF,t_70,g_se,x_16)
**因此我们可以使用改变总值的方法，来切换任务，这就是任务切换的核心思想。**


> <font color=red>**未完待续…
实时操作系统系列将持续更新
创作不易希望朋友们点赞，转发，评论，关注。
您的点赞，转发，评论，关注将是我持续更新的动力
作者：李巍
Github：liyinuoman2017**
