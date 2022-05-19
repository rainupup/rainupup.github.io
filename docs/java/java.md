# JAVA基础
## 基本语法

**获取随机值函数：**

​	`` Math.random() ``获取\[0,1)之间的值

​	``(int)(Math.random() * (b - a + 1) + a)``获取\[a,b) 之间的值



### 变量

**变量按照数据类型分为：**

1. 基本数据类型：

   整数型：`byte，short，int，long`

   浮点型：`float，double`

   字符型：`char`

   布尔型：`boolean`

 2. 引用数据类型
    类(`class`)

    ​		接口(`interface`)

    ​		数组(\[\])

**基本数据类型介绍**

1. 整型：`byte(1字节) \\ short(2字节) \\ int(4字节) \\ long(8字节)`

2. 浮点型：`float(4字节) \\ double(8字节)`

3. 字符型：`char (1字符=2字节)`

   	byte范围：-128 ~ 127
   	声明long型变量，必须以"l"或"L"结尾
   	float表示数值的范围比long还大
   	定义float类型变量时，变量要以"f"或"F"结尾


**自动类型转换**

1. **自动类型提升：**

   结论：当容量小的数据类型的变量与容量大的数据类型的变量做运算时，结果自动提升为容量大的数据类型。

   `byte 、char 、short --> int --> long --> float --> double`

   特别的：当`byte、char、short`三种类型的变量做运算时，结果为`int`型

2. **强制类型转换：自动类型提升运算的逆运算。**
   需要使用强转符：()

   注意点：强制类型转换，可能导致精度损失。

````java
long l1 = 21332423235234123;    //错误
long l2 = 21332423235234123L;   //正确
        
        
float f1 = 12.3;                //编译失败
float f2 = 12.3F;               //正确
float f3 = (float)12.3;         //正确
        

byte b = 12;
byte b1 = b + i;                //编译失败
float f4 = b + 12.3;            //编译失败
````



**String类型变量的使用**

1.  `String`可以和8种基本数据类型变量做运算，且运算只能是连接运算：+
2.  运算的结果仍然是`String`类型

````java
char c = 'a';//97 A:65
int num = 10;
String str = "hello";
System.out.println(c + num + str);    //107hello
System.out.println(c + str + num);    //ahello10
System.out.println(c + (num + str));  //a10hello
System.out.println((c + num) + str);  //107hello
System.out.println(str + num + c);    //hello10a
````

### 运算符

| 比较运算符 |                    |
| ---------- | ------------------ |
| instanceof | 检查是否是类的对象 |

| 逻辑运算符 |                                                              |
| ---------- | ------------------------------------------------------------ |
| &          | 逻辑与，左右都参与运算                                       |
| &&         | 短路与，如果左边为真，右边参与运算，如果左边为假，那么右边不参与运算 |
| \|         | 逻辑或，左右都参与运算                                       |
| \|\|       | 短路或，当左边为真，右边不参与运算                           |
| ！         | 非                                                           |
| ^          | 异或                                                         |

| 位运算符       |            |                                                              |
| -------------- | ---------- | ------------------------------------------------------------ |
| <<             | 左移       | 空位补0，被移除的高位丢弃，空缺位补0。                       |
| >>             | 右移       | 被移位的二进制最高位是0，右移后，空缺位补0；最高位是1，空缺位补1。 |
| >>>            | 无符号右移 | 被移位二进制最高位无论是0或者是1，空缺位都用0补。            |
| &              | 与运算     | 二进制位进行&运算，只有1&1时结果是1，否则是0;                |
| \|             | 或运算     | 二进制位进行\| 运算，只有0 \| 0时结果是0，否则是1            |
| ^              | 异或运算   | 相同二进制位进行^ 运算，结果是0；1^1=0 , 0^0=0不相同二进制位^ 运算结果是1。1^0=1 , 0^1=1 |
| ~              | 取反运算   | 正数取反，各二进制码按补码各位取反负数取反，各二进制码按补码各位取反 |

**三元运算符**

``(条件表达式)?表达式1：表达式2；``条件表达式为真：运算的结果为表达式1,条件表达式为假：运算的结果为表达式2



### 获取输入

具体实现步骤：

1.导包：``import java.util.Scanner;``

2.``Scanner``的实例化:``Scanner 变量 = new Scanner(System.in);``

3.调用``Scanner``类的相关方法（``next() / nextXxx()``），来获取指定类型的变量

注意：

1. 需要根据相应的方法，来输入指定类型的值。如果输入的数据类型与要求的类型不匹配时，会报异常：``InputMisMatchException``导致程序终止。

2. 对于`char`型的获取，`Scanner`没有提供相关的方法。只能获取一个字符串

   ````java
   Scanner scan = new Scanner(System.in);	//实例化定义变量scan
   int num = scan.nextInt();               //调用变量的Scanner类方法
   System.out.println(num);
   ````



### 流程控制

**if - else**

**switch - case** (`break,continue,default`)

① 根据`switch`表达式中的值，依次匹配各个`case`中的常量。一旦匹配成功，则进入相应`case`结构中， MLML调用其执行语句。当调用完执行语句以后，则仍然继续向下执行其他`case`结构中的执行语句，直到遇到`break`关键字或此`switch-case`结构末尾结束为止。

② `break`,可以使用在`switch-case`结构中，表示一旦执行到此关键字，就跳出`switch-case`结构

③` switch`结构中的表达式，只能是如下的6种数据类型之一(没有浮点型)： ​	``byte 、short、char、int、枚举类型(JDK5.0新增)、String类型(JDK7.0新增)``

④ `case `之后只能声明常量。不能声明范围。(例：case 1 \~ 10 错误)

⑤` break`关键字是可选的。

⑥ `default`:相当于if-else结构中的else.default结构是可选的，而且位置是灵活的。

**for 和 while**

**break标签**

````java
int i =0;
int j = 0;
label:while(true){
    //我是第一层循环
    while(true){
        //我是第二层循环
        if(j*i == 81)
        break label; // continue label
        j++;
    }
    i++;
}
````

``label``:就是标签 要终止的位置

``break label`` ：终止结束到标签 结束语句

``continue label``: 终止本次循环 跳到标签位置进行接下来的循环



### 数组

遍历数组函数``Arrays.tostring(数组)``

#### 数组的概述

特点：

- 数组属于引用数据类型的变量。数组的元素，既可以是基本数据类型，也可以是引用数据类型

- 创建数组对象会在内存中开辟一整块连续的空间

- 数组的长度一旦确定，就不能修改。




#### 一维数组

1. 初始化

   ​	``静态：数据类型[]变量名 = new 数据类型[]{元素1，元素2……}；``

   ​	``动态：数据类型[]变量名 = new 数据类型[长度];``

   ​	注意：赋值不写长度，写长度不赋值

2. 默认值

   - 数组元素是整型：0

   - 数组元素是浮点型：0.0


   - 数组元素是`cha`r型：0或'\\u0000'，而非'0'


   - 数组元素是`boolean`型：`false`


   - 数组元素是引用数据类型：`null`



#### 二维数组

1. 概述：

​		数组属于引用数据类型

​		数组的元素也可以是引用数据类型

2. 初始化

​		``静态：数据类型[][] 变量 = new 数据类型[][]{{元素1，元素2……}，{元素1，元素2……}}``

​		``动态：数据类型[][] 变量 = new 数据类型[长度][长度];``

3. 默认值

   - 外层元素的初始化值为：地址值

   - 内层元素的初始化值为：与一维数组初始化情况相同

````java
int[][] arr = new int[4][3];
System.out.println(arr[0]);   //[I@15db9742 
System.out.println(arr[0][0]);//0
````



#### Arrays 工具类的使用

具体实现步骤：

导包：``import java.util.Arrays;``

常用方法：

````java
boolean equals(int[] a,int[] b);     //判断两个数组是否相等
String toString(int[] a)             //输出数组的信息
void fill(int[] a,int val)           //将指定值填充到数组之中。
void sort(int[] a)                   //对数组进行排序。
int binarySearch(int[] a,int key)    //对排序后的数组进行二分法检索指定的值，返回下标。
````

````java
int[] arr3 = new int[10];
Arrays.fill(arr3,3);
````







# 面向对象

## 类和对象

`Java`类及类的成员：属性、方法、构造器；代码块、内部类 

面向对象的大特征：封装性、继承性、多态性、(抽象性)

关键字：``this、super、static、final、abstract、interface、package、import``等



### 属性与局部变量

属性：可以在声明属性时，指明其权限，使用权限修饰符。

​	常用的权限修饰符：``private、public、缺省、protected`` --->封装性

局部变量：不可以使用权限修饰符。没默认初始化值。意味着，我们在调用局部变量之前，一定要显式赋值。

内存中加载的位置：

​         属性：加载到堆空间中  （非``static``）

​         局部变量：加载到栈空间

 

## 方法

**格式：**

````java
权限修饰符  返回值类型  方法名(形参列表){
         方法体                           
}
````

> 方法的重载

定义：在**同一个类**中，允许存在一个以上的同名方法，只要它们的参数个数或者参数类型不同即可。（与返回类型无关）



> 可变个数形参

**格式**

````java
//数据类型 ... 变量名
public void show(String ... strs){
    ……
}
````



## 封装性

四种权限修饰符

​	权限从小到大顺序为：`private < 缺省(default) < protected < public`

| 修饰符            | 类内部 | 同一个包 | 不同包的子类 | 同个工程 |
| ----------------- | ------ | -------- | ------------ | -------- |
| private(私有)     | yes    |          |              |          |
| 缺省(默认)        | yes    | yes      |              |          |
| protected(受保护) | yes    | yes      | yes          |          |
| public (公开)     | yes    | yes      | yes          | yes      |



4种权限都可以用来修饰类的内部结构：属性、方法、构造器、内部类

修饰类的话，只能使用：缺省、public



## 继承性

**格式**

````java
class 子类 extends 父类 {
	……
} 
````



> 方法的重写

子类重写方法的方法名和形参列表与父类相同，权限修饰符大于父类，不能重写父类中声明为private权限的方法

返回值类型：

- 父类返回值类型是void，则子类返回值类型只能是void
- 父类返回值类型是A类型，则子类返回值类型可以是A类或A类的子类
- 父类返回值类型是基本数据类型(比如：double)，则子类返回值类型必须是相同的基本数据类型(必须也是double)
- 子类重写的方法抛出的异常类型不大于父类被重写的方法抛出的异常类型
- 子类和父类中的同名同参数的方法要么都声明为非static的（考虑重写），要么都声明为static的（不是重写)。    





## 多态性

父类的引用指向子类的对象（或子类的对象赋给父类的引用）

有了对象的多态性以后，我们在编译期，只能调用父类中声明的方法，但在运行期，我们实际执行的是子类重写父类的方法。



对象的多态性，只适用于方法，不适用于属性（编译和运行都看左边）

原因：

  * 属性是静态绑定

- 方法是动态绑定



**instanceof的使用：**

​	格式：``变量名instanceof 类名``,判断变量是否是类名的实例；

````java
class Base {
    int count = 10;
    public void display() {
        System.out.println(this.count);
    }
}

class Sub extends Base {
    int count = 20;
    public void display() {
        System.out.println(this.count);
    }
}
public class FieldMethodTest {
    public static void main(String[] args) {
        Sub s = new Sub();
        System.out.println(s.count);   //20
        s.display();                   //20

        Base b = s;//多态性
        //==：对于引用数据类型来讲，比较的是两个引用数据类型变量的地址值是否相同
        System.out.println(b == s);    //true
        System.out.println(b.count);   //10 多态性不适用属性
        b.display();                   //20
    }
}
````



## Object类

**java.lang.Object类**

Object类是所有Java类的根父类

如果在类的声明中未使用extends关键字指明其父类，则默认父类为java.lang.Object类 

方法：`equals() / toString() / getClass() /hashCode() / clone() / finalize() / wait() / notify() / notifyAll()`



**Object类方法如下：**

![图形用户界面, 文本, 应用程序  描述已自动生成](java.assets/clip_image001.png)



 **1.getClass方法**

获取运行时类型,返回值为`Class`对象

**2.hashCode方法**

返回该对象的哈希码值，是为了提高哈希表的性能（`HashTable`）

**3.equals方法**

判断两个对象是否相等，在`Object`源码中`equals`就是使用==去判断，所以在`Object`中`equals`是等价于==的，但是在`String`及某些类对`equals`进行了重写，实现不同的比较。

**4.clone方法**

主要是JAVA里除了8种基本类型传参数是值传递，其他的类对象传参数都是引用传递，我们有时候不希望在方法里将参数改变，这时就需要在类中复写`clone`方法。

如果在`clone`方法中调用`super.clone()`方法需要实现`Cloneable`接口,否则会抛出`CloneNotSupportedException`。

此方法只实现了一个浅层拷贝,对于基本类型字段成功拷贝,但是如果是嵌套对象,只做了赋值,也就是只把地址拷贝了,所以没有成功拷贝,需要自己重写`clone`方法进行深度拷贝。

**5.toString方法**

返回一个`String`字符串,用于描述当前对象的信息,可以重写返回对自己有用的信息，默认返回的是`当前对象的类名+hashCode`的16进制数字。

**6.wait方法**

多线程时用到的方法，作用是让当前线程进入等待状态，同时也会让当前线程释放它所持有的锁。直到其他线程调用此对象的 `notify() `方法或 `notifyAll() `方法，当前线程被唤醒

**7.notify方法**

多线程时用到的方法，唤醒该对象等待的某个线程

**8.notifyAll方法**

多线程时用到的方法，唤醒该对象等待的所有线程

**9.finalize**

对象在被GC释放之前一定会调用`finalize`方法，对象被释放前最后的挣扎,因为无法确定该方法什么时候被调用，很少使用。

 

## 包装类

为了使基本数据类型的变量具有类的特征，引入包装类。

| 基本数据类型 | 包装类    |
| ------------ | --------- |
| byte         | Byte      |
| shout        | Shout     |
| int          | Integer   |
| long         | Long      |
| float        | Floa      |
| double       | Double    |
| boolean      | Boolean   |
| char         | Character |



### 类型转换

> 基本数据类型<--->包装类，JDK 5.0 新特性：**自动装箱 与自动拆箱**



> 基本数据类型、包装类--->String

格式：``String 变量 = String.valueOf(数据或变量)；``



> String--->基本数据类型、包装类

格式：``数据类型 变量 = 数据类型包装类.parseXxx(String S)；``

````java
String S = "123456";
int in1 = Integer.parseInt(S);
````



**integer**

`Integer`内部定义了`IntegerCache`结构，`IntegerCache`中定义了`Integer[]`,保存了从-128~127范围的整数。如果我们使用自动装箱的方式，给`Integer`赋值的范围在

-128~127范围内时，可以直接使用数组中的元素，不用再去`new`了。目的：提高效率

````java
@Test
public void test3() {
    Integer m = 1;
    Integer n = 1;
    System.out.println(m == n);     //true,在-128~127范围内
    
    /*大于127*/
    Integer x = 128;                //相当于new了一个Integer对象
    Integer y = 128;                //相当于new了一个Integer对象
    System.out.println(x == y);     //false
}
````







## 关键字

**this**



**super**

> super调用构造器：

- 可以在子类的构造器中显式的使用"`super(形参列表)`"的方式，调用父类中声明的指定的构造器
- "`super(形参列表)`"的使用，必须声明在子类构造器的首行！
- 我在类的构造器中，针对于"`this(形参列表)`"或"`super(形参列表)`"只能二选一，不能同时出现
- 在构造器的首行，没显式的声明"`this(形参列表)`"或"`super(形参列表)`"，则默认调用的是父类中空参的构造器：`super()`
- 在类的多个构造器中，至少一个类的构造器中使用了"`super(形参列表)`"，调用父类中的构造器



**package**“ 

包”，指：类所在的包


**import**

“引入”，指：引入类中需要的类。

`import static`: 导入指定类或接口中的静态结构、属性或方法。 