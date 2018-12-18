---
title: SQL VM 到 Azure 搜尋服務的連線 | Microsoft Docs
description: 啟用加密的連線並設定防火牆，以允許在 Azure 虛擬機器 (VM) 上從 Azure 搜尋服務的索引子連接到 SQL Server。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 01/23/2017
ms.author: heidist
ms.openlocfilehash: 7800e83891cb336bb896299b8fd4d6b3ba590178
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34366455"
---
# <a name="configure-a-connection-from-an-azure-search-indexer-to-sql-server-on-an-azure-vm"></a>在 Azure VM 上設定從 Azure 搜尋服務索引子到 SQL Server 的連線
如[使用索引子將 Azure SQL Database 連接至 Azure 搜尋服務](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#faq)中所述，Azure 搜尋服務支援針對 **Azure VM 上的 SQL Server** (或簡稱 **SQL Azure VM**) 建立索引子，但必須先處理幾個安全性相關的必要條件。 

**工作持續時間︰** 約 30 分鐘，假設您已經在 VM 上安裝憑證。

## <a name="enable-encrypted-connections"></a>啟用加密的連線
Azure 搜尋服務會針對透過公用網際網路連接的所有索引子要求加密的通道。 本節列出執行這項工作的步驟。

1. 檢查憑證的屬性，以驗證主體名稱為 Azure VM 的完整網域名稱 (FQDN)。 您可以使用 CertUtils 之類的工具或憑證嵌入式管理單元來檢視屬性。 您可以在 [Azure 入口網站](https://portal.azure.com/)的 [公用 IP 位址/DNS 名稱標籤] 欄位中，從 VM 服務刀鋒視窗的 [程式集] 區段中取得 FQDN。
   
   * 對於使用較新的 **Resource Manager** 範本建立的 VM，FQDN 的格式為 `<your-VM-name>.<region>.cloudapp.azure.com`。 
   * 對於建立為**傳統** VM 的較舊 VM，FQDN 的格式為 `<your-cloud-service-name.cloudapp.net>`。 
2. 使用登錄編輯器 (regedit) 設定 SQL Server 以使用憑證。 
   
    雖然通常會在這項工作中使用 SQL Server 組態管理員，但您在此案例中無法使用它。 它不會尋找匯入的憑證，因為在 Azure 上 VM 的 FQDN 不符合由 VM 所決定的 FQDN (它會將網域識別為本機電腦或其已加入的網路網域)。 當名稱不相符時，使用 regedit 來指定憑證。
   
   * 在 regedit 中，瀏覽此登錄金鑰︰`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\[MSSQL13.MSSQLSERVER]\MSSQLServer\SuperSocketNetLib\Certificate`。
     
     `[MSSQL13.MSSQLSERVER]` 部分將視版本和執行個體名稱而有所不同。 
   * 將**憑證**金鑰的值設定為您匯入 VM 的 SSL 憑證**指紋**。
     
     有數種方式可以取得指紋，有些方式較其他的更好。 如果您將它從 MMC 中的 **憑證** 嵌入式管理單元進行複製，您可能會挑選不可見的開頭字元 ( [如本支援文章中所述](https://support.microsoft.com/kb/2023869/))，如此當您嘗試連接時便會導致錯誤。 有數個因應措施可修正此問題。 最簡單的方法是按退格鍵，然後重新輸入指紋的第一個字元，在 regedit 中移除金鑰值欄位中的開頭字元。 或者，您可以使用不同的工具來複製指紋。
3. 授與權限給服務帳戶。 
   
    務必要將 SSL 憑證私密金鑰的適當權限授與 SQL Server 服務帳戶。 如果您忽略此步驟，SQL Server 將會無法啟動。 針對這項工作，您可以使用**憑證**嵌入式管理單元或 **CertUtils**。
4. 重新啟動 SQL Server 服務。

## <a name="configure-sql-server-connectivity-in-the-vm"></a>在 VM 中設定 SQL Server 連線
在您設定由 Azure 搜尋服務所要求的加密連接之後，Azure VM 上的 SQL Server 內建函式會有額外設定步驟。 如果您尚未這麼做，則下一個步驟是使用這些文件其中之一來完成設定︰

* 若為 **Resource Manager** VM，請參閱 [使用 Resource Manager 連接到 Azure 上的 SQL Server 虛擬機器](../virtual-machines/windows/sql/virtual-machines-windows-sql-connect.md)。 
* 若為 **傳統** VM，請參閱 [連線到 Azure 上的 SQL Server 虛擬機器](../virtual-machines/windows/classic/sql-connect.md)。

特別是，檢閱每個文章中的「透過網際網路連接」一節。

## <a name="configure-the-network-security-group-nsg"></a>設定網路安全性群組 (NSG)
設定 NSG 與相對應的 Azure 端點或存取控制清單 (ACL)，讓他人能夠存取您的 Azure VM。 您很可能過去已完成此設定，允許自己的應用程式邏輯連接至您的 SQL Azure VM。 這和 Azure 搜尋服務連線至 SQL Azure VM 並無不同。 

下列連結提供 VM 部署的 NSG 組態相關指示。 使用下列 Azure 搜尋服務端點根據其 IP 位址的 ACL 指示。

> [!NOTE]
> 若為背景，請參閱 [什麼是網路安全性群組？](../virtual-network/security-overview.md)
> 
> 

* 若為 **Resource Manager** VM，請參閱 [如何針對 ARM 部署建立 NSG](../virtual-network/tutorial-filter-network-traffic.md)。 
* 若為 **傳統** VM，請參閱 [如何針對傳統部署建立 NSG](../virtual-network/virtual-networks-create-nsg-classic-ps.md)。

IP 位址可能會造成一些挑戰，如果您知道問題和可能的因應措施則可輕易克服。 其餘各節提供與 ACL 中的 IP 位址相關的問題處理建議。

#### <a name="restrict-access-to-the-search-service-ip-address"></a>限制對搜尋服務 IP 位址的存取
強烈建議您在 ACL 中限制對搜尋服務 IP 位址的存取，而不是讓 SQL Azure VM 對任何連接要求來者不拒。 您可以 Ping 搜尋服務的 FQDN (例如， `<your-search-service-name>.search.windows.net`)，輕鬆地找出 IP 位址。

#### <a name="managing-ip-address-fluctuations"></a>管理 IP 位址的變動
如果您的搜尋服務只有一個搜尋單位 (也就是有一個複本和一個分割區)，在例行服務重新啟動期間，IP 位址會變更，使用您搜尋服務的 IP 位址讓現有的 ACL 失效。

要避免後續連線能力錯誤的方法之一，是在 Azure 搜尋服務中使用多個複本和一個資料分割。 這樣做會增加成本，但它也可以解決 IP 位址的問題。 在 Azure 搜尋服務中，當您有多個搜尋單位時，IP 位址不會變更。

第二個方法是讓連線失敗，然後再重新設定 NSG 中的 ACL。 平均而言，您可以預期 IP 位址每隔幾週會進行變更。 對於頻繁地進行控制編製索引的客戶，此方法可能可行。

第三個可行 (但並不特別安全) 的方法，是針對搜尋服務佈建所在的 Azure 區域，指定 IP 位址範圍。 將公用 IP 位址配置給 Azure 資源的 IP 範圍清單已發佈於 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。 

#### <a name="include-the-azure-search-portal-ip-addresses"></a>包含 Azure 搜尋服務入口網站 IP 位址
如果您使用 Azure 入口網站建立索引子，Azure 搜尋服務入口網站邏輯也需要在建立期間存取您的 SQL Azure VM。 Ping `stamp2.search.ext.azure.com`，即可找到 Azure 搜尋服務入口網站 IP 位址。

## <a name="next-steps"></a>後續步驟
解決組態之後，您現在就可以在 Azure VM 指定 SQL Server 作為 Azure 搜尋服務索引子的資料來源。 如需詳細資訊，請參閱 [使用索引子將 Azure SQL Database 連接至 Azure 搜尋服務](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md) 。

