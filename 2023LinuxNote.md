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
在windows(主機)建立一個含圖片及文字的html檔(用word)  


#### 用ngrok建立可在外網使用的網頁

#### 課本

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
