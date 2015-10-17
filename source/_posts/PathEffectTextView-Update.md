title: PathEffectTextView Update
date: 2015-10-17 14:11:43
categories:
  - android
tags:
---

更新了一下[PathEffectTextView](https://github.com/dkmeteor/PathEffectTextView)

- 添加了字体大小 颜色 字重(bold) 阴影 等设置项.

- 更新了下Demo.

- 做了几个新的Demo Gif

- 发布到Jcenter



## TODO:

- 等有空的时候,添加各种笔刷.

---

#Screenshot

Please waiting for loading the gif...

![](https://github.com/dkmeteor/PathEffectTextView/blob/master/path1.gif?raw=true)

![](https://github.com/dkmeteor/PathEffectTextView/blob/master/path2.gif?raw=true)

![](https://github.com/dkmeteor/PathEffectTextView/blob/master/path3.gif?raw=true)

![](https://github.com/dkmeteor/PathEffectTextView/blob/master/path4.gif?raw=true)

#How to use

Step 1: add denpendence


    compile('com.dk.view.patheffect:Library:0.1.1@aar')
    

    
If you are still using `Eclipse`, you can just copy source code or jar file to you project.



Step 2: add view to your layout:

    <com.dk.view.patheffect.PathTextView
        android:id="@+id/path"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

step 3: call `init` method like this:

    PathTextView mPathTextView = (PathTextView) findViewById(R.id.path);
    mPathTextView.init("Hello World");

Option settings:

    mPathTextView.setPaintType(PathTextView.Type.MULTIPLY);
    mPathTextView.setTextColor(color);
    mPathTextView.setTextSize(size);
    mPathTextView.setTextWeight(weight);
    mPathTextView.setDuration(duration);
	mPathTextView.setShadow(radius, dx, dy, shadowColor);

#NOTE

- Only Support capital letter, you can check this file for [`Path Data`](https://github.com/dkmeteor/PathEffectTextView/blob/master/Library/src/main/java/com/dk/view/patheffect/MatchPath.java), the data comes from [android-Ultra-Pull-To-Refresh](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh/)

- the size unit is px , 72 means 72px*72px.

- the text weight unit is px.