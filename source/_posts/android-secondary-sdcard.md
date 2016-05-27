title: Android中如何获取Secondary External Storage路径(android secondary sdcard setting)
date: 2016-05-23 19:01:23
tags: android
---

前置条件

当Android同时拥有内置存储区和外置SD卡时,外置存储卡会被识别为 `Secondary External Storage`.
本文讨论的是该情况下 `Secondart External Storage`的识别和读写问题,如果你没有SD卡,或没有内置存储区的设备,只要简单使用`Environment.getExternalStorageDirectory()` 就好了


- 如何获得外置SD卡路径

`Environment.getExternalStorageDirectory()` 获得的只是系统逻辑上的存储区,在目前的大部分手机上,返回的其实是内置存储区路径
Google也没有提供一个明确的方法来或者外置存储卡路径.

1.对于Android 4.4 及以上设备使用 `Application.getExternalFilesDirs(null)`,然后返回的数组中就包含内置存储区路径和SD卡路径,简单判断一下就好了.
需要注意的是,返回的路径应当是 /storage/sdcard1/Android/data/your.packagename 的一个路径,在SD卡内,只有该路径下应用是拥有写权限的,其它路径下你都只有只读权限.
相关文档请参考这个:
https://developer.android.com/reference/android/content/ContextWrapper.html#getExternalFilesDirs(java.lang.String)

简单来说,4.4以后你就不能在SD卡上瞎几把放东西了,只能存储在系统给你指定的地方


 
2. 对于低于4.4版本的设备,只有一些歪门邪道的方法

一种方式是可以通过`mount`命令的返回来搜索SD卡路径,不是特别靠谱
相关代码可以参考 http://stackoverflow.com/questions/5694933/find-an-external-sd-card-location/9406276#9406276
但是请注意,我看到过4~5种不一样的匹配方法,感觉和manufacture关系很大,国内瞎几把定制ROM不确定性更大,请配合`File.canRead()`验证使用.

代码贴一下:

    import java.io.BufferedReader;
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FilenameFilter;
	import java.io.IOException;
	import java.io.InputStreamReader;

	import android.util.Log;


	/**
	 * @author ajeet
	 *05-Dec-2014  2014
	 *
	 */
	public class StorageUtil {

	    public boolean isRemovebleSDCardMounted() {
	        File file = new File("/sys/class/block/");
	        File[] files = file.listFiles(new MmcblkFilter("mmcblk\\d$"));
	        boolean flag = false;
	        for (File mmcfile : files) {
	            File scrfile = new File(mmcfile, "device/scr");
	            if (scrfile.exists()) {
	                flag = true;
	                break;
	            }
	        }
	        return flag;
	    }

	    public String getRemovebleSDCardPath() throws IOException {
	        String sdpath = null;
	        File file = new File("/sys/class/block/");
	        File[] files = file.listFiles(new MmcblkFilter("mmcblk\\d$"));
	        String sdcardDevfile = null;
	        for (File mmcfile : files) {
	            Log.d("SDCARD", mmcfile.getAbsolutePath());
	            File scrfile = new File(mmcfile, "device/scr");
	            if (scrfile.exists()) {
	                sdcardDevfile = mmcfile.getName();
	                Log.d("SDCARD", mmcfile.getName());
	                break;
	            }
	        }
	        if (sdcardDevfile == null) {
	            return null;
	        }
	        FileInputStream is;
	        BufferedReader reader;

	        files = file.listFiles(new MmcblkFilter(sdcardDevfile + "p\\d+"));
	        String deviceName = null;
	        if (files.length > 0) {
	            Log.d("SDCARD", files[0].getAbsolutePath());
	            File devfile = new File(files[0], "dev");
	            if (devfile.exists()) {
	                FileInputStream fis = new FileInputStream(devfile);
	                reader = new BufferedReader(new InputStreamReader(fis));
	                String line = reader.readLine();
	                deviceName = line;
	            }
	            Log.d("SDCARD", "" + deviceName);
	            if (deviceName == null) {
	                return null;
	            }
	            Log.d("SDCARD", deviceName);

	            final File mountFile = new File("/proc/self/mountinfo");

	            if (mountFile.exists()) {
	                is = new FileInputStream(mountFile);
	                reader = new BufferedReader(new InputStreamReader(is));
	                String line = null;
	                while ((line = reader.readLine()) != null) {
	                    // Log.d("SDCARD", line);
	                    // line = reader.readLine();
	                    // Log.d("SDCARD", line);
	                    String[] mPonts = line.split("\\s+");
	                    if (mPonts.length > 6) {
	                        if (mPonts[2].trim().equalsIgnoreCase(deviceName)) {
	                            if (mPonts[4].contains(".android_secure")
	                                    || mPonts[4].contains("asec")) {
	                                continue;
	                            }
	                            sdpath = mPonts[4];
	                            Log.d("SDCARD", mPonts[4]);

	                        }
	                    }

	                }
	            }

	        }

	        return sdpath;
	    }

	    static class MmcblkFilter implements FilenameFilter {
	        private String pattern;

	        public MmcblkFilter(String pattern) {
	            this.pattern = pattern;

	        }

	        @Override
	        public boolean accept(File dir, String filename) {
	            if (filename.matches(pattern)) {
	                return true;
	            }
	            return false;
	        }

	    }

	}

	
以红米1举例,插入SD卡后,mount返回如下

    rootfs / rootfs ro,relatime 0 0
    tmpfs /dev tmpfs rw,seclabel,nosuid,relatime,size=437680k,nr_inodes=109420,mode=755 0 0
    devpts /dev/pts devpts rw,seclabel,relatime,mode=600 0 0
    proc /proc proc rw,relatime 0 0
    sysfs /sys sysfs rw,seclabel,relatime 0 0
    selinuxfs /sys/fs/selinux selinuxfs rw,relatime 0 0
    debugfs /sys/kernel/debug debugfs rw,relatime 0 0
    none /acct cgroup rw,relatime,cpuacct 0 0
    none /sys/fs/cgroup tmpfs rw,seclabel,relatime,size=437680k,nr_inodes=109420,mode=750,gid=1000 0 0
    none /sys/fs/cgroup/memory cgroup rw,relatime,memory 0 0
    tmpfs /mnt/asec tmpfs rw,seclabel,relatime,size=437680k,nr_inodes=109420,mode=755,gid=1000 0 0
    tmpfs /mnt/obb tmpfs rw,seclabel,relatime,size=437680k,nr_inodes=109420,mode=755,gid=1000 0 0
    none /dev/cpuctl cgroup rw,relatime,cpu 0 0
    /dev/block/platform/msm_sdcc.1/by-name/system /system ext4 ro,seclabel,relatime,data=ordered 0 0
    /dev/block/platform/msm_sdcc.1/by-name/userdata /data ext4 rw,seclabel,nosuid,nodev,relatime,discard,noauto_da_alloc,data=ordered 0 0
    /dev/block/platform/msm_sdcc.1/by-name/cache /cache ext4 rw,seclabel,nosuid,nodev,relatime,discard,noauto_da_alloc,data=ordered 0 0
    /dev/block/platform/msm_sdcc.1/by-name/persist /persist ext4 rw,seclabel,nosuid,nodev,relatime,discard,noauto_da_alloc,data=ordered 0 0
    /dev/block/platform/msm_sdcc.1/by-name/modem /firmware vfat ro,relatime,uid=1000,gid=1000,fmask=0337,dmask=0227,codepage=cp437,iocharset=iso8859-1,shortname=lower,errors=remount-ro 0 0
    /dev/fuse /mnt/shell/emulated fuse rw,nosuid,nodev,relatime,user_id=1023,group_id=1023,default_permissions,allow_other 0 0
    /dev/fuse /storage/emulated/legacy fuse rw,nosuid,nodev,relatime,user_id=1023,group_id=1023,default_permissions,allow_other 0 0
    /dev/block/vold/179:65 /mnt/media_rw/sdcard1 vfat rw,dirsync,nosuid,nodev,noexec,relatime,uid=1023,gid=1023,fmask=0007,dmask=0007,allow_utime=0020,codepage=cp437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro 0 0
    /dev/fuse /storage/sdcard1 fuse rw,nosuid,nodev,relatime,user_id=1023,group_id=1023,default_permissions,allow_other 0 0

可以看到,以上面的字符串匹配或者stackoverflow上其它几种验证方法(asec / vfat / void),都匹配不到最后那条记录

另一种方法:
读取 `/system/etc/vold.fstab`

示例代码如下

        try {
                Scanner scanner = new Scanner(new File("/system/etc/vold.fstab"));
                while (scanner.hasNext()) {
                    String line = scanner.nextLine();
                    if (line.startsWith("dev_mount")) {
                        String[] lineElements = line.split(" ");
                        String element = lineElements[2];
    
                        if (element.contains(":"))
                            element = element.substring(0, element.indexOf(":"));
    
                        if (element.contains("usb"))
                            continue;
    
                        // don't add the default vold path
                        // it's already in the list.
                        if (!out.contains(element))
                            //这里就是你要的SD卡路径了
                            out.add(element);
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            

还是以红米1为例,贴一下'/system/etc/vold.fstab'的内容

    # Copyright (c) 2012, The Linux Foundation. All rights reserved.
    #
    # Redistribution and use in source and binary forms, with or without
    # modification, are permitted provided that the following conditions are
    # met:
    #     * Redistributions of source code must retain the above copyright
    #       notice, this list of conditions and the following disclaimer.
    #     * Redistributions in binary form must reproduce the above
    #       copyright notice, this list of conditions and the following
    #       disclaimer in the documentation and/or other materials provided
    #       with the distribution.
    #     * Neither the name of The Linux Foundation nor the names of its
    #       contributors may be used to endorse or promote products derived
    #       from this software without specific prior written permission.
    #
    # THIS SOFTWARE IS PROVIDED "AS IS" AND ANY EXPRESS OR IMPLIED
    # WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
    # MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT
    # ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS
    # BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
    # CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
    # SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR
    # BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
    # WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
    # OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
    # IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
    
    dev_mount sdcard /storage/sdcard1 auto /devices/msm_sdcc.2/mmc_host

保险起见,我目前是以上2种方法同时使用,再用`file.canRead()`检查是否存在

