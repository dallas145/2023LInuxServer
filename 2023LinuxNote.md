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
* 將`centos7-1`的ip位址改為`192.168.xxx.100`，`centos7-2`的ip位址改為`192.168.xxx.101`
* **記得設定dns**
* `ping 8.8.8.8`及`ping www.google.com`作為測試

#### 更改 /etc/hosts 的內容
* 使用vim在`/etc/hosts`文件最後加上以下內容並存檔
```
192.168.xxx.100 centos7-1 centos7-1.test.com
192.168.xxx.101 centos7-2 centos7-2.test.com
```



----------
[linux0912-1]: https://github.com/dallas145/2023LInuxServer/blob/51b05b58162480da8937799b2c8a96a70cf68f0f/source/linux0912-1.png
[rpm_package_manager]: https://zh.wikipedia.org/zh-tw/RPM套件管理員
