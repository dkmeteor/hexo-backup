title: TO DO LIST
id: 241
categories:
  - Blog
date: 2015-03-03 13:21:36
tags:
---

<div>
<div>1.Bubble-notification</div>
<div></div>
draggableflagview
<div>[https://github.com/wangjiegulu/DraggableFlagView/blob/master/src/com/wangjie/draggableflagview/DraggableFlagView.java](https://github.com/wangjiegulu/DraggableFlagView/blob/master/src/com/wangjie/draggableflagview/DraggableFlagView.java)</div>
<div>BezierDemo</div>
<div>[https://github.com/chenupt/BezierDemo](https://github.com/chenupt/BezierDemo)</div>
<div></div>
<div>参考一下 Bezier 的另一种画法.</div>
<div>现在回想一下...当初我用2段带宽度的Bezier曲线来组合这个效果有点脑洞太大了....</div>
<div>直接用path不就完事了.......</div>
<div></div>
<div>顺便将该库迁移到Android Studio,发布aar以后,如果没有其它问题,应该不会再更新了</div>
<div></div>
<div>2.ScreenTransitionTool</div>
<div></div>
<div>ActivityOptionsICS</div>
<div>[https://github.com/tianzhijiexian/ActivityOptionsICS](https://github.com/tianzhijiexian/ActivityOptionsICS)</div>
<div></div>
<div>以前也做过这个效果,结果遇到了一些问题结果没有做完.</div>
<div>这几天发现有人已经做好了...</div>
<div>赶紧clone下来研究一下实现机制.</div>
<div></div>
<div>这个作者实现思路和我当初如出一辙....</div>
<div>将前一个Activity需要做动画的View截取Bitmap传到下一个Activity中</div>
<div></div>
<div>不过他还做了另一个ScreenAnimation的功能,我当时完全没有想到这个问题.</div>
<div></div>
<div>既然已经花了很多时间,考虑把ScreenTransitionTool完善一下.</div>
<div>只更新下共享组件动画好了.....</div>
<div></div>
<div>
ActivityOptionsICS</div>
<div>本身更着重于对 5.0新的ActivityOptions的低版本是配合实现.</div>
<div></div>
<div>那我的ScreenTransitionTool就做 成单独的 共享组件动画好了..</div>
<div></div>
<div>ActivityOptionsICS目前还只提供了基本的几种动画.</div>
<div>源代码还没有仔细研究,动画部分应该是可扩展的</div>
<div></div>
<div></div>
<div>3.Folder-reside-menu</div>
<div></div>
<div>本来是写好了</div>
<div>Folder-DrawerLayout</div>
<div>[https://github.com/dkmeteor/Folder-DrawerLayout](https://github.com/dkmeteor/Folder-DrawerLayout)</div>
<div></div>
<div>作为DrawerLayout的一个扩展,可以正常使用.</div>
<div>但是作为一个拟物化的控件,在边界控制上遇到了一个交互上的问题.</div>
<div></div>
<div>对于DrawerLayout来说,手指移动的距离和Drawer移动的距离是类似 甚至可能更短的...</div>
<div></div>
<div></div>
<div>但是对于 窗帘而言....窗帘移动的距离 往往只有 手指移动 距离的 一半左右</div>
<div>在家扯了半天窗帘</div>
<div></div>
<div>所以最后收尾那个效果很反直觉,虽然能用,但是不是很满意.</div>
<div></div>
<div>正好open-project-analysis 二期开始,那干脆做个 Reside-menu 的分析好了</div>
<div>做完了正好我可以把 这个 Folder 效果在 Reside-menu实现一下</div>
<div>一举两得把</div>
<div></div>
<div>目前大致浏览了一下 Reside-menu 效果实现机制,替换一部分效果控制逻辑就好,感觉上没有什么难度.</div>
<div></div>
<div></div>
<div>顺便</div>
<div>bitmapMesh</div>
<div>[https://github.com/7heaven/bitmapMesh](https://github.com/7heaven/bitmapMesh)</div>
<div></div>
<div>看到一个这样的库.  效果和我的差不多</div>
<div>不过verts矩阵他开的很小(30*10),导致边缘棱角很严重</div>
<div></div>
<div>而且这个地方当初我写的时候就想着...这TM写出来估计没人看得懂这一段矩阵计算在做什么...</div>
<div></div>
<div>虽然这个库做出来的效果和我的一样的...果然我也很难看懂他的运算逻辑....</div>
<div></div>
<div>X轴的压缩系数似乎是一个xy相关的函数,目测和我的不大一样</div>
<div></div>
<div>我是个y相关的二次函数来做出弧形的效果...</div>
<div></div>
<div>不过他这个 逻辑 和 调理   比我那一团乱麻清楚多了..</div>
<div></div>
<div>我要重构一下我这块代码...至少要重构到能让人看懂的地步.</div>
<div></div>
<div>另外个注意到他是通过<span style="color: #333333;">drawBitmapMesh附带的colors矩阵来添加的阴影,</span></div>
<div><span style="color: #333333;">我是通过shader添加的阴影,不过看起来效果没区别.</span></div>
<div><span style="color: #333333;"> </span></div>
<div><span style="color: #333333;"> </span></div>
<div><span style="color: #333333;"> </span></div>
<div><span style="color: #333333;">4\. HeartBeats</span></div>
<div><span style="color: #333333;"> </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">UI还没什么眉目.</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">打算先做过纯扁平化的UI.纯色.大字.</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">主色调红 绿 蓝 里选一个..</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;"> </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">先把核心逻辑写完吧.UI能看就行,最后再慢慢调.</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;"> </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;"> </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">大致安排是这样</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;"> </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">1.camera preview</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">2.renderscript YUV</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">3.charts (simple)</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">     这里先做个简单的能看的...用真实数据出的心率图会很难看...应该要处理一下...15fps的数据图都是棱角,没法看的</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">4.FFT</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">     暂定.FFT可以出频率.但是怀疑低频小数据量时用FFT是否合适.</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">     是否直接取采样点,做加权平均,或者趋势分析之类的来计算频率更准确更快.</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">     考虑到变化趋势,比如稳定5~10s就出数据,可能直接做时域数据分析更快?</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">5.UI</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">     5.1 charts这部分重做一下.</span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">     5.2 尝试一些动效     </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;"> </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;"> </span></div>
<div><span style="color: #333333; font-family: Consolas, 'Liberation Mono', Menlo, Courier, monospace; font-size: small;">6.</span> android-open-project-analysis</div>
<div></div>
<div>AndroidResideMenu</div>
<div>https://github.com/SpecialCyCi/AndroidResideMenu</div>
<div></div>
<div>这个库比较简单,分析起来会比较快,而且正好我要做Folder-reside-menu,一举两得吧.</div>
<div></div>
<div>方法分析其实并没有用,我觉得从结构 或者 调用流程上做分析更恰当</div>
<div>然后对 调用流程 上的关键点做分析</div>
<div></div>
<div>所以 预计是  做若干个流程图</div>
<div>然后对 流程图中 每一个节点做分析</div>
<div></div>
<div>全部重心放到流程分析上,方法分析作为流程图的补充</div>
<div></div>
<div>然后如果Folder-reside-menu能在结束前完结的话,就把扩展Folder-reside-menu的过程带一点进去作为夹带私货把.</div>
<div></div>
<div></div>
</div>