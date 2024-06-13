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
        ```
        yum install -y davfs2
        ```
    2. 掛載WebDAV磁碟  
        ```
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

## Week 5 (2024/03/20)

### 通配符/萬用字元 (Wildcard character)

* **通配符**用來批配檔案名稱，**正則表達式**用來匹配內容

| 通配符 | 作用 |
| :--- | :--- |
| `?` | 匹配一個任意字元 |
| `*` | 匹配零個或一個或多個任意字元 |
| `[]` | 匹配單個範圍內的任意字元 |

* ex:  
    ![](source/linux0320-1.png)

* 參考資料：[通配符| Linux基础概要](https://abcfy2.gitbooks.io/linux_basic/content/first_sense_for_linux/command_learning/wildcard.html)

### 正則表達式/正規表示式 (Regular Expression / RE)

* ex: 找出系統的帳號  
    ```
    grep -n nologin$ /etc/passwd
    ```
    ![](source/linux0320-2.png)

* 參考資料：[linux 正则表达式详解](https://cloud.tencent.com/developer/article/1683785), [正規表示式-維基百科](https://zh.wikipedia.org/wiki/正则表达式)

### linux 三劍客： `awk`, `grep`, `sed` - 2

#### grep

看[上面](#正則表達式正規表示式-regular-expression--re)

#### sed

取代用法類似於`vim`的搜尋並取代字串  

* 參考資料：[Linux文字三劍客超詳細教學---grep、sed、awk](https://tw511.com/a/01/11537.html)

#### awk

* 參考資料：[【CHT智能稅務案】教學筆記 03：awk 指令介紹](https://hackmd.io/@cht-fia-project/ryuiv7O2Y)

#### 練習

Windows Powershell 中的 `Get-Process`指令輸出格式為：  
![](source/linux0320-4.png)  

* 先用`sed`得到`Get-Process`指令的目標欄位位置（`process name`和`pid`），再用`grep`和`awk`指令找到`process name`為 **pythonw** 的 `pid`  
    ![](source/linux0320-3.png)

* 更簡單的方法：  
    ![](source/linux0320-5.png)  

## Week 6 (2024/03/27)
### Docker
#### 安裝最新版：  
##### Uninstall old versions:  
```
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-commom \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```  

##### Install from rpm repository:  
> Set up the repository
```
sudo yum install -y yum-utiils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```  

> Install Docker Engine
```
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```  

#### 開啟Docker服務
```
sudo systemctl enable docker
sudo systemctl start docker
```  

#### 測試是否安裝成功
```
sudo docker run hello-world
```

#### Docker 常用指令
* **Docker 指令都要在管理員權限下執行**

* 下載image:  
    ```
    docker pull [user_name/]{repo_name}[:tag]
    ```

* 完整的Docker image名稱格式：  
    `registry_name/user_name/repo_name:tag`  
    預設的`Registry`是官方的`Docker Hub`  
    官方的映像檔（如`apache`、`ubuntu`等）的名稱格式通常只會有`repo_name`和`tag`

* 建立並執行container
    ```
    docker run {image name}
    ```

* 執行已建立的container
    ```
    docker start {container_name}
    ```

* 進入可互動的container
    ```
    docker attach {container name}
    ```

* 離開互動中的container  
    `Ctrl + p` -> `Ctrl + q`

* 列出執行中的container
    ```
    docker ps [-a] [-q]
    ```
    * `-a`：列出未執行的container
    * `-q`：只列出container id

* 停止執行中的container
    ```
    docker stop {container id or name}
    ```  
    停止所有執行中的container  
    ```
    docker stop `docker ps -q`
    ```

* 將容器commit為image  
    ```
    docker commit {container name or id} {image name:tag}
    ```  
    ex:  
    ```
    docker commit mybusybox1 mybusybox:test
    ```

* 參考資料
  * [[Day6]-新手學Docker 0x2](https://ithelp.ithome.com.tw/articles/10241572)
  * [快速建立你的第一個Docker服務](https://joshhu.gitbooks.io/dockercommands/content/Containers/DockerRun.html)
  * [了解 docker run 指令](https://ithelp.ithome.com.tw/articles/10239672)
  * [为什么在 docker 中运行 apache 时总离不开“-D FOREGROUND”？](https://wangyiguo.medium.com/为什么在-docker-中运行-apache-时总离不开-d-foreground-76dcd8c86f17)
 
## Week 7 (2024/04/04)
***春假***

## Week 8 (2024/04/10)
### Docker 續

* 建立並執行container  
    ```
    docker run [-i] [-t] [-d] [--name] [--rm] [-p] [-v] {image} [command] [args]
    ```
    * `-i`：`--interactive`，開啟標準輸入。
    * `-t`：連接終端機。
    * `-d`：背景執行。
    * `--name`：指定container名稱。
    * `--rm`：退出container後自動刪除。
    * `-p`：連接埠號。
    * `-v`：掛載路徑。  
    * `-e`：將環境變數傳入容器
    

    * ex:
        ```
        docker run -d --name www1 --rm -p 8000:80 -v /web:/var/www/html centos7:httpd2 /usr/sbin/apachectl -DFOREGROUND
        ```
        > 建立並啟動`centos7:httpd2`，將容器取名為`www1`，背景執行，關閉後自動刪除，連接主機`8000 port`到容器的`80 port`，掛載主機的`/web`路徑到容器的`/var/www/html`路徑，執行`centos7:httpd2`image中的`/usr/sbin/apachectl`（開啟apache伺服器），`-DFOREGROUND`使apache 伺服器在容器內的前景執行，讓容器不會在離開後自動關閉。  

        ![](source/linux0410-1.png)


* 刪除所有container  
  ```
  docker rm `docker ps -aq`
  ```  

* 刪除所有container（包含執行中的）  
  ```
  docker rm -f `docker ps -aq`
  ```  

#### 使用Dockerfile建立image

ex:
```
mkdir testdocker
cd testdocker
touch index.html
echo "Hello World!" > index.html
vim Dockerfile
```
Dockerfile:
```
FROM centos:centos7
RUN yum -y install httpd
EXPOSE 80
ADD index.html /var/www/html/
```

```
docker build -t centos7:httpd .
```
> 指令格式：  
> docker build -t {image名稱:tag} {build執行位置}

執行建立的image:
```
docker run -itd -p 8080:80 centos7:httpd /usr/sbin/apachectl -DFOREGROUND
```

測試：
```
curl 127.0.0.1:8080/index.html
```
有抓到`hello 2024`表示成功

![](./source/linux0410-2.png)

#### Dockerfile CMD(可覆蓋)、ENTRYPOINT(不可覆蓋)

```
FROM centos:centos7
RUN yum -y install httpd
EXPOSE 80
ADD index.html /var/www/html/
CMD ["/usr/sbin/apachectl", "-DFOREGROUND"]
```
建立：
```
docker build -t centos7:httpd2 .
```
執行：
```
docker run -itd -p 8081:80 centos7:httpd2
```
測試：
```
curl 127.0.0.1:8081/index.html
```

#### 參考資料
[Docker Dockerfile | 菜鸟教程](https://www.runoob.com/docker/docker-dockerfile.html)

#### Docker 擴容
docker啟動腳本 `testweb.sh`：
```bash
#!/usr/bin/bash

for i in {1..5}
do
    portno=`expr 8000 + $i`
    docker run -d -p $portno:80 -v /webdata:/var/www/html centos7:httpd2
done
```

```
chmod +x testweb.sh
```

* 執行
```
./testweb.sh
```
![](./source/linux0410-3.png)

## Week 9 (2024/04/17)
### docker 備份和搬移

#### docker save & docker load

```
docker save {image name[:tag name]} > {filename}.tar
```

ex:
```
docker save centos7:httpd2 > test.tar
```

用`scp`把檔案複製到另一臺虛擬機
```
scp ./test.tar root@192.168.241.101:/tmp
```

在另一臺虛擬機載入備份
```
docker load < {filename}.tar
```

ex:
```
docker load < test.tar
```  
![](./source/linux0417-1.png)

#### upload to docker hub

**先在docker hub註冊帳號，再到虛擬機裡登入**
```
docker login
```

以`busybox`為例操作：
```
docker pull busybox
```

```
docker run -it --name test1 busybox /bin/sh
```

在container中建立一些檔案作為修改：
```
/ # cd /tmp
/tmp # touch 1 2 3 4 5
/tmp # ls
1 2 3 4 5
/tmp # exit
```

使用`docker commit`建立image（dallas145是docker hub的username）：
```
docker commit test1 dallas145/mybusybox:0.1
```

> 補充：`docker tag`可為image產生別名  
> ex: `docker tag busybox mybusybox`

使用`docker push`將image推送至docker hub
```
docker push dallas145/mybusybox:0.1
```

完成後到docker hub查看
![](./source/linux0417-2.png)

從docker hub下載：
```
docker pull dallas145/mybusybox:0.1
```

**補充：[用 Harbor 架設私有 Docker 倉庫. 用 Harbor 架設一個僅供公司內網存取的 Docker Registry | by 被蛇咬到的魯卡 | Starbugs Weekly 星巴哥技術專欄 | Medium](https://medium.com/starbugs/用-harbor-架設私有-docker-倉庫-9e7eb2bbf769)**

### load balancer
使用腳本建立伺服器：
```bash
for i in {1..5}
do
    mkdir -p /web$i
    echo $i > /web$i/hi.htm
    portno=`expr 8000 + $i`
    docker run -d -p $portno:80 -v /web$i:/var/www/html centos7:httpd2 /usr/sbin/apachectl -DFOREGROUND
done
```  
![](./source/linux0417-3.png)

`haproxy`：
```
mkdir haproxy
cd haproxy
vim haproxy.cfg
```

haproxy.cfg:
```
defaults
  mode http
  timeout client 10s
  timeout connect 5s
  timeout server 10s
  timeout http-request 10s

frontend myfrontend
  bind 0.0.0.0:8080
  default_backend myservers

backend myservers
  balance roundrobin
  server server1 127.0.0.1:8001
  server server2 127.0.0.1:8002
  server server3 127.0.0.1:8003
  server server4 127.0.0.1:8004
  server server5 127.0.0.1:8005
```

執行haproxy：
```
docker run -d -p 8080:8080 --name my-haproxy -v /root/hptest:/usr/local/etc/haproxy:ro haproxy
```

測試：
![](./source/linux0417-4.png)

### docker network
查看docker網路：
```
docker network ls
```

查看docker網路詳細資訊：
```
docker network inspect {NETWORK ID}
```

建立docker network：
```
docker network create -d {type} {name}
```

**補充：[Docker四种网络模式（Bridge，Host，Container，None） - 大数据老司机 - 博客园](https://www.cnblogs.com/liugp/p/16328904.html)**  
* 用預設的`docker 0 bridge`網路，container之間只能用ip互連
* 用自己建立的`bridge`網路，container之間可以用ip或name互連

## Week 10 (2024/04/24)
### docker network
使用預設的 `docker 0` 網路，容器間只能使用 `ip` 連接；使用新增的網路，可使用 `container name` 連接。

### docker-compose
* install:
    
```
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose && docker-compose --version
```

![](./source/linux0424-1.png)

#### 實驗
使用兩台虛擬機，分別架設`MySQL`、`php`

##### 建立docker network
```
docker network create -d bridge mybr
```

##### MySQL
```
docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.24
```

啟動成功後，建立資料表與資料

```sql
/* 顯示目前有的資料庫 */
show databases;   
/* 創建資料庫 */
create database testdb;   
/*  使用資料庫 */
use testdb;  
/* 創建資料表 */
create table addrbook(name varchar(50) not null, phone char(10));
/* 加入資料 */
insert into addrbook(name, phone) values ("tom", "0912345678");
insert into addrbook(name, phone) values ("mary", "0987654321");
/* 選擇資料 */
select name,phone from addrbook;
/* */
update addrbook set phone="0987465123" 
```

![](./source/linux0424-2.png)

##### php
test.php:
```php
<?php
$servername="mydb";
$username="root";    
$password="123456";
$dbname="testdb";

$conn = new mysqli($servername, $username, $password, $dbname);

if($conn->connect_error){
    die("connection failed: " . $conn->connect_error);
}
else{
    echo "connect OK!" . "<br>";
}

$sql="select name,phone from addrbook";
$result=$conn->query($sql);

if($result->num_rows>0){
    while($row=$result->fetch_assoc()){
        echo "name: " . $row["name"] . "\tphone: " . $row["phone"] . "<br>";
    }
} else {
    echo "0 record";
}
?>
    
```
執行
```
docker run -d -p 8080:8080 --name my-apache-php-app --network mybr -v "/root/test-docker/php-code":/var/www/html php:7.2-apache
```

使用 `curl 127.0.0.1:8080/test.php` 測試會發現錯誤：

![](./source/linux0424-3.jpeg)

* 解法：
進入 `php` container 並安裝 `mysqli` 套件：

![](./source/linux0424-4.png)

##### 成功結果
![](./source/linux0424-5.png)

### docker volume

[Docker 實戰系列（三）：使用 Volume 保存容器內的數據 | by Larry Lu | Larry・Blog](https://larrylu.blog/using-volumn-to-persist-data-in-container-a3640cc92ce4)

### docker namespace

[理解Docker容器网络之Linux Network Namespace | Tony Bai](https://tonybai.com/2017/01/11/understanding-linux-network-namespace-for-docker-network/)

#### 練習
![](./source/linux0424-6.png)

> 0424 done

## Week ? (2024/05/15)
### Docker swawrm



-----

[linuxModuleManagement]: https://blog.csdn.net/yangjizhen1533/article/details/112239092
[linuxModuleManagement1]: https://mingyi-ulinux.blogspot.com/2009/01/insmod-modprobe-rmmod.html
