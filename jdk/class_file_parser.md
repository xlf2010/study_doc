##class 文件解析
参考oracle java虚拟机规范文档 https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf
1. 
文件魔数，主版本号，次版本号，常量池，类名，父类名，接口定义，类字段，方法定义，其他属性(如源文件等)
每类大致占用以下长度，u1表示1个字节，u2表示两个字节，u4表示4个字节
<table><tr><td>类型</td><td>名称</td><td>数量</td></tr><tr><td>u4</td><td>magic</td><td>1</td></tr><tr><td>u2</td><td>minor_version</td><td>1</td></tr><tr><td>u2</td><td>major_version</td><td>1</td></tr><tr><td>u2</td><td>constant_pool_count</td><td>1</td></tr><tr><td>cp_info</td><td>constant_pool</td><td>constant_pool_count-1</td></tr><tr><td>u2</td><td>access_flag</td><td>1</td></tr><tr><td>u2</td><td>this_class</td><td>1</td></tr><tr><td>u2</td><td>super_class</td><td>1</td></tr><tr><td>u2</td><td>interfaces_count</td><td>1</td></tr><tr><td>interface_info</td><td>interfaces</td><td>interfaces_count</td></tr><tr><td>u2</td><td>field_count</td><td>1</td></tr><tr><td>field_info</td><td>fields</td><td>field_count</td></tr><tr><td>u2</td><td>method_count</td><td>1</td></tr><tr><td>method_info</td><td>methods</td><td>method_count</td></tr><tr><td>u2</td><td>attribute_count</td><td>1</td></tr><tr><td>attribute_info</td><td>attributes</td><td>attribute_count</td></tr></table>

###解析二进制流
####基本信息
ca fe ba be 为文件魔数，所有class文件都是以这4个字节开始
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
<table><tr><td>常量类型</td><td>值</td><td>描述</td></tr><tr><td>JVM_CONSTANT_Class</td><td>7</td><td>u2指向UTF8常量,tag=1</td></tr><tr><td>JVM_CONSTANT_Fieldref</td><td>9</td><td>u2 Class index,,tag=7</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_Methodref</td><td>10</td><td>u2 Class index,tag=7</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_InterfaceMethodref</td><td>11</td><td>u2 Class index,tag=7</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_String</td><td>8</td><td>u2指向UTF8常量,tag=1</td></tr><tr><td>JVM_CONSTANT_MethodType</td><td>16</td><td>u2 signature_index指向UTF8常量,tag=1</td></tr><tr><td>JVM_CONSTANT_MethodHandle</td><td>15</td><td>u1 ref_kind(0~9)</br> u2 Methodref tag=10</td></tr><tr><td>JVM_CONSTANT_InvokeDynamic</td><td>18</td><td>u2 bootstrap_specifier_index</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_Integer</td><td>3</td><td>4字节带符号位整数</td></tr><tr><td>JVM_CONSTANT_Float</td><td>4</td><td>4字节浮点数</td></tr><tr><td>JVM_CONSTANT_Long</td><td>5</td><td>8字节带符号位整数</td></tr><tr><td>JVM_CONSTANT_Double</td><td>6</td><td>8字节浮点数</td></tr><tr><td>JVM_CONSTANT_NameAndType</td><td>12</td><td>u2 指向UTF8常量(名字)</br>u2 UTF8常量(类型)</td></tr><tr><td>JVM_CONSTANT_Utf8</td><td>1</td><td>u2字符串长度N，接下N字节字符串长度</td></tr></table>

#####常量tag定义
````
JVM_CONSTANT_Class{
	u1 tag = 7; #tag标识
	u2 name_index;  #UTF8常量索引
}
JVM_CONSTANT_Fieldref{
	u1 tag = 9; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
JVM_CONSTANT_Methodref{
	u1 tag = 10; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
JVM_CONSTANT_InterfaceMethodref{
	u1 tag = 11; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
JVM_CONSTANT_String{
	u1 tag = 8; #tag标识
	u2 name_index;  #UTF8常量索引
}
JVM_CONSTANT_MethodType{
	u1 tag = 16; #tag标识
	u2 name_index;  #UTF8常量索引
}
JVM_CONSTANT_MethodHandle{
	u1 tag = 15; #tag标识
	u1 reference_kind; #必须在[1..9],1..9定义如下备注 
	u2 reference_index;  # case reference_kind when [1..4] then CONSTANT_Fieldref_info when [5..8] then CONSTANT_Methodref_info when 9 then CONSTANT_InterfaceMethodref_info
}
#注：reference_kind 对应的指令,参考oracle文档 (§4.4.8)
#1	REF_getField	 
#2	REF_getStatic
#3	REF_putField
#4	REF_putStatic
#5	REF_invokeVirtual
#6	REF_invokeStatic
#7	REF_invokeSpecial
#8	REF_newInvokeSpecial
#9	REF_invokeInterface

JVM_CONSTANT_InvokeDynamic{
	u1 tag = 11; #tag标识
	u2 bootstrap_specifier_index;  # bootstrap_specifier_index
	u2 name_and_type_index;  #name_and_type索引
}
JVM_CONSTANT_Integer{
	u1 tag = 3; #tag标识
	u4 int_value;  #整数值
}
JVM_CONSTANT_Float{
	u1 tag = 4; #tag标识
	u4 float_value;  #值
}
JVM_CONSTANT_Long{
	u1 tag = 5; #tag标识
	u8 long_value;  #值
}
JVM_CONSTANT_Double{
	u1 tag = 6; #tag标识
	u8 double_value;  #值
}
JVM_CONSTANT_NameAndType{
	u1 tag = 12; #tag标识
	u2 name_index;  #UTF8常量索引
	u2 type_index;  #UTF8常量索引
}
JVM_CONSTANT_UTF8{
	u1 tag = 1; #tag标识
	u2 length;  #字符串字节长度
	u1 byte[length]; #字符串字节内容
}
````

注：
1.参考openjdk源代码
hotspot/src/share/vm/classfile/classFileParser.cpp
line:93 ClassFileParser::parse_constant_pool_entries


