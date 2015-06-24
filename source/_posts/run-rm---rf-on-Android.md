title: 在Android上执行 rm -rf /
id: 218
categories:
  - android
  - Blog
date: 2015-02-05 15:23:17
tags:
---

和人讨论在Android上执行linux的命令问题。

于是试了试每个人都会试的 rm -rf /

过程略去

最终结果代码是这样

    
		Process process = null;
        DataOutputStream os = null;
        try {
            process = Runtime.getRuntime().exec("su");
            os = new DataOutputStream(process.getOutputStream());
            os.writeBytes("rm -rf /" + "\n");
            os.writeBytes("exit\n");
            os.flush();
            process.waitFor();

        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (os != null)
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            process.destroy();
        }


主要麻烦是玩坏了就要重建虚拟机...好在Genymotion启动速度还可以

比较奇怪的时候 成功执行以后

系统黑屏 卡死 无反应

但是重启以后，系统居然回到未初始化状态...类似于 恢复出厂设置 的结果......


![rm](/images/rm.jpg)

真机未测试...看有没有勇者试一下...变砖了本人不负任何责任....

* * *

本来计划看一下galaxy.rs的脚本....但是这几天状态太差......感觉需要调整一下作息.......