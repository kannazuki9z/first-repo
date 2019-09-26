# RHEL以開發者訂閱建立本地套件庫
RHEL在2017年（如果沒記錯的話）開放了開發者訂閱，這個訂閱允許開發者使用大部分RHEL Server版本的功能。請注意，此訂閱針對Server版本，Desktop、Workstation版本並不包含在內。更多相關訊息請見[FAQs for no-cost Red Hat Enterprise Linux](https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux/)
如果想要自Red Hat官方套件庫下載、更新、安裝套件，必須要將系統註冊於具有有效訂閱的帳戶底下，未註冊或未具有有效訂閱的系統與帳戶無法使用Red Hat的Repository。此篇命題為建立本地套件庫，先決條件必定是要解決訂閱問題，剛好，開發者訂閱可以滿足需求。

## 註冊帳戶
到[Red Hat Customer Portal](https://access.redhat.com/)申請帳戶，詳細步驟不贅述，請依照網頁指示申請帳戶。請牢記申請時所填寫的ID與密碼。請注意，是ID而非電子郵件。提交申請後，需至填寫的電子郵件信箱點選啟用帳戶連結。
![](https://i.imgur.com/zIYnIQ7.png)
![](https://i.imgur.com/DsVaZ1y.png)



## 申請開發者訂閱
在網頁搜尋引擎輸入red hat developer subscription作為關鍵字，找到[No-Cost RHEL Developer Subscription now available](https://developers.redhat.com/blog/2016/03/31/no-cost-rhel-developer-subscription-now-available/)。進入網頁後，網頁下拉，找到Go get it!段落，並點選[Download](http://red.ht/1Suvt82)連結至下載ISO檔案頁面，直接點擊畫面中TRY IT下方的Download。在彈出畫面中填入缺漏的Company欄位並勾選最下方的核取方塊後，點擊SUBMIT即可取得訂閱並開始下載系統安裝ISO檔。若不需要安裝檔，則可直接取消下載。
![](https://i.imgur.com/uIT4z7l.png)
![](https://i.imgur.com/J4MoQgP.png)


## 檢查訂閱
回到[Red Hat Customer Portal](https://access.redhat.com/)，登入帳戶後，進入主頁下方My Subscriptions即可查看訂閱狀況。除了訂閱狀態外，下方的系統即是註冊於帳戶下的系統。
![](https://i.imgur.com/OrPj4W8.png)


## 註冊系統
參考[How to register and subscribe a system to the Red Hat Customer Portal using Red Hat Subscription-Manager](https://access.redhat.com/solutions/253273)說明內容註冊系統。請注意，命令中的--username需填入的內容為ID而非電子郵件。已註冊系統會列在帳戶的訂閱管理。在Red Hat Subscription-Manager，註冊和訂閱的使用實際上是兩件事情：先登錄系統，再套用訂閱。可以使用下列內容以一道命令同時完成兩件事情：

```
subscription-manager register --username **** --password **** --auto-attach
```

或者先以subscription-manager register將系統登錄至帳戶下，然後透過Customer Portal分配訂閱，或以subscription-manager attach命令操作。

若要註銷系統，可用下列命令完成：

```
subscription-manager remove --all
subscription-manager unregister
subscription-manager clean
```
![](https://i.imgur.com/T2aFmFN.png)
![](https://i.imgur.com/q7UZVvv.png)


## 同步Red Hat套件庫
參考How to create a local mirror of the latest update for Red Hat Enterprise Linux 5, 6, 7, 8 without using Satellite server?建立本地套件庫。本Lab使用/repo掛載點作為存放本地套件庫的路徑：
```
reposync -g -l --downloadcomps --download-metadata -p /repo/ -r rhel-7-server-rpms
```

## 修改yum配置文件
同步Repository過程中，太多下載失敗檔案，每個檔案都得卡個30秒才會再繼續下一個檔案。如果連續卡個20個檔案，整整十分鐘毫無任何下載進度。以map page查詢yum.conf說明文件，在/etc/yum.conf內加入參數timeout=3把預設的逾時時間30秒更改為3秒，快速略過下載失敗的檔案。

## 建立允許用戶使用群組功能的本地套件資料庫
若只單純想要各套件可獨立安裝，可參考How to setup your own package repository以repocreate命令建立本地資料庫所需的相關內容（比方說sqlite DB、metadata之類的內容）；如果想要能夠群組安裝（yum groupinstall），那麼一樣參考前述連結內容，以官方套件庫提供的comps.xml檔案建立sqlite DB。執行下列命令：

```
createrepo -v /repo/rhel-7-server-rpms/ -g /repo/rhel-7-server-rpms/comps.xml
```

命令完成後，會發現多出一個repodata目錄。

## 自訂本地套件庫repo文件
接著在/etc/yum.repos.d/目錄下編輯本地套件庫的配置文件，注意檔名任選，但必須是.repo結尾。如果不會寫，還可以抄啊！在這個目錄下有個redhat.repo，就直接抄這份檔案內容就好。因為這個檔案內寫了一～大～堆的套件庫（每一個以[******]起始的區段就是一個套件庫），事實上預設啟用的也只有一個，那就隨便複製一段來改吧！
```=
[rhel-7-server-v2vwin-1-debug-rpms]
metadata_expire = 86400
enabled_metadata = 0
sslclientcert = /etc/pki/entitlement/872508134122952497.pem
baseurl = https://cdn.redhat.com/content/dist/rhel/server/7/$releasever/$basearch/v2vwin/debug
ui_repoid_vars = releasever basearch
sslverify = 1
name = Red Hat Virt V2V Tool for RHEL 7 (Debug RPMs)
sslclientkey = /etc/pki/entitlement/872508134122952497-key.pem
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
enabled = 0
sslcacert = /etc/rhsm/ca/redhat-uep.pem
gpgcheck = 1
```

1. 第1行中括號框起的是套件庫ID，這個內容可以任意填入，但別使用除了字母、數字、-（dash）以外的字元，可能會有問題的。
1. 第5行baseurl指的是此套件庫的連結路徑，因為是本機路徑，所以以file://表示，而絕對路徑是/repo/rhel-7-server-rpms/（注意這個路徑是指向repodata所在目錄，也就是前面repocreate所指定的路徑），這兩個組合起來就是file:///repo/rhel-7-server-rpms/，是三個斜線，請勿忽略。
1. 第8行name的部分是對這個套件庫的描述，可以填入比較詳細的介紹內容，此Lab填入Red Hat 7 Local。
1. 第11行enabled，0表示禁用、1表示啟用該套件庫，請改成1吧。
1. 第4、7、9、12行的部分，由於原本套件庫配置文件內容是以https作為套件庫連結協定，但此Lab以本地資料作為套件庫為本機系統所用，因此不需要SSL憑證檢查。另外，假使此套件庫將來作為https Server端供其他Client連入，這些SSL選項指定的相關憑證路徑也無法正常使用（難道你有Red Hat的私鑰？），所以保留這些配置項目並沒有意義，在該行前面加上井號（#）註解，或是直接整行刪除。如果不改、不刪會怎樣？不會怎樣，反正它又不是走https協定，根本不會使用SSL！
1. 第13行的gpgcheck，如果參考過其他網路上的內容，大多會建議直接設置成0禁用GPG校驗。但是因為前述的套件庫同步已檢查GPG簽署，所以可以保留相關配置。至於GPG是什麼，那又是另一個故事了，這裡暫不討論。
1. 其他項目對套件庫能否正常運作影響不大，如有興趣可查詢相關資料、yum.conf的man page，或者直接刪掉眼不見為淨。

```
1. [rhel-7-local]
1. metadata_expire = 86400
1. enabled_metadata = 0
1. #sslclientcert = /etc/pki/entitlement/872508134122952497.pem
1. baseurl = file:///repo/rhel-7-server-rpms/
1. ui_repoid_vars = releasever basearch
1. #sslverify = 1
1. name = Red Hat 7 Local
1. #sslclientkey = /etc/pki/entitlement/872508134122952497-key.pem
1. gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
1. enabled = 1
1. #sslcacert = /etc/rhsm/ca/redhat-uep.pem
1. gpgcheck = 1
```

## 測試
都弄好之後就可以開始測試啦～完成註冊後的系統，在有對外網路的情況下是會一直啟用RHN套件庫的，就算把/etc/yum.repos.d/redhat.repo刪除、搬移、更名，都會再「長出」一份新的！所以可用下列幾種方式測試：
1. 使用--diablerepo和--enablerepo選項來選擇啟用哪些套件庫。
1. 直接修改enabled選項，將不需要的套件庫直接禁用。
1. 在套件庫配置文件內加入priority選項，數字小者優先。


## 排程更新套件庫
排程也不見得必要，只是可以減少手動去做將來勢必會做的事情。如果不設定排程工作，手動執行下列命令也是可以更新套件庫：

```
reposync -g -l -m --download-metadata -p /repo/ -r rhel-7-server-rpms
createrepo --update -g /repo/rhel-7-server-rpms/comps.xml /repo/rhel-7-server-rpms
```
而要進行排程也很容易，只需要將上面的內容編寫為shell script並加入排程即可。排程工作如果要使用單次排程就用at；要週期執行可用cron或anacron。參考例行性工作排程。

## 異常問題
在排程工作過後，以yum repolist檢視本地套件庫與RHN套件庫，發現整體套件數量有差異。以yum clean all清除快取並以yum repolist重新檢視即正常。因排查期間重新以createrepo命令建立套件資料庫，故此問題須再觀察是--update選項未起作用，或是必須以重建資料庫的方式刷新資料庫內容。
9/23將shell script修正為下列內容後，排程工作結果正常。
```=
reposync -g -l -m --download-metadata -p /repo/ -r rhel-7-server-rpms
createrepo --update -g /repo/rhel-7-server-rpms/comps.xml /repo/rhel-7-server-rpms
yum clean all
```

## 系統更新
```
# 更新系統上除了Kernel相關套件以外的所有套件
yum update --exclude kernel*
# 移除不需要的Kernel，count選項指定保留最新的N個版本
yum install yum-utils
package-cleanup --oldkernels --count=1
```



