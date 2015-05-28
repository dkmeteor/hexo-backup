title: android-Ultra-Pull-To-Refresh动画效果简要分析
id: 175
categories:
  - Blog
date: 2015-01-13 09:58:35
tags:
---

#android-Ultra-Pull-To-Refresh动画效果简要分析

[android-Ultra-Pull-To-Refresh](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh)

![image](https://camo.githubusercontent.com/d3fbe757c87fddc94e998ebdd08ac55956aed1cf/687474703a2f2f737261696e2d6769746875622e71696e6975646e2e636f6d2f756c7472612d7074722f73746f72652d686f7573652d737472696e672e676966)

核心类StoreHousePath
    package in.srain.cube.views.ptr.header;
    
    import android.util.SparseArray;
    
    import java.util.ArrayList;
    
    /**
    * Created by srain on 11/7/14.
    */
    public class StoreHousePath {
    
    private static final SparseArray&lt;float[]&gt; sPointList;
    
    static {
    sPointList = new SparseArray&lt;float[]&gt;();
    float[][] LETTERS = new float[][]{
    new float[]{
    // A
    24, 0, 1, 22,
    1, 22, 1, 72,
    24, 0, 47, 22,
    47, 22, 47, 72,
    1, 48, 47, 48
    },
    
    new float[]{
    // B
    0, 0, 0, 72,
    0, 0, 37, 0,
    37, 0, 47, 11,
    47, 11, 47, 26,
    47, 26, 38, 36,
    38, 36, 0, 36,
    38, 36, 47, 46,
    47, 46, 47, 61,
    47, 61, 38, 71,
    37, 72, 0, 72,
    },
    
    new float[]{
    // C
    47, 0, 0, 0,
    0, 0, 0, 72,
    0, 72, 47, 72,
    },
    
    new float[]{
    // D
    0, 0, 0, 72,
    0, 0, 24, 0,
    24, 0, 47, 22,
    47, 22, 47, 48,
    47, 48, 23, 72,
    23, 72, 0, 72,
    },
    
    new float[]{
    // E
    0, 0, 0, 72,
    0, 0, 47, 0,
    0, 36, 37, 36,
    0, 72, 47, 72,
    },
    
    new float[]{
    // F
    0, 0, 0, 72,
    0, 0, 47, 0,
    0, 36, 37, 36,
    },
    
    new float[]{
    // G
    47, 23, 47, 0,
    47, 0, 0, 0,
    0, 0, 0, 72,
    0, 72, 47, 72,
    47, 72, 47, 48,
    47, 48, 24, 48,
    },
    
    new float[]{
    // H
    0, 0, 0, 72,
    0, 36, 47, 36,
    47, 0, 47, 72,
    },
    
    new float[]{
    // I
    0, 0, 47, 0,
    24, 0, 24, 72,
    0, 72, 47, 72,
    },
    
    new float[]{
    // J
    47, 0, 47, 72,
    47, 72, 24, 72,
    24, 72, 0, 48,
    },
    
    new float[]{
    // K
    0, 0, 0, 72,
    47, 0, 3, 33,
    3, 38, 47, 72,
    },
    
    new float[]{
    // L
    0, 0, 0, 72,
    0, 72, 47, 72,
    },
    
    new float[]{
    // M
    0, 0, 0, 72,
    0, 0, 24, 23,
    24, 23, 47, 0,
    47, 0, 47, 72,
    },
    
    new float[]{
    // N
    0, 0, 0, 72,
    0, 0, 47, 72,
    47, 72, 47, 0,
    },
    
    new float[]{
    // O
    0, 0, 0, 72,
    0, 72, 47, 72,
    47, 72, 47, 0,
    47, 0, 0, 0,
    },
    
    new float[]{
    // P
    0, 0, 0, 72,
    0, 0, 47, 0,
    47, 0, 47, 36,
    47, 36, 0, 36,
    },
    
    new float[]{
    // Q
    0, 0, 0, 72,
    0, 72, 23, 72,
    23, 72, 47, 48,
    47, 48, 47, 0,
    47, 0, 0, 0,
    24, 28, 47, 71,
    },
    
    new float[]{
    // R
    0, 0, 0, 72,
    0, 0, 47, 0,
    47, 0, 47, 36,
    47, 36, 0, 36,
    0, 37, 47, 72,
    },
    
    new float[]{
    // S
    47, 0, 0, 0,
    0, 0, 0, 36,
    0, 36, 47, 36,
    47, 36, 47, 72,
    47, 72, 0, 72,
    },
    
    new float[]{
    // T
    0, 0, 47, 0,
    24, 0, 24, 72,
    },
    
    new float[]{
    // U
    0, 0, 0, 72,
    0, 72, 47, 72,
    47, 72, 47, 0,
    },
    
    new float[]{
    // V
    0, 0, 24, 72,
    24, 72, 47, 0,
    },
    
    new float[]{
    // W
    0, 0, 0, 72,
    0, 72, 24, 49,
    24, 49, 47, 72,
    47, 72, 47, 0
    },
    
    new float[]{
    // X
    0, 0, 47, 72,
    47, 0, 0, 72
    },
    
    new float[]{
    // Y
    0, 0, 24, 23,
    47, 0, 24, 23,
    24, 23, 24, 72
    },
    
    new float[]{
    // Z
    0, 0, 47, 0,
    47, 0, 0, 72,
    0, 72, 47, 72
    },
    };
    final float[][] NUMBERS = new float[][]{
    new float[]{
    // 0
    0, 0, 0, 72,
    0, 72, 47, 72,
    47, 72, 47, 0,
    47, 0, 0, 0,
    },
    new float[]{
    // 1
    24, 0, 24, 72,
    },
    
    new float[]{
    // 2
    0, 0, 47, 0,
    47, 0, 47, 36,
    47, 36, 0, 36,
    0, 36, 0, 72,
    0, 72, 47, 72
    },
    
    new float[]{
    // 3
    0, 0, 47, 0,
    47, 0, 47, 36,
    47, 36, 0, 36,
    47, 36, 47, 72,
    47, 72, 0, 72,
    },
    
    new float[]{
    // 4
    0, 0, 0, 36,
    0, 36, 47, 36,
    47, 0, 47, 72,
    },
    
    new float[]{
    // 5
    0, 0, 0, 36,
    0, 36, 47, 36,
    47, 36, 47, 72,
    47, 72, 0, 72,
    0, 0, 47, 0
    },
    
    new float[]{
    // 6
    0, 0, 0, 72,
    0, 72, 47, 72,
    47, 72, 47, 36,
    47, 36, 0, 36
    },
    
    new float[]{
    // 7
    0, 0, 47, 0,
    47, 0, 47, 72
    },
    
    new float[]{
    // 8
    0, 0, 0, 72,
    0, 72, 47, 72,
    47, 72, 47, 0,
    47, 0, 0, 0,
    0, 36, 47, 36
    },
    
    new float[]{
    // 9
    47, 0, 0, 0,
    0, 0, 0, 36,
    0, 36, 47, 36,
    47, 0, 47, 72,
    }
    };
    // A - Z
    for (int i = 0; i &lt; LETTERS.length; i++) {
    sPointList.append(i + 65, LETTERS[i]);
    }
    // a - z
    for (int i = 0; i &lt; LETTERS.length; i++) {
    sPointList.append(i + 65 + 32, LETTERS[i]);
    }
    // 0 - 9
    for (int i = 0; i &lt; NUMBERS.length; i++) {
    sPointList.append(i + 48, NUMBERS[i]);
    }
    // blank
    addChar(' ', new float[]{});
    // -
    addChar('-', new float[]{
    0, 36, 47, 36
    });
    // .
    addChar('.', new float[]{
    24, 60, 24, 72
    });
    }
    
    public static void addChar(char c, float[] points) {
    sPointList.append(c, points);
    }
    
    public static ArrayList&lt;float[]&gt; getPath(String str) {
    return getPath(str, 1, 14);
    }
    
    /**
    * @param str
    * @param scale
    * @param gapBetweenLetter
    * @return ArrayList of float[] {x1, y1, x2, y2}
    */
    public static ArrayList&lt;float[]&gt; getPath(String str, float scale, int gapBetweenLetter) {
    ArrayList&lt;float[]&gt; list = new ArrayList&lt;float[]&gt;();
    float offsetForWidth = 0;
    for (int i = 0; i &lt; str.length(); i++) {
    int pos = str.charAt(i);
    int key = sPointList.indexOfKey(pos);
    if (key == -1) {
    continue;
    }
    float[] points = sPointList.get(pos);
    int pointCount = points.length / 4;
    
    for (int j = 0; j &lt; pointCount; j++) {
    float[] line = new float[4];
    for (int k = 0; k &lt; 4; k++) {
    float l = points[j * 4 + k];
    // x
    if (k % 2 == 0) {
    line[k] = (l + offsetForWidth) * scale;
    }
    // y
    else {
    line[k] = l * scale;
    }
    }
    list.add(line);
    }
    offsetForWidth += 57 + gapBetweenLetter;
    }
    return list;
    }
    }

对数字和坐标敏感的话,看到上面这几个数组应该就能理解这部分的作用.
这里将 数字 和 字母 分解成 对应笔画的 Path , 每2个数字 作为一个坐标点
每一行的2个坐标点作为一个笔画

所以,该库不支持 这里定义之外的 其它字符 比如汉字 某些特殊符号之类的...
虽然提供了一个方法让你以xml的形式的自定义path进来...但是需要自定义path还是算了...

作为库分析来说...其实看到这里就已经猜到后面的实现方式了......

顺藤摸瓜在 StoreHouseHeader里找到了对 StoreHousePath 的调用
这里先把path封装成了 StoreHouseBarItem 对象方便处理
    
    @Override
    public void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        float progress = mProgress;
        int c1 = canvas.save();
        int len = mItemList.size();

        for (int i = 0; i &lt; len; i++) {

            canvas.save();
            StoreHouseBarItem storeHouseBarItem = mItemList.get(i);
            float offsetX = mOffsetX + storeHouseBarItem.midPoint.x;
            float offsetY = mOffsetY + storeHouseBarItem.midPoint.y;

            if (mIsInLoading) {
                storeHouseBarItem.getTransformation(getDrawingTime(), mTransformation);
                canvas.translate(offsetX, offsetY);
            } else {

                if (progress == 0) {
                    storeHouseBarItem.resetPosition(horizontalRandomness);
                    continue;
                }

                float startPadding = (1 - internalAnimationFactor) * i / len;
                float endPadding = 1 - internalAnimationFactor - startPadding;

                // done
                if (progress == 1 || progress &gt;= 1 - endPadding) {
                    canvas.translate(offsetX, offsetY);
                    storeHouseBarItem.setAlpha(mBarDarkAlpha);
                } else {
                    float realProgress;
                    if (progress &lt;= startPadding) {
                        realProgress = 0;
                    } else {
                        realProgress = Math.min(1, (progress - startPadding) / internalAnimationFactor);
                    }
                    offsetX += storeHouseBarItem.translationX * (1 - realProgress);
                    offsetY += -mDropHeight * (1 - realProgress);
                    Matrix matrix = new Matrix();
                    matrix.postRotate(360 * realProgress);
                    matrix.postScale(realProgress, realProgress);
                    matrix.postTranslate(offsetX, offsetY);
                    storeHouseBarItem.setAlpha(mBarDarkAlpha * realProgress);
                    canvas.concat(matrix);
                }
            }
            storeHouseBarItem.draw(canvas);
            canvas.restore();
        }
        if (mIsInLoading) {
            invalidate();
        }
        canvas.restoreToCount(c1);
    }
    
    
    //StoreHouseBarItem中对应的draw方法
    public void draw(Canvas canvas) {
        canvas.drawLine(mCStartPoint.x, mCStartPoint.y, mCEndPoint.x, mCEndPoint.y, mPaint);
    }
    
 
 

根据startPadding 和 progress的值对每个笔画进行平移 旋转 缩放 以及 alpha值的调整.

注意这里startPadding实际上是通过笔画在数组中的位置计算出来的...

所以简单来看 每一个笔画的 realProgress 是不同的
realProgress 受 笔画在数组中的位置 i 和 progress 影响

---

剩下个高光效果直接用Shader渲染的..不分析了.....