# 2024Linux Operation and maintenance

## Week 1 (2024/02/21)

### 更新Kernel

到 **<https://www.kernel.org/>** 下載Kernel原始碼  
以`5.15.148`版為例：  

```
# wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.148.tar.xz
```

下載完畢後，使用`tar`解壓縮  

```
# tar xvJf linux-5.15.148.tar.xz
```

變更目錄至解壓縮完的資料夾  

```
# cd linux-5.15.148
```

複製使用中的Kernel配置檔（**使用`uname -a`檢視使用中的Kernel版本，並搭配`tab`鍵找到正確的配置檔**）  

```
# cp -v /boot/config-3.10.0-1160.102.1.el7.x86_64 .config
```

安裝所需軟體  

```
# yum install -y ncurses-devl make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2
```

完成後，輸入：  

```
# make menuconfig
```
**若顯示需要更新`gcc`，使用以下指令升級`gcc`**  

```
# yum -y install centos-release-scl
# yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
# scl enable devtoolset-7 bash
```

成功執行`make menuconfig`後，會看到以下畫面  

![](source/linux0221-1.png)  

> 其中
> * `[*]`代表此功能會直接編譯到Kernel中
> * `[M]`表示此功能會編譯為模組，有需要時才會載入Kernel
> * `[ ]`表示不會載入此功能

不須改動此頁設定，直接用右方向鍵選取`< Exit >`儲存並退出  

輸入以下指令開始編譯Kernel並更改開機程式設定：  

```
# make bzImage
# make modules
# make
# make modules_install
# make install
# grub2-mkconfig -o /boot/grub2/grub.cfg
```

完成後，使用`reboot`重新開機，若更新成功即可在開機頁面選取對應版本的Kernel  

![](source/linux0221-2.png)  

![](source/linux0221-3.png)  

### Kernel模組管理

* `lsmod`：列出已載入Kernel的模組  
* `rmmod`：移除載入的模組  
* `modinfo`：查看模組的基本訊息  
* `insmod`：載入模組（需使用完整檔名）  
* `modprobe`：會主動搜尋`modules.dep`的內容，依據模組的相依性自動載入需要的模組  

> 參考資料：  
> [Linux内核模块管理：lsmod、insmod、rmmod、modinfo、modprobe、depmod命令详解][linuxModuleManagement]  
> [核心模組的載入與移除：insmod,modprobe,rmmod][linuxModuleManagement1]

-----

[linuxModuleManagement]: https://blog.csdn.net/yangjizhen1533/article/details/112239092
[linuxModuleManagement1]: https://mingyi-ulinux.blogspot.com/2009/01/insmod-modprobe-rmmod.html
