Python最强大的一个特性是能和C语言编写的程序和库互动。通过载入C动态库访问的功能，可以使用第三方通用扩展模块ctypes，它是Python内置的一个模块，允许调用C语言编写的动态库函数，只需要写些纯Python代码，从而实现Python和C的混合编程。

ctypes提供了调用C动态库接口的API，与C语言兼容的数据类型，这也可以理解为ctypes是一个适配模块，通过只修改Python代码来实现调用C动态库函数。调用函数就要涉及传递参数，处理返回值基本问题，下面分析如何使用ctypes实现C库函数的调用。

### 加载动态库

使用dll加载器的LoadLibrary方法加载动态库，返回一个动态库对象：

    >>> from ctypes import *
    >>> libc = cdll.LoadLibrary("/usr/lib64/libc.so.6")

接下来调用库函数是通过访问动态库对象属性的方式来实现：

    >>> libc.time  #函数指针对象
    <_FuncPtr object at 0x7fb54ae3e2c0>
    >>> print libc.time(None) #调用库函数
    1494488034
    
调用函数必然要面临的问题是如何传参和获取返回值。

### 传参

- 数据类型

ctypes定义了一些与C语言兼容的数据类型。

如下所示为ctypes，C语言和Python的数据类型对比：

![](http://i.imgur.com/NL1MZxt.png)

- 值传递

Python中一切都是对象，ctypes的数据类型也不例外。实例化这些数据类型时可以使用正确的值做初始化。

    >>> from ctypes import *
    >>> i = c_int(3) #初始值为3
    >>> print i
    c_int(3)
    >>> print i.value
    3
    >>> i.value = 5  #修改
    >>> print i.value
    5
    >>> s = c_char_p("Hello world") #初始化字符串指针
    >>> s
    c_char_p(140416622172500)
    >>> s.value
    'Hello world'

上面示例中定义了一个整型和字符指针，它们可以用来作为函数参数进行值传递。

- 传递指针

函数调用除了传值，传递传递指针变量也是很常见的。上例中定义了一个字符串指针，ctypes还提供了byref()函数来实现传递指针。

如下所示声明了库函数：

    c_lib_module.h
    #动态库接口函数原型
    uint32_t read_usbkey(char *label, uint8_t *p_data, uint64_t *p_data_len);

在Python中调用库函数，传递指针(引用)：

    call_c_lib.py
    from ctypes import *
    
    handle = CDLL('./libtest.so')
    label = "key label"
    #创建指定大小的字符缓冲区
    buf = create_string_buffer(64*1024) 
    length = c_ulong()
    length.value = 64*1024
    #函数调用传递指针
    ret = handle.read_usbkey(label, byref(buf), byref(length))

示例说明：

1.create_string_buffer()和byref()函数是ctypes提供的工具函数，前者用来创建字符缓冲区，后者用来获取变量的指针(引用)。

### 返回值

默认情况，ctypes认为函数返回的是C语言的int型，如果返回其它类型，需要使用函数对象的restype属性指定类型。看下面的示例：

    >>> from ctypes import *
    >>> libc = CDLL("/ust/lib64/libc.so.6")
    >>> strchr = libc.strchr
    >>> strchr("abcdef", ord("d")
    8059983
    >>> strchr.restype = c_char_p
    >>> strchr("abcdef", ord("d")
    'def'

说明：

1.ord()是Python内置函数，返回字符对应的整数值，例如ord('a')返回97。

2.还可以将函数作为restype属性值，在函数中做返回值检查和异常处理。

### 其它

上面介绍的是ctypes部分特性，用来实现对C语言动态库函数的调用。还有数组，结构体等特性可参考[ctypes文档](https://docs.python.org/2/library/ctypes.html)。
