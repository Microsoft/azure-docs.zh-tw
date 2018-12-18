---
title: 在 Azure 中 Windows 上安裝 MongoDB | Microsoft Docs
description: 了解如何在 Azure VM (執行以 Resource Manager 部署範本建立的 Windows Server 2012 R2) 上安裝 MongoDB。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: 53faf630-8da5-4955-8d0b-6e829bf30cba
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: a45d6a6064173cea7ed15065ab3464718cc8578e
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37931407"
---
# <a name="install-and-configure-mongodb-on-a-windows-vm-in-azure"></a>在 Azure 中的 Windows VM 上安裝及設定 MongoDB
[MongoDB](http://www.mongodb.org) 是受歡迎的高效能開放原始碼 NoSQL 資料庫。 這篇文章會逐步引導您安裝和設定 Azure 中 Windows Server 2016 虛擬機器 (VM) 上的 MongoDB。 您也可以[在 Azure 中的 Linux VM 上安裝 MongoDB](../linux/install-mongodb.md)。

## <a name="prerequisites"></a>先決條件
在安裝及設定 MongoDB 之前，您必須建立 VM，並且最好將資料磁碟新增至其中。 請參閱下列文章，以建立 VM 並且新增資料磁碟︰

* 使用 [Azure 入口網站](quick-create-portal.md)或 [Azure PowerShell](quick-create-powershell.md) 建立 Windows Server VM。
* 使用 [Azure 入口網站](attach-managed-disk-portal.md)或 [Azure PowerShell](attach-disk-ps.md)將資料磁碟連結到 Windows Server VM。

若要開始安裝和設定 MongoDB，請使用遠端桌面[登入您的 Windows Server VM](connect-logon.md)。

## <a name="install-mongodb"></a>安裝 MongoDB
> [!IMPORTANT]
> MongoDB 安全性功能，例如驗證和 IP 位址繫結，均非預設為已啟用。 安全性功能應該在將 MongoDB 部署到生產環境前加以啟用。 如需詳細資訊，請參閱 [MongoDB 安全性和驗證](http://www.mongodb.org/display/DOCS/Security+and+Authentication)。


1. 使用遠端桌面連線到 VM 之後，請從工作列開啟 Internet Explorer。
2. Internet Explorer 第一次開啟時，選取 [使用建議的安全性、隱私權與相容性設定]，然後按一下 [確定]。
3. 預設會啟用 Internet Explorer 增強式安全性設定。 將 MongoDB 網站新增至允許的網站清單︰
   
   * 選取右上方的 [工具] 圖示。
   * 在 [網際網路選項] 中，選取 [安全性] 索引標籤，然後選取 [受信任的網站] 圖示。
   * 按一下 [網站] 按鈕。 將 *https://\*.mongodb.com* 新增至受信任的網站清單，然後關閉對話方塊。
     
     ![設定 Internet Explorer 安全性設定](./media/install-mongodb/configure-internet-explorer-security.png)
4. 瀏覽至 [MongoDB - 下載](http://www.mongodb.com/downloads)頁面 (http://www.mongodb.com/downloads)。
5. 如果需要，選取 [Community Server] 版本，然後選取適用於「Windows Server 2008 R2 64 位元和更新版本」的目前最新穩定版本。 若要下載安裝程式，請按一下 [下載 (msi)]。
   
    ![下載 MongoDB 安裝程式](./media/install-mongodb/download-mongodb.png)
   
    下載完成之後，請執行安裝程式。
6. 閱讀並接受授權合約。 當系統提示時，選取 [完整] 安裝。
7. 如有需要，您可以選擇同時安裝 MongoDB 的圖形介面 Compass。
8. 在最後畫面上，按一下 [安裝]。

## <a name="configure-the-vm-and-mongodb"></a>設定 VM 和 MongoDB
1. 路徑變數不會被 MongoDB 安裝程式更新。 在您的路徑變數中沒有 MongoDB `bin` 位置，您必須在每次使用 MongoDB 可執行檔時指定完整路徑。 若要將位置新增至路徑變數︰
   
   * 使用滑鼠右鍵按一下 [開始] 功能表，然後選取 [系統]。
   * 按一下 [進階系統設定]，然後按一下 [環境變數]。
   * 在 [系統變數] 底下，選取 [路徑]，然後按一下 [編輯]。
     
     ![設定路徑變數](./media/install-mongodb/configure-path-variables.png)
     
     將路徑新增至您的 MongoDB `bin` 資料夾。 MongoDB 通常安裝在 C:\Program Files\MongoDB。 請確認您的 VM 上的安裝路徑。 下列範例會將預設 MongoDB 安裝位置新增至 `PATH` 變數︰
     
     ```
     ;C:\Program Files\MongoDB\Server\3.6\bin
     ```
     
     > [!NOTE]
     > 請務必新增開頭分號 (`;`) 來指出您要將位置新增至 `PATH` 變數。

2. 在資料磁碟上建立 MongoDB 資料和記錄檔目錄。 在 [開始] 功能表中，選取 [命令提示字元]。 下列範例會在磁碟機 F: 上建立目錄
   
    ```
    mkdir F:\MongoData
    mkdir F:\MongoLogs
    ```
3. 以下列命令啟動 MongoDB 執行個體，並且據以調整您的資料和記錄檔目錄︰
   
    ```
    mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log
    ```
   
    MongoDB 可能需要花費數分鐘來配置日誌檔案，並開始接聽連線。 在 `mongod.exe` 伺服器啟動和配置日誌檔案時，所有記錄訊息都會傳送至 *F:\MongoLogs\mongolog.log* 檔案。
   
   > [!NOTE]
   > 當您的 MongoDB 執行個體正在執行時，命令提示字元會專注於這項工作。 保持命令提示字元視窗開啟以繼續執行 MongoDB。 或者，安裝 MongoDB 做為服務，如下一個步驟所述。

4. 為了更穩固的 MongoDB 體驗，請安裝 `mongod.exe` 做為服務。 建立服務表示您不需要在每次想要使用 MongoDB 時都保持命令提示字元執行。 如下所示建立服務，據以調整您的資料和記錄檔目錄的路徑︰
   
    ```
    mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log --logappend  --install
    ```
   
    上述命令會建立一個名為 MongoDB 的服務，其說明為 "Mongo DB"。 同時指定下列參數：
   
   * `--dbpath` 選項指出資料目錄的位置。
   * `--logpath` 選項必須用來指定記錄檔，因為執行中的服務沒有命令視窗可以顯示輸出。
   * `--logappend` 選項指出重新啟動服務會導致輸出附加在現有的記錄檔案中。
   
   若要啟動 MongoDB 服務，請執行下列命令︰
   
    ```
    net start MongoDB
    ```
   
    如需建立 MongoDB 服務的詳細資訊，請參閱[設定 MongoDB 的 Windows 服務](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/#mongodb-as-a-windows-service)。

## <a name="test-the-mongodb-instance"></a>測試 MongoDB 執行個體
當 MongoDB 執行為單一執行個體或安裝為服務，您現在可以開始建立和使用您的資料庫。 要啟動 MongoDB 管理殼層，請從 [開始] 功能表中開啟另一個命令提示字元視窗，並輸入下列命令：

```
mongo  
```

您可以使用 `db` 命令列出資料庫。 插入一些資料，如下所示︰

```
db.foo.insert( { a : 1 } )
```

搜尋資料，如下所示︰

```
db.foo.find()
```

輸出類似於下列範例：

```
{ "_id" : "ObjectId("57f6a86cee873a6232d74842"), "a" : 1 }
```

結束 `mongo` 主控台，如下所示︰

```
exit
```

## <a name="configure-firewall-and-network-security-group-rules"></a>設定防火牆和網路安全性群組規則
現在，MongoDB 已安裝並正在執行，在 Windows 防火牆中開啟一個連接埠，才能遠端連線至 MongoDB。 若要建立新的輸入規則以允許 TCP 連接埠 27017，開啟系統管理 PowerShell 提示字元並輸入下列命令︰

```powerahell
New-NetFirewallRule `
    -DisplayName "Allow MongoDB" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 27017 `
    -Action Allow
```

您也可以使用**具有進階安全性的 Windows 防火牆** 圖形化管理工具來建立規則。 建立新的輸入規則以允許 TCP 連接埠 27017。

如有需要，建立網路安全性群組規則以允許從現有 Azure 虛擬網路子網路外部存取 MongoDB。 您可以使用 [Azure 入口網站](nsg-quickstart-portal.md) 或 [Azure PowerShell](nsg-quickstart-powershell.md) 來建立網路安全性群組規則。 如同 Windows 防火牆規則，允許 TCP 連接埠 27017 連接至 MongoDB VM 的虛擬網路介面。

> [!NOTE]
> TCP 連接埠 27017 是 MongoDB 使用的預設連接埠。 您可以在手動啟動或從服務啟動 `mongod.exe` 時，使用 `--port` 參數變更此連接埠。 如果您變更連接埠，請確定在先前步驟中更新 Windows 防火牆和網路安全性群組規則。


## <a name="next-steps"></a>後續步驟
在本教學課程中，您了解如何在 Windows VM 上安裝及設定 MongoDB。 您現在可以遵循 [MongoDB 文件](https://docs.mongodb.com/manual/) 中的進階主題，以便存取 Windows VM 上的 MongoDB。

