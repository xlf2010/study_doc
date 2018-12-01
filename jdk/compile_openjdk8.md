#编译openjdk8，centos7下编译
1. 下载源码：
 hg clone http://hg.openjdk.java.net/jdk8/jdk8 openjdk8

2. 安装依赖
2.1 下载配置jdk，
2.2 安装编译所需的依赖：
````
yum install gcc gdb gcc-c++ libXtst-devel libXt-devel libXrender-devel cups-devel freetype-devel alsa-lib-devel 
````
2.3 yum没能安装到ccache，到官网下载ccache：
````
wget https://www.samba.org/ftp/ccache/ccache-3.4.2.tar.bz2
````
解压安装 
````
tar -jxf ccache-3.4.2.tar.bz2 && cd ccache-3.4.2 && ./configure --prefix=/usr/local/ccache && make && make install
````

3. 编译安装
3.1 进入openjdk8源码目录，运行配置脚本
````
bash ./configure --prefix=/usr/local/openjdk8/ --with-debug-level=slowdebug  2>&1 | tee configure.log
````
--with-ccache-dir 表示刚刚安装的ccache目录，prefix为openjdk编译后的安装目录
更多参数请使用 bash ./configure --help 查看

````
	Configuration summary:
	* Debug level:    release
	* JDK variant:    normal
	* JVM variants:   server
	* OpenJDK target: OS: linux, CPU architecture: x86, address length: 64
	
	Tools summary:
	* Boot JDK:       java version "1.8.0_181" Java(TM) SE Runtime Environment (build 1.8.0_181-b13) Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)  (at /usr/local/jdk)
	* C Compiler:     gcc (GCC) 4.8.5 20150623 (Red Hat-28) version 4.8.5-28) (at /usr/bin/gcc)
	* C++ Compiler:   g++ (GCC) 4.8.5 20150623 (Red Hat-28) version 4.8.5-28) (at /usr/bin/g++)
	
	Build performance summary:
	* Cores to use:   1
	* Memory limit:   1838 MB
	* ccache status:  not installed (consider installing)
	
	WARNING: The result of this configuration has overridden an older
	configuration. You *should* run 'make clean' to make sure you get a
	proper build. Failure to do so might result in strange build problems.
````
3.2 编译：运行命令 make all,编译时间比较长，耐心等待

问题点：
编译nashorn模块时出现异常：
````
	## Starting nashorn
Compiling 435 files for BUILD_NASHORN
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Compiling 111 files for BUILD_NASGEN
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
Running nasgen
Exception in thread "main" java.lang.VerifyError: class jdk.nashorn.internal.objects.ScriptFunctionImpl overrides final method setPrototype.(Ljava/lang/Object;)V
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
	at java.net.URLClassLoader.defineClass(URLClassLoader.java:467)
	at java.net.URLClassLoader.access$100(URLClassLoader.java:73)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:368)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:362)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:361)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at jdk.nashorn.internal.tools.nasgen.StringConstants.<clinit>(StringConstants.java:85)
	at jdk.nashorn.internal.tools.nasgen.MemberInfo.verify(MemberInfo.java:250)
	at jdk.nashorn.internal.tools.nasgen.ScriptClassInfo.verify(ScriptClassInfo.java:227)
	at jdk.nashorn.internal.tools.nasgen.Main.process(Main.java:108)
	at jdk.nashorn.internal.tools.nasgen.Main.processAll(Main.java:88)
	at jdk.nashorn.internal.tools.nasgen.Main.main(Main.java:62)
gmake[1]: *** [/root/source/openjdk8/build/linux-x86_64-normal-server-release/nashorn/classes/_the.nasgen.run] Error 1

````

解决办法：
修改：vim nashorn/make/BuildNashorn.gmk 
80行原来 -cp 修改为：-Xbootclasspath/p

原来代码：
````
 73 # Copy classes to final classes dir and run nasgen to modify classes in jdk.nashorn.internal.objects package
 74 $(NASHORN_OUTPUTDIR)/classes/_the.nasgen.run: $(BUILD_NASGEN)
 75     $(ECHO) Running nasgen
 76     $(MKDIR) -p $(@D)
 77     $(RM) -rf $(@D)/jdk $(@D)/netscape
 78     $(CP) -R -p $(NASHORN_OUTPUTDIR)/nashorn_classes/* $(@D)/
 79     $(FIXPATH) $(JAVA) \
 80         -cp "$(NASHORN_OUTPUTDIR)/nasgen_classes$(PATH_SEP)$(NASHORN_OUTPUTDIR)/nashorn_classes" \
 81         jdk.nashorn.internal.tools.nasgen.Main $(@D) jdk.nashorn.internal.objects $(@D)
 82     $(TOUCH) $@

````
修改后：
````
 73 # Copy classes to final classes dir and run nasgen to modify classes in jdk.nashorn.internal.objects package
 74 $(NASHORN_OUTPUTDIR)/classes/_the.nasgen.run: $(BUILD_NASGEN)
 75         $(ECHO) Running nasgen
 76         $(MKDIR) -p $(@D)
 77         $(RM) -rf $(@D)/jdk $(@D)/netscape
 78         $(CP) -R -p $(NASHORN_OUTPUTDIR)/nashorn_classes/* $(@D)/
 79         $(FIXPATH) $(JAVA) \
 80             -Xbootclasspath/p:"$(NASHORN_OUTPUTDIR)/nasgen_classes$(PATH_SEP)$(NASHORN_OUTPUTDIR)/nashorn_classes" \
 81             jdk.nashorn.internal.tools.nasgen.Main $(@D) jdk.nashorn.internal.objects $(@D)
 82         $(TOUCH) $@

````

修改后重新执行 make all，编译成功
执行make install 安装到指定目录。

4. 编译成功后，会生成build目录
5. 调试java
写个hello world编译：
````
	public class HelloWorld{
		public static void main(String[] args){
			System.out.println("Hello world");
		}
	}
````
编译命令：
````
	build/linux-x86_64-normal-server-release/jdk/bin/javac HelloWorld.java 
````
编译成功后生成HelloWorld.class



调试前先将libjvm.so调试信息解压
openjdk8/build/linux-x86_64-normal-server-slowdebug/jdk/lib/amd64/server/libjvm.so
````
build/linux-x86_64-normal-server-slowdebug/jdk/lib/amd64/server$ unzip libjvm.diz
````

调试java命令
````
	gdb build/linux-x86_64-normal-server-release/jdk/bin/java
	// 设置参数
	(gdb) set args HelloWorld
	// 启动程序
	(gdb) start
	Temporary breakpoint 1 at 0x400540: file /root/source/openjdk8/jdk/src/share/bin/main.c, line 94.
	Starting program: /root/source/openjdk8/build/linux-x86_64-normal-server-release/jdk/bin/java HelloWorld
	[Thread debugging using libthread_db enabled]
	Using host libthread_db library "/lib64/libthread_db.so.1".
	
	Temporary breakpoint 1, main (argc=2, argv=0x7fffffffe438) at /root/source/openjdk8/jdk/src/share/bin/main.c:94
	94	{
	(gdb) n
	125	    return JLI_Launch(margc, margv,
	(gdb) bt
	#0  main (argc=2, argv=0x7fffffffe438) at /root/source/openjdk8/jdk/src/share/bin/main.c:125

	// 可以看到刚刚设置的参数
	(gdb) p argv[1]
	$1 = 0x7fffffffe6f2 "HelloWorld"
````


