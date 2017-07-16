## Matplotlib快速入门

Matplotlib是一个Python开源项目，提供数据绘图软件包，使用十分广泛。它使得用Python可以快速的将数据可视化，产生各种格式并达到出版级质量的输出。

#### 依葫芦画瓢

Matplotlib项目的文档和[示例](https://matplotlib.org/gallery.html)资料十分丰富，参考一下丰富的示例，然后就直接使用它来绘制图并非难事。快速入门的第一步是依葫芦画瓢，是模仿的过程中入门。用它来解决一个问题，绘制一个图形，过程中涉及到的概念暂时不必深究，猜猜大概意思就行。例如使用Matplotlib来绘制两组股票历史报价和月增长率曲线图，最终绘制的效果如下图所示：


![](http://i.imgur.com/1rEVNWE.png)

​                                                                                       图1

如何一边查资料，一边参考官方示例来绘制出如上所示的曲线图，在这里就不详细介绍。写到这好像这篇文章也可以结束了，能够画出图1所示的内容也算是摸到门了，但如果觉得依葫芦画瓢的学习曲线有点陡，特别是对计算机编程画图没有任何经验的情况下尤为突出，那可以看看下面的内容，跟着做一遍，或许对你更有帮助。

下面先了解最少的必要核心概念，然后有一个小示例实际操作一下，对概念会有更直观的了解。

#### 基本概念

Matplotlib将绘图所需的基本概念封装成对象，通过对象方法提供的功能来绘制图形元素，设置元素属性。如下图所示，图中列出了绘制图形时会涉及的图形元素。

![](http://i.imgur.com/SB4MTPV.png)

​                                                                                    图2

Figure对象代表整个图形。

[Axes](https://matplotlib.org/api/axes_api.html)对象表示绘图区域，包含主要的图形元素，例如坐标轴，标题等，有时也称一个绘图区域为子图(subplot)。Figure对象可以包含一个或多个Axes对象，一个Axes对象只能被包含在一个Figure对象中。

Axis对象用来表示坐标轴，坐标轴限定了数据范围，包含均匀分布的坐标轴刻度(ticks)，刻度标签(tick label)，坐标轴标签(axis label)。

Spines对象是数据展示区域的边界线。

#### 动手实践

了解了核心的基本概念，接下来看看如何使用Python编程语言来使用这些概念绘制出图形。核心对象提供了许多设置属性的方法，初学者不可能全都记下来，把[官方文档](http://matplotlib.org/contents.html)当手册按需查询，例如后面示例中大量使用到的Axes对象相关方法就包含在The Matplotlib API一节的Axes class中。

- 创建图形对象和子图对象

pyplot模块提供的subplots方法可以用来创建一个图形对象和一个或多个子图。子图的布局方法是按行分列放置，subplots方法有两个默认参数nrows和ncols，它们用来确定图形中包含多少个子图，分别按几行几列放置。如下示例创建了4个子图，从左到右，从上到下排成了两行两列。第5行代码取出放在第一行第二列位置上的子图。

    >>> import numpy as np
    >>> import matplotlib
    >>> matplotlib.use('Agg')   #使用PNG图形渲染引擎,输出PNG格式图像
    >>> import matplotlib.pyplot as plt
    >>> fig, axs = plt.subplots(2, 2)    #创建图形对象和4个子图对象
    >>> ax = axs[0, 0]    #从数组中取出第一行第一列的子图对象
    >>> plt.subplots_adjust(hspace=0.5, wspace=0.3)    #设置子图行间距
    #使用numpy模块产生(x,y)坐标数据
    >>> x = np.linspace(0.0, 5.0, 100)
    >>> y = np.cos(2 * np.pi * x) * np.exp(-x)
    >>> ax.plot(x, y, "k")    #在子图中根据数据描点画线,"k"表示线条颜色为黑色

- 设置子图中的图形元素

使用Axes对象的方法设置子图元素：

    #定义字体格式
    >>> font = {"family": "serif", "color": "darkred", "weight": "normal", "size":16,}
    #在子图中通过坐标定位添加一段文本
    >>> ax.text(2, 0.65, "cos(2%st)exp(-t)" % (u"\u03C0"), fontdict=font) 
    >>> ax.set_title("Damped exponential decay", fontdict=font)    #设置子图标题
    >>> ax.grid(True)    #设置网格
    #不显示子图上面和右面的边框线
    >>> ax.spines["top"].set_visible(False)
    >>> ax.spines["right"].set_visible(False)
    #只显示左边纵轴和下方横轴上的刻度
    >>> ax.yaxis.set_ticks_position("left")
    >>> ax.xaxis.set_ticks_position("bottom")
    #设置x轴和y轴标签
    >>> ax.set_xlabel("time (s)", fontdict=font)
    >>> ax.set_ylabel("voltage (mV)", fontdict=font)
    >>> plt.savefig("demo.png")    #保存图片

最后保存的图片如下：

![](http://i.imgur.com/rh745zF.png)

#### 后记

Matplotlib的功能很丰富，有需求和应用场景，勤加练习方能生巧。