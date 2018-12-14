##class 文件解析
参考深入理解《深入理解Java虚拟机 JVM高级特性与最佳实践》第三部分第6章
1. 先写个简单的Hello world class文件
````java
	public class Hello{
	public static void main(String[] args){
		System.out.println("Hello World");
	}
}
````
javac 编译
````
javac -g Hello.java   #g参数表示生成调试信息
````
之后得到对应的class文件Hello.class
先用javap 看下文件结构，javap 讲class文件生成java字节码
````shell
javap -v Hello
````
将会输出一堆信息,主要分为几部分
 文件基本信息，常量池字符，方法执行字节码，源文件
````
javap -v HelloWorld
#文件基本信息
Classfile /root/java/HelloWorld.class
  Last modified Dec 10, 2018; size 533 bytes
  MD5 checksum 66b2780fc2408f1e92e98e77f09d781b
  Compiled from "HelloWorld.java"
public class HelloWorld
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
#常量池
Constant pool:
   #1 = Methodref          #6.#20         // java/lang/Object."<init>":()V
   #2 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #23            // Hello world
   #4 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #26            // HelloWorld
   #6 = Class              #27            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               LHelloWorld;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               SourceFile
  #19 = Utf8               HelloWorld.java
  #20 = NameAndType        #7:#8          // "<init>":()V
  #21 = Class              #28            // java/lang/System
  #22 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
  #23 = Utf8               Hello world
  #24 = Class              #31            // java/io/PrintStream
  #25 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
  #26 = Utf8               HelloWorld
  #27 = Utf8               java/lang/Object
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
{
#构造方法
  public HelloWorld();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   LHelloWorld;
#main方法
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello world
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 3: 0
        line 4: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
}
SourceFile: "HelloWorld.java"
````

接下来从二进制流来看class文件结构
````shell
hexdump -C HelloWorld.class 
00000000  ca fe ba be 00 00 00 34  00 22 0a 00 06 00 14 09  |.......4."......|
00000010  00 15 00 16 08 00 17 0a  00 18 00 19 07 00 1a 07  |................|
00000020  00 1b 01 00 06 3c 69 6e  69 74 3e 01 00 03 28 29  |.....<init>...()|
00000030  56 01 00 04 43 6f 64 65  01 00 0f 4c 69 6e 65 4e  |V...Code...LineN|
00000040  75 6d 62 65 72 54 61 62  6c 65 01 00 12 4c 6f 63  |umberTable...Loc|
00000050  61 6c 56 61 72 69 61 62  6c 65 54 61 62 6c 65 01  |alVariableTable.|
00000060  00 04 74 68 69 73 01 00  0c 4c 48 65 6c 6c 6f 57  |..this...LHelloW|
00000070  6f 72 6c 64 3b 01 00 04  6d 61 69 6e 01 00 16 28  |orld;...main...(|
00000080  5b 4c 6a 61 76 61 2f 6c  61 6e 67 2f 53 74 72 69  |[Ljava/lang/Stri|
00000090  6e 67 3b 29 56 01 00 04  61 72 67 73 01 00 13 5b  |ng;)V...args...[|
000000a0  4c 6a 61 76 61 2f 6c 61  6e 67 2f 53 74 72 69 6e  |Ljava/lang/Strin|
000000b0  67 3b 01 00 0a 53 6f 75  72 63 65 46 69 6c 65 01  |g;...SourceFile.|
000000c0  00 0f 48 65 6c 6c 6f 57  6f 72 6c 64 2e 6a 61 76  |..HelloWorld.jav|
000000d0  61 0c 00 07 00 08 07 00  1c 0c 00 1d 00 1e 01 00  |a...............|
000000e0  0b 48 65 6c 6c 6f 20 77  6f 72 6c 64 07 00 1f 0c  |.Hello world....|
000000f0  00 20 00 21 01 00 0a 48  65 6c 6c 6f 57 6f 72 6c  |. .!...HelloWorl|
00000100  64 01 00 10 6a 61 76 61  2f 6c 61 6e 67 2f 4f 62  |d...java/lang/Ob|
00000110  6a 65 63 74 01 00 10 6a  61 76 61 2f 6c 61 6e 67  |ject...java/lang|
00000120  2f 53 79 73 74 65 6d 01  00 03 6f 75 74 01 00 15  |/System...out...|
00000130  4c 6a 61 76 61 2f 69 6f  2f 50 72 69 6e 74 53 74  |Ljava/io/PrintSt|
00000140  72 65 61 6d 3b 01 00 13  6a 61 76 61 2f 69 6f 2f  |ream;...java/io/|
00000150  50 72 69 6e 74 53 74 72  65 61 6d 01 00 07 70 72  |PrintStream...pr|
00000160  69 6e 74 6c 6e 01 00 15  28 4c 6a 61 76 61 2f 6c  |intln...(Ljava/l|
00000170  61 6e 67 2f 53 74 72 69  6e 67 3b 29 56 00 21 00  |ang/String;)V.!.|
00000180  05 00 06 00 00 00 00 00  02 00 01 00 07 00 08 00  |................|
00000190  01 00 09 00 00 00 2f 00  01 00 01 00 00 00 05 2a  |....../........*|
000001a0  b7 00 01 b1 00 00 00 02  00 0a 00 00 00 06 00 01  |................|
000001b0  00 00 00 01 00 0b 00 00  00 0c 00 01 00 00 00 05  |................|
000001c0  00 0c 00 0d 00 00 00 09  00 0e 00 0f 00 01 00 09  |................|
000001d0  00 00 00 37 00 02 00 01  00 00 00 09 b2 00 02 12  |...7............|
000001e0  03 b6 00 04 b1 00 00 00  02 00 0a 00 00 00 0a 00  |................|
000001f0  02 00 00 00 03 00 08 00  04 00 0b 00 00 00 0c 00  |................|
00000200  01 00 00 00 09 00 10 00  11 00 00 00 01 00 12 00  |................|
00000210  00 00 02 00 13                                    |.....|
00000215
````

上面二进制大致分为几类：
文件魔数，主版本号，次版本号，常量池，类名，父类名，接口定义，类字段，方法定义，其他属性(如源文件等)
每类大致占用以下长度，u1表示1个字节，u2表示两个字节，u4表示4个字节
<table>
<tr><td>类型</td><td>名称</td><td>数量</td></tr>
<tr><td>u4</td><td>magic</td><td>1</td></tr>
<tr><td>u2</td><td>minor_version</td><td>1</td></tr>
<tr><td>u2</td><td>major_version</td><td>1</td></tr>
<tr><td>u2</td><td>constant_pool_count</td><td>1</td></tr>
<tr><td>cp_info</td><td>constant_pool</td><td>constant_pool_count-1</td></tr>
<tr><td>u2</td><td>access_flag</td><td>1</td></tr>
<tr><td>u2</td><td>this_class</td><td>1</td></tr>
<tr><td>u2</td><td>super_class</td><td>1</td></tr>
<tr><td>u2</td><td>interface_count</td><td>1</td></tr>
<tr><td>interface_info</td><td>interfaces</td><td>interface_count</td></tr>
<tr><td>u2</td><td>field_count</td><td>1</td></tr>
<tr><td>field_info</td><td>fields</td><td>field_count</td></tr>
<tr><td>u2</td><td>method_count</td><td>1</td></tr>
<tr><td>method_info</td><td>methods</td><td>method_count</td></tr>
<tr><td>u2</td><td>attribute_count</td><td>1</td></tr>
<tr><td>attribute_info</td><td>attributes</td><td>attribute_count</td></tr>
</table>

###解析二进制流
####基本信息
ca fe ba be 为文件魔数，所有class文件都是以这4个字节开始，又名"咖啡宝贝"
00 00 为次版本号
00 34 为主版本号，转换成10进制为52，jdk1.0开始为45，以后每个大版本都会往上+1，到1.8为52，高版本jdk能执行低版本的class，低版本不能执行高版本的class。
####常量池
00 22 表示常量池constant_pool有33个常量，常量池第0个元素没有被使用，第0项常量具有特殊意义，如果某些指向常量池索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况可以将索引值置为0来表示
常量信息结构体
````
struct cp_info{
	u1 tag;	# 常量类型枚举值		
	u1 info[];  # 枚举值后的常量索引
}
````
常量tag说明
<table>
<tr><td>常量类型</td><td>值</td><td>描述</td></tr>
<tr><td>CONSTANT\_UTF8</td><td>1 </td><td>u2字符串长度，接下N字节字符串长度</td></tr>
<tr><td>CONSTANT\_Integer</td><td>3 </td><td>4字节带符号位整数</td></tr>
<tr><td>CONSTANT\_Float</td><td>4 </td><td>4字节浮点数</td></tr>
<tr><td>CONSTANT\_Long</td><td>5 </td><td>8字节带符号位整数</td></tr>
<tr><td>CONSTANT\_Double</td><td>6 </td><td>8字节浮点数</td></tr>
<tr><td>CONSTANT\_Class</td><td>7 </td><td>u2指向UTF8常量,tag=1</td></tr>
<tr><td>CONSTANT\_String</td><td>8 </td><td>u2指向UTF8常量,tag=1</td></tr>
<tr><td>CONSTANT\_Fieldref</td><td>9 </td><td>u2 Class index,,tag=7<br> u2 NameAndType index,tag=12</td></tr>
<tr><td>CONSTANT\_Methodref</td><td>10</td><td>u2 Class index,tag=7<br> u2 NameAndType index,tag=12</td></tr>
<tr><td>CONSTANT\_Interfaceref</td><td>11</td><td>u2 Class index,tag=7<br> u2 NameAndType index,tag=12</td></tr>
<tr><td>CONSTANT\_NameAndType</td><td>12</td><td>u2 指向UTF8常量(名字)<br> u2 UTF8常量(类型) </td></tr>
<tr><td>CONSTANT\_MethodHandle/td><td>15</td><<td>待补充</td></tr>
<tr><td>CONSTANT\_MethodType</td><td>16</td><td>待补充</td></tr>
<tr><td>CONSTANT\_InvokeDynamic</td><td>18</td><td>待补充</td></tr>
</table>

#####常量tag定义
````
#CONSTANT_UTF8
struct CONSTANT_UTF8{
	u1 tag = 1; #tag标识
	u2 length;  #字符串字节长度
	u1 byte[length]; #字符串字节内容
}
#CONSTANT_Integer
struct CONSTANT_Integer{
	u1 tag = 3; #tag标识
	u4 int_value;  #整数值
}
#CONSTANT_Float
struct CONSTANT_Float{
	u1 tag = 4; #tag标识
	u4 float_value;  #值
}
#CONSTANT_Long
struct CONSTANT_Long{
	u1 tag = 5; #tag标识
	u8 long_value;  #值
}
#CONSTANT_Double
struct CONSTANT_Double{
	u1 tag = 6; #tag标识
	u8 double_value;  #值
}
#CONSTANT_Class
struct CONSTANT_Class{
	u1 tag = 7; #tag标识
	u2 name_index;  #UTF8常量索引
}
#CONSTANT_String
struct CONSTANT_String{
	u1 tag = 8; #tag标识
	u2 name_index;  #UTF8常量索引
}
#CONSTANT_Fieldref
struct CONSTANT_Fieldref{
	u1 tag = 9; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
#CONSTANT_Methodref
struct CONSTANT_Methodref{
	u1 tag = 10; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
#CONSTANT_Interfaceref
struct CONSTANT_Interfaceref{
	u1 tag = 11; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
#CONSTANT_NameAndType
struct CONSTANT_NameAndType{
	u1 tag = 12; #tag标识
	u2 name_index;  #UTF8常量索引
	u2 type_index;  #UTF8常量索引
}
````
