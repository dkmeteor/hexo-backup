title: Android Studio Gradle构建速度优化
id: 247
categories:
  - android
  - Blog
date: 2015-03-16 16:48:01
tags:
---

<div>
<div>1.</div>
<div></div>
<div>[https://www.timroes.de/2013/09/12/speed-up-gradle/](https://www.timroes.de/2013/09/12/speed-up-gradle/)</div>
<div></div>
<div>create a file named **gradle.properties** in the following directory:</div>
<div></div>

*   /home/&lt;username&gt;/.gradle/ (_Linux_)
*   /Users/&lt;username&gt;/.gradle/ (_Mac_)
*   C:\Users\&lt;username&gt;\.gradle (_Windows_)
Add this line to the file:
<pre style="color: #b9bdb6;">org.gradle.daemon=true
</pre>
<div></div>
<div>From now on Gradle will use a daemon to build, whether you are using Gradle from command line or building in Android Studio. You could also place the **gradle.properties** file to the root directory of your project and commit it to your SCM system. But you would have to do this, for every project (if you want to use the daemon in every project).</div>
</div>
<div></div>
<div></div>
<div>
<div></div>
<div>关于这个写法有几个变种，都尝试了一下，并没有用。</div>
</div>
<div></div>
<div>参考Android Studio配置选项</div>
<div></div>
<div>[![Image](http://dk-exp.com/wp-content/uploads/2015/03/Image-1024x565.png)](http://dk-exp.com/wp-content/uploads/2015/03/Image.png)</div>
<div></div>
<div>Android Studio 1.1.0+版本中该配置已经默认打开。不</div>
<div>

* * *

</div>
<div>2.</div>
<div>用命令行Build</div>
<div></div>
<div>可以参考这个讨论串</div>
<div>[https://plus.google.com/u/0/+RicardoAmaral/posts/e9PG6vSN5w3](https://plus.google.com/u/0/+RicardoAmaral/posts/e9PG6vSN5w3)</div>
<div></div>
<div>gradle assembleDebug</div>
<div>实测速度快50%左右 原理不明</div>
<div>回头写个脚本 build完成再自动安装运行应该就好了</div>
<div></div>
<div>更新下</div>
<div></div>
<div>gradle installDebug 就可以了，不用自己在gradle里搞task了，省事.</div>
<div>明天测试下性能</div>
<div></div>
<div></div>
<div>

* * *

</div>
<div>3.</div>
<div>Android模块化编程之引用本地的aar</div>
<div>[http://stormzhang.com/android/2015/03/01/android-reference-local-aar/](http://stormzhang.com/android/2015/03/01/android-reference-local-aar/)</div>
<div></div>
<div>打算回头把 subModule全部做成本地aar，减少编译的文件，优化一下编译速度</div>
<div></div>
<div></div>
<div></div>
<div>

* * *

</div>
<div>4.</div>
<div>我觉得TMD还是把公司的破电脑扔了换个新的才能从根本上解决问题</div>