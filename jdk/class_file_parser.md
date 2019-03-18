##class 文件解析
参考oracle java虚拟机规范文档 https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf
表格转换地址：http://pressbin.com/tools/excel_to_html_table/index.html
1. 文件魔数，主版本号，次版本号，常量池，类名，父类名，接口定义，类字段，方法定义，其他属性(如源文件等)<br>每类大致占用以下长度，u1表示1个字节，u2表示两个字节，u4表示4个字节
<table><tr><td>类型</td><td>名称</td><td>数量</td></tr><tr><td>u4</td><td>magic</td><td>1</td></tr><tr><td>u2</td><td>minor_version</td><td>1</td></tr><tr><td>u2</td><td>major_version</td><td>1</td></tr><tr><td>u2</td><td>constant_pool_count</td><td>1</td></tr><tr><td>cp_info</td><td>constant_pool</td><td>constant_pool_count-1</td></tr><tr><td>u2</td><td>access_flag</td><td>1</td></tr><tr><td>u2</td><td>this_class</td><td>1</td></tr><tr><td>u2</td><td>super_class</td><td>1</td></tr><tr><td>u2</td><td>interfaces_count</td><td>1</td></tr><tr><td>interface_info</td><td>interfaces</td><td>interfaces_count</td></tr><tr><td>u2</td><td>field_count</td><td>1</td></tr><tr><td>field_info</td><td>fields</td><td>field_count</td></tr><tr><td>u2</td><td>method_count</td><td>1</td></tr><tr><td>method_info</td><td>methods</td><td>method_count</td></tr><tr><td>u2</td><td>attribute_count</td><td>1</td></tr><tr><td>attribute_info</td><td>attributes</td><td>attribute_count</td></tr></table>

---
###常量池
常量池用一个字节标识类型，然后跟着字节数组标识常量信息。
````
cp_info{
	u1 tag;	# 常量类型枚举值		
	u1 info[];  # 枚举值后的常量索引
}
````
常量tag枚举值说明
<table><tr><td>常量类型</td><td>值</td><td>描述</td></tr><tr><td>JVM_CONSTANT_Class</td><td>7</td><td>u2指向UTF8常量,tag=1</td></tr><tr><td>JVM_CONSTANT_Fieldref</td><td>9</td><td>u2 Class index,tag=7</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_Methodref</td><td>10</td><td>u2 Class index,tag=7</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_InterfaceMethodref</td><td>11</td><td>u2 Class index,tag=7</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_String</td><td>8</td><td>u2指向UTF8常量,tag=1</td></tr><tr><td>JVM_CONSTANT_MethodType</td><td>16</td><td>u2 signature_index指向UTF8常量,tag=1</td></tr><tr><td>JVM_CONSTANT_MethodHandle</td><td>15</td><td>u1 ref_kind(0~9)</br> u2 Methodref tag=10</td></tr><tr><td>JVM_CONSTANT_InvokeDynamic</td><td>18</td><td>u2 bootstrap_specifier_index</br>u2 NameAndType index,tag=12</td></tr><tr><td>JVM_CONSTANT_Integer</td><td>3</td><td>4字节带符号位整数</td></tr><tr><td>JVM_CONSTANT_Float</td><td>4</td><td>4字节浮点数</td></tr><tr><td>JVM_CONSTANT_Long</td><td>5</td><td>8字节带符号位整数</td></tr><tr><td>JVM_CONSTANT_Double</td><td>6</td><td>8字节浮点数</td></tr><tr><td>JVM_CONSTANT_NameAndType</td><td>12</td><td>u2 指向UTF8常量(名字)</br>u2 UTF8常量(类型)</td></tr><tr><td>JVM_CONSTANT_Utf8</td><td>1</td><td>u2字符串长度N，接下N字节字符串长度</td></tr></table>

####常量tag定义
````
CONSTANT_Class_info {
	u1 tag = 7; #tag标识
	u2 name_index;  #UTF8常量索引
}
CONSTANT_Fieldref_info {
	u1 tag = 9; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
CONSTANT_Methodref_info {
	u1 tag = 10; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
CONSTANT_InterfaceMethodref_info {
	u1 tag = 11; #tag标识
	u2 class_index;  #class索引
	u2 name_and_type_index;  #name_and_type索引
}
CONSTANT_String_info {
	u1 tag = 8; #tag标识
	u2 name_index;  #UTF8常量索引
}
CONSTANT_MethodType_info{
	u1 tag = 16; #tag标识
	u2 name_index;  #UTF8常量索引
}
CONSTANT_MethodHandle_info{
	u1 tag = 15; #tag标识
	u1 reference_kind; #必须在[1..9],1..9定义如下备注 
	u2 reference_index;  # case reference_kind when [1..4] then CONSTANT_Fieldref_info when [5..8] then CONSTANT_Methodref_info when 9 then CONSTANT_InterfaceMethodref_info
}
/**
* 注：reference_kind 对应的指令,参考oracle文档 (§4.4.8)
* #1	REF_getField	 
* #2	REF_getStatic
* #3	REF_putField
* #4	REF_putStatic
* #5	REF_invokeVirtual
* #6	REF_invokeStatic
* #7	REF_invokeSpecial
* #8	REF_newInvokeSpecial
* #9	REF_invokeInterface
*/
CONSTANT_InvokeDynamic_info{
	u1 tag = 11; #tag标识
	u2 bootstrap_specifier_index;  # bootstrap_specifier_index
	u2 name_and_type_index;  #name_and_type索引
}
CONSTANT_Integer_info {
	u1 tag = 3; #tag标识
	u4 int_value;  #整数值
}
CONSTANT_Float_info {
	u1 tag = 4; #tag标识
	u4 float_value;  #值
}
CONSTANT_Long_info{
	u1 tag = 5; #tag标识
	u4 high_bytes; 
	u4 low_bytes;
}
CONSTANT_Double_info{
	u1 tag = 6; #tag标识
	u4 high_bytes;
	u4 low_bytes;
}
CONSTANT_NameAndType_info{
	u1 tag = 12; #tag标识
	u2 name_index;  #UTF8常量索引
	u2 type_index;  #UTF8常量索引
}
CONSTANT_Utf8_info{
	u1 tag = 1; #tag标识
	u2 length;  #字符串字节长度
	u1 byte[length]; #字符串字节内容
}
````

注：
1.参考openjdk源代码
hotspot/src/share/vm/classfile/classFileParser.cpp
line:93 ClassFileParser::parse_constant_pool_entries

---
###Class Access Flag
<table><tr><td>Flag Name</td><td>Value</td><td>Interpretation</td><td>说明</td></tr><tr><td>ACC_PUBLIC</td><td>0x0001</td><td>Declared public; may be accessed from outside its package.</td><td>public 修饰</td></tr><tr><td>ACC_FINAL</td><td>0x0010</td><td>Declared final; no subclasses allowed.</td><td>final 修饰，不允许有子类</td></tr><tr><td>ACC_SUPER</td><td>0x0020</td><td>Treat superclass methods specially when invoked by the invokespecial instruction.</td><td>允许使用invokespecial指令</td></tr><tr><td>ACC_INTERFACE</td><td>0x0200</td><td>Is an interface, not a class.</td><td>是否接口</td></tr><tr><td>ACC_ABSTRACT</td><td>0x0400</td><td>Declared abstract; must not be instantiated.</td><td>是否抽象类</td></tr><tr><td>ACC_SYNTHETIC</td><td>0x1000</td><td>Declared synthetic; not present in the source code.</td><td>由编译器产生的，不在源码</td></tr><tr><td>ACC_ANNOTATION</td><td>0x2000</td><td>Declared as an annotation type.</td><td>注解类型</td></tr><tr><td>ACC_ENUM</td><td>0x4000</td><td>Declared as an enum type.</td><td>枚举类型</td></tr></table>

---
###Interface Info
指向CONSTANT_Class_info 索引

---
###Field Info
````
field_info {
	 u2 access_flags;
	 u2 name_index;	#UTF8常量索引
	 u2 descriptor_index;	 #UTF8常量索引
	 u2 attributes_count;
	 attribute_info attributes[attributes_count];	#属性
}
````
####field_info.access_flags
<table><tr><td>Flag Name</td><td>Value </td><td>Interpretation</td><td>说明</td></tr><tr><td>ACC_PUBLIC</td><td>0x0001</td><td>Declared public; may be accessed from outside its package.</td><td>public 公共可见</td></tr><tr><td>ACC_PRIVATE</td><td>0x0002</td><td>Declared private; usable only within the defining class.</td><td>private 类可见</td></tr><tr><td>ACC_PROTECTED</td><td>0x0004 </td><td>Declared protected; may be accessed within subclasses.</td><td>protected 类与子类可见</td></tr><tr><td>ACC_STATIC</td><td>0x0008 </td><td>Declared static.</td><td>static</td></tr><tr><td>ACC_FINAL</td><td>0x0010 </td><td>Declared final; never directly assigned to after object construction (JLS §17.5).</td><td>final 修饰</td></tr><tr><td>ACC_VOLATILE</td><td>0x0040 </td><td>Declared volatile; cannot be cached.</td><td>volatile 易变的</td></tr><tr><td>ACC_TRANSIENT</td><td>0x0080</td><td>Declared transient; not written or read by a persistent object manager.</td><td>transient 非序列化</td></tr><tr><td>ACC_SYNTHETIC</td><td>0x1000 </td><td>Declared synthetic; not present in the source code.</td><td>synthetic 非代码编译</td></tr><tr><td>ACC_ENUM</td><td>0x4000 </td><td>Declared as an element of an enum</td><td>枚举值</td></tr></table>

注：
1. ACC_PUBLIC,ACC_PRIVATE,ACC_PROTECTED 这三个只能有其中一个或没有
2. ACC_FINAL,ACC_VOLATILE 这两个只能有其中一个或没有
3. Interface内的Field Info都是ACC_PUBLIC,ACC_STATIC,ACC_FINAL修饰
4. ACC_SYNTHETIC 表示这个Field Info是由编译器产生，不体现在源代码
5. ACC_ENUM 表示这是一个枚举值的引用

---
###Method Info

```
method_info {
	 u2 access_flags;
	 u2 name_index;	#UTF8常量索引
	 u2 descriptor_index;	 #UTF8常量索引
	 u2 attributes_count;
	 attribute_info attributes[attributes_count];	#属性
}
```
####method_info.access_flags
<table><tr><td>Flag Name </td><td>Value</td><td> Interpretation</td><td>说明</td></tr><tr><td>ACC_PUBLIC</td><td>0x0001</td><td>Declared public; may be accessed from outside its package.</td><td>public 公共可见</td></tr><tr><td>ACC_PRIVATE</td><td>0x0002</td><td>Declared private; accessible only within the defining class.</td><td>private 类可见</td></tr><tr><td>ACC_PROTECTED</td><td>0x0004</td><td>Declared protected; may be accessed within subclasses.</td><td>protected 类与子类可见</td></tr><tr><td>ACC_STATIC</td><td>0x0008</td><td>Declared static.</td><td>static</td></tr><tr><td>ACC_FINAL</td><td>0x0010</td><td>Declared final; must not be overridden (§5.4.5).</td><td>final 修饰，子类不能重写</td></tr><tr><td>ACC_SYNCHRONIZED</td><td>0x0020</td><td>Declared synchronized; invocation is wrapped by a monitor use.</td><td>同步方法</td></tr><tr><td>ACC_BRIDGE</td><td>0x0040</td><td>A bridge method, generated by the compiler.</td><td>桥接方法，由编译器产生(泛型调用)</td></tr><tr><td>ACC_VARARGS</td><td>0x0080</td><td>Declared with variable number of arguments.</td><td>不定参数个数方法(变长参数)</td></tr><tr><td>ACC_NATIVE</td><td>0x0100</td><td>Declared native; implemented in a language other than Java.</td><td>本地方法，其他语言实现</td></tr><tr><td>ACC_ABSTRACT</td><td>0x0400</td><td>Declared abstract; no implementation is provided.</td><td>抽象方法</td></tr><tr><td>ACC_STRICT</td><td>0x0800</td><td>Declared strictfp; floating-point mode is FPstrict.</td><td>方法是否为strictfp,严格按照IEEE754计算float或double</td></tr><tr><td>ACC_SYNTHETIC</td><td>0x1000</td><td>Declared synthetic; not present in the source code.</td><td>synthetic 非代码编译</td></tr></table>

注：
1. ACC_PUBLIC,ACC_PRIVATE,ACC_PROTECTED 这三个只能有其中一个或没有
2. Interface 方法不能是ACC_PROTECTED, ACC_FINAL, ACC_SYNCHRONIZED, ACC_NATIVE <br>在jdk版本小于52.0(1.8)之前，方法修饰符都必须有ACC_PUBLIC与ACC_ABSTRACT，在52.0(1.8)包括52.0之后，则可以是ACC_PUBLIC或者是ACC_PRIVATE**(private方法貌似没有在jdk1.8编译通过，在openjdk9编译过了)** <br>参考openjdk8代码
> hotspot/src/share/vm/classfile/classFileParser.cpp.verify_legal_method_modifiers(line 4671)

```cpp
 // Class file version is JAVA_8_VERSION or later Methods of
      // interfaces may set any of the flags except ACC_PROTECTED,
      // ACC_FINAL, ACC_NATIVE, and ACC_SYNCHRONIZED; they must
      // have exactly one of the ACC_PUBLIC or ACC_PRIVATE flags set.
      if ((is_public == is_private) || /* Only one of private and public should be true - XNOR */
          (is_native || is_protected || is_final || is_synchronized) ||
          // If a specific method of a class or interface has its
          // ACC_ABSTRACT flag set, it must not have any of its
          // ACC_FINAL, ACC_NATIVE, ACC_PRIVATE, ACC_STATIC,
          // ACC_STRICT, or ACC_SYNCHRONIZED flags set.  No need to
          // check for ACC_FINAL, ACC_NATIVE or ACC_SYNCHRONIZED as
          // those flags are illegal irrespective of ACC_ABSTRACT being set or not.
          (is_abstract && (is_private || is_static || is_strict))) {
        is_illegal = true;
      }
```
以上代码说明 
- ACC_PUBLIC与ACC_PRIVATE只能有其中一个为true,**说明可以是ACC_PRIVATE**
- 不能是ACC_NATIVE,ACC_PROTECTED,ACC_FINAL,ACC_SYNCHRONIZED
- ACC_ABSTRACT情况下不能是ACC_PRIVATE,ACC_STATIC,ACC_STRICT


--- 
###Attribute Info
Attribute 用于 ClassFile , field_info , method_info , and Code_attribute
```
attribute_info {
	u2 attribute_name_index; 	#UTF8常量索引
	u4 attribute_length;		#属性长度
	u1 info[attribute_length];
}
```
attribute_name名称可以分为三类：
- jvm能正确解析class文件5个属性：
ConstantValue
Code
StackMapTable
Exceptions
BootstrapMethods
- java SE类库能正确解析class文件的12个属性
InnerClasses
EnclosingMethod
Synthetic
Signature
RuntimeVisibleAnnotations
RuntimeInvisibleAnnotations
RuntimeVisibleParameterAnnotations
RuntimeInvisibleParameterAnnotations
RuntimeVisibleTypeAnnotations
RuntimeInvisibleTypeAnnotations
AnnotationDefault
MethodParameters
- 不是jvm或java SE类库必须的属性，但是比较有用的6个属性
SourceFile
SourceDebugExtension
LineNumberTable
LocalVariableTable
LocalVariableTypeTable
Deprecated

<table><tr><td>Attribute</td><td>class file</td><td>Java SE</td><td>Location</td></tr><tr><td>ConstantValue</td><td>45.3</td><td>1.0.2</td><td>field_info</td></tr><tr><td>Code</td><td>45.3</td><td>1.0.2</td><td>method_info</td></tr><tr><td>StackMapTable</td><td>50</td><td>6</td><td>Code</td></tr><tr><td>Exceptions</td><td>45.3</td><td>1.0.2</td><td>method_info</td></tr><tr><td>InnerClasses</td><td>45.3</td><td>1.1</td><td>ClassFile</td></tr><tr><td>EnclosingMethod</td><td>49</td><td>5</td><td>ClassFile</td></tr><tr><td>Synthetic</td><td>45.3</td><td>1.1</td><td>ClassFile,field_info,method_info</td></tr><tr><td>Signature</td><td>49</td><td>5</td><td>ClassFile,field_info,method_info</td></tr><tr><td>SourceFile</td><td>45.3</td><td>1.0.2</td><td>ClassFile</td></tr><tr><td>SourceDebugExtension</td><td>49</td><td>5</td><td>ClassFile</td></tr><tr><td>LineNumberTable</td><td>45.3</td><td>1.0.2</td><td>Code</td></tr><tr><td>LocalVariableTable</td><td>45.3</td><td>1.0.2</td><td>Code</td></tr><tr><td>LocalVariableTypeTable</td><td>49</td><td>5</td><td>Code</td></tr><tr><td>Deprecated</td><td>45.3</td><td>1.1</td><td>ClassFile,field_info,method_info</td></tr><tr><td>RuntimeVisibleAnnotations</td><td>49</td><td>5</td><td>ClassFile,field_info,method_info</td></tr><tr><td>RuntimeInvisibleAnnotations</td><td>49</td><td>5</td><td>ClassFile,field_info,method_info</td></tr><tr><td>RuntimeVisibleParameterAnnotations</td><td>49</td><td>5</td><td>method_info</td></tr><tr><td>RuntimeInvisibleParameterAnnotations</td><td>49</td><td>5</td><td>method_info</td></tr><tr><td>RuntimeVisibleTypeAnnotations</td><td>52</td><td>8</td><td>ClassFile,field_info,method_info,Code</td></tr><tr><td>RuntimeInvisibleTypeAnnotations</td><td>52</td><td>8</td><td>ClassFile,field_info,method_info,Code</td></tr><tr><td>AnnotationDefault</td><td>49</td><td>5</td><td>method_info</td></tr><tr><td>BootstrapMethods</td><td>51</td><td>7</td><td>ClassFile</td></tr><tr><td>MethodParameters</td><td>52</td><td>8</td><td>method_info</td></tr></table>

####1.ConstantValue
ConstantValue位于field_info中，表示常量(final修饰)，一个field_info最多只能有一个ConstantValue，如果是static表示是接口或类常量
````
ConstantValue_attribute {
	 u2 attribute_name_index;	#常量池UTF8索引，名字为ConstantValue
	 u4 attribute_length;	#属性长度，值为2,不包括attribute_name_index+attribute_length 6字节长度
	 u2 constantvalue_index;	#属性指向UTF8常量值
}
````
constantvalue_index 值如下，只有基础类型与String才能是ConstantValue
<table><tr><td>Field Type</td><td> Entry Type</td></tr><tr><td>long </td><td>CONSTANT_Long</td></tr><tr><td>float </td><td>CONSTANT_Float</td></tr><tr><td>double </td><td>CONSTANT_Double</td></tr><tr><td>int, short, char, byte, boolean </td><td>CONSTANT_Integer</td></tr><tr><td>String </td><td>CONSTANT_String</td></tr></table>

####2.Code
Code是method_info的属性，是java字节码执行指令，如果是native或abstract，则没有这个属性，否则必须要有Code属性
Code属性结构如下：
````
Code_attribute {
	 u2 attribute_name_index; 	#常量池UTF8索引，名字为Code
	 u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	 u2 max_stack;				#方法栈最大深度
	 u2 max_locals;				#最大局部变量数，long或double占用两个local索引，最大索引变量为long或double时，最大索引值为max_locals-2,其他类型为为max_locals-1
	 u4 code_length;			#指令长度需在[1..65535]之间
	 u1 code[code_length];		#jvm执行的指令集合
	 u2 exception_table_length;	#异常个数
	 { 
		 u2 start_pc;			#异常起始，包括当前值
		 u2 end_pc;				#异常结束，不包括当前值,end_pc>start_pc
		 u2 handler_pc;			#出现异常时处理指令地址
		 u2 catch_type;			#异常类型，指向CONSTANT_Class_info常量池,如果值为0则表示无条件执行，可用于编译finally语句
	 } exception_table[exception_table_length];
	 u2 attributes_count;		#属性长度
	 attribute_info attributes[attributes_count];	#属性，属性类型StackMapTable,LineNumberTable,LocalVariableTable,LocalVariableTypeTable
}
````
举个栗子：
````java
public class TestException{
	public void test(){
		try{
			int i=1;
		}catch(RuntimeException e){
			int j=1;
		}catch(Exception e){
			int k=1;
		}
	}
}
//编译之后 javap -v -p TestException查看字节码信息
 public void test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=3, args_size=1
         0: iconst_1
         1: istore_1
         2: goto          14
         5: astore_1
         6: iconst_1
         7: istore_2
         8: goto          14
        11: astore_1
        12: iconst_1
        13: istore_2
        14: return
      Exception table:
         from    to  target type
             0     2     5   Class java/lang/RuntimeException	//在[0,2)指令，也就是执行iconst_1,istore_1时出现RuntimeException则跳转到指令5(astore_1)开始执行
             0     2    11   Class java/lang/Exception		   //在[0,2)指令，也就是执行iconst_1,istore_1时出现Exception则跳转到指令11(astore_1)开始执行

````
####3.StackMapTable
StackMapTable是位于Code属性的一个属性值，在虚拟机类加载的类型阶段被使用。结构如下：
````
StackMapTable_attribute {
    u2              attribute_name_index;	#常量池UTF8索引，名字为StackMapTable
    u4              attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
    u2              number_of_entries;		#entries表中的成员stack_map_frame数量
    stack_map_frame entries[number_of_entries];	
}

// stack_map_frame定义
union stack_map_frame {
    same_frame;				#
    same_locals_1_stack_item_frame;
    same_locals_1_stack_item_frame_extended;
    chop_frame;
    same_frame_extended;
    append_frame;
    full_frame;
}

same_frame {
      u1 frame_type = SAME; /* 0-63 */
}

same_locals_1_stack_item_frame
 {
      u1 frame_type = SAME_LOCALS_1_STACK_ITEM;/* 64-127 */
      verification_type_info stack[1];
}

same_locals_1_stack_item_frame_extended {
	u1 frame_type = SAME_LOCALS_1_STACK_ITEM_EXTENDED;/* 247 */
	u2 offset_delta;
	verification_type_info stack[1];
}

chop_frame {
      u1 frame_type = CHOP; /* 248-250 */
      u2 offset_delta;
}

same_frame_extended {
      u1 frame_type = SAME_FRAME_EXTENDED; /* 251 */
      u2 offset_delta;
}

append_frame {
      u1 frame_type = APPEND; /* 252-254 */
      u2 offset_delta;
      verification_type_info locals[frame_type - 251];
}

full_frame {
    u1 frame_type = FULL_FRAME; /* 255 */
    u2 offset_delta;
    u2 number_of_locals;
    verification_type_info locals[number_of_locals];
    u2 number_of_stack_items;
    verification_type_info stack[number_of_stack_items];
}
````

````
union verification_type_info {    
      Top_variable_info;
      Integer_variable_info;
      Float_variable_info;
      Long_variable_info;
      Double_variable_info; 
      Null_variable_info;
      UninitializedThis_variable_info;
      Object_variable_info;
      Uninitialized_variable_info;
}

Top_variable_info {
	u1 tag = ITEM_Top; /* 0 */
}

Integer_variable_info {
	u1 tag = ITEM_Integer; /* 1 */
}

Float_variable_info {
	u1 tag = ITEM_Float; /* 2 */
}

Long_variable_info {
	u1 tag = ITEM_Long; /* 4 */
}

Double_variable_info {
	u1 tag = ITEM_Double; /* 3 */ 
}

Null_variable_info {
	u1 tag = ITEM_Null; /* 5 */
}

UninitializedThis_variable_info {
	u1 tag = ITEM_UninitializedThis; /* 6 */
}

Object_variable_info {
	u1 tag = ITEM_Object; /* 7 */
	u2 cpool_index;
}

Uninitialized_variable_info {
	u1 tag = ITEM_Uninitialized /* 8 */
	u2 offset;
}
````

####4.Exceptions
抛出异常至少是以下条件中的一个
要抛出的是RuntimeException或其子类的实例。
要抛出的是Error或其子类的实例。
要抛出的是在exception_index_table[]数组中申明的异常类或其子类的实例。
Exception结构如下：
````
Exceptions_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为Exceptions
	u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u2 number_of_exceptions;	#异常个数
	u2 exception_index_table[number_of_exceptions];	#指向CONSTANT_Class_info类型索引
}
````

####5.InnerClasses
````
InnerClasses_attribute {  
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为InnerClasses
	u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u2 number_of_classes;		#class成员个数
	{ 
		u2 inner_class_info_index;	#内部类索引，指向CONSTANT_Class_info
		u2 outer_class_info_index;	#宿主类索引，指向CONSTANT_Class_info，顶层类(与class并列)，局部类(方法内部)，匿名类则为0
		u2 inner_name_index;		#类名，指向CONSTANT_Utf8_info常量，匿名类为0
		u2 inner_class_access_flags;	#访问标志
	} classes[number_of_classes];
}
````
修饰符
<table><tr><td>Flag Name </td><td>Value </td><td>Interpretation</td><td>描述</td></tr><tr><td>ACC_PUBLIC </td><td>0x0001 </td><td>Marked or implicitly public in source.</td><td>public修饰</td></tr><tr><td>ACC_PRIVATE </td><td>0x0002 </td><td>Marked private in source.</td><td>private修饰</td></tr><tr><td>ACC_PROTECTED </td><td>0x0004 </td><td>Marked protected in source.</td><td>protected修饰</td></tr><tr><td>ACC_STATIC </td><td>0x0008 </td><td>Marked or implicitly static in source.</td><td>static修饰</td></tr><tr><td>ACC_FINAL </td><td>0x0010 </td><td>Marked final in source.</td><td>final修饰</td></tr><tr><td>ACC_INTERFACE</td><td>0x0200 </td><td>Was an interface in source.</td><td>接口</td></tr><tr><td>ACC_ABSTRACT</td><td>0x0400 </td><td>Marked or implicitly abstract in source.</td><td>抽象类</td></tr><tr><td>ACC_SYNTHETIC </td><td>0x1000 </td><td>Declared synthetic; not present in the source code.</td><td>非源码生成</td></tr><tr><td>ACC_ANNOTATION </td><td>0x2000 </td><td>Declared as an annotation type.</td><td>注解</td></tr><tr><td>ACC_ENUM </td><td>0x4000 </td><td>Declared as an enum type.</td><td>枚举</td></tr></table>

####6.EnclosingMethod
当且仅当Class为局部类或匿名类时才有EnclosingMethod属性
````
EnclosingMethod_attribute {
      u2 attribute_name_index;	#CONSTANT_Utf8_info,值为EnclosingMethod
      u4 attribute_length;		#固定值为4
      u2 class_index			#指向CONSTANT_Class_info
      u2 method_index;			#CONSTANT_NameAndType_info，如果不是方法内部则为0
}
````

####7.Synthetic
位于ClassFile,field_info, 或者 method_info 表示不在源码出现，例外情况：由编译器产生的实例化方法，类初始方法，Enum.values()，Enum.valueOf()等不需要Synthetic标志
````
Synthetic_attribute {
      u2 attribute_name_index;	#CONSTANT_Utf8_info,值为Synthetic
      u4 attribute_length;		#固定值0
}
````

####8.Signature
泛型签名信息，位于ClassFile,field_info, 或者 method_info，如果一个类，接口，构造方法，字段包含泛型变量，则保存泛型信息，结构如下：
````
Signature_attribute {
	 u2 attribute_name_index;	#CONSTANT_Utf8_info,值为Signature
	 u4 attribute_length;	#固定值2
	 u2 signature_index;	#CONSTANT_Utf8_info
}

````

####9.SourceFile
可选定长字段，位于ClassFile，记录源文件名，结构如下：
````
SourceFile_attribute {
	 u2 attribute_name_index;	#CONSTANT_Utf8_info,值为SourceFile
	 u4 attribute_length;		#固定值2
	 u2 sourcefile_index;		#CONSTANT_Utf8_info
}
````

####10.SourceDebugExtension 
可选属性,保存扩展调试信息
````
SourceDebugExtension_attribute {
	 u2 attribute_name_index;	#CONSTANT_Utf8_info,值为SourceDebugExtension
	 u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	 u1 debug_extension[attribute_length];	#用于保存调试信息，指向CONSTANT_Utf8_info
}
````

####11.LineNumberTable
可选变长属性，位于Code属性表，对应code数组在源文件中的位置
````
LineNumberTable_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为LineNumberTable
	u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u2 line_number_table_length;	#line_number_table个数
	{ 
		u2 start_pc;				#code 数组的索引，表示源文件中新的行的起点,需小于code长度
		u2 line_number;				#源文件行数
	} line_number_table[line_number_table_length];
}
````

####12.LocalVariableTable
可选变长属性，位于Code属性表中，用于调试保存局部变量信息,每个code属性最多只能有一个LocalVariableTable属性
````
LocalVariableTable_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为LocalVariableTable
	u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u2 local_variable_table_length;		#局部变量个数
	{ 
		u2 start_pc;			#code 数组的索引起始值
		u2 length;				#局部变量在code范围长度，为 [start_pc+length)
		u2 name_index;			#局部变量名，CONSTANT_Utf8_info
		u2 descriptor_index;	#局部变量类型描述符，CONSTANT_Utf8_info
		u2 index;				#局部变量在栈帧中的索引，如果是long或double则占用index和index+1两个索引
	} local_variable_table[local_variable_table_length];
}
````

####13.LocalVariableTypeTable
用于给调试器确定方法在执行中局部变量的信息，在Code属性的属性表中，与**LocalVariableTable**不同，这里提供方法签名信息，这仅在泛型参数情况下有效，泛型类型会同时出现在LocalVariableTable，LocalVariableTypeTable中，其他仅存在LocalVariableTable中
```
LocalVariableTypeTable_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为LocalVariableTable
	u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u2 local_variable_type_table_length;	#局部变量个数
	{ 
		u2 start_pc;		#code 数组的索引起始值
		u2 length;			#局部变量在code范围长度，为 [start_pc+length)
		u2 name_index;		#局部变量名，CONSTANT_Utf8_info
		u2 signature_index;	#局部变量类型的字段签名，CONSTANT_Utf8_info
		u2 index;			#局部变量在栈帧中的索引，如果是long或double则占用index和index+1两个索引
	} local_variable_type_table[local_variable_type_table_length];
}

```

####14.Deprecated
Deprecated属性是可选定长属性,位于ClassFile, field_info, 或 method_info中，表示过期属性，将来会被其他替换
结构如下：
````
Deprecated_attribute {
	 u2 attribute_name_index;	#CONSTANT_Utf8_info,值为Deprecated
	 u4 attribute_length;		#固定值0
}
````

####15.RuntimeVisibleAnnotations
变长属性位于ClassFile, field_info, method_info，表示运行时可见注解，用于反射API获取。
结构如下：
````
RuntimeVisibleAnnotations_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为RuntimeVisibleAnnotations
	u4 attribute_length;	#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u2 num_annotations;		#运行时可见注解的数量
	annotation annotations[num_annotations];	#表示每个注解
}

annotation {    
	u2 type_index;    	#CONSTANT_Utf8_info,表示注解类型
	u2 num_element_value_pairs;    #key-value个数
	{   
		u2 element_name_index;     #key对应的CONSTANT_Utf8_info索引
		element_value value;    #value
	} element_value_pairs[num_element_value_pairs];
}

element_value {
	u1 tag;
	union {
		u2 const_value_index;		#表示基础类型常量，或String字面常量
		{
			u2 type_name_index;		#CONSTANT_Utf8_info，表示枚举常量类型
			u2 const_name_index;	#CONSTANT_Utf8_info,表示枚举常量类型简单名称
		} enum_const_value;
		u2 class_info_index;		#CONSTANT_Utf8_info,表示类名称
		annotation annotation_value;	#annotation类型
		{
			u2 num_values;			#数组元素个数
			element_value values[num_values];	#数组元素值
		} array_value;				#数组类型
	} value;
}
注：
class_info_index表示如下：
1. 如果是类名称，接口或数组，则为类型名称，如Object.class表示为Ljava/lang/Object
2. 如果是基础类型，则为对应的缩写，如int.class表示为I
3. 如果是void，则为V
````
element_value.tag枚举如下：
<table><tr><td>tag值</td><td>类型</td><td>union联合体值</td><td>常量类型</td></tr><tr><td>B </td><td>byte </td><td>const_value_index </td><td>CONSTANT_Integer</td></tr><tr><td>C </td><td>char </td><td>const_value_index </td><td>CONSTANT_Integer</td></tr><tr><td>D </td><td>double </td><td>const_value_index </td><td>CONSTANT_Double</td></tr><tr><td>F </td><td>float </td><td>const_value_index </td><td>CONSTANT_Float</td></tr><tr><td>I </td><td>int </td><td>const_value_index </td><td>CONSTANT_Integer</td></tr><tr><td>J </td><td>long </td><td>const_value_index </td><td>CONSTANT_Long</td></tr><tr><td>S </td><td>short </td><td>const_value_index </td><td>CONSTANT_Integer</td></tr><tr><td>Z </td><td>boolean </td><td>const_value_index </td><td>CONSTANT_Integer</td></tr><tr><td>s </td><td>String </td><td>const_value_index </td><td>CONSTANT_Utf8</td></tr><tr><td>e </td><td>Enum type </td><td>enum_const_value </td><td>Not applicable</td></tr><tr><td>c </td><td>Class </td><td>class_info_index </td><td>Not applicable</td></tr><tr><td>@ </td><td>Annotation type </td><td>annotation_value </td><td>Not applicable</td></tr><tr><td>[ </td><td>Array type </td><td>array_value </td><td>Not applicable</td></tr></table>

####16.RuntimeInvisibleAnnotations
RuntimeInvisibleAnnotations跟RuntimeVisibleAnnotations类似，RuntimeInvisibleAnnotations表示在运行时不可见注解，不能通过反射API获取，除非通过某些特殊方式(如命令行参数)获取，JVM虚拟机会忽略这些参数，结构参考RuntimeVisibleAnnotations

####17.RuntimeVisibleParameterAnnotations
RuntimeVisibleParameterAnnotations表示在运行时可见参数注解，能通过反射API获取，是method_info的一个属性，method_info最多只有一个RuntimeVisibleParameterAnnotations。
结构如下:
````
RuntimeVisibleParameterAnnotations_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为RuntimeVisibleParameterAnnotations
	u4 attribute_length;	#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u1 num_parameters;    #方法参数个数
	{ 
		u2 num_annotations;		#注解的数量
		annotation annotations[num_annotations];	#注解值
	} parameter_annotations[num_parameters];
}
````


####18. RuntimeInvisibleParameterAnnotations
RuntimeInvisibleParameterAnnotations跟RuntimeVisibleParameterAnnotations类似，RuntimeInvisibleParameterAnnotations表示方法参数不可见注解，不能通过反射API获取，除非通过某些特殊方式(如命令行参数)获取，JVM虚拟机会忽略这些参数，结构参考RuntimeVisibleParameterAnnotations

####19. RuntimeVisibleTypeAnnotations
RuntimeVisibleTypeAnnotations是一个变长属性，位于ClassFile, field_info,  method_info，或者code属性中，记录了可见注解在ClassFile, field_info,  method_info中的正确位置，或者方法体表达式的位置，JVM必须能够识别这个注解以便被反射API获取到。

####20. RuntimeInvisibleTypeAnnotations


####21. AnnotationDefault
AnnotationDefault位于某些特定method_info的变长属性，表示注解结构的默认值，如一个注解内定义一个方法： int i() default 1;
结构如下：
```
AnnotationDefault_attribute {    
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为AnnotationDefault
	u4 attribute_length;	#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	element_value default_value;	#默认值
}
```

####22. BootstrapMethods
变长属性，位于ClassFile结构的属性表中。用于保存invokedynamic指令引用的引导方法限定符。
如果常量池中至少有一个CONSTANT_InvokeDynamic_info，则ClassFile中有且只能有一个BootstrapMethods属性。
结构如下：
**疑问点：为何必须为6或8？**
````
BootstrapMethods_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为BootstrapMethods
	u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u2 num_bootstrap_methods;	#bootstrap_methods个数
	{ 
		u2 bootstrap_method_ref;	#CONSTANT_MethodHandle_info
		u2 num_bootstrap_arguments;	#参数个数
		u2 bootstrap_arguments[num_bootstrap_arguments];	#指向常量池索引
	} bootstrap_methods[num_bootstrap_methods];
}
注：
1. bootstrap_method_ref指向的CONSTANT_MethodHandle_info.reference_kind为6(REF_invokeStatic)或者8(REF_newInvokeSpecial)，否则可能导致不能执行方法。
2. bootstrap_arguments常量池类型为以下其中一种：
	CONSTANT_String_info
	CONSTANT_Class_info
	CONSTANT_Integer_info
	CONSTANT_Long_info
	CONSTANT_Float_info
	CONSTANT_Double_info
	CONSTANT_MethodHandle_info
	CONSTANT_MethodType_info
````

####23. MethodParameters
MethodParameters是位于method_info 的一个变长属性，用于记录方法参数信息，如名字，访问权限等。
结构如下：
```
MethodParameters_attribute {
	u2 attribute_name_index;	#CONSTANT_Utf8_info,值为MethodParameters
	u4 attribute_length;		#属性长度，不包括attribute_name_index+attribute_length 6字节长度
	u1 parameters_count;	#方法参数个数
	{ 
		u2 name_index;	方法名称索引：为0表示没名字，大于0表示常量池索引
		u2 access_flags;
	} parameters[parameters_count];
}
注：access_flags
0x0010 (ACC_FINAL)	final修饰
0x1000 (ACC_SYNTHETIC)	非源码编译，不出现在源码文件中
0x8000 (ACC_MANDATED)	源码编译，出现在源码文件中

```