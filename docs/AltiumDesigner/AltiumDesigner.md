# 创建文件

![1694054679805](AltiumDesigner.assets/1694054679805.png)



原理图`.SchDoc`

PLC`.PcbDoc`

原理图库`.SchLib`

PCB库`.PcbLib`





芯片 U？	IC

电阻 R？	RES

电容 C？	CAP

排针 J？	HEADER

二极管 D？

光耦 U？

电阻的表示方式是R，电容的表示方式是C，电感的表示方式是L，[二极管](https://zhidao.baidu.com/search?word=二极管&fr=iknow_pc_qb_highlight)的表示方式是D

## **安装库文件**

安装所需要的库文件，可以下载别人的库，进行导入

![ee91d789d3a1d34bfe28dc838493fd4](AltiumDesigner.assets/ee91d789d3a1d34bfe28dc838493fd4.png)



# 原理图

## **格点**

设置格点大小

![1694055250875](AltiumDesigner.assets/1694055250875.png)



## **网络标号**

![1693981170084](AltiumDesigner.assets/1693981170084.png)

按Table键，设置网络标号名字

## 筛选

![4ba8ffd4cfb24110a63949e67861ecc](AltiumDesigner.assets/4ba8ffd4cfb24110a63949e67861ecc.png)



## **反转**

空格：90度反转

X：水平反转

Y：竖直反转



需选中：鼠标按住不放再按快捷键



## **画线**

![1693981504051](AltiumDesigner.assets/1693981504051.png)

按住空格可以画直线

右键结束画线



## **添加文字**

![1693981599450](AltiumDesigner.assets/1693981599450.png)

## **放置尺寸**

![1694428531913](AltiumDesigner.assets/1694428531913.png)





## **标号**

标号管理器

![1694422539409](AltiumDesigner.assets/1694422539409.png)







**一键生成标号**

![1693981828078](AltiumDesigner.assets/1693981828078.png)

先清空，在生成

## **封装**

封装需要自己画，或者使用别人的封装，都需要导入封装文件，文件后缀为.Pcblib

版本不一样，有些许差别，只需要找到Footprint添加封装即可

![image-20230906143915814](AltiumDesigner.assets/image-20230906143915814.png)



T + J 调用封装管理器：可以查看元件的封装，或修改封装

![1694424092424](AltiumDesigner.assets/1694424092424.png)



shift C



# PCB

## 元器件摆放

### **元器件摆放位置**

![1694427939288](AltiumDesigner.assets/1694427939288.png)



Arranger Within Rectangle：在矩形框中摆放

Arranger Outside Board：摆放在板子周围



### **设置原点**

![1693984691699](AltiumDesigner.assets/1693984691699.png)



快捷键 E O S设置原点

### **设置板子大小**

先画好线，可以借助原点

shift选择四周的线，再按图操作

![1693985125266](AltiumDesigner.assets/1693985125266.png)

### **交叉选择模式**

![1694437981851](AltiumDesigner.assets/1694437981851.png)

在原理图中框选的元器件，在PCB内就会被框选。如果会框选到其他引脚，打开筛选即可

### **摆放元器件**

XY轴放置元器件：先框选元器件，按M 选择按XY轴放置。是元器件源坐标为基点



### **锁定元器件**

锁定元器件，不让元器件挪动

选中元器件

![1693985224535](AltiumDesigner.assets/1693985224535.png)

### **对齐**

![1693983974021](AltiumDesigner.assets/1693983974021.png)







## **层**

![1694436929397](AltiumDesigner.assets/1694436929397.png)



![c8951243be1cf2c2ed2a20c20c5ea7b](AltiumDesigner.assets/c8951243be1cf2c2ed2a20c20c5ea7b.png)

正片层（英文名为Layer或者signal）就是在整版的绝缘体上走线，走线的地方是铜，不走线的地方是绝缘体。

负片层（英文名为Plane）就是在整版的铜片上走线，走线的地方是绝缘体，不走线的地方是铜。（GND,VCC）

Core 芯板材

Prepreg PP压材



|       层       |                            描述                             |
| :------------: | :---------------------------------------------------------: |
|   top layer    |                       顶层，用来走线                        |
|  bottom layer  |                       底层，用来走线                        |
|   mechanical   |                机械层，用来定义PCB形状和尺寸                |
| keepout layer  |                 禁止布线层，用来绘制禁布区                  |
|  top overlay   |                    顶层丝印层，绘制丝印                     |
| bottom overlay |                    底层丝印层，绘制丝印                     |
|   top paste    |     顶层助焊层，用来开钢网时，让焊锡落到板子上，助焊用      |
|  bottom paste  |                         底层助焊层                          |
|   top solder   |             顶层阻焊层，用于描述绿油的覆盖区域              |
| bottom solder  |                         底层阻焊层                          |
|  drill guide   |                         过孔引导层                          |
| drill drawing  |                         过孔钻孔层                          |
|   multilayer   | 多层，如下文自己添加的层，也即除了顶层底层以外的正/负片层。 |

以四层板为例：其中两层专门设置VCC/GND，剩下两层为走线层



**切换层**

移动元器件时按L键 即可将元器件放在底层





## **过孔**

小键盘的`*`号键：放置过孔 并 切换层



2号键 放置过孔

L键 切换层 



盲孔：只露出一面表层

埋孔：不露出表层

通孔：贯穿所有层



**设置过孔规则**

设置为24mil、12mil

![1693988048791](AltiumDesigner.assets/1693988048791.png)

过孔大小一般为12mil(便宜)

过孔盘 =  2*过孔大小+-2



过孔设置规则后不会生效，需要

![1694491646531](AltiumDesigner.assets/1694491646531.png)

Tented：过孔盖油



**埋孔**

在不同层之间打孔

![1694494699038](AltiumDesigner.assets/1694494699038.png)

如果没有properties：panels-properties

需要先保存，再选择埋孔属性

点击过孔，设置埋孔属性

![1694495270629](AltiumDesigner.assets/1694495270629.png)

**缝合孔**

批量添加过孔

![1694497021042](AltiumDesigner.assets/1694497021042.png)

孔缝合是一种用于将不同层上较大的铜区域连接在一起的技术，实际上可以通过电路板结构创建牢固的垂直连接，有助于保持低阻抗和短返回环路。通孔缝合也可用于将可能与其网络隔离的铜区域连接到该网络。





## 飞线

### **隐藏飞线**

快捷键N

![1693985304161](AltiumDesigner.assets/1693985304161.png)

### **隐藏电源线**

为了方便起见，可以隐藏GND、VCC等

快捷键 D C

创建一个新网络类

![image-20230912121404253](AltiumDesigner.assets/image-20230912121404253.png)

隐藏class

![image-20230912121424409](AltiumDesigner.assets/image-20230912121424409.png)



### **布线**

Ctrl + W布线



**交互式布线**

对多跟相邻的线同时布线，先选中

![1694492730307](AltiumDesigner.assets/1694492730307.png)

### **自动布线**

![1693988397948](AltiumDesigner.assets/1693988397948.png)



## 设置规则

### Class

为飞线分类，快捷键D C

![image-20230912120000610](AltiumDesigner.assets/image-20230912120000610.png)



### **设置间距**

一般最低设置为6，因为在不加钱的情况下，厂家最低为6

designer-rules

![1693988218099](AltiumDesigner.assets/1693988218099.png)

可以进行高级设置，Advanced，设置各个种类之间的间距



**为某类线设置规则**，注意优先级，在上面优先级越高，注意使能

以PWR分类为例

![image-20230912120403564](AltiumDesigner.assets/image-20230912120403564.png)







### **设置线规则**

以设置线宽度为例

可以设置：整个板、1个网络、多个网络等

![image-20230906161555675](AltiumDesigner.assets/image-20230906161555675.png)

比如单独设置一个5V的线，选择1 Net进行下面的设置

![1693987804498](AltiumDesigner.assets/1693987804498.png)



## **规则检查**

![1694427211562](AltiumDesigner.assets/1694427211562.png)

设置检查规则，是否忽略规则

### **报错**

T M复位绿色报错

### **工程选项**

![1694424570071](AltiumDesigner.assets/1694424570071.png)

可以设置错误提醒、忽略错误等

### **电气规则检查**

检查错误

![image-20230906165230466](AltiumDesigner.assets/image-20230906165230466.png)

寻找错误，在PCB页面双击错误就可以放大出错的地方

![c0eaf5f219f970e0a9eaa867755a1ce](AltiumDesigner.assets/c0eaf5f219f970e0a9eaa867755a1ce.png)

















## **铺铜**

铺铜的目的，可以进行散热



1. 画好区域进行铺铜

![65df92d906737796dbb4324165e75c0](AltiumDesigner.assets/65df92d906737796dbb4324165e75c0.png)



2. 设置网络，对GND进行铺铜

![994b3d5a72a178b91e36564bde834af](AltiumDesigner.assets/994b3d5a72a178b91e36564bde834af.png)



![9e190125629d779a2d17156bed40b6a](AltiumDesigner.assets/9e190125629d779a2d17156bed40b6a.png)

remove dead copper：删除死区铺铜，即删除没有用到的铺铜区



**多边形挖空**

![1694493863292](AltiumDesigner.assets/1694493863292.png)

删除掉多边形内的铺铜，先画出删除的部分，再重新铺铜



铺铜管理器、重新铺铜

![ea18f2778ee31ce21e8c1d0e833cd0f](AltiumDesigner.assets/ea18f2778ee31ce21e8c1d0e833cd0f.png)

**隐藏铺铜**

快捷键L

![1697957841089](AltiumDesigner.assets/1697957841089.png)







## **设置丝印**

选中一个丝印，右击，查找相似对象

![image-20230906164449182](AltiumDesigner.assets/image-20230906164449182.png)



## **2D/3D切换**

2号键：切换2D

3号键：切换3D



## 联合

组合多个元器件：选择多个元器件，单击

![1694496240716](AltiumDesigner.assets/1694496240716.png)

## 脚本

![1694496078551](AltiumDesigner.assets/1694496078551.png)



## 拼板

拼板分为：

* V-cut：切一刀
* 邮票孔：打多个孔

拼板步骤：

创建一个新的PCB文件

![image-20230912134222282](AltiumDesigner.assets/image-20230912134222282.png)



PCB Document：要拼板的源文件

Column Count：横向个数

Row Count：竖向个数



## 生成文件

### 装配图

![image-20230912135403071](AltiumDesigner.assets/image-20230912135403071.png)



### BOM表

清单

![image-20230912135728952](AltiumDesigner.assets/image-20230912135728952.png)



### Gerber

**生成Gerber文件**

![image-20230912140713831](AltiumDesigner.assets/image-20230912140713831.png)

**钻孔文件**

![image-20230912140831921](AltiumDesigner.assets/image-20230912140831921.png)



**坐标文件**

![image-20230912141013380](AltiumDesigner.assets/image-20230912141013380.png)

**IPC网表**

生产厂家使用此表，再次对开短路核对；虽然我们已经DRC检查过了，这样双保险

![image-20230912141243878](AltiumDesigner.assets/image-20230912141243878.png)



## **快捷键**

设置快捷键：Ctrl + 鼠标点击，即可设置对应快捷键

Ctrl + D ：设置显示的内容(比如显示丝印、铺铜等)



左键选中元器件 按方向键可以进行微调



Ctrl + M ：测量距离



快捷键A ：对齐方式



Shift + s 切换单层显示 

Shift + 左键 ：高亮网络

E->J->C,然后输入标号就行:按位号查找



mm（毫米）和mil（米尔）的切换：点击PCB空白处后，然后通过快捷方式“V”+“U”即可完成切换，第二种方法是通过快捷方式“Q”直接进行切换











# 原理图库

## 创建原理图库

![1693991908535](AltiumDesigner.assets/1693991908535.png)

后缀`.SchLib`



## 手动添加

**添加原理图**

两种方法

![image-20230906172147173](AltiumDesigner.assets/image-20230906172147173.png)



**放置矩形**

![1693992290638](AltiumDesigner.assets/1693992290638.png)

也可以放置其他形状



**放置引脚**

两种方法

![image-20230906172716790](AltiumDesigner.assets/image-20230906172716790.png)

按Table修改引脚信息



修改描述信息的方向

![1694056792143](AltiumDesigner.assets/1694056792143.png)

上划杆

比如要在描述EN上添加上划杆，这`\E\N\`,即可





注意：要将pin引脚的小白点(放大可以看到)朝外，因为小白点方向是引出引脚



**阵列粘贴**

![9d2cf5c2f423cf4eb352aad48b1cb08](AltiumDesigner.assets/9d2cf5c2f423cf4eb352aad48b1cb08.png)



![b9bbbe4f26f25f9b6cdfa8e6333196c](AltiumDesigner.assets/b9bbbe4f26f25f9b6cdfa8e6333196c.png)





itemCount ：粘贴个数

Primary increment ：pin号增量，即每次增加几个数

Secondary increment ：此增量，即pin角描述每次增加几个数

## 使用向导

**使用向导添加**

使用上面方法，需手动添加，比较麻烦，还可以选择向导进行快速添加

![b83918fc040dff4597d9863b98564af](AltiumDesigner.assets/b83918fc040dff4597d9863b98564af.png)



![1693993108477](AltiumDesigner.assets/1693993108477.png)

可以使用Excel类似的操作方式，批量操作，引脚类型最好选择Passive，直接放置即可



**修改**

![1693993375643](AltiumDesigner.assets/1693993375643.png)

修改页面不能像Excel类似的操作方式，所以尽量在添加页面设置好



## 使用嘉立创

在嘉立创EDA的元件库中搜索自己想要的元器件原理图，导出为Altium Designer。

在AD中打开，这样就可以复制元器件原理图，粘贴到原理图中了(直接复制嘉立创的内容是没办法粘贴导原理图库中的)

虽然能在AD中使用嘉立创的原理图了，但是并没有创建为原理图库，不方便后续使用。



原本下载的文件后缀为`.schdoc`，并不是`.schlib`

![1693994827567](AltiumDesigner.assets/1693994827567.png)



使用Make Schematic Library即可将嘉立创下载的文件转为原理图库

这个选项也可以将一个大原理图生成多个原理图库(在原理图界面使用此命令，即获得别人的原理图后就能生成原理图库)

## 导入原理图库

安装所需要的库文件，可以下载别人的库，进行导入

![ee91d789d3a1d34bfe28dc838493f](AltiumDesigner.assets/ee91d789d3a1d34bfe28dc838493fd4.png)



# 封装库

## 创建封装库

![1693996181963](AltiumDesigner.assets/1693996181963.png)

后缀`.PcbLib`





## PCB生成PCB库

通过别人的PCB生成PCB库，在PCB页面

![1694424767118](AltiumDesigner.assets/1694424767118.png)

会自动生成PCB封装库，然后将生成的PCB封装库复制到自己的库中即可



复制单个PCB封装：在PCB页面，直接复制，在PCB库页面的列表直接粘贴即可





## 绘制3D模型

![1694425900099](AltiumDesigner.assets/1694425900099.png)

属性设置

![1694425543400](AltiumDesigner.assets/1694425543400.png)



Generic：可以导入其他软件绘制的模型

Extruded：绘制不规则图形

Cylinder：绘制圆形

Sphere：绘制球体



OverallHeight：整体高度

Standoff Height：浮空高度



**导入其他软件绘制的模型**

![1694425866777](AltiumDesigner.assets/1694425866777.png)

或者直接放置3D body，效果一样







# 其他

**电容**

黄色

电容之间的换算公式：

（1）1F（法拉）=1000 mF（毫法）

（2）1mF（毫法）=1000 μF（微法）

（3）1μF（微法）=1000 nF（纳法）

（4）1nF（纳法）=1000 pF（皮法）



**电阻**

黑色

电阻单位为[欧姆](https://zhidao.baidu.com/search?word=欧姆&fr=iknow_pc_qb_highlight)，用[希腊字母](https://zhidao.baidu.com/search?word=希腊字母&fr=iknow_pc_qb_highlight)“Ω”表示。电阻从大到小的单位都有MΩ（兆欧）、kΩ（千欧）、Ω（欧）、mΩ（毫欧）、μΩ（微欧）

公制的单位，进制以千为步长。电阻的是：
1GΩ=1000MΩ
1MΩ=1000KΩ
1KΩ=1000Ω
1Ω=1000mΩ
1mΩ=1000nΩ
1nΩ=1000pΩ

知道1mΩ是多少Ω了吧，千分之一Ω。





**在VCC电源旁边 接电容接地通常有以下几个目的：**

1. 去噪声和滤波：这个电容器通常被称为去耦电容或维持电容。它可以帮助滤除电源线上的高频噪声，确保电源电压在稳定的直流电平上。这对于确保电路的正常工作非常重要，尤其是在数字电路中。

2. 电源稳定性： 当某些组件或部分电路需要瞬时更多的电流时（例如，逻辑门的瞬态开关），这个电容可以提供额外的电流支持，以确保电源电压的稳定性。这对于防止电源电压崩溃或闪烁是有益的。

3. 降低电磁干扰（EMI）： 电容也可以减少电路中的电磁干扰，帮助确保电路的性能和可靠性。

一般来说，将适当大小的电容器连接到电源电压旁边是一个常见的做法，以确保电源质量，减少电路的噪声敏感性，并维持电路的稳定性。选择电容的具体大小通常取决于电路的需求和工作频率