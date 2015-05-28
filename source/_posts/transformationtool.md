title: 'TransformationTool '
id: 190
categories:
  - Blog
date: 2015-01-14 05:42:24
tags:
---


既然想到脚本...翻了下...把我原来写的dp百分比换算脚本翻了出来...  反正java也可以用来写写小工具嘛

作用是把所有 dp/sp 单位进行百分比换算....例如在 720X1280 & density = 2 时

1dp = 2px

在 480*800 &amp; density =1.5 时

2/720*480/1.5 = 0.8888889 dp

目的是保证所有元素在不同分辨率上保持比例完全一致

但是大家都知道 android本身设计逻辑不是这样的......dp是物理尺寸相关的单位...所以最后我把UX说服了没用这么个奇怪的方案....不过脚本当时已经写好了....记录一下免得丢了..



至于为什么输入数据是dp单位是因为我写这个脚本的时候界面已经用dp适配好了....否则就该严格按照px先写1920*1080分辨率的...然后用脚本生成所有其它分辨率的.....

    import java.io.BufferedReader;
    import java.io.File;
    import java.io.FileReader;
    import java.io.FileWriter;
    import java.io.IOException;
    import java.util.HashMap;
    import java.util.regex.Matcher;
    import java.util.regex.Pattern;
    
    /**
     * 
     * @author Peter.Ding
     * 
     */
    public class TransformationTool {
        private static final String resourceFolder = "C:\\Users\\peter.ding\\Desktop\\src_folder";
        private static final String targetFolder = "C:\\Users\\peter.ding\\Desktop\\target_folder";
    
        private static final Pattern PATTERN_DP = Pattern.compile("[0-9]{1,3}dp");
        private static final Pattern PATTERN_SP = Pattern.compile("[0-9]{1,3}sp");
    
        private static final String SOLUTION_480X800 = "480x800";
        private static final String SOLUTION_720X1280 = "720x1280";
        private static final String SOLUTION_1080x1920 = "1080x1920";
    
        private static HashMap&lt;String, Float&gt; densityMap = new HashMap&lt;&gt;();
    
        private static String SRC_SOLUTION = SOLUTION_720X1280;
        private static String TARGET_SOLUTION = SOLUTION_480X800;
    
        private static int SRC_SOLUTION_WIDTH;
        private static int TARGET_SOLUTION_WIDTH;
    
        public static void main(String[] args) throws IOException {
    
            SRC_SOLUTION_WIDTH = Integer.parseInt(SRC_SOLUTION.split("x")[0]);
            TARGET_SOLUTION_WIDTH = Integer.parseInt(TARGET_SOLUTION.split("x")[0]);
    
            densityMap.put(SOLUTION_720X1280, 2f);
            densityMap.put(SOLUTION_480X800, 1.5f);
            densityMap.put(SOLUTION_1080x1920, 3.0f);
    
            File directory = new File(resourceFolder);
            File[] files = directory.listFiles();
    
            for (File srcFile : files) {
                FileReader fr = new FileReader(srcFile);
                BufferedReader br = new BufferedReader(fr);
    
                String content = "";
                String line = null;
                while ((line = br.readLine()) != null) {
                    content += line;
                }
                br.close();
                fr.close();
    
                Matcher matcher = PATTERN_DP.matcher(content);
                while (matcher.find()) {
                    String srcString = matcher.group();
                    int originDp = Integer.parseInt(srcString.substring(0, srcString.length() - 2));
                    String targetString = originDp * densityMap.get(SRC_SOLUTION) * TARGET_SOLUTION_WIDTH / SRC_SOLUTION_WIDTH
                            / densityMap.get(TARGET_SOLUTION) + "dp";
                    content = content.replace(srcString, targetString);
                }
    
                Matcher matcher_sp = PATTERN_SP.matcher(content);
                while (matcher_sp.find()) {
                    String srcString = matcher_sp.group();
                    int originDp = Integer.parseInt(srcString.substring(0, srcString.length() - 2));
                    String targetString = originDp * densityMap.get(SRC_SOLUTION) * TARGET_SOLUTION_WIDTH / SRC_SOLUTION_WIDTH
                            / densityMap.get(TARGET_SOLUTION) + "sp";
                    content = content.replace(srcString, targetString);
                }
    
                File targetFile = new File(targetFolder + File.separatorChar + srcFile.getName());
                FileWriter fw = new FileWriter(targetFile);
                fw.write(content);
                fw.flush();
                fw.close();
            }
    
        }
    }
