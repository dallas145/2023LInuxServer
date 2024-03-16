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

## Week 2 (2024/02/28)

***放假***

## Week 3 (2024/03/06)

### 網頁翻譯擴充元件

* [沈浸式翻譯：雙語對照網頁翻譯＆PDF文檔翻譯](https://chromewebstore.google.com/detail/immersive-translate-web-p/bpoadfkcbjbfhfodiogcnhhhpibjhbnh?utm_source=ext_app_menu)

### Nmap

使用Nmap掃描網路

* [Nmap入門教學](https://medium.com/璿的筆記本/nmap-入門教學-36ed094d6ef8)

### 使用Hydra攻擊ssh伺服器

* [Centos7安裝Hydra](https://www.cnblogs.com/ellisonzhang/p/13440614.html)

**後面太麻煩我就沒做了（攻擊的部份）**

### 關閉ssh的帳號密碼登入,讓使用者採用rsa key進行登入

> server -> `centos7-2`, ip: 192.168.241.101  
> client -> `centos7-1`, ip: 192.168.241.100

1. 在client端產生ssh rsa key（如果沒有的話）
    ```
    ssh-keygen
    ```

2. 將client端的公鑰複製到server端  
    * ex:
        ```
        scp ~/.ssh/id_rsa.pub user@192.168.241.100:/home/user/.ssh/centos7-1.pub
        ```

3. 在server端更改`authorized_keys`檔案  
    * ex:
        ```
        cat .ssh/centos7-1.pub >> authorized_keys
        ```

4. 關閉ssh伺服器的密碼驗證功能（server端）
    * 修改`/etc/ssh/sshd_config`，其中的：
        ```
        PasswordAuthentication yes
        ```
        將`yes`改為`no`  
        存檔後重啟sshd  
        ```
        systemctl restart sshd
        ```

* 若**無法登入**：
    1. 檢查`selinux` and `firewalld`
    2. 檢查`.ssh`資料夾及`authorized_keys`檔案權限
        * `.ssh` 權限須為`700`
        * `authorized_keys`權限須為`600`
    3. 檢查server端的家目錄權限，須為`755`
        * ex:  
            ```
            drwxr-xr-x. 34 user user 4096 Mar 15 16:18 /home/user
            ```

* 參考資料：[SSH Authentication Refused: Bad Ownership or Modes for Directory](https://chemicloud.com/kb/article/ssh-authentication-refused-bad-ownership-or-modes-for-directory/)

#### 練習

使用`powershell 7`以進行`ssh`連線  
方法相同  
![](source/linux0306.png)

### linux 驅動程式 hello world

1. 建立`hello`資料夾

2. 建立`hello/hello.c`檔案
    ```c
    #include <linux/init.h>
    #include <linux/module.h>

    MODULE_DESCRIPTION("Hello_world");
    MODULE_LICENSE("GPL");

    static int hello_init(void)
    {
            printk(KERN_INFO "Hello world !\n");
                return 0;
    }

    static void hello_exit(void)
    {
            printk(KERN_INFO "ByeBye !\n");
    }

    module_init(hello_init);

    module_exit(hello_exit);
    ```

3. 建立`hello/Makefile`檔案
    ```make
    obj-m += hello.o
    KVERSION := $(shell uname -r)

    all:
        $(MAKE) -C /lib/modules/$(KVERSION)/build M=$(PWD) modules

    clean:
        $(MAKE) -C /lib/modules/$(KVERSION)/build M=$(PWD) clean
    ```

4. 在`hello`資料夾中使用`make`指令

5. 使用`insmod {path/to/hello/hello.ko}`指令載入模組

6. 使用`rmmod hello`移除模組

7. 使用`dmesg`檢查是否成功  
    ![](source/linux0306-2.png)  

* 發現如下圖錯誤  
    ![](source/linux0306-3.png)  
    進入`/usr/src/kernels/`檢查有沒有相應的內核開發工具，如果沒有  
    ```sh
    UNAME=$(uname -r)
    yum install gcc kernel-devel-${UNAME%.*}
    ```

**記得要使用新版gcc**  
* gcc更新指令：  
    ```sh
    yum -y install centos-release-scl
    yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
    scl enable devtoolset-7 bash
    ```  

* 參考資料：
    * [[Linux Kernel] 撰寫簡單 Hello, World module (part 1).](https://blog.wu-boy.com/2010/06/linux-kernel-driver-撰寫簡單-hello-world-module-part-1%2f)
    * [make: \*** /lib/modules/3.10.0-693.el7.x86_64/build: 没有那个文件或目录](https://blog.csdn.net/u012343297/article/details/79141878)

## Week 4 (2023/03/13)

### WebDAV

1. `yum install -y epel-release httpd`
2. `httpd -M | grep dav`  
    確定有這三個模組  
    ```
    dav_module
    dav_fs_module
    dav_lock_module
    ``` 
3. `mkdir /var/www/html/webdav`
4. `cd /var/www/html/webdav`
5. `touch {a..d}.txt`
6. `vim /etc/httpd/conf.d/webdav.conf`
    ```apacheconf
    DavLockDB /var/www/html/DavLock
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/webdav/
        ErrorLog /var/log/httpd/error.log
        CustomLog /var/log/httpd/access.log combined
        Alias /webdav /var/www/html/webdav
        <Directory /var/www/html/webdav>
            DAV On
            #AuthType Basic
            #AuthName "webdav"
            #AuthUserFile /etc/httpd/.htpasswd
            #Require valid-user
        </Directory>
    </VirtualHost>
    ```
7. `chmod -R 755 /var/www/html` 變更資料夾權限  
8. `chown -R apache:apache /var/www/html` 變更資料夾擁有者  
9. `systemctl restart httpd.service` 重啟伺服器  

* 使用Windows檔案總管連線存取  
    1. 在`本機`按右鍵，選擇`連線網路磁碟機...`  
        ![](source/linux0313.png)  
    2. 選擇磁碟機代號並輸入伺服器ip
        ![](source/linux0313-2.png)

* 使用另一台linux虛擬機連線存取  
    1. 安裝`davfs2`  
        ```bash
        yum install -y davfs2
        ```
    2. 掛載WebDAV磁碟  
        ```bash
        mkdir /cloud
        mount -t davfs http://192.168.241.101 /cloud
        ```

* 掛載範例  
    windows:  
    ![](source/linux0313-3.png)  
    
    linux:  
    ![](source/linux0313-4.png)

* 參考資料：  
    [學姐筆記](https://hackmd.io/@jenny126/By8OS6Fas/%2FiiMobAxzR3--0JSa_JNwVw)  
    [Linux将WebDAV为本地磁盘](https://blog.lincloud.pro/archives/36.html)

### linux 三劍客： `awk`, `grep`, `sed`

#### sed

* 參考資料：  
    [學姐筆記](https://hackmd.io/@jenny126/By8OS6Fas/%2FiiMobAxzR3--0JSa_JNwVw#Windows-Sedstream-editor)  
    [Linux sed 字串取代用法與範例](https://shengyu7697.github.io/linux-sed/)  
    [Linux 以 sed 指令搜尋、取代檔案內容教學與範例](https://officeguide.cc/linux-sed-find-and-replace-text-in-file-tutorial-examples/)  
    [Linux 指令 SED 用法教學、取代範例、詳解](https://terryl.in/zh/linux-sed-command/)  

-----

[linuxModuleManagement]: https://blog.csdn.net/yangjizhen1533/article/details/112239092
[linuxModuleManagement1]: https://mingyi-ulinux.blogspot.com/2009/01/insmod-modprobe-rmmod.html
