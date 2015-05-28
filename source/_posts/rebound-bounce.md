title: Rebound中bounce函数分析
id: 291
categories:
  - Algorithm
  - android
  - Blog
date: 2015-04-16 16:54:07
tags:
---

<div>花了点时间分析Rebound源代码，原本打算用代码反推一下公式，然后把公式掏出来用，结果最后发现是用多段函数组合出来的...这就有点麻烦...还是直接整个使用比较方便。</div>

<div></div>

<div></div>

<div></div>

<div>直观点分析的话，大概是这样</div>

<div></div>

<div>首先是 friction</div>

<div>当friction为5时，曲线是这样...恩...friction就影响曲线上凸起来的部分。</div>

<div></div>

<div></div>

<div>![](file:///C:/Users/DK/AppData/Local/Temp/enhtmlclip/Image.png)[![friction_5](http://dk-exp.com/wp-content/uploads/2015/04/friction_5.png)](http://dk-exp.com/wp-content/uploads/2015/04/friction_5.png)</div>

<div></div>

<div></div>

<div></div>

<div>friction为10时</div>

<div></div>

<div>![](file:///C:/Users/DK/AppData/Local/Temp/enhtmlclip/Image(1).png)[![friction_10](http://dk-exp.com/wp-content/uploads/2015/04/friction_10.png)](http://dk-exp.com/wp-content/uploads/2015/04/friction_10.png)</div>

<div></div>

<div>随着friction增大，OverScroll部分越来越小，渐进部分斜率越来越小</div>

<div></div>

<div>---</div>

<div></div>

<div>然后是 Tension  弹性 ，影响曲线变化的斜率。</div>

<div></div>

<div>Tension等于100时</div>

<div>![](file:///C:/Users/DK/AppData/Local/Temp/enhtmlclip/Image(2).png)[![tension_100](http://dk-exp.com/wp-content/uploads/2015/04/tension_100.png)](http://dk-exp.com/wp-content/uploads/2015/04/tension_100.png)</div>

<div></div>

<div>Tension等于70时</div>

<div>![](file:///C:/Users/DK/AppData/Local/Temp/enhtmlclip/Image(3).png)</div>

<div>[![tension_70](http://dk-exp.com/wp-content/uploads/2015/04/tension_70.png)](http://dk-exp.com/wp-content/uploads/2015/04/tension_70.png)</div>

<div></div>

<div>其实这个比较好理解，看代码实际上是一个 速度/时间 相关的变量，在Rebound中通过这个控制动画的时间。所以你不能直接指定动画的时间。</div>

<div>嘛。。比较好理解吧。。。移动10px复位动画的时间和移动100px复位动画的时间当然应当是不一致的，所以在Rebound系统中传入的参数是speed而不是动画时间。</div>

<div></div>

<div>本来是想把公式导出来用，反正代换一下贴到google里就能直接看到函数曲线</div>

<div>比如这个b3_friction3的曲线是这样</div>

<div></div>

<div>![](file:///C:/Users/DK/AppData/Local/Temp/enhtmlclip/Image(4).png)[![go_image](http://dk-exp.com/wp-content/uploads/2015/04/go_image.png)](http://dk-exp.com/wp-content/uploads/2015/04/go_image.png)</div>

<div></div>

<div></div>

<div>[https://www.google.com.hk/search?newwindow=1&amp;safe=strict&amp;q=%280.00000045+*x%5E3%29+-+%280.000332+*x%5E2%29+%2B+0.1078+*x%2B+5.84&amp;oq=%280.00000045+*x%5E3%29+-+%280.000332+*x%5E2%29+%2B+0.1078+*x%2B+5.84&amp;gs_l=serp.12...1469765.1474096.0.1474763.3.3.0.0.0.0.974.1647.3-2j6-1.3.0.msedr...0...1c.1.64.serp..3.0.0.H7nsfVzoOT4](https://www.google.com.hk/search?newwindow=1&amp;safe=strict&amp;q=%280.00000045+*x%5E3%29+-+%280.000332+*x%5E2%29+%2B+0.1078+*x%2B+5.84&amp;oq=%280.00000045+*x%5E3%29+-+%280.000332+*x%5E2%29+%2B+0.1078+*x%2B+5.84&amp;gs_l=serp.12...1469765.1474096.0.1474763.3.3.0.0.0.0.974.1647.3-2j6-1.3.0.msedr...0...1c.1.64.serp..3.0.0.H7nsfVzoOT4)</div>

<div></div>

<div></div>

<div></div>

<div>不过因为函数嵌套比较多，而且Tension那还是个分段函数，得找张大点的草稿纸都代换一下把原始公式组合出来，不过最后发现没有必要 Rebound Core里面本来就没有多余的东西,直接使用就好了。</div>

<div></div>

<div>周末可以把CircleAnimation的bounce效果用Rebound更新下.....我原来那个bounce系数还是从daimajia的AndroidViewAnimations里弄出来的......位移小的时候还看不出来，位移大了以后挺难看的...</div>

&nbsp;