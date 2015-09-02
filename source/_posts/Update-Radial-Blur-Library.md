title: Update Radial Blur Library
date: 2015-09-03 02:24:42
tags:
---
因为看了@drakeet的FingerTransparentView想起来去年自己也写过一个类似的动态模糊(Motion Blur)的库

于是翻出来看了一下,花了点时间迁移到了Android Studio

在看的时候自己发现一个问题,这TM根本不是 `Motion Blur` ,我自己写的代码实际上实现的效果是 径向模糊(`Radial Blur`)

嘛...我还是要说一下 MotionBlur和RadialBlur看起来很像,但实际上是不一样的,我之前也是误解了,后来仔细看了些效果图,下面提到的几个库算法实现虽然名字都是叫MotionBlur,但是实现的效果其实都是RadialBlur

Demo效果

![Examples list](https://raw.githubusercontent.com/dkmeteor/MotionBlur-Android/master/release/blur_center.png)

因为涉及到重复draw,当时我写得时候就记得性能很一般,这次迁移顺便也打算看看这个问题

核心代码其实很简单

      public static Bitmap doRadialBlur(Bitmap src, int centerX, int centerY, float factor, int times) {

        Bitmap dst = Bitmap.createBitmap(src.getWidth(), src.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(dst);
        Matrix matrix = new Matrix();
        Paint paint = new Paint();
        canvas.drawBitmap(src, matrix, paint);
        paint.setAlpha(51);

        for (int i = 0; i < times; i++) {
            matrix.setScale(i * factor + 1, i * factor + 1, centerX, centerY);
            canvas.drawBitmap(src, matrix, paint);
        }
        return dst;
    }
    
    
为了实现效果需要反复的Scale并重复的draw同一张Bitmap,性能当然狠捉急.
测试时发现渲染一次需要 1400ms 左右.

图片是2400*2400的,换成800*800的以后,减少到200ms左右,作为一个滤镜来讲,尚可接受吧,一时间也想不到什么优化的方法.
优化到16ms以下可以做成实时滤镜,否则的话,似乎意义不大.

想起jhlab也有 径向模糊 的滤镜算法,于是拔出来代码看了下


     public BufferedImage filter( BufferedImage src, BufferedImage dst ) {
        if ( dst == null )
            dst = createCompatibleDestImage( src, null );
        BufferedImage tsrc = src;
        float cx = (float)src.getWidth() * centreX;
        float cy = (float)src.getHeight() * centreY;
        float imageRadius = (float)Math.sqrt( cx*cx + cy*cy );
        float translateX = (float)(distance * Math.cos( angle ));
        float translateY = (float)(distance * -Math.sin( angle ));
        float scale = zoom;
        float rotate = rotation;
        float maxDistance = distance + Math.abs(rotation*imageRadius) + zoom*imageRadius;
        int steps = log2((int)maxDistance);

    	translateX /= maxDistance;
		translateY /= maxDistance;
		scale /= maxDistance;
		rotate /= maxDistance;
		
        if ( steps == 0 ) {
            Graphics2D g = dst.createGraphics();
            g.drawRenderedImage( src, null );
            g.dispose();
            return dst;
        }
        
        BufferedImage tmp = createCompatibleDestImage( src, null );
        for ( int i = 0; i < steps; i++ ) {
            Graphics2D g = tmp.createGraphics();
            g.drawImage( tsrc, null, null );
			g.setRenderingHint( RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON );
			g.setRenderingHint( RenderingHints.KEY_INTERPOLATION, RenderingHints.VALUE_INTERPOLATION_BILINEAR );
			g.setComposite( AlphaComposite.getInstance( AlphaComposite.SRC_OVER, 0.5f ) );

            g.translate( cx+translateX, cy+translateY );
            g.scale( 1.0001+scale, 1.0001+scale );  // The .0001 works round a bug on Windows where drawImage throws an ArrayIndexOutofBoundException
            if ( rotation != 0 )
                g.rotate( rotate );
            g.translate( -cx, -cy );

            g.drawImage( dst, null, null );
            g.dispose();
            BufferedImage ti = dst;
            dst = tmp;
            tmp = ti;
            tsrc = dst;

            translateX *= 2;
            translateY *= 2;
            scale *= 2;
            rotate *= 2;
        }
        return dst;
    }
    
Jhlabs比较古老,BufferedImage是awt里面,做Android的如果不了解的话当Bitmap理解就好了.
可以看到Jhlabs的代码逻辑也没什么区别,一样是通过反复draw image实现的径向模糊效果.

参考意义不大.


另外在Github上搜索了一下,一个叫做AndroidFastImageProcessing的库也支持MotionBlur,其实现机制比较特别

核心代码是这样

    protected String getFragmentShader() {
    	return 
				 "precision mediump float;\n" 
				+"uniform sampler2D "+UNIFORM_TEXTURE0+";\n"  
				+"varying vec2 "+VARYING_TEXCOORD+";\n"	
				+"uniform float "+UNIFORM_TEXELWIDTH+";\n"
				+"uniform float "+UNIFORM_TEXELHEIGHT+";\n"
				
		  		+"void main(){\n"
		  		+"   vec2 step = vec2("+UNIFORM_TEXELWIDTH+", "+UNIFORM_TEXELHEIGHT+");\n"
		  		+"   vec4 fragColour = texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+") * 0.18;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" + step) * 0.15;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" - step) * 0.15;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" + step * 2.0) * 0.12;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" - step * 2.0) * 0.12;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" + step * 3.0) * 0.09;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" - step * 3.0) * 0.09;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" + step * 4.0) * 0.05;\n"
		  		+"   fragColour += texture2D("+UNIFORM_TEXTURE0+", "+VARYING_TEXCOORD+" - step * 4.0) * 0.05;\n"
		  		+"   gl_FragColor = fragColour;\n"
		  		+"}\n";
	}
    
可以看到 这是生成了一段代码

然后对应的执行部分

    GLES20.glShaderSource(fragmentShaderHandle, fragmentShader);
    GLES20.glCompileShader(fragmentShaderHandle);

好吧,其实我压根就不懂OpenGl.大概是把图片作为Shader进行了多次渲染来实现MontionBlur的效果.



这个研究基本上没有结果,往下层去看都是native方法,OpenGl可能是一个可行的优化方法,可惜是我不懂OpenGL,看了一点点实在没什么头绪,可能需要系统的从头学习一下.

---

然后过程中还发现个比较奇特的问题

将项目发布到jcenter以后,打算自己测试一下,但是一直编译不过,提示如下:

    Error:A problem occurred configuring project ':app'.
    > Could not resolve all dependencies for configuration ':app:_debugCompile'.
       > Could not resolve com.dk.image.process.radialblur:library:0.1.0.
         Required by:
             MotionBlur-Android:app:unspecified
          > Could not resolve com.dk.image.process.radialblur:library:0.1.0.
             > Could not get resource 'https://jcenter.bintray.com/com/dk/image/process/radialblur/library/0.1.0/library-0.1.0.pom'.
                > Could not HEAD 'https://jcenter.bintray.com/com/dk/image/process/radialblur/library/0.1.0/library-0.1.0.pom'. Received status code 400 from server: Bad Request

我自己反复确认了几次, https://jcenter.bintray.com/com/dk/image/process/radialblur/library/0.1.0/library-0.1.0.pom 绝对是可以访问的.

然后我一想,可能是代理的问题,但是我确定自己肯定没给Android Studio配过代理,因为我自己用的代理是 红杏,需要全局代理的时候我用的mxvpm需要客户端登陆,根本不会在Android Studio里配置,
不过还是检查了下Android Studio的代理配置,代理配置确实是空的,但是最顶部有一个Warnning提示,提示我JVM里配置了一个Proxy

`mirrors.opencas.cn`

我其实不大理解什么叫JVM PROXY,不过这个代理服务器地址我一看就知道是我在SDK Manager里使用的镜像的地址,我也不知道为什么Android Studio会把SDK Manager里的配置读过来,之前是从来没有过的.
打开SDK Manager,删掉镜像配置,重启Android Studio,恢复了正常,denpendices可以正常sync.

关掉Android Studio,再打开SDK Manager,把镜像重新配回去.重启Android Studio,检查代理设置,正常.

这我就不理解了,什么情况下Android Studio会读取SDK Mananger的代理配置,什么时候不会?

出现问题的版本 Android Studio版本1.3.1 

之前从来没有遇到过,因为我SDK Manager里一直是挂着镜像的地址.




