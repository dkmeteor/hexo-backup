title: Bubble-Notification Update
date: 2015-10-25 16:48:18
categories:
  - android
tags:
---
# Bubble-Notification Update


[Bubble-Notification](https://github.com/dkmeteor/Bubble-Notification)

https://github.com/dkmeteor/Bubble-Notification

![](/images/bubble_notification_heart.gif)


做了一次大更新

记录一下相关内容

#1.Bug fix

## 1.1 贝塞尔曲线调整

看了这篇文章

[QQ手机版 5.0“一键下班”设计小结](http://isux.tencent.com/qq-mobile-off-duty.html)以后,修改了一下贝塞尔曲线的绘制方式

这文章作者是个设计师,并不是程序员

我也没有完全按他的去画这个贝塞尔曲线,优化了一下曲线函数,主要贝塞尔曲线参考点的修改,修复几个参考点计算错误的问题


### 1.2 修复由StatusBar高度导致的位置计算偏移

检讨一下,由于这部分公式计算比较复杂,且两种绘图方式坐标不统一,代码中有大量的位置换算,导致了很多小的误差.


#2. 新功能


##2.1 添加特效

最近看了 [ExplosionField](https://github.com/tyrantgit/ExplosionField) ,  [ParticleLayout](https://github.com/ZhaoKaiQiang/Par
ticleLayout)等项目,有了一些新的想法,打算更新一下Bubble-Notification的特效,添加几个新的.

由于Bubble-Notification的Explosion本身设计上是相当于一个微型粒子引擎,通过添加新粒子类型,给粒子添加更多属性和字段,可以挺方便的定制各种特效.

不过凑巧,那天正好戳到一个AE教程,稍微看了一下,然后我意识到, AE作为专业的特效工具,其提供的功能远远不是目前这种简陋的效果能比的.

结合起来也许能做一些更有趣的东西.

##2.2 添加GifRender

AE导出的效果只能是movie gif 或 序列帧,目前看来使用比较方便的只有gif和序列帧
先添加一个Gif支持来试试水

由于所有效果都直接绘制在SurfaceView上,需要自身控制,所以不方便使用第三方Gif库,好在通过android.graphics.Movie自己撸一个也很简单.

写了GifRender包装了一下,为了和Explosion保持一致,添加了GifUpdateThread用来刷新Gif

#3. 重构

添加GifRender过程后,由于以后可能还会添加新的特效模式,于是重构了下代码,将最后Explosion特效相关的代码从DropCover类中全部抽离,DropCover只负责贝塞尔曲线和圆形的绘制.

相关杂七杂八过程控制代码都丢入CoverMananger类中,这不是一个特别好的方案,不过把200行代码按OOP的方针分割成4~5个类也不是特别好的方案.暂且保持这样.如果控制过程的代码在将来会膨胀的话,到时候必须将CoverMananger中的代码拆分成各个独立的功能模块,不过目前没有必要.

---

### 具体算法过程描述:

这个一定要写一下,看半年前的代码完全看不懂了,到处都是坑,90%的问题都是 坐标点换算造成的.

        private void drawDrop() {
        Canvas canvas = getHolder().lockCanvas();
        if (canvas != null) {
            canvas.drawColor(Color.TRANSPARENT, Mode.CLEAR);

            if (isDraw) {
                double distance = Math.sqrt(Math.pow(mBaseX - mTargetX, 2) + Math.pow(mBaseY - mTargetY, 2));
                mPaint.setColor(0xffff0000);
                mPaint.setStyle(Paint.Style.FILL);
                if (distance < mMaxDistance) {
                    //计算圆形半径
                    mStrokeWidth = (float) ((1f - distance / mMaxDistance) * mRadius);
                    //绘制 原位置的圆形,会随着拖动距离还减小,注意mBaseX,mBaseY是圆心坐标
                    canvas.drawCircle(mBaseX, mBaseY, mStrokeWidth / 2, mPaint);
                    drawBezier(canvas);
                }
                //绘制拖动后的图形,注意mTargetX,mTargetY是左上角坐标
                canvas.drawBitmap(mDest, mTargetX, mTargetY, mPaint);
            }
            getHolder().unlockCanvasAndPost(canvas);
        }
    }

注意`drawCircle`和`drawBitmap` 使用的坐标是不一样的,一个是圆心,一个是图片左上角.
进行坐标计算时使用的都是 圆心坐标,可以参考下面的图片

注意换算,非常操蛋.

 ---
 # 算法详细
 

可以参考[QQ手机版 5.0“一键下班”设计小结](http://isux.tencent.com/qq-mobile-off-duty.html),基本上是一致的,就是贝塞尔曲线的参考点不会移动


计算2根贝塞尔曲线的4个起点/终点

看代码可能不明白,还原下其实就是简单的三角函数

    /**
     * ax=by=0 x^2+y^2=s/2
     * <p/>
     * ==>
     * <p/>
     * x=a^2/(a^2+b^2)*s/2
     *
     * @param start
     * @param end
     * @return
     */
    private Point[] calculate(Point start, Point end) {
        float a = end.x - start.x;
        float b = end.y - start.y;

        float y1 = (float) Math.sqrt(a * a / (a * a + b * b) * (mStrokeWidth / 2f) * (mStrokeWidth / 2f));
        float x1 = -b / a * y1;

        float y2 = (float) Math.sqrt(a * a / (a * a + b * b) * (targetWidth / 2f) * (targetHeight / 2f));
        float x2 = -b / a * y2;


        Point[] result = new Point[4];

        result[0] = new Point(start.x + x1, start.y + y1);
        result[1] = new Point(end.x + x2, end.y + y2);

        result[2] = new Point(start.x - x1, start.y - y1);
        result[3] = new Point(end.x - x2, end.y - y2);

        return result;
    }



---

绘制贝塞尔曲线

贝塞尔曲线使用的参考点为 centerX,centerY,为两圆圆心连线的中点


    private void drawBezier(Canvas canvas) {

        Point[] points = calculate(new Point(mBaseX, mBaseY), new Point(mTargetX + mDest.getWidth() / 2f, mTargetY + mDest.getHeight() / 2f));

        float centerX = (points[0].x + points[1].x + points[2].x + points[3].x) / 4f;
        float centerY = (points[0].y + points[1].y + points[2].y + points[3].y) / 4f;

        Path path1 = new Path();
        path1.moveTo(points[0].x, points[0].y);
        path1.quadTo(centerX, centerY, points[1].x, points[1].y);
        path1.lineTo(points[3].x, points[3].y);

        path1.quadTo(centerX, centerY, points[2].x, points[2].y);
        path1.lineTo(points[0].x, points[0].y);
        canvas.drawPath(path1, mPaint);
    }

按腾讯那文章来看,参考点随距离移动效果可能会更好,我尝试了一下,看不出明显的区别. 

---
加了个Note:

     The origin circle drawn by  canvas.drawCircle(mBaseX, mBaseY, mStrokeWidth / 2, mPaint);
    
     so mBaseX, mBaseY are center position.
    
     The Moved circle drawn by  canvas.drawBitmap(mDest, mTargetX, mTargetY, mPaint);
    
     so mTargetX,mTargetY are left top corner position.
     
     
计算过程中时刻记得这2点,之前好多bug就是因为这个造成的.


---

Explosion

其实这个东西非常简单.

一个简单的粒子生成器,实例化直接生成所有粒子

随机生成每个例子的颜色 x速度 y速度

然后由ExplosionUpdateThread刷新每个粒子的坐标并绘图

 
正常的 粒子引擎 功能要复杂的多的多, 不过上cocos2d或者别的图形/游戏引擎感觉没有必要
 

 
 
 
    
