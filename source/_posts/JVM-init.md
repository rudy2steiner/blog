---
title: 从Hotspot对象实例化过程，理解类加载
toc: true
tags:
  - JVM
  - 类加载
date: 2019-05-28 08:46:08
---


Java 对象的实例化JVM都做了那些事情？在日常编写Java项目的时候，实例化过无数多的对象，随时都在与JVM交互！对JVM的认识总是停留在只是知道它能解析识别，并加载运行Java编写的程序，好奇心驱使着我开始关注JVM是如何实现这一系列功能的。

即使读过《深入理解Java虚拟机》，对JVM的理解也只是达到一个入门的程度，开源的JVM实现凤毛菱角，为大家所熟知的OpenJDK自然成为了想一探究竟的入口，经过一番的搜寻，发现另外一本《Hotspot实战》让我看到了一丝希望。这本书买过来在我的书桌有一段时间了，最近终于有空仔细阅读了前面三章的内容，但纸上得来终觉浅，源码面前无秘密，跟着这本书终于满足我对JVM的一丝好奇心。

面对Hotspot如此庞大的工程，并且C++语言对大多数人也不太熟悉，项目的入口在哪里？这个我目前也不太清楚；我最关心的是什么？main、Launcher、new关键字大概是我最想了解的，类加载恰好可以串联它们。第一篇JVM相关的博客，看书和写博客都是都是慢长的过程（个人感觉），有些东西不值得看，也不值得写，JVM属于另外一类，那就抽空看，再慢慢写。


## 类加载

一个class文件，需要依次完成加载、链接、初始化三个阶段，才可以在JVM中正常的使用。如创建实例，访问类的静态方法和域。
主要步骤如下：

1 class 文件加载完成hook，提供parse前改变字节码的能力
2 读取魔数、大版本和小版本号、类的常量池(符号引用)、访问标示
3 解析类的接口、域、方法及父类
4 计算vtable 和 itable 大小
5 创建InstanceKlass
6 创建Java 镜像类,并初始化静态成员变量


```c++
instanceKlassHandle ClassFileParser::parseClassFile(Symbol* name,
                                                ClassLoaderData* loader_data,
                                                Handle protection_domain,
                                                KlassHandle host_klass,
                                                GrowableArray<Handle>* cp_patches,
                                                TempNewSymbol& parsed_name,
                                                bool verify,
                                                TRAPS) {
    ......

    JvmtiExport::post_class_file_load_hook(...); // class file load hook

    ......

     // Magic value
     u4 magic = cfs->get_u4_fast();
     // Version numbers
     u2 minor_version = cfs->get_u2_fast();
     u2 major_version = cfs->get_u2_fast();

     ......

     // Constant pool
    constantPoolHandle cp = parse_constant_pool(...);

     ......

    // Access flags
    access_flags.set_flags(flags);

    ......

    // Interfaces
    Array<Klass*>* local_interfaces =parse_interfaces(...);

    ......

     // Fields (offsets are filled in later)
    Array<u2>* fields = parse_fields(...);

    ......

    // Methods
    Array<Method*>* methods = parse_methods(...);

    ......

    // Finalize the Annotations metadata object,
    // now that all annotation arrays have been created.
    create_combined_annotations(CHECK_(nullHandle));

    ......

    // Supper class
    Klass* k = SystemDictionary::resolve_super_or_fail(...);

    // Compute the transitive list of all unique interfaces implemented by this class
    transitive_interfaces =compute_transitive_interfaces(super_klass, local_interfaces, CHECK_(nullHandle));

     // vtable and itable size
     klassVtable::compute_vtable_size_and_num_mirandas(...)
     klassItable::compute_itable_size(transitive_interfaces);

    // We can now create the basic Klass* for this klass
    InstanceKlass* kclass=InstanceKlass::allocate_instance_klass(...);
    instanceKlassHandle this_klass(THREAD, klass);

    // Allocate mirror and initialize static fields
    java_lang_Class::create_mirror(this_klass, class_loader,...);


    // Notify class load  
    ClassLoadingService::notify_class_loaded(InstanceKlass::cast(this_klass()),false /* not shared class */);


}


```

## 类的实例化

  从虚拟机层面来看，所有的Java构造函数都显示为具有特殊名称[ &lt;init&gt; ](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-2.html)的实例初始化方法。Hotspot中调用&lt;init&gt;的代码如下：

vmSymbols.h 头文件包含object_initializer_name：

```c++

  /* common method and field names */           
  template(object_initializer_name,                "<init>")  
  template(class_initializer_name,               "<clinit>")    

```

jvm.cpp

```c++
  JVM_AllocateNewObject(JNIEnv *env,
                        jobject receiver,
                        jclass currClass,
                        jclass initClass){
    ......                      
    methodHandle m(THREAD,init_klass->find_method(vmSymbols::object_initializer_name(),vmSymbols::void_method_signature()));
    Handle obj = curr_klass->allocate_instance_handle(CHECK_NULL);

    // Call constructor m. This might call a constructor higher up in the hierachy
    JavaCalls::call_default_constructor(thread, m, obj, CHECK_NULL);
}

```  

待续...


## 追问

* Java如何实现重载以及重写的？

* 一个对象最少占用多大的内存？
