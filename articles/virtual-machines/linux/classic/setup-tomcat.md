---
title: 在 Linux 虛擬機器上設定 Apache Tomcat | Microsoft Docs
description: 了解如何使用執行 Linux 的 Azure 虛擬機器設定 Apache Tomcat7。
services: virtual-machines-linux
documentationcenter: ''
author: NingKuang
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: 45ecc89c-1cb0-4e80-8944-bd0d0bbedfdc
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 12/15/2015
ms.author: ningk
ms.openlocfilehash: 161a56a019f8c2c8ce5e3890e73ad5c5710e7b82
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30841610"
---
# <a name="set-up-tomcat7-on-a-linux-virtual-machine-with-azure"></a>使用 Azure 在 Linux 虛擬機器上設定 Tomcat7
Apache Tomcat (或直接稱為 Tomcat，以往也稱為 Jakarta Tomcat) 是 Apache Software Foundation (ASF) 開發的開放原始碼 Web 伺服器和 Servlet 容器。 Tomcat 會實作 Sun Microsystems 提供的 Java Servlet 和 JavaServer Pages (JSP) 規格。 Tomcat 提供用來執行 Java 程式碼的純 Java HTTP 網頁伺服器環境。 在最簡單的組態中，Tomcat 會在單一作業系統處理序中執行。 此程序會執行 Java 虛擬機器 (JVM)。 從瀏覽器到 Tomcat 的每個 HTTP 要求都會以 Tomcat 程序中個別的執行緒形式予以處理。  

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有兩種：[Azure Resource Manager](../../../resource-manager-deployment-model.md) 和傳統。 本文說明如何使用傳統部署模型。 我們建議讓大部分的新部署使用 Resource Manager 模型。 若要透過 Resource Manager 範本使用 Open JDK 與 Tomcat 部署 Ubuntu VM 的詳細資訊，請參閱[本文](https://azure.microsoft.com/documentation/templates/openjdk-tomcat-ubuntu-vm/)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

在本文中，您將在 Linux 映像上安裝 Tomcat7 並且部署於 Azure 中。  

您將了解：  

* 如何在 Azure 中建立虛擬機器。
* 如何準備 Tomcat7 的虛擬機器。
* 如何安裝 Tomcat7。

假設您已經有 Azure 訂用帳戶。  如果沒有，您可以在 [Azure 網站](https://azure.microsoft.com/)上註冊免費試用版。 如果您擁有 MSDN 訂用帳戶，請參閱 [Microsoft Azure 特價：MSDN、MPN 及 BizSpark 優惠](https://azure.microsoft.com/pricing/member-offers/msdn-benefits/?c=14-39)。 若要深入了解 Azure，請參閱 [什麼是 Azure？](https://azure.microsoft.com/overview/what-is-azure/)。

本文假設您有 Tomcat 和 Linux 的基本操作知識。  

## <a name="phase-1-create-an-image"></a>第 1 階段：建立映像。
在這個階段，您將在 Azure 中使用 Linux 映像建立虛擬機器。  

### <a name="step-1-generate-an-ssh-authentication-key"></a>步驟 1：產生 SSH 驗證金鑰
SSH 對系統管理員而言是很重要的工具。 不過，不建議根據人為決定的密碼來設定存取安全性。 惡意使用者可以依據使用者名稱和弱式密碼侵入系統。

好消息是，有辦法保持遠端存取開啟，而且不需要擔心密碼。 此方法是由使用非對稱密碼編譯的驗證所組成。 使用者的私密金鑰就是一種授與驗證的金鑰。 您甚至可以鎖定使用者的帳戶，而不允許密碼驗證。

這個方法的另一個優點是，您不需要不同的密碼來登入不同的伺服器。 您可以使用所有伺服器上的個人私密金鑰進行驗證，因此不需要記住多組密碼。



依照下列步驟來產生 SSH 驗證金鑰。

1. 從下列位置下載並安裝 PuTTYgen：[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
2. 執行 Puttygen.exe。
3. 按一下 [產生]  來產生金鑰。 在這個程序中，您可以在視窗中的空白區域移動滑鼠來提高隨機性。  
   ![PuTTY 金鑰產生器螢幕擷取畫面，其中顯示 [產生新金鑰] 按鈕][1]
4. 在產生程序之後，Puttygen.exe 會顯示新的公開金鑰。  
   ![PuTTY 金鑰產生器螢幕擷取畫面，其中顯示新的公開金鑰和 [儲存私密金鑰] 按鈕][2]
5. 選取並複製公開金鑰，並將它儲存在名為 publicKey.pem 的檔案中。 不要按 [儲存公開金鑰] ，因為儲存的公開金鑰檔案格式與我們想要的公開金鑰不同。
6. 按一下 [儲存私密金鑰]，並將它儲存在名為 privateKey.ppk 的檔案中。

### <a name="step-2-create-the-image-in-the-azure-portal"></a>步驟 2：在 Azure 入口網站中建立映像
1. 在[入口網站](https://portal.azure.com/)中，按一下工作列中的 [建立資源] 來建立映像。 然後根據您的需求選擇 Linux 映像。 下列範例使用 Ubuntu 14.04 映像。
![顯示 [新增] 按鈕的入口網站螢幕擷取畫面][3]

2. 對於 [主機名稱]，指定您和網際網路用戶端將用來存取此虛擬機器的 URL 名稱。 定義 DNS 名稱的最後一個部分，例如 tomcatdemo。 Azure 會接著產生如 tomcatdemo.cloudapp.net 的 URL。  

3. 對於 [SSH 驗證金鑰]，從 publicKey.pem 檔案複製金鑰值，其中包含 PuTTYgen 所產生的公開金鑰。  
![入口網站中的 [SSH 驗證金鑰] 方塊][4]

4. 視需要設定其他設定，然後按一下 [建立]。  

## <a name="phase-2-prepare-your-virtual-machine-for-tomcat7"></a>第 2 階段：準備用於 Tomcat7 的虛擬機器
在這個階段，您將設定 Tomcat 流量的端點，然後連線到新的虛擬機器。

### <a name="step-1-open-the-http-port-to-allow-web-access"></a>步驟 1：開啟允許 Web 存取的 HTTP 連接埠
Azure 中的端點包含 TCP 或 UDP 通訊協定，以及公用和私人連接埠。 私人連接埠是服務在虛擬機器上接聽的連接埠。 公用連接埠是 Azure 雲端服務接聽外部內送、網際網路流量的連接埠。  

TCP 連接埠 8080 是 Tomcat 用於接聽的預設連接埠號碼。 如果使用 Azure 端點開啟這個連接埠，您和其他網際網路用戶端即可存取 Tomcat 頁面。  

1. 在入口網站中，按一下 [瀏覽]  > [虛擬機器]，然後按一下您建立的虛擬機器。  
   ![虛擬機器目錄的螢幕擷取畫面][5]
2. 若要將端點新增至虛擬機器，請按一下 [端點]  方塊。
   ![顯示 [端點] 方塊的螢幕擷取畫面][6]
3. 按一下 [新增] 。  

   1. 對於端點，在 [端點] 中輸入端點的名稱，然後在 [公用連接埠] 中輸入 80。  

      如果設定為 80，您就不需要在用來存取 Tomcat 的 URL 中包含連接埠號碼。 例如： http://tomcatdemo.cloudapp.net。    

      如果您將它設定為另一個值 (例如 81)，您就必須將此連接埠號碼新增 URL 才能存取 Tomcat。 例如，http://tomcatdemo.cloudapp.net:81/。
   2. 在 [私人連接埠] 中輸入 8080。 Tomcat 預設會接聽 TCP 連接埠 8080。 如果您變更 Tomcat 的預設接聽連接埠，則必須更新 [私人連接埠]，使其與 Tomcat 接聽連接埠相同。  
      ![UI 的螢幕擷取畫面，其中顯示 [新增] 命令、[公用連接埠] 和 [私人連接埠]][7]
4. 按一下 [確定]  ，將端點加入您的虛擬機器。

### <a name="step-2-connect-to-the-image-you-created"></a>步驟 2：連線到您建立的映像
您可以選擇任何 SSH 工具來連線到虛擬機器。 在此範例中，我們使用 PuTTY。  

1. 從入口網站取得您虛擬機器的 DNS 名稱。
    1. 按一下 [瀏覽] > [虛擬機器]。
    2. 選取虛擬機器的名稱，然後按一下 [屬性]。
    3. 在 [屬性] 圖格中，查看 [網域名稱] 方塊以取得 DNS 名稱。  

2. 從 [SSH] 方塊取得 SSH 連線的連接埠號碼。  
![顯示 SSH 連線連接埠號碼的螢幕擷取畫面][8]

3. 下載 [PuTTY](http://www.putty.org/)。  

4. 下載之後，按一下可執行檔 Putty.exe。 在 PuTTY 組態中，使用從虛擬機器屬性取得的主機名稱和連接埠號碼設定基本選項。   
![顯示 PuTTY 組態主機名稱和連接埠選項的螢幕擷取畫面][9]

5. 在左窗格中，按一下 [連線]  > [SSH]  > [驗證]，然後按一下 [瀏覽] 來指定 privateKey.ppk 檔案的位置。 PrivateKey.ppk 檔案包含 PuTTYgen 稍早在本文的＜第 1 階段：建立映像＞一節中產生的私密金鑰。  
![顯示連線目錄階層和 [瀏覽] 按鈕的螢幕擷取畫面][10]

6. 按一下 [開啟] 。 您可能會收到警告訊息方塊。 如果您已正確設定 DNS 名稱和連接埠號碼，按一下 [是] 。
![顯示通知的螢幕擷取畫面][11]

7. 系統會提示您輸入使用者名稱。  
![螢幕擷取畫面，其中顯示輸入使用者名稱的位置][12]

8. 輸入您稍早在本文的＜第 1 階段：建立映像＞一節中用來建立虛擬機器的使用者名稱。 您會看到類似下列的畫面：  
![顯示驗證確認的螢幕擷取畫面][13]

## <a name="phase-3-install-software"></a>第 3 階段：安裝軟體
在這個階段，您會安裝 Java 執行階段環境、Tomcat7 和其他 Tomcat7 元件。  

### <a name="java-runtime-environment"></a>Java 執行階段環境
Tomcat 是以 Java 撰寫的。 Java 開發套件 (JDK) 有兩種類型 (OpenJDK 和 Oracle JDK)。 您可以選擇所需的其中一個。  

> [!NOTE]
> 這兩個 JDK 對於 Java API 中的類別有幾乎相同的程式碼，但是對於虛擬機器的程式碼則有所不同。 OpenJDK 傾向使用開放程式庫，而 Oracle JDK 傾向於使用非開放程式庫。 Oracle JDK 有較多的類別和一些已修復的錯誤，而 Oracle JDK 則比 OpenJDK 穩定。

#### <a name="install-openjdk"></a>安裝 OpenJDK  

使用下列命令來下載 OpenJDK。   

    sudo apt-get update  
    sudo apt-get install openjdk-7-jre  


* 若要建立包含 JDK 檔案的目錄：  

        sudo mkdir /usr/lib/jvm  
* 若要將 JDK 檔案擷取到 /usr/lib/jvm/ 目錄：  

        sudo tar -zxf jdk-8u5-linux-x64.tar.gz  -C /usr/lib/jvm/

#### <a name="install-oracle-jdk"></a>安裝 Oracle JDK


使用下列命令從 Oracle 網站下載 Oracle JDK。  

     wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u5-b13/jdk-8u5-linux-x64.tar.gz  
* 若要建立包含 JDK 檔案的目錄：  

        sudo mkdir /usr/lib/jvm  
* 若要將 JDK 檔案擷取到 /usr/lib/jvm/ 目錄：  

        sudo tar -zxf jdk-8u5-linux-x64.tar.gz  -C /usr/lib/jvm/  
* 若要將 Oracle JDK 設定為預設 Java 虛擬機器︰  

        sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_05/bin/java 100  

        sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_05/bin/javac 100  

#### <a name="confirm-that-java-installation-is-successful"></a>確認 Java 安裝成功
您可以使用如下所示的命令，測試是否已正確安裝 Java 執行階段環境：  

    java -version  

如果您已安裝 OpenJDK，應該會看見如下的訊息：![OpenJDK 安裝成功訊息][14]

如果您已安裝 Oracle JDK，應該會看見如下的訊息：![Oracle JDK 安裝成功訊息][15]

### <a name="install-tomcat7"></a>安裝 Tomcat7
使用下列命令來安裝 Tomcat7。  

    sudo apt-get install tomcat7  

如果您不使用 Tomcat7，請適度修改此命令。  

#### <a name="confirm-that-tomcat7-installation-is-successful"></a>確認 Tomcat7 安裝成功
若要檢查是否已成功安裝 Tomcat7，請瀏覽至 Tomcat 伺服器的 DNS 名稱。 在本文中，範例 URL 是 http://tomcatexample.cloudapp.net/。 如果您看到類似下列的訊息，則表示 Tomcat7 安裝正確。
![Tomcat7 安裝成功訊息][16]

### <a name="install-other-tomcat7-components"></a>安裝其他 Tomcat7 元件
您也可以安裝其他選用 Tomcat 元件。  

使用 **sudo apt-cache search tomcat7** 命令來查看所有可用的元件。 使用下列命令來安裝一些實用的元件。  

    sudo apt-get install tomcat7-admin      #admin web applications

    sudo apt-get install tomcat7-user         #tools to create user instances  

## <a name="phase-4-configure-tomcat7"></a>第 4 階段：設定 Tomcat7
在這個階段，您可以管理 Tomcat。

### <a name="start-and-stop-tomcat7"></a>啟動和停止 Tomcat7
Tomcat7 伺服器會在您進行安裝時自動啟動。 您也可以使用下列命令啟動它：   

    sudo /etc/init.d/tomcat7 start

若要停止 Tomcat7：

    sudo /etc/init.d/tomcat7 stop

若要檢視 Tomcat7 的狀態：

    sudo /etc/init.d/tomcat7 status

若要重新啟動 Tomcat 服務： 

    sudo /etc/init.d/tomcat7 restart

### <a name="tomcat7-administration"></a>Tomcat7 系統管理
您可以編輯 Tomcat 使用者組態檔，以設定您的系統管理員認證。 使用下列命令：  

    sudo vi  /etc/tomcat7/tomcat-users.xml   

下列是一個範例：  
![顯示 sudo vi 命令輸出的螢幕擷取畫面][17]  

> [!NOTE]
> 建立系統管理員使用者名稱的強式密碼。  

編輯此檔案之後，您應該使用下列命令重新啟動 Tomcat7 服務，以確保變更生效：  

    sudo /etc/init.d/tomcat7 restart  

開啟瀏覽器，並輸入 **http://<your tomcat server DNS name>/manager/html** 做為 URL。 針對本文中的範例，URL 是 http://tomcatexample.cloudapp.net/manager/html。  

連接之後，您應該會看到類似下面的內容：  
![Tomcat Web 應用程式管理員的螢幕擷取畫面][18]

## <a name="common-issues"></a>常見問題
### <a name="cant-access-the-virtual-machine-with-tomcat-and-moodle-from-the-internet"></a>無法從網際網路存取使用 Tomcat 和 Moodle 的虛擬機器
#### <a name="symptom"></a>徵狀  
  Tomcat 正在執行中，但無法使用瀏覽器來查看 Tomcat 預設頁面。
#### <a name="possible-root-cause"></a>可能的根本原因   

  * Tomcat 接聽連接埠與虛擬機器上用於 Tomcat 流量的端點私人連接埠不同。  

     檢查您的公用連接埠和私人連接埠端點設定，並確定私人連接埠與 Tomcat 接聽連接埠相同。 請參閱本文＜第 1 階段：建立映像＞一節，以取得為虛擬機器設定端點的指示。  

     若要決定 Tomcat 接聽連接埠，請開啟 /etc/httpd/conf/httpd.conf (Red Hat 版本) 或 /etc/tomcat7/server.xml (Debian 版本)。 Tomcat 接聽連接埠預設為 8080。 下列是一個範例：  

        <Connector port="8080" protocol="HTTP/1.1"  connectionTimeout="20000"   URIEncoding="UTF-8"            redirectPort="8443" />  

     如果您使用 Debian 或 Ubuntu 之類的虛擬機器，而且您要變更 Tomcat 接聽的預設連接埠 (例如 8081)，您也應該對於作業系統開啟該連接埠。 首先，開啟設定檔：  

        sudo vi /etc/default/tomcat7  

     然後移除最後一行的註解，並將「否」變更為「是」。  

        AUTHBIND=yes
  2. 防火牆已停用 Tomcat 的接聽連接埠。

     您只可以從本機主機看見 Tomcat 預設頁面。 此問題很可能是防火牆封鎖了 Tomcat 所接聽的連接埠。 您可以使用 w3m 工具瀏覽網頁。 下列命令會安裝 w3m，並瀏覽至 Tomcat 預設頁面：  


        sudo yum  install w3m w3m-img


        w3m http://localhost:8080  
#### <a name="solution"></a>解決方法

  * 如果 Tomcat 接聽連接埠與虛擬機器上用於流量的端點私人連接埠不同，您需要將私人連接埠變更為與 tomcat 接聽連接埠相同。   
  2. 如果問題是由防火牆/iptables 造成，請在 /etc/sysconfig/iptables 中新增下列幾行。 https 流量才需要第二行：  

      -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT

      -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT  

     > [!IMPORTANT]
     > 確定上述幾行位於會全域限制存取的任何程式行上方，例如下列這一行：-A INPUT -j REJECT --reject-with icmp-host-prohibited



若要重新載入 iptables，請執行下列命令：

    service iptables restart

這在 CentOS 6.3 上已經過測試。

### <a name="permission-denied-when-you-upload-project-files-to-varlibtomcat7webapps"></a>將專案檔案上傳到 /var/lib/tomcat7/webapps/ 時權限遭拒
#### <a name="symptom"></a>徵狀
  當您使用 SFTP 用戶端 (例如 FileZilla) 連線到虛擬機器，並瀏覽到 /var/lib/tomcat7/webapps/ 以發佈網站時，您收到類似下列的錯誤訊息：  

     status:    Listing directory /var/lib/tomcat7/webapps
     Command:    put "C:\Users\liang\Desktop\info.jsp" "info.jsp"
     Error:    /var/lib/tomcat7/webapps/info.jsp: open for write: permission denied
     Error:    File transfer failed
#### <a name="possible-root-cause"></a>可能的根本原因
  您沒有存取 /var/lib/tomcat7/webapps 資料夾的權限。  
#### <a name="solution"></a>解決方法  
  您需要從 root 帳戶取得權限。 您可以將該資料夾的擁有權從 root 變更為佈建機器時所用的使用者名稱。 下列是一個 azureuser 帳戶名稱的範例：  

     sudo chown azureuser -R /var/lib/tomcat7/webapps

  使用-R 選項對目錄內的所有檔案也一併套用權限。  

  此命令也適用於目錄。 -R 選項會變更目錄內所有檔案和目錄的權限。 下列是一個範例：  

     sudo chown -R username:group directory  

  此命令會變更目錄內所有檔案和目錄的擁有權 (使用者和群組兩者)。  

  下列命令只會變更資料夾目錄的權限。 但不會變更目錄內的檔案和資料夾。  

     sudo chown username:group directory

[1]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-01.png
[2]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-02.png
[3]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-03.png
[4]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-04.png
[5]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-05.png
[6]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-06.png
[7]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-07.png
[8]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-08.png
[9]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-09.png
[10]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-10.png
[11]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-11.png
[12]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-12.png
[13]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-13.png
[14]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-14.png
[15]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-15.png
[16]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-16.png
[17]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-17.png
[18]:media/setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-18.png
