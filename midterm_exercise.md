# 10到12章習題

## Ch.10

1. RPM規格是誰提出的？
    * RedHat

2. 有關RPM以下為真？
    * RPM是預先編譯好的軟體

3. 以下哪一個指令可列出主機內安裝的rpm軟體清單：
    * `rpm -qa`

4. 有一個rpm檔案名稱為`httpd-2.2.15-1.el5.noarch.rpm`，以下何者為真？
    * 不限特定主機皆可安裝

5. 想要知道主機內已安裝的`php`套件的整體資訊，應使用指令為？
    * `rpm -qi php`

6. 以下何者非安裝rpm的途徑？
    * 由BBS下載安裝

7. RPM安裝時有時會出現以下哪個問題？
    * 相依於某特定套件

8. 管理者可以利用rpm指令完成以下哪一項工作？
    * 更新套件

9. 有關`yum`，以下何者不為真？
    * 可更新系統時間

10. 利用`yum`可達到以下哪項功能？
    * 可搜尋含有特定關鍵字套件

11. 在Linux中安裝軟體不能使用以下哪個方式？
    * 下載setup.exe後安裝

## Ch.11

1. 如果管理員想知道`/home`目錄底下總共使用的多少空間 ，可使用哪一個指令得到結果？
    * `du -s /home`

2. 如果管理員想知道`/home`目錄底下第一層目錄的空間使用狀況，可使用哪個指令？
    * `du --max-depth=1 /home

3. 管理員可使用哪一個指令得到目前掛載分割區的使用情況？
    * `df`

4. 欲產生測試用一個10MB的空白檔案，可使用哪個指令得到？
    * `dd`

5. 哪個工具可計算出檔案內共有幾行？
    * `wc`

6. 哪個工具指令可產生循序性的數字？
    * `seq`

7. 哪個工具指令可用來刪除某特定字元？
    * `tr`

8. 哪個工具指令可用來取代某特定字元？
    * `tr`

9. 如果在兩台主機中都有帳號，使用哪個指令可以跨主機進行檔案複製？
    * `scp`

## Ch.12

1. 什麼是`shell`? 取名為`shell`的原因為何？

2. 在`shell`中，使用雙引號「" "」與單引號「' '」有何不同？
    * 使用雙引號（"），若引號中的內容含有「$變數名稱」會將變數的值印出；單引號（'）則不會

3. 建立一個新別名的指令是？
    * `alias`

4. 在命令列上印出字串「Hello all」的指令是？
    * `echo 'Hello all'`

5. 以下兩行指令的執行結果是？
    ```sh
    n=30
    echo 'I am $n years old'
    ```

    * I am $n years old

6. 可由哪個指令得到以下執行結果？
    ```
    first line
    2nd line
    ```

    * `echo -e 'first line\n2nd line'`

7. 想取得目前檔案「搜尋路徑」，可以在bash裡使用哪一個內建變數？
    * `PATH`

8. 以下哪個指令可以產生一個亂數？
    * `echo $RANDOM`

9. 以下指令依序執行後，最後`$n`變數的值為？
    ```sh
    m='Raining'
    n="$m"
    ```

    * `Raining`

10. 以下指令最後「`echo $?`」的結束為？
    * `0`
    
11. 以下是某目錄的檔案清單，若執行bash指令「test -d file1」，結果為？
    ```
    [root@localhost testdir] # ls -l
    總計 30388
    -rw-r--r--1 root root 66191 2017-03-01 00:00 file1
    -rw-------1 root root 1812 2017-03-01 00:00 file2
    -rw-r--r--1 root root 14084 2017-03-01 00:00 file3
    -rw-r--r--1 root root 30986240 2017-06-28 23:09 myetc.tar
    -rw-r--r--1 root root 17812 2017-06-28 23:11 myfiles.tar.gz
    -rw-r--r--1 root root 17812 2017-06-28 23:12 myfiles.tgz
    [root@localhost testdir] # test -d file1
    [root@localhost testdir] # echo $?
    ```

    * 1

12. 判斷檔案是否擁有可執行權限的shell指令為？
    * `test -x file1`
