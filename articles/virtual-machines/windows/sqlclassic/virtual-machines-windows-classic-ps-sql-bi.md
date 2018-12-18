---
title: SQL Server Business Intelligence | Microsoft Docs
description: 本主題使用以傳統部署模型建立的資源，並描述 Azure 虛擬機器 (VM) 上執行的 SQL Server 提供的商業智慧 (BI) 功能。
services: virtual-machines-windows
documentationcenter: na
author: markingmyname
manager: erikre
editor: monicar
tags: azure-service-management
ms.assetid: c681e7a7-eeda-48aa-bc35-6277f4828244
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/30/2017
ms.author: maghan
ms.openlocfilehash: a41dcd5f2c93e5c1279e1c7511e10e6d72574b3b
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37098741"
---
# <a name="sql-server-business-intelligence-in-azure-virtual-machines"></a>Azure 虛擬機器中的 SQL Server Business Intelligence
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。

Microsoft Azure 虛擬機器資源庫含有包含 SQL Server 安裝的映像。 資源庫映像中支援的 SQL Server 版本與您可以在內部部署電腦與虛擬機器中安裝的安裝檔案相同。 本主題摘要說明映像上安裝的 SQL Server 商業智慧 (BI) 功能，以及佈建虛擬機器後所需的組態步驟。 本主題也描述 BI 功能支援的部署拓撲和最佳做法。

## <a name="license-considerations"></a>授權考量
有兩種方式可在 Microsoft Azure 虛擬機器中授權 SQL Server：

1. 屬於軟體保證的授權機動性優點。 如需詳細資訊，請參閱 [Azure 上透過軟體保證的授權機動性](https://azure.microsoft.com/pricing/license-mobility/)。
2. 支付安裝了 SQL Server 的 Azure 虛擬機器每小時的費用。 請參閱 [虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/#Sql)中的＜SQL Server＞一節。

如需有關授權和目前費率的詳細資訊，請參閱 [虛擬機器授權常見問題集](https://azure.microsoft.com/pricing/licensing-faq/)。

## <a name="sql-server-images-available-in-azure-virtual-machine-gallery"></a>Azure 虛擬機器資源庫中提供 SQL Server 映像
Microsoft Azure 虛擬機器資源庫涵蓋數個包含 Microsoft SQL Server 的映像。 虛擬機器映像上安裝的軟體因作業系統版本與 SQL Server 版本而異。 Azure 虛擬機器資源庫中提供的映像清單經常變更。

<!--![SQL image in azure VM gallery](./media/virtual-machines-windows-classic-ps-sql-bi/IC741367.png)-->
![Azure VM 資源庫中的 SQL 映像](./media/virtual-machines-windows-classic-ps-sql-bi/vm-sql-images.png)

![PowerShell](./media/virtual-machines-windows-classic-ps-sql-bi/IC660119.gif) 下列 PowerShell 指令碼會傳回 ImageName 中包含 “SQL-Server” 的 Azure 映像的清單：

    # assumes you have already uploaded a management certificate to your Microsoft Azure Subscription. View the thumbprint value from the "Subscriptions" menu in Azure portal.

    $subscriptionID = ""    # REQUIRED: Provide your subscription ID.
    $subscriptionName = "" # REQUIRED: Provide your subscription name.
    $thumbPrint = "" # REQUIRED: Provide your certificate thumbprint.
    $certificate = Get-Item cert:\currentuser\my\$thumbPrint # REQUIRED: If your certificate is in a different store, provide it here.-Ser  store is the one specified with the -ss parameter on MakeCert

    Set-AzureSubscription -SubscriptionName $subscriptionName -Certificate $certificate -SubscriptionID $subscriptionID

    Write-Host -foregroundcolor green "List of available gallery images where imagename contains 2016"
    Write-Host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
    get-azurevmimage | where {$_.ImageName -Like "*SQL-Server-2016*"} | select imagename,category, location, label, description

    Write-Host -foregroundcolor green "List of available gallery images where imagename contains 2014"
    Write-Host -foregroundcolor green ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
    get-azurevmimage | where {$_.ImageName -Like "*SQL-Server-2014*"} | select imagename,category, location, label, description

如需有關版本與 SQL Server 所支援的功能的詳細資訊，請參閱下列各項：

* [SQL Server 版本](https://www.microsoft.com/sql-server/sql-server-2017-editions)
* [SQL Server 2016 的版本所支援的功能](https://msdn.microsoft.com/library/cc645993.aspx)

### <a name="bi-features-installed-on-the-sql-server-virtual-machine-gallery-images"></a>SQL Server 虛擬機器資源庫映像上安裝的 BI 功能
下表摘要說明常見的 Microsoft Azure 虛擬機器資源庫映像上所安裝 SQL Server 的商務智慧功能：

* SQL Server 2016 SP1 Enterprise
* SQL Server 2016 SP1 Standard
* SQL Server 2014 SP2 Enterprise
* SQL Server 2014 SP2 Standard
* SQL Server 2012 SP3 Enterprise
* SQL Server 2012 SP3 Standard

| SQL Server BI 功能 | 在資源庫映像上安裝 | 注意 |
| --- | --- | --- |
| **Reporting Services 原生模式** |yes |已安裝但需要組態，包括報表管理員 URL。 請參閱 [設定 Reporting Services](#configure-reporting-services)一節。 |
| **Reporting Services SharePoint 模式** |否 |Microsoft Azure 虛擬機器資源庫映像庫不包含 SharePoint 或 SharePoint 安裝檔案。 <sup>1</sup> |
| **Analysis Services 多維度和資料採礦 (OLAP)** |yes |已安裝並設定為預設的 Analysis Services 執行個體 |
| **Analysis Services 表格式** |否 |SQL Server 2012、2014 和 2016 映像中支援，但預設不會安裝。 安裝另一個執行個體的 Analysis Services。 請參閱本主題中的＜安裝其他 SQL Server 服務和功能＞一節。 |
| **適用於 SharePoint 的 Analysis Services Power Pivot** |否 |Microsoft Azure 虛擬機器資源庫映像庫不包含 SharePoint 或 SharePoint 安裝檔案。 <sup>1</sup> |

<sup>1</sup> 如需有關 SharePoint 和 Azure 虛擬機器的詳細資訊，請參閱[適用於 SharePoint 2013 的 Microsoft Azure 架構](https://technet.microsoft.com/library/dn635309.aspx)和 [Microsoft Azure 虛擬機器上的 SharePoint 部署](https://www.microsoft.com/download/details.aspx?id=34598)。

![PowerShell](./media/virtual-machines-windows-classic-ps-sql-bi/IC660119.gif) 執行下列 PowerShell 命令來取得服務名稱中包含 "SQL" 的已安裝服務清單。

    get-service | Where-Object{ $_.DisplayName -like '*SQL*' } | Select DisplayName, status, servicetype, dependentservices | format-Table -AutoSize

## <a name="general-recommendations-and-best-practices"></a>一般建議和最佳作法
* 使用 SQL Server Enterprise Edition 時，虛擬機器的最小建議大小為 **A3** 。 Analysis Services 和 Reporting Services 的 SQL Server BI 部署建議的虛擬機器大小為 **A4** 。
  
    如需關於目前 VM 大小的詳細資訊，請參閱 [Azure 的虛擬機器大小](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* 磁碟管理的最佳作法是儲存資料、記錄與備份 **C**: 和 **D**: 以外磁碟機上的檔案。 例如，建立資料磁碟 **E**: 和 **F**:。
  
  * 預設磁碟機 **C**: 的磁碟機的快取原則不是使用資料的最佳選項。
  * **D**: 磁碟機是主要用於分頁檔的暫存磁碟機。 **D**: 磁碟機不會永久保存，並且不會儲存在 blob 儲存體中。 管理工作 (例如變更虛擬機器大小) 會重設 **D**: 磁碟機。 建議您**不要**將 **D**: 磁碟機用於資料庫檔案，包括 tempdb。
    
    如需建立及連接磁碟的詳細資訊，請參閱 [如何將資料磁碟連接至虛擬機器](../classic/attach-disk-classic.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。
* 停止或解除安裝您不打算使用的服務。 例如，如果虛擬機器僅用於 Reporting Services，請停止或解除安裝 Analysis Services 和 SQL Server Integration Services。 下圖是預設會啟動的服務的範例。
  
    ![SQL Server 服務](./media/virtual-machines-windows-classic-ps-sql-bi/IC650107.gif)
  
  > [!NOTE]
  > 支援的 BI 案例中需要 SQL Server 資料庫引擎。 在單一伺服器 VM 拓撲中，需要資料庫引擎才能在相同的 VM 上執行。
  
    如需詳細資訊，請參閱下列：[解除安裝 Reporting Services](https://msdn.microsoft.com/library/hh479745.aspx) 和[解除安裝 Analysis Services 的執行個體](https://msdn.microsoft.com/library/ms143687.aspx)。
* 檢查 **Windows Update** 以取得新的「重要更新」。 Microsoft Azure 虛擬機器映像經常更新；不過，在前一次重新整理 VM 映像之後，重要更新可能可從 **Windows Update** 取得。

## <a name="example-deployment-topologies"></a>部署拓撲範例
下面是使用 Microsoft Azure 虛擬機器的部署範例。 這些圖表中的拓撲只是一些您可以搭配 SQL Server BI 功能與 Microsoft Azure 虛擬機器使用的可能拓撲。

### <a name="single-virtual-machine"></a>單一虛擬機器
Analysis Services、Reporting Services、SQL Server Database Engine 和資料來源在單一虛擬機器上。

![具有 1 部虛擬機器的 BI IaaS 案例](./media/virtual-machines-windows-classic-ps-sql-bi/IC650108.gif)

### <a name="two-virtual-machines"></a>兩部虛擬機器
* Analysis Services、Reporting Services 和 SQL Server Database Engine 在單一虛擬機器上。 此部署包含報表伺服器資料庫。
* 資料來源在第二部 VM 上。 第二部 VM 包含 SQL Server Database Engine 當做資料來源。

![具有 2 部虛擬機器的 BI IaaS 案例](./media/virtual-machines-windows-classic-ps-sql-bi/IC650109.gif)

### <a name="mixed-azure--data-on-azure-sql-database"></a>混合的 Azure - 資料在 Azure SQL 資料庫上
* Analysis Services、Reporting Services 和 SQL Server Database Engine 在單一虛擬機器上。 此部署包含報表伺服器資料庫。
* 資料來源是 Azure SQL 資料庫。

![BI IaaS 案例 VM 以及 AzureSQL 做為資料來源](./media/virtual-machines-windows-classic-ps-sql-bi/IC650110.gif)

### <a name="hybrid-data-on-premises"></a>混合 - 資料內部部署
* 在此範例部署中，Analysis Services、Reporting Services 和 SQL Server Database Engine 是在單一虛擬機器上執行。 虛擬機器會裝載報表伺服器資料庫。 虛擬機器會透過 Azure 虛擬網路或一些其他 VPN 通道解決方案加入內部部署網域。
* 資料來源是在內部部署。

![BI IaaS 案例 VM 以及內部部署資料來源](./media/virtual-machines-windows-classic-ps-sql-bi/IC654384.gif)

## <a name="reporting-services-native-mode-configuration"></a>Reporting Services 原生模式組態
SQL Server 的虛擬機器資源庫映像包含 Reporting Services 原生模式安裝，但是未設定報表伺服器。 本小節中的步驟會設定 Reporting Services 報表伺服器。 如需有關詳細設定 Reporting Services 原生模式的詳細資訊，請參閱 [安裝 Reporting Services 原生模式報表伺服器 (SSRS)](https://msdn.microsoft.com/library/ms143711.aspx)。

> [!NOTE]
> 如需使用 Windows PowerShell 指令碼來設定報表伺服器的類似內容，請參閱 [使用 PowerShell 建立具有原生模式報表伺服器的 Azure VM](../classic/ps-sql-report.md)。

### <a name="connect-to-the-virtual-machine-and-start-the-reporting-services-configuration-manager"></a>連接到虛擬機器並啟動 Reporting Services 組態管理員
連接到 Azure 虛擬機器有兩個常見的工作流程：

* 若要連接，請按一下虛擬機器的名稱，然後按一下 [連接] 。 遠端桌面連線會開啟，並自動填入電腦名稱。
  
    ![連接至 Azure 虛擬機器](./media/virtual-machines-windows-classic-ps-sql-bi/IC650112.gif)
* 透過 Windows 遠端桌面連線連接到虛擬機器。 在遠端桌面的使用者介面中：
  
  1. 輸入 **雲端服務名稱** 做為電腦名稱。
  2. 輸入冒號 (:) 以及針對 TCP 遠端桌面端點設定的公用連接埠號碼。
     
      Myservice.cloudapp.net:63133
     
      如需詳細資訊，請參閱 [什麼是雲端服務？](https://azure.microsoft.com/manage/services/cloud-services/what-is-a-cloud-service/)。


**啟動 Reporting Services 組態管理員**

在 **Windows Server 2012/2016** 中：

1. 從 [開始] 畫面上，輸入 **Reporting Services** 來查看應用程式的清單。
2. 以滑鼠右鍵按一下 [Reporting Services 組態管理員]，然後按一下 [以系統管理員身分執行]。

在 **Windows Server 2008 R2**中：

1. 按一下 [開始]，然後按一下 [所有程式]。
2. 按一下 [Microsoft SQL Server 2016] 。
3. 按一下 [組態工具] 。
4. 以滑鼠右鍵按一下 [Reporting Services 組態管理員]，然後按一下 [以系統管理員身分執行]。

或：

1. 按一下 [啟動] 。
2. 在 [搜尋程式與檔案] 對話方塊中，輸入 **reporting services**。 如果 VM 執行 Windows Server 2012，請在 Windows Server 2012 [開始] 畫面輸入 **reporting services** 。
3. 以滑鼠右鍵按一下 [Reporting Services 組態管理員]，然後按一下 [以系統管理員身分執行]。
   
    ![搜尋 ssrs 組態管理員](./media/virtual-machines-windows-classic-ps-sql-bi/IC650113.gif)

### <a name="configure-reporting-services"></a>設定 Reporting Services
**服務帳戶及 Web 服務 URL：**

1. 確認 [伺服器名稱] 為本機伺服器名稱，然後按一下 [連接]。
2. 記下空白的 **報表伺服器資料庫名稱**。 組態完成時會建立資料庫。
3. 確認 [報表伺服器狀態] 是 [已啟動]。 如果您想要在 Windows 伺服器管理員中確認服務，服務便是 **SQL Server Reporting Services** Windows 服務。
4. 按一下 [服務帳戶] 並視需要變更帳戶。 如果虛擬機器是用在非加入網域的環境中，內建的 **ReportServer** 帳戶便已足夠。 如需有關服務帳戶的詳細資訊，請參閱[服務帳戶](https://msdn.microsoft.com/library/ms189964.aspx)。
5. 按一下左窗格中的 [ **Web 服務 URL** ]。
6. 按一下 [套用]  來設定預設值。
7. 記下 **報表伺服器 Web 服務的 URL**。 請注意，預設 TCP 連接埠是 80 且屬於 URL。 在稍後步驟中，您可以為連接埠建立 Microsoft Azure 虛擬機器端點。
8. 在 [結果]  窗格中，確認動作已順利完成。

**資料庫：**

1. 按一下左窗格中的 [資料庫]  。
2. 按一下 [變更資料庫] 。
3. 確認 [建立新的報表伺服器資料庫]  選取，然後按 [下一步]。
4. 確認 [伺服器名稱]，然後按一下 [測試連接]。
5. 如果結果是 [測試連接成功]，請按一下 [確定]，然後按 [下一步]。
6. 請注意，資料庫名稱是 **ReportServer**，而 [報表伺服器模式] 是**原生**，然後按 [下一步]。
7. 按一下 [開始]  on the  。
8. 按一下 [開始]  on the  。
9. 按一下 [開始]  on the  。

**入口網站 URL 或 2012 和 2014 版的報表管理員 URL：**

1. 在左窗格中，按一下 [Web Portal URL] \(入口網站 URL) 或 2012 和 2014 版的 [報表管理員 URL]。
2. 按一下 [套用]。
3. 在 [結果]  窗格中，確認動作已順利完成。
4. 按一下 [結束] 。

如需報表伺服器權限的詳細資訊，請參閱 [在原生模式報表伺服器上授與權限](https://msdn.microsoft.com/library/ms156014.aspx)。

### <a name="browse-to-the-local-report-manager"></a>瀏覽至本機報表管理員
若要驗證組態，瀏覽至 VM 上的報表管理員。

1. 在 VM 上，以系統管理員權限啟動 Internet Explorer。
2. 在 VM 上瀏覽至 http://localhost/reports。

### <a name="to-connect-to-remote-web-portal-or-report-manager-for-2014-and-2012"></a>若要連接遠端入口網站或 2012 和 2014 版的報表管理員
如果您想要從遠端電腦連接到虛擬機器上的入口網站或 2012 和 2014 版報表管理員，請建立新的虛擬機器 TCP 端點。 根據預設，報表伺服器會接聽 **連接埠 80**上的 HTTP 要求。 如果您將報表伺服器 URL 設定為使用不同的連接埠，您必須在下列指示中指定該連接埠編號。

1. 為虛擬機器的 TCP 連接埠 80 建立端點。 如需詳細資訊，請參閱這份文件中的 [虛擬機器端點和防火牆連接埠](#virtual-machine-endpoints-and-firewall-ports) 一節。
2. 在虛擬機器防火牆中開啟連接埠 80。
3. 使用 Azure 虛擬機器 **DNS 名稱** 做為 URL 中的伺服器名稱，瀏覽至入口網站或報表管理員。 例如︰
   
    **報表伺服器**：http://uebi.cloudapp.net/reportserver**Web 入口網站**：http://uebi.cloudapp.net/reports
   
    [為報表伺服器存取設定防火牆](https://msdn.microsoft.com/library/bb934283.aspx)

### <a name="to-create-and-publish-reports-to-the-azure-virtual-machine"></a>建立並將報表發佈至 Azure 虛擬機器
下表摘要列出一些選項，可將現有報表從內部部署電腦發佈至 Microsoft Azure 虛擬機器上託管的報表伺服器：

* **報表產生器**：虛擬機器包含適用於 SQL 2012 和 2014 的 Click Once 版本 Microsoft SQL Server 報表產生器。 若要在虛擬機器上首次啟動搭配 SQL 2016 的報表產生器：
  
  1. 以管理權限啟動瀏覽器。
  2. 瀏覽至虛擬機器上的入口網站，並選取右上角的 **下載** 圖示。
  3. 選取 [報表產生器] 。
     
     如需詳細資訊，請參閱 [啟動報表產生器](https://msdn.microsoft.com/library/ms159221.aspx)。
* **SQL Server Data Tools**：VM：SQL Server Data Tools 安裝在虛擬機器上，並可用在虛擬機器上建立**報表伺服器專案**和報表。 SQL Server Data Tools 可將報表發佈至虛擬機器上的報表伺服器。
* **SQL Server Data Tools：遠端**：在本機電腦上，以 SQL Server Data Tools 建立含有 Reporting Services 報告的 Reporting Services 專案。 設定專案以連接至 Web 服務 URL。
  
    ![SSRS 專案的 ssdt 專案屬性](./media/virtual-machines-windows-classic-ps-sql-bi/IC650114.gif)
* 建立包含報表的 .VHD 硬碟機，然後上傳並連接磁碟機。
  
  1. 在您的本機電腦上建立包含報表的 .VHD 硬碟機。
  2. 建立及安裝管理憑證。
  3. 使用 Add-AzureVHD Cmdlet 將 VHD 檔案上傳至 Azure [建立及上傳 Windows Server VHD 至 Azure](../classic/createupload-vhd.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。
  4. 將磁碟連接至虛擬機器。

## <a name="install-other-sql-server-services-and-features"></a>安裝其他 SQL Server 服務和功能
若要在表格式模式中安裝其他 SQL Server 服務 (例如 Analysis Services)，請執行 SQL Server 安裝精靈。 安裝程式檔案是在虛擬機器的本機磁碟上。

1. 按一下 [開始]，然後按一下 [所有程式]。
2. 按一下 [Microsoft SQL Server 2016]、[Microsoft SQL Server 2014] 或 [Microsoft SQL Server 2012]，然後按一下 [組態工具]。
3. 按一下 [SQL Server 安裝中心] 。

或執行 C:\SQLServer_13.0_full\setup.exe、C:\SQLServer_12.0_full\setup.exe 或 C:\SQLServer_11.0_full\setup.exe

> [!NOTE]
> 第一次執行 SQL Server 安裝程式時，可能會下載更多安裝檔，且需要將虛擬機器重新開機和重新啟動 SQL Server 安裝程式。
> 
> 如果您需要重複自訂從 Microsoft Azure 虛擬機器選取的映像，請考慮建立您自己的 SQL Server 映像。 Analysis Services SysPrep 功能是隨著 SQL Server 2012 SP1 CU2 啟用。 如需詳細資訊，請參閱[使用 SysPrep 安裝 SQL Server 的考量](https://msdn.microsoft.com/library/ee210754.aspx)和 [Sysprep Support for Server Roles (伺服器角色的 Sysprep 支援)](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)。
> 
> 

### <a name="to-install-analysis-services-tabular-mode"></a>安裝 Analysis Services 表格式模式
本節中的步驟 **摘要** Analysis Services 表格式模式安裝。 如需詳細資訊，請參閱下列：

* [以表格式模式安裝 Analysis Services](https://msdn.microsoft.com/library/hh231722.aspx)
* [表格式模型化 (Adventure Works 教學課程)](https://msdn.microsoft.com/library/140d0b43-9455-4907-9827-16564a904268)

**若要安裝 Analysis Services 表格式模式：**

1. 在 SQL Server 安裝精靈中，按一下左窗格中的 [安裝]，然後按一下 [新的 SQL 伺服器安裝或將功能加入到現有安裝]。
   
   * 如果您看到 [瀏覽資料夾]，請瀏覽至 c:\SQLServer_13.0_full、c:\SQLServer_12.0_full 或 c:\SQLServer_11.0_full，然後按一下 [確定]。
2. 在產品更新頁面上，按 [下一步]  。
3. 在 [安裝類型] 頁面上，選取 [執行 SQL Server 的新安裝]，然後按 [下一步]。
4. 在 [安裝程式角色] 頁面上，按一下 [SQL Server 功能安裝]。
5. 在 [特徵選取] 頁面上，按一下 [Analysis Services]。
6. 在 [執行個體組態] 頁面上，輸入描述性名稱 (例如**表格式**) 到 [具名執行個體] 和 [執行個體識別碼] 文字方塊。
7. 在 [Analysis Services 組態] 頁面上，選取 [表格式模式]。 將目前使用者加入至管理權限清單。
8. 完成並關閉 SQL Server 安裝精靈。

## <a name="analysis-services-configuration"></a>Analysis Services 組態
### <a name="remote-access-to-analysis-services-server"></a>遠端存取 Analysis Services 伺服器
Analysis Services 伺服器僅支援 Windows 驗證。 若要從用戶端應用程式 (例如 SQL Server Management Studio 或 SQL Server Data Tools) 遠端存取 Analysis Services，虛擬機器必須使用 Azure 虛擬網路加入至您的本機網域。 如需詳細資訊，請參閱 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md)。

Analysis Services 的**預設執行個體**會接聽 TCP 連接埠 **2383**。 在虛擬機器防火牆中開啟連接埠。 Analysis Services 的叢集具名執行個體也會接聽連接埠 **2383**。

針對 Analysis Services 的 **具名執行個體** ，需要 SQL Server Browser 服務才能管理連接埠存取。 SQL Server Browser 預設組態為連接埠 **2382**。

在虛擬機器防火牆中，開啟連接埠 **2382** 並建立靜態 Analysis Services 具名執行個體連接埠。

1. 若要確認 VM 上已在使用的連接埠，以及正在使用連接埠的處理序，請以管理權限執行下列命令：
   
        netstat /ao
2. 使用 SQL Server Management Studio，藉由更新表格式 AS 執行個體一般屬性中的「連接埠」值來建立靜態的 Analysis Services 具名執行個體連接埠。 如需詳細資訊，請參閱 [設定 Windows 防火牆以允許 Analysis Services 存取](https://msdn.microsoft.com/library/ms174937.aspx#bkmk_fixed)中的＜對預設或具名執行個體使用固定的連接埠＞。
3. 重新啟動 Analysis Services 服務的表格式執行個體。

如需詳細資訊，請參閱這份文件中的 **虛擬機器端點和防火牆連接埠** 一節。

## <a name="virtual-machine-endpoints-and-firewall-ports"></a>虛擬機器端點和防火牆連接埠
本節摘要說明要建立的 Microsoft Azure 虛擬機器端點，以在虛擬機器防火牆中開啟的連接埠。 下表摘要說明要建立端點的 **TCP** 連接埠，以及要在虛擬機器防火牆中開啟的連接埠。

* 如果您使用單一 VM，而下列兩項條件成立，您不需要建立 VM 端點，並且不需要在 VM 上的防火牆開啟連接埠。
  
  * 您未從遠端存取連接到 VM 上的 SQL Server 功能。 建立對 VM 的遠端桌面連線並存取 VM 本機上的 SQL Server 功能，不會被視為對 SQL Server 功能的遠端連線。
  * 您未透過 Azure 虛擬網路或其他 VPN 通道解決方案將 VM 加入內部部署網域。
* 如果虛擬機器未加入網域，但您想要從遠端連接到 VM 上的 SQL Server 功能：
  
  * 在 VM 上開啟防火牆的連接埠。
  * 為標示的連接埠 (*) 建立虛擬機器端點。
* 如果虛擬機器是使用 VPN 通道 (例如 Azure 虛擬網路) 加入網域，那麼並不需要端點。 不過，請在 VM 上的防火牆中開啟連接埠。
  
  | Port | 類型 | 說明 |
  | --- | --- | --- |
  | **80** |TCP |報表伺服器遠端存取 (\*)。 |
  | **1433** |TCP |SQL Server Management Studio (\*)。 |
  | **1434** |UDP |SQL Server Browser。 在 VM 加入網域時所需。 |
  | **2382** |TCP |SQL Server Browser。 |
  | **2383** |TCP |SQL Server Analysis Services 預設執行個體和叢集具名執行個體。 |
  | **使用者定義** |TCP |為您選擇的連接埠編號建立靜態 Analysis Services 具名執行個體連接埠，然後在防火牆中解除封鎖連接埠號碼。 |

如需有關如何建立端點的詳細資訊，請參閱下列各項：

* 建立端點：[如何設定虛擬機器的端點](../classic/setup-endpoints.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。
* SQL Server：請參閱 [在 Azure 上佈建 SQL Server 虛擬機器](../sql/virtual-machines-windows-portal-sql-server-provision.md)的＜完成在另一部電腦上使用 SQL Server Management Studio 連接到虛擬機器的組態步驟＞一節。

下圖說明要在 VM 防火牆中開啟，以允許遠端存取 VM 上的功能和元件的連接埠。

![要在 Azure VM 中為 BI 應用程式開啟的連接埠](./media/virtual-machines-windows-classic-ps-sql-bi/IC654385.gif)

## <a name="resources"></a>資源
* 檢閱 Azure 虛擬機器環境中使用的 Microsoft 伺服器軟體的支援原則。 下列主題摘要說明 BitLocker、容錯移轉叢集和網路負載平衡等功能的支援。 [Azure 虛擬機器的 Microsoft 伺服器軟體支援](http://support.microsoft.com/kb/2721672)。
* [Azure 虛擬機器上的 SQL Server 概觀](../sql/virtual-machines-windows-sql-server-iaas-overview.md)
* [虛擬機器](https://azure.microsoft.com/documentation/services/virtual-machines/)
* [在 Azure 上佈建 SQL Server 虛擬機器](../sql/virtual-machines-windows-portal-sql-server-provision.md)
* [如何將資料磁碟連接至虛擬機器](../classic/attach-disk-classic.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)
* [將資料庫移轉至 Azure VM 上的 SQL Server](../sql/virtual-machines-windows-migrate-sql.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fsqlclassic%2ftoc.json)
* [判斷 Analysis Services 執行個體的伺服器模式](https://msdn.microsoft.com/library/gg471594.aspx)
* [多維度模型化 (Adventure Works 教學課程)](https://technet.microsoft.com/library/ms170208.aspx)
* [Azure 文件中心](https://azure.microsoft.com/documentation/)
* [在混合式環境中使用 Power BI](https://msdn.microsoft.com/library/dn798994.aspx)

> [!NOTE]
> [透過 Microsoft SQL Server Connect 提交意見和連絡資訊](https://connect.microsoft.com/SQLServer/Feedback)

### <a name="community-content"></a>社群內容
* [使用 PowerShell 管理 Azure SQL Database](https://azure.microsoft.com/blog/windows-azure-sql-database-management-with-powershell/)

