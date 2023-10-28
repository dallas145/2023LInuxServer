# NQUCSIE 2023 Linux 2
# Linux 伺服器架設

### (2023/09/12)
#### 更改hostname
```
# hostnamectl set-hostname {the-hostname-you-want}
```

* 作業一：將第一台虛擬機hostname改為`centos7-1`，第二台虛擬機hostname改為`centos7-2`
![linux0912-1][linux0912-1]

#### 補充

* 用rpm檢查sshd是否有正確安裝
```
# rpm -qa | grep ssh
```

[**rpm維基百科**][rpm_package_manager]

#### 用圖形化界面修改網路內容
* 作業二：將`centos7-1`的ip位址改為`192.168.xxx.100`，`centos7-2`的ip位址改為`192.168.xxx.101`
* **記得設定dns**
* `ping 8.8.8.8`及`ping www.google.com`作為測試
![linux0912-2][linux0912-2]

#### 兩台虛擬機互ping
* 作業二：用`centos7-1`ping`centos7-2`的ip位址
![linux0912-3][linux0912-3]

#### 更改 /etc/hosts 的內容
* 使用vim在`/etc/hosts`文件最後加上以下內容並存檔
```
192.168.xxx.100 centos7-1 centos7-1.test.com
192.168.xxx.101 centos7-2 centos7-2.test.com
```

![linux0912-4][linux0912-4]

* 測試

![linux0912-5][linux0912-5]

#### 在一號機使用ssh連上二號機
* 作業三：兩台虛擬機用ssh互連

![linux0912-6][linux0912-6]

### (2023/09/19)
#### 複習：ssh伺服器無法連接除錯

1. 確認伺服器已開啟
    * `systemctl status sshd`
    * 若未開啟，使用`systemctl start sshd`
2. 確認port number
    * `netstat -tulnp | grep sshd`
    * 若要修改，修改`/etc/ssh/sshd_config`
3. 確認selinux已關閉
    * `getenforce`
    * 若要更改，修改`/etc/selinux`
4. 確認防火牆已關閉
    * `systemctl status firewalld`
    * 若未關閉，使用`systemctl stop firewalld`，並用`systemctl disable firewalld`使其不會自動開啟

**修改完記得重開機(reboot)**

#### 複習：使用winscp及PuTTY在windows連上虛擬機
* 作業一：
![linux0919-1][linux0919-1]

#### 操作無密碼登入
1. 若`/.ssh`已存在：`rm -rf /.ssh`
2. `ssh-keygen`
3. `ssh-copy-id [user]@[host]` ex: `ssh-copy-id root@centos7-2`

* 作業二：
![linux0919-2][linux0919-2]

#### scp指令操作
* `scp` `欲複製檔案` `貼上路徑`
* ex: `scp /etc/hosts root@centos7-2:/etc/hosts`
* ex: `scp -r root@centos7-2:/root/testdir/ /root/`
* 圖片：
![linux0919-3][linux0919-3]
![linux0919-4][linux0919-4]

#### ssh執行單一指令
* `ssh [user]@[host] [command]`
* ex: `ssh root@centos7-2 ls /root`

#### 更改sshd port number
* 修改`/etc/ssh/sshd_config`
* 作業三：
![linux0919-5][linux0919-5]

#### 課本第十章
#### RPM - Redhat Package Manager
* 查詢指令

| 選項 | 功能 |
|:----:|:----:|
| -qa  | 查詢系統已安裝套件清單 |
| -qi  | 查詢特定套件的安裝資訊 |
| -ql  | 查詢套件所安裝的檔案清單 |
| -qf  | 查詢系統特定檔案的來源安裝套件 |

### (2023/09/26)

#### 在內網建立網頁
1. 在windows(主機)建立一個含圖片及文字的html檔(用word)  
2. 將檔案透過winscp上傳到虛擬機  
3. 將html檔移動至`/var/www/html`目錄中  
4. 在虛擬機的瀏覽器網址列中輸入虛擬機`ip位址 + /檔名.html`即可查看頁面  

![linux0926-1][linux0926-1]

#### 用ngrok建立可在外網使用的網頁
1. 登入`ngrok`
2. 下載`ngrok`主程式
3. 依照官網指示操作設定`ngrok`
4. 輸入指令
```
# ./ngrok http 80
```

即可使用命令列中顯示的網址查看頁面  

![linux0926-2][linux0926-2]

#### 課本
##### 從原始碼編譯htop

1. 從[這裡][htop_source]下載htop原始碼
2. 將下載到的原始碼壓縮檔解壓縮
3. 執行`configure`執行檔
```
# ./configure
```
4. 執行`make`
5. 執行`make install`
6. 執行`htop`即可開啟程式，按下`q`鍵可以退出

##### du - 目錄空間使用量

**常用參數：**
* `-s`:顯示該目錄的總用量，不顯示子目錄
* `-h`:以磁碟單位顯示空間用量
* `--max-depth=N`:限制只顯示至第N層子目錄

##### df - 掛載的磁碟分割資訊

**常用參數**
* `-h`:以磁碟單位顯示空間用量

![linux0926-3][linux0926-3]

### (2023/10/03)
#### 建立nfs伺服器

1. Server端設置
    * 安裝NFS

    ```
    # yum install nfs-utils
    ```

    **安裝`nfs-utils`會自動安裝`rpcbind`**

    * 設置NFS開機自動啟動

    ```
    # systemctl enable rpcbind
    # systemctl enable nfs
    ```

    * 啟動NFS

    ```
    # systemctl start rpcbind
    # systemctl start nfs
    ```  

**課堂上使用的虛擬機為求實驗方便已將防火牆關閉，故無須設置防火牆**

* 設置共享目錄
```
# mkdir /data -p
```  

**`-p`參數表示若目錄存在不動作若不存在則建立目錄，即不產生錯誤訊息**

* 配置導出目錄

```
# vim /etc/exports
```

* 在`/etc/exports`檔案中新增以下內容

```
/data   192.168.241.0/24(rw,sync,no_root_squash,no_all_squash)
```

* 內容說明：

    1. `/data`:共享目錄位置
    2. `192.168.241.0/24`:客戶端IP範圍
    3. `rw`:設置讀寫權限
    4. `sync`:同步共享目錄
    5. `no_root_squash`:可以使用root權限
    6. `no_all_squash`:可以使用普通用戶權限

* `:wq`存檔後，重新啟動NFS

```
# systemctl restart nfs
```

* 檢查本機的共享目錄

```
# showmount -e localhost
```

若配置成功會顯示：

```
Export list for localhost:
/data 192.168.241.0/24
```

2. Client端設置

* 安裝NFS

```
# yum install nfs-utils
```

**安裝`nfs-utils`會自動安裝`rpcbind`**

* 設置NFS開機自動啟動

```
# systemctl enable rpcbind
# systemctl enable nfs
```

* 啟動NFS

```
# systemctl start rpcbind
# systemctl start nfs
```

* 新增掛載資料夾

```
# mkdir /nfs-data
```

* 掛載

```
# mount -t nfs 192.168.241.100:/data /nfs-data
```

* 成果展示
![linux1003-1][linux1003-1]

#### 課本

##### dd - 讀取檔案並輸出
早期在UNIX環境下，用來拷貝磁片內容，現在常用來產生特定大小的測試檔案，如需要產生3MB大小的測試檔案，可使用以下指令：

```
# dd if=/dev/zero of=file3m bs=1M count=3
```

* `if`參數用來指定來源檔案，`/dev/zero`是一個會不斷產生0的檔案

* `of`參數用來指定目的檔案名稱

* `bs=1M`代表產生1M區塊大小

* `count=3`代表產生3個區塊

##### wc - 統計檔案行數與字數
`wc`指令可用來統計檔案有多少行(newline/ line feed/ LF)、多少個英文字節(word)與多少個位元組(byte /letters)

**常用參數**
* `-l`:只顯示行數

* `-c`:只顯示字元數

* `-w`:只顯示英文字節

###### 測試
```
# cat test.txt
hello world
tom marry john

1234 nqu
peter
# cat -n test.txt
1  hello world
2  tom marry john
3  
4  1234 nqu
5  peter
# wc test.txt
5  8 42 test.txt
```

* `cat`指令加上`-n`參數會加上行號

* `wc`指令輸出格式為
```
行數  英文字節數  byte數(letters)  檔案名稱
```

##### tr - 取代或刪除字元
tr指令可將標準輸入字串的特定字元取代或刪除，並輸入取代結果

* [參考資料：tr 命令，Linux tr 命令详解：将字符进行替换压缩和删除 -  Linux 命令搜索引擎](https://wangchujiang.com/linux-command/c/tr.html)

###### 測試1
```
# echo "ABCDEFG"
ABCDEFG
# echo "ABCDEFG" | tr "ABC" "xyz"
xyzDEFG
```

###### 測試2
```
# echo "ABCDEFG" | tr [:upper:] [:lower:]
abcdefg
```

tr定義的常用集合：
* `[:alnum:]`:所有大小寫字母與數字的集合

* `[:alpha:]`:所有大小寫字母的集合

* `[:blank:]`:空白

* `[:digit:]`:代表所有數字的集合

* `[:upper:]`:代表所有大寫字母的集合

* `[:lower:]`:代表所有小寫字母的集合

###### 測試3
```
# echo ABC | tr -d 'A'
BC
```

* `-d`參數可刪除特定字元

###### 測試4
```
# cat test.txt
hello world     
tom    marry   john

1234 nqu
peter
# cat -T -E test.txt
hello world     $
tom^Imarry^Ijohn$
$
1234 nqu$
peter$
# cat test.txt | tr "\t" " "
hello world
tom marry john

1234 nqu
peter
```

* `cat`指令的`-T`參數可將tab顯示為`^I`，`-E`參數可在每行的結尾加上`$`

* `\t`代表字串中的`tab`

###### 測試4
```
# echo aa.,a 1 b#$bb 2 c*/cc 3 ddd 4 | tr -d -c '0-9 \n'
1  2  3  4
```

* `-c`參數表示除了`'0-9 \n'`（0到9、空格、換行）此字符集以外的字符

##### seq - 產生序列數字
用法如下
```
seq 起始值 [累加值] 結束值
```

ex:

```
# seq 1 10
1
2
3
4
5
6
7
8
9
10 
```

```
# seq 10 -1 0
10 9
8
7
6
5
4
3
2
1
0
```

* `-w`參數可補0讓每個數字一樣寬度

```
# seq -w 1 2 11
01
03
05
07
09
11
```

###### 測試
```
# seq -s "+" 1 10 | bc
55
```

* `-s`參數可以指定分隔符號

* `bc`可以計算字串中的運算式

### (2023/10/17)
#### samba

* 安裝
```
# yum install samba samba-client samba-common -y
```

設置SAMBA server:  

1. 建立目標資料夾：
```
# cd /
# mkdir test_samba
```

2. 更改資料夾狀態：
```
# chown nobody /test_samba
# chmod 777 /test_samba
```

3. 編輯SAMBA設定檔：
```
# vim /etc/samba/smb.conf
```

* 設定內容：
```
[test]
    comment = for test # 註解
    path = /test_samba # 資料夾路徑（須在root下）
    read only = no     # 設定為可寫入
    guest ok = yes     # 可以給一般使用者使用
    browseable = yes   # 可以瀏覽的
```

4. 測試設定的參數
```
testparm
```

5. 啟動SAMBA  
**記得要關閉防火牆**
```
# systemctl start smb
```

6. 查看使用到的port
```
# netstat -tunlp | grep smb
```

* 使用445、139 port

7. 建立SAMBA存取密碼  
```
# smbpasswd -a user
```

只要在Windows的檔案總管的路徑欄輸入`\\Linux IP`即可存取SAMBA server內容

* 結果展示：  
![linux1017-1][linux1017-1]

#### 作業1：
1. 在`centos7-1`建立一個`nfs`及`samba`伺服器，將共享資料夾名稱設為`test123`  
2. 在Windows中，使用`data`資料夾存取`centos7-1`中的`test123`資料夾  
3. 在`centos7-2`中，使用`data`資料夾存取`centos7-1`中的`test123`資料夾  

* SAMBA server 設置
```
# cd /
# mkdir test123
# vim /etc/samba/smb.conf
# systemctl restart smb
```

* `smb.conf`新增內容：
```
[data]
    comment = test123 to data 
    path = /test123
    read only = no     # 設定為可寫入
    guest ok = yes     # 可以給一般使用者使用
    browseable = yes   # 可以瀏覽的
```

* NFS server 設置  
* 配置導出目錄  
```
# vim /etc/exports
# systemctl restart nfs
```

* 在`/etc/exports`檔案中新增以下內容
```
/test123   192.168.241.0/24(rw,sync,no_root_squash,no_all_squash)
```

* NFS client 設置
```
# mkdir /data
# mount -t nfs 192.168.241.100:/test123 /data
```

* 結果展示  
![linux1017-2][linux1017-2]

**補充**  
若要在Windows切換SAMBA的使用者，需要在命令提示字元（CMD）或PowerShell輸入以下指令中斷遠端連線：
```
net use * /delete
```

* 結果展示：
![linux1017-3][linux1017-3]

#### 課本
##### sort - 文字檔內容排序
* `sort`預設會以每行第一個字元的ASCII碼比較，若相同則比較第二個字元

* 若要比較數字可以加入`-g`參數

* 若要改變排序方向可加入`-r`參數

* 若要以不同欄位進行比較，如
    ```
    123 0 abc
    325 9 cae
    385 2 cop
    883 10 ceon
    ```
    以上內容若要使用第二欄進行排序可加入`-k{n}`參數，n為第n欄

* 可使用`-t{符號}`來指定欄位的分割符號

* 可使用`tr`指令將分割符號去除再使用`sort`排序

##### uniq - 過濾重複
ex:  
```
# cat doc
2
3
3
5
7
1
5
4
6
# sort doc
1
2
3
3
4
5
5
6
7
# sort doc | uniq
1
2
3
4
5
6
7
```

##### cut - 擷取子字串
ex:  
```
# cat doc
tom, 22, 31000
jack, 21, 29500
eric, 18, 42000
# cut -d',' -f 2 doc
22
21
18
```  

* 裁切字串：  
```
# echo "12345" | cut -b2-4
234
```  

* 產生隨機密碼：  
```
# echo $RANDOM | md5sum | cut -b1-8
```  
以上指令可產生隨機八位數密碼

##### split - 分割檔案
ex:  
```
# dd if=/dev/zero of=file3m bs=1M count=3
3+0 records in
3+0 records out
3145728 bytes (3.1 MB) copied, 0.00227186 s, 1.4 GB/s
# ls -h -l
-rw-r--r-- 1 root root 3.0M Oct 17 03:59 file3m
# split -b 1m file3m
# ls -h -l
-rw-r--r-- 1 root root 3.0M Oct 17 03:59 file3m
-rw-r--r-- 1 root root 1.0M Oct 17 04:02 xaa
-rw-r--r-- 1 root root 1.0M Oct 17 04:02 xab
-rw-r--r-- 1 root root 1.0M Oct 17 04:02 xac
```

* 可使用`cat`進行還原：  
```
# cat 1.txt
Hello1
# cat 2.txt
Hello2
# cat 3.txt
Hello3
# cat 1.txt 2.txt 3.txt > original.txt
Hello1
Hello2
Hello3
```

**補充**  
可使用`diff`指令比較文件差異

##### ping - 請求網路主機回應

##### traceroute - 追蹤網路主機路徑

##### hostname - 主機名稱
```
# hostname
centos7-1
```  

* 改變主機名稱：  
```
# hostnamectl set-hostname {hostname}
```  

##### mail - 簡易電子郵件指令
ex:  
```
# mail -s "test" s111010511@student.nqu.edu.tw
This is a test.
Testing.
EOT(^D)
```  

* `-s` 為主旨

* 主旨後為目標郵箱

* 輸入郵箱後可編輯內文，結束後換行並按下`Ctrl + d`

* **若沒有正確架設則Gmail無法接收**

### (2023/10/24)
#### ipv6

* 使用手機網路取得ipv6位址

* 在Windows CMD or PowerShell使用`ping -6`來測試是否取得ipv6位址：  

![linux1024-1][linux1024-1]

* 即可使用ipv6位址連上linux系統的http伺服器：  

![linux1024-2][linux1024-2]

#### 課本

##### alias - 別名
* 為複雜指令取一個簡單的名稱：
```
# alias
alias cp='cp -i'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --sho -tilde'
```  
* 直接執行會顯示已存在的別名

* 在指令前加入`反斜線（\）`可以使用不使用別名

* 新增別名：
ex:  
```
alias la='ls -a -l'
```  

* 刪除別名：
ex:  
```
unalias la
```  

* 效果只會存在於當前終端機視窗（Session），若要永久生效可將其加入shell的配置檔如`~/.bashrc`檔案中

* 若要將別名套用至整個系統則須修改`/etc/profile`檔案

* 使用`echo $$`可以顯示當前行程pid

##### echo
echo是一個印出指令（output），後面的文字會預設輸出在螢幕上  
ex:  
```
# echo welcome
welcome
# echo welcome to linux
welcome to linux
```

* 使用雙引號（"），若引號中的內容含有「$變數名稱」會將變數的值印出；單引號（'）則不會

* 特殊字元（須搭配`-e`參數）

| 特殊字元 | 說明 |
|:---:|:---:|
| `\a` | 警示音（嗶） |
| `\b` | backspace鍵 |
| `\\` | 代表無特殊意義的反斜線 |
| `\n` | 跳行字元 |
| `\t` | 水平定位點，與tab鍵同義 |
| `\v` | 垂直定位點 |
| `\'` | 代表無特殊意義的單引號 |
| `\"` | 代表無特殊意義的雙引號 |

ex:  
```
# echo "hello\tworld"
hello\tworld
# echo -e "hello\tworld"
hello   world
```  

ex2:  
```
# echo "line1\nline2\nline3"
line1\nline2\nline3
# echo -e "line1\nline2\nline3"
line1
line2
line3
```  

##### bash預設變數

* HOME
    * 目前使用者的家目錄

* IFS
    * 用來分隔欄位的字元清單

* PATH
    * 用分號分隔的一連串目錄，代表執行指令時的搜尋路徑清單

* USER
    * 目前使用者帳號名稱

* UID
    * 目前使用者的uid

##### 將路徑加入環境變數（PATH）

以將`~/bin`加入PATH為例：  
```
export PATH=/home/user/bin:$PATH
```  

以上指令直接執行效果僅限當前session，若要永久生效可將其加入shell的配置檔如`~/.bashrc`中

##### shell簡單判斷式
ex:  
```
[[ $USER == 'root' ]]&&echo 1||echo 0
```  
若`[[`  `]]`中的內容為真，執行`&&`後的指令，若為假則執行`||`後的指令

----------
[linux0912-1]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0912-1.png?raw=tru
[rpm_package_manager]: https://zh.wikipedia.org/zh-tw/RPM套件管理員 
[linux0912-2]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0912-2.png?raw=tru
[linux0912-3]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0912-3.png?raw=tru
[linux0912-4]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0912-4.png?raw=tru
[linux0912-5]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0912-5.png?raw=tru
[linux0912-6]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0912-6.png?raw=tru
[linux0919-1]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0919-1.png?raw=tru
[linux0919-2]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0919-2.png?raw=tru
[linux0919-3]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0919-3.png?raw=tru
[linux0919-4]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0919-4.png?raw=tru
[linux0919-5]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0919-5.png?raw=tru
[linux0926-1]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0926-1.png?raw=tru
[linux0926-2]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0926-2.png?raw=tru
[linux0926-3]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux0926-3.png?raw=tru
[htop_source]: https://src.fedoraproject.org/lookaside/extras/htop/htop-2.2.0.tar.gz/sha512/ec1335bf0e3e0387e5e50acbc508d0effad19c4bc1ac312419dc97b82901f4819600d6f87a91668f39d429536d17304d4b14634426a06bec2ecd09df24adc62e/
[linux1003-1]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux1003-1.png?raw=tru
[linux1017-1]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux1017-1.png?raw=tru
[linux1017-2]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux1017-2.png?raw=tru
[linux1017-3]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux1017-3.png?raw=tru
[linux1017-4]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux1017-4.png?raw=tru
[linux1024-1]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux1024-1.png?raw=tru
[linux1024-2]: https://github.com/dallas145/2023LInuxServer/blob/main/source/linux1024-2.png?raw=tru
