---
title: 建立執行 MySQL 的傳統 Azure VM | Microsoft Docs
description: 使用傳統部署模型，建立執行 Windows Server 2012 R2 和 MySQL 資料庫的 Azure 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ms.assetid: 98fa06d2-9b92-4d05-ac16-3f8e9fd4feaa
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: cynthn
ms.openlocfilehash: be2a4232fdb59deb87d6d9340fd22909707294b8
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30918110"
---
# <a name="install-mysql-on-a-virtual-machine-created-with-the-classic-deployment-model-running-windows-server-2016"></a>在以傳統部署模型建立且執行 Windows Server 2016 的虛擬機器上安裝 MySQL
[MySQL](https://www.mysql.com) 是一種很受歡迎的開放原始碼 SQL 資料庫。 本教學課程示範如何在執行 **Windows Server 2016** 的虛擬機器上安裝並執行 **MySQL 5.7.18 社群版本**作為 MySQL 伺服器。 對於非 MySQL 或 Windows Server 的版本，操作方式可能略有不同。

如需在 Linux 上安裝 MySQL 的指示，請參閱[如何在 Azure 上安裝 MySQL](../../linux/mysql-install.md)。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

## <a name="create-a-virtual-machine-running-windows-server-2016"></a>建立執行 Windows Server 2016 的虛擬機器
如果您還沒有執行 Windows Server 2016 的 VM，則可以使用這個[教學課程](./tutorial.md)來建立虛擬機器。

## <a name="attach-a-data-disk"></a>連接資料磁碟
建立虛擬機器之後，您可以選擇連接資料磁碟。 建議對生產工作負載新增資料磁碟，這可避免包含作業系統的 OS 磁碟機 (C:) 空間不足。

請參閱[如何將資料磁碟附加至 Windows 虛擬機器](../attach-managed-disk-portal.md)，並依照指示連接空的磁碟。 將主機快取設定設為 [無] 或 [唯讀]。

## <a name="log-on-to-the-virtual-machine"></a>登入虛擬機器
接著，您將[登入虛擬機器](./connect-logon.md)，以安裝 MySQL。

## <a name="install-and-run-mysql-community-server-on-the-virtual-machine"></a>在虛擬機器上安裝和執行 MySQL Community Server
請依照下列步驟安裝、設定和執行 MySQL Server 社群版本：

> [!NOTE]
> 當使用 Internet Explorer 下載項目時，您可以將 IE 的 [增強式安全性設定] 設為 [關閉]，並簡化下載程序。 從 [開始] 功能表，按一下 [系統管理工具]/[伺服器管理員]/[本機伺服器]，然後按一下 IE [增強式安全性設定] 並將組態設定為 [關閉]。
>
>

1. 使用遠端桌面連線到虛擬機器之後，按一下 [開始] 畫面中的 [Internet Explorer]  。
2. 選取右上角的 [工具] 按鈕 (齒輪圖示)，然後按一下 [網際網路選項]。 依序按一下 [安全性] 索引標籤和 [信任的網站] 圖示，然後按一下 [網站] 按鈕。 將 http://*.mysql.com 新增至受信任的網站清單。 按一下 [關閉]，然後按一下 [確定]。
3. 在 Internet Explorer 網址列中，輸入 https://dev.mysql.com/downloads/mysql/。
4. 使用 MySQL 網站尋找並下載 Windows 版 MySQL 最新版安裝程式。 選擇 [MySQL 安裝程式] 時，請下載具有完整檔案集的版本 (例如檔案大小為 352.8 MB 的 mysql-installer-community-5.7.18.0.msi)，然後儲存該安裝程式。
5. 安裝程式下載完成之後，請按一下 [執行]  來啟動安裝。
6. 在 [授權合約] 頁面上接受授權合約，然後按 [下一步]。
7. 在 [選擇安裝類型] 頁面上，按一下所需的安裝類型，然後按 [下一步]。 下列步驟假設選取 [僅限伺服器]  安裝類型。
8. 如果 [檢查需求] 頁面顯示，請按一下 [執行] 讓安裝程式安裝任何遺漏的元件。 遵循顯示的任何指示，例如 C++ 可轉散發套件執行階段。
9. 在 [安裝] 頁面上，按一下 [執行]。 安裝完成時，按 [下一步] 。

10. 在 [產品組態] 頁面上，按 [下一步]。

11. 在 [類型和網路] 頁面上，視需要指定所需的組態類型和連線選項，包括 TCP 連接埠。 選取 [顯示進階選項]，然後按 [下一步]。
    ![](./media/mysql-2008r2/MySQL_TypeNetworking.png)

12. 在 [帳戶及角色]  頁面上，指定強式的 MySQL 根密碼。 視需要新增其他 MySQL 使用者帳戶，然後按 [下一步] 。

    ![](./media/mysql-2008r2/MySQL_AccountsRoles_Filled.png)
13. 在 [Windows 服務] 頁面上，視需要對於執行 MySQL 伺服器作為 Windows 服務的預設設定指定變更，然後按 [下一步]。

    ![](./media/mysql-2008r2/MySQL_WindowsService.png)
14. [Plugins and Extensions (外掛程式與延伸模組)] 頁面上的選項為選擇性。 選取 [下一步] 以繼續操作。
15. 在 [進階選項] 頁面上，視需要對於記錄選項指定變更，然後按 [下一步]。

    ![](./media/mysql-2008r2/MySQL_AdvOptions.png)
16. 在 [套用伺服器組態] 頁面上，按一下 [執行]。 完成組態步驟時，按一下 [完成] 。
17. 在 [產品組態] 頁面上，按 [下一步]。
18. 如果您想要以後再檢查，請在 [安裝完成] 頁面上，按一下 [將記錄複製到剪貼簿]，然後按一下 [完成]。
19. 從 [開始] 畫面，輸入 **mysql**，然後按一下 [MySQL 5.7 命令列用戶端]。
20. 輸入您在步驟 12 中指定的根密碼，系統會提示您發出命令來設定 MySQL。 如需命令和語法的詳細資訊，請參閱 [MySQL 參考手冊](https://dev.mysql.com/doc/refman/5.7/en/server-configuration.html)。

    ![](./media/mysql-2008r2/MySQL_CommandPrompt.png)
21. 您也可以設定伺服器組態預設設定，例如基底、資料目錄和磁碟機。 如需詳細資訊，請參閱 [6.1.2 伺服器組態預設值](https://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html)。

## <a name="configure-endpoints"></a>設定端點

對於可供網際網路上用戶端電腦使用的 MySQL 服務，您必須為 TCP 連接埠設定端點，並建立 Windows 防火牆規則。 MySQL Server 服務用來接聽 MySQL 用戶端的預設連接埠值為 3306。 只要該連接埠與 [Type and Networking (類型與網路)] 頁面上提供的值一致 (上個程序的步驟 11)，您就能指定另一個連接埠。

> [!NOTE]
> 若要用於實際執行環境，請考量讓 MySQL Server 服務可供網際網路上所有電腦使用的安全性。 您可以定義來源 IP 位址的集合，以便使用具備存取控制清單 (ACL) 的端點。 如需詳細資訊，請參閱[如何設定虛擬機器的端點](setup-endpoints.md)。
>
>

若要設定 MySQL Server 服務的端點：

1. 在 Azure 入口網站中，依序按一下 [虛擬機器 (傳統)]、MySQL 虛擬機器的名稱以及 [端點]。
2. 在命令列中，按一下 [新增] 。
3. 在 [新增端點] 頁面上，為 [名稱] 輸入唯一名稱。
4. 選取 **TCP** 做為通訊協定。
5. 在 [公用連接埠] 與 [私人連接埠] 中輸入連接埠號碼，例如 **3306**，然後按一下 [確定]。

## <a name="add-a-windows-firewall-rule-to-allow-mysql-traffic"></a>新增允許 MySQL 流量的 Windows 防火牆規則
若要新增允許來自網際網路之 MySQL 流量的「Windows 防火牆」規則，請在 MySQL 伺服器虛擬機器上，於_提升權限的 Windows PowerShell 命令提示字元_中執行下列命令。

    New-NetFirewallRule -DisplayName "MySQL57" -Direction Inbound –Protocol TCP –LocalPort 3306 -Action Allow -Profile Public

## <a name="test-your-remote-connection"></a>測試您的遠端連線
若要測試對執行 MySQL Server 服務的 Azure VM 其遠端連線，您必須提供包含 VN 之雲端服務的 DNS 名稱。

1. 在 Azure 入口網站中，按一下 [虛擬機器 (傳統型)]，然後按一下您的 MySQL 伺服器虛擬機器名稱，再按一下 [概觀]。
2. 在虛擬機器儀表板中，記下 [DNS 名稱] 值。 下列是一個範例：

   ![](media/mysql-2008r2/MySQL_DNSName.png)
3. 在執行 MySQL 或 MySQL 用戶端的本機電腦中執行下列命令，以使用 MySQL 使用者的身分登入：

     mysql -u <yourMysqlUsername> -p -h <yourDNSname>

   例如，您可以為虛擬機器使用 MySQL 使用者名稱 _dbadmin3_ 與 _testmysql.cloudapp.net_ DNS 名稱，以使用下列命令啟動 MySQL：

     mysql -u dbadmin3 -p -h testmysql.cloudapp.net

## <a name="next-steps"></a>後續步驟
若要深入了解如何執行 MySQL，請參閱 [MySQL 文件](http://dev.mysql.com/doc/)。
