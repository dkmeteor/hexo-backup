title: 关于CheckBox的PaddingLeft
id: 280
categories:
  - Algorithm
date: 2015-03-26 16:30:09
tags:
---

在不同Android版本中对PaddingLeft的计算方式不同

在Android 4.1.2 之前,Checkbox的 paddingleft 是从View最左侧开始计算的

而Android 4.1.2 之后,Checkbox的 paddingLeft 是从drawableLeft的右侧开始计算起的

2个版本之间的 paddingLeft会差一个 drawableLeft的宽度

一个兼容实现方式

    //View hack for fucking difference between versions
            if (Build.VERSION.SDK_INT &lt;= 16) {//4.1.2
                mCheckBox.setPadding(ViewUtils.getPixels(20 + 10, mContext), 0, 0, 0);
                } else {
                mCheckBox.setPadding(ViewUtils.getPixels(10,mContext), 0, 0, 0);
            }

讨厌这种写法。
