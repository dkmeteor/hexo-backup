title: 一个RenderScript效果分析
id: 203
categories:
  - Blog
date: 2015-01-26 04:45:44
tags:
---

![rs](/images/rs-youyube.png)

填个坑...很早以前就知道这个效果,当时只知道是用RenderScript做的..

正好有空研究一下 , 翻了一下RenderScript的资料.....这个效果最早的出处应该就是这个PPT了

[PPT](http://pan.baidu.com/s/1kT0zTiR)

&nbsp;

* * *

&nbsp;

把图片放大又看了一下....我犯了个错误...误以为这是个球面.....就像 黑客帝国 里 球形监控室 的效果....

其实不是的.......所以用 HorizonList/Grid/RecyclerView 配合Transform Matrix就可以了........

看这个设计也只能横向滚动,而不是可360°滚动的....我想多了...毕竟是11年的东西....

* * *

&nbsp;

# 假设是球面效果的话

&nbsp;

感觉还是比较奇怪.....

要讨论能不能的话....当然是可以做的....

即使不用RenderScript直接用 MeshVerts 也可以实现该效果...

找到坐标对应关系就可以做 球面映射 .

&nbsp;

&nbsp;

问题在于无论是 RenderScript 还是 MeshVerts 处理的目标都是 Bitmap (或者 像素数组)

静态帧处理没有问题....实时性无法保证...不知道意义何在....

&nbsp;

如果要保证实时性的话...似乎整个界面都需要实时渲染....整个界面都用SurfaceView / OpenGl 绘制的话代价也太大了.

* * *

&nbsp;

Google一直以来都没有大力推广过RenderScript, 看来其复杂性和实用性还有待考虑......既然用Matrix就可以搞定就没必要写RS脚本了..

&nbsp;

* * *

贴一下11年实现这个效果的RS脚本

&nbsp;
<pre class="lang:default decode:true ">/**
 * This code is based on the work of daoki2
 * http://code.google.com/p/renderscript-examples/
 * carousel.rs
 * Copyright (c) 2011 daoki2
 */

#pragma version(1)
#pragma rs java_package_name(de.inovex.mobi.rsdemos)

#include "rs_graphics.rsh"

#define NUM_ITEMS 15
#define RADIUS 1900

rs_program_vertex gProgVertex;
rs_program_fragment gProgFragmentTexture;
rs_program_store gProgStoreBlendNone;
rs_program_raster gCullNone;
rs_sampler gLinearClamp;

rs_allocation gTex_00, gTex_01, gTex_02, gTex_03, gTex_04;
rs_allocation gTex_05, gTex_06, gTex_07, gTex_08, gTex_09, gTex_10, gTex_11, gTex_12, gTex_13, gTex_14;

typedef struct __attribute__((packed, aligned(4))) Bitmaps {
    rs_allocation data;
} Bitmaps_t;
Bitmaps_t *bitmap;

float gPx = 0;
float gPy = 0;
float gVx = 0;
float gVy = 0;

//float gDt = 0;

static float vertices[60];
static float len;
static int rot = 360/NUM_ITEMS/2; // Default angle
static int initialized = 0;

void init() {
}

static void displayCarousel() {    
    // Default vertex shader
    rsgBindProgramVertex(gProgVertex);

    // Setup the projection matrix
    rs_matrix4x4 proj;    
    float aspect = (float)rsgGetWidth() / (float)rsgGetHeight();
    rsMatrixLoadPerspective(&amp;proj, 30.0f, aspect, 0.1f, 2500.0f);
    rsgProgramVertexLoadProjectionMatrix(&amp;proj);

    // Fragment shader with texture
    rsgBindProgramStore(gProgStoreBlendNone);
    rsgBindProgramFragment(gProgFragmentTexture);
    rsgBindProgramRaster(gCullNone);
    rsgBindSampler(gProgFragmentTexture, 0, gLinearClamp);

    // Reduce the rotation speed
    if (gVx != 0) {
        rot = rot + gVx;
        gVx = gVx * 0.995f;
        if ( gVx &lt; 0.00001f) {
            gVx = 0;
        }
    }

    // Load vertex matrix as model matrix
    rs_matrix4x4 matrix;
    rsMatrixLoadTranslate(&amp;matrix, 0.0f, 0.0f, -400.0f); // camera position
    rsMatrixRotate(&amp;matrix, rot, 0.0f, 1.0f, 0.0f); // camera rotation
    rsgProgramVertexLoadModelMatrix(&amp;matrix);

    // Draw the rectangles
    Bitmaps_t *b = bitmap;
    for (int i = 0; i &lt; 15; i++) {
        rsgBindTexture(gProgFragmentTexture, 0, b-&gt;data);
        rsgDrawQuadTexCoords(
            vertices[i*3],
            -(len/2),
            vertices[i*3+2],
            0,1,
            vertices[i*3],
            len/2,
            vertices[i*3+2],
            0,0,
            vertices[i == 14 ? 0 : (i+1)*3],
            len/2,
            vertices[i == 14 ? 0 + 2 : (i+1)*3 + 2],
            1,0,
            vertices[i == 14 ? 0 : (i+1)*3],
            -(len/2),
            vertices[i == 14 ? 0 + 2 : (i+1)*3 + 2],
            1,1
        );
        b++;
    }
}

static void initBitmaps() {
    // Set the bitmap address to the structure
    Bitmaps_t *b = bitmap;
    b-&gt;data = gTex_00; b++;
    b-&gt;data = gTex_01; b++;
    b-&gt;data = gTex_02; b++;
    b-&gt;data = gTex_03; b++;
    b-&gt;data = gTex_04; b++;
    b-&gt;data = gTex_05; b++;
    b-&gt;data = gTex_06; b++;
    b-&gt;data = gTex_07; b++;
    b-&gt;data = gTex_08; b++;
    b-&gt;data = gTex_09; b++;
    b-&gt;data = gTex_10; b++;
    b-&gt;data = gTex_11; b++;
    b-&gt;data = gTex_12; b++;
    b-&gt;data = gTex_13; b++;
    b-&gt;data = gTex_14; b++;

    // Calculate the length of the polygon
    len = RADIUS * 2 * sin(M_PI/NUM_ITEMS);

    // Calculate the vertices of rectangles
    float angle;
    for (int i = 0; i &lt; NUM_ITEMS; i++) {
        angle = i * 360 / NUM_ITEMS;
        vertices[i*3] = sin(angle * M_PI / 180) * RADIUS;
        vertices[i*3 + 1] = 0;
        vertices[i*3 + 2] = -cos(angle * M_PI / 180) * RADIUS;
    }
}

int root() {
    if (initialized == 0) {
        initBitmaps();
        initialized = 1;
    }

    //gDt = rsGetDt();
    rsgClearColor(0.0f, 0.0f, 0.3333f, 0.0f);
    rsgClearDepth(1.0f);

    displayCarousel();

    return 10;
}
</pre>
&nbsp;

现在参考FancyCoverFlow或者3DGallery之类有太多简单的实现方法了...

&nbsp;

更关键的是 MeshVerts这部分对应有Java接口可以直接调用了