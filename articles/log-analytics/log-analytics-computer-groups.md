---
title: Azure Log Analytics 記錄搜尋中的電腦群組 | Microsoft Docs
description: Log Analytics 中的電腦群組可讓您將記錄檔搜尋範圍限於一組特定的電腦。  這篇文章說明可用來建立電腦群組的不同方法，以及如何將它們用在記錄檔搜尋中。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: a28b9e8a-6761-4ead-aa61-c8451ca90125
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/03/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 23ac75b4afb07c3f8d5f0d90755a5cf2087087e1
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37130670"
---
# <a name="computer-groups-in-log-analytics-log-searches"></a>Log Analytics 記錄檔搜尋中的電腦群組

Log Analytics 中的電腦群組中可讓您將[記錄檔搜尋](log-analytics-log-search-new.md)範圍限於一組特定的電腦。  使用您所定義的查詢，或從不同來源匯入群組，將電腦填入每個群組中。  當記錄檔搜尋包含群組時，結果就僅限於與群組中的電腦相符的記錄。

## <a name="creating-a-computer-group"></a>建立電腦群組
您可以使用下表的任何方法，在 Log Analytics 中建立電腦群組。  下列各節提供每個方法的詳細資料。 

| 方法 | 說明 |
|:--- |:--- |
| 記錄搜尋 |建立記錄檔搜尋來傳回電腦清單。 |
| 記錄檔搜尋 API |使用記錄檔搜尋 API，根據記錄檔搜尋結果，以程式設計方式建立電腦群組。 |
| Active Directory |自動掃描屬於 Active Directory 網域的任何代理程式電腦的群組成員資格，並為每個安全性群組在 Log Analytics 中建立一個群組。 |
| 組態管理員 | 從 System Center Configuration Manager 匯入集合，並在 Log Analytics 中為每個集合建立群組。 |
| Windows Server Update Services |自動掃描 WSUS 伺服器或用戶端來找出目標群組，並為每個群組在 Log Analytics 中建立一個群組。 |

### <a name="log-search"></a>記錄搜尋
從記錄搜尋建立的電腦群組會包含您定義的查詢所傳回的所有電腦。  每次使用電腦群組時都會執行此查詢，因此會反映自建立群組以來的任何變更。  

您可以對電腦群組使用任何查詢，但它必須使用 `distinct Computer` 傳回一組不同的電腦。  以下是您可以用來作為電腦群組的典型搜尋範例。

    Heartbeat | where Computer contains "srv" | distinct Computer

下表描述定義電腦群組的屬性。

| 屬性 | 說明 |
|:---|:---|
| 顯示名稱   | 要在入口網站中顯示的搜尋名稱。 |
| 類別       | 用來在入口網站中組織搜尋的類別。 |
| 查詢          | 電腦群組的查詢。 |
| 函式別名 | 用來識別查詢中電腦群組的唯一別名。 |

使用下列程序在 Azure 入口網站中透過記錄搜尋建立電腦群組。

2. 開啟 [記錄搜尋]，然後按一下畫面頂端的 [已儲存的搜尋]。
3. 按一下 [新增]，然後提供電腦群組的每一個屬性的值。
4. 選取 [將此查詢儲存為電腦群組]，然後按一下 [確定]。



### <a name="active-directory"></a>Active Directory
當您設定匯入 Log Analytics 來匯入 Active Directory 群組成員資格時，它會分析已加入網域且裝有 OMS 代理程式的任何電腦的群組成員資格。  針對 Active Directory 中的每個安全性群組，Log Analytics 中會建立一個電腦群組，而每一部電腦會新增至對應到它們所屬安全性群組的電腦群組。  此成員資格持續地每 4 小時更新一次。  

您可以在 Azure 入口網站中，將 Log Analytics 設定為從 Log Analytics [進階設定] 匯入 Active Directory 安全性群組。  依序選取 [電腦群組]、[Active Directory] 和 [從電腦匯入 Active Directory 群組成員資格]。  不需要進一步的組態。

![來自 Active Directory 的電腦群組](media/log-analytics-computer-groups/configure-activedirectory.png)

匯入群組後，此功能表會列出已偵測到群組成員資格的電腦數目，以及匯入的群組數目。  您可以按一下任一連結，以連同此資訊傳回 **ComputerGroup** 記錄。

### <a name="windows-server-update-service"></a>Windows Server Update Service
當您設定匯入 Log Analytics 來匯入 WSUS 群組成員資格時，它會分析裝有 OMS 代理程式的任何電腦的目標群組成員資格。  如果您使用用戶端目標，則任何連線至 Log Analytics 且屬於任何 WSUS 目標群組的電腦，其群組成員資格都會匯入至 Log Analytics。 如果您使用伺服器端目標，則 WSUS 伺服器上應該安裝 OMS 代理程式，才能將群組成員資格資訊匯入至 Log Analytics。  此成員資格持續地每 4 小時更新一次。 

您可以在 Azure 入口網站中，將 Log Analytics 設定為從 Log Analytics [進階設定] 匯入 WSUS 群組。  依序選取 [電腦群組]、[WSUS] 和 [匯入 WSUS 群組成員資格]。  不需要進一步的組態。

![來自 WSUS 的電腦群組](media/log-analytics-computer-groups/configure-wsus.png)

匯入群組後，此功能表會列出已偵測到群組成員資格的電腦數目，以及匯入的群組數目。  您可以按一下任一連結，以連同此資訊傳回 **ComputerGroup** 記錄。

### <a name="system-center-configuration-manager"></a>System Center Configuration Manager
當您設定 Log Analytics 以匯入 Configuration Manager 集合成員資格時，它會為每個集合建立電腦群組。  每 3 個小時就會擷取一次集合成員資格資訊，以便電腦群組會隨時保持最新狀態。 

您必須先[將 Configuration Manager 連線至 Log Analytics](log-analytics-sccm.md)，才能匯入 Configuration Manager 集合。  接著，您可以在 Azure 入口網站中從 Log Analytics [進階設定] 設定匯入。  依序選取 [電腦群組]、[SCCM] 和 [匯入 Configuration Manager 集合成員資格]。  不需要進一步的組態。

![來自 SCCM 的電腦群組](media/log-analytics-computer-groups/configure-sccm.png)

匯入集合後，此功能表會列出已偵測到群組成員資格的電腦數目，以及匯入的群組數目。  您可以按一下任一連結，以連同此資訊傳回 **ComputerGroup** 記錄。

## <a name="managing-computer-groups"></a>管理電腦群組
您可以從 Azure 入口網站的 Log Analytics [進階設定]，檢視透過記錄搜尋或記錄搜尋 API 所建立的電腦群組。  依序選取 [電腦群組] 和 [已儲存的群組]。  

按一下 [移除] 欄的 [x] 來刪除電腦群組。  按一下群組的 [檢視成員] 圖示，以執行群組的記錄檔搜尋來傳回其成員。  您無法修改電腦群組，而是必須先加以刪除再以修改後的設定重建。

![已儲存的電腦群組](media/log-analytics-computer-groups/configure-saved.png)


## <a name="using-a-computer-group-in-a-log-search"></a>在記錄檔搜尋中使用電腦群組
您可以將電腦群組的別名當作函式，在查詢中使用從記錄檔搜尋建立的電腦群組，所使用的語法一般如下：

  `Table | where Computer in (ComputerGroup)`

例如，您可以使用下列語法，僅傳回 mycomputergroup 電腦群組中之電腦的 UpdateSummary 記錄。
 
  `UpdateSummary | where Computer in (mycomputergroup)`


匯入的電腦群組及其包含的電腦會儲存在 **ComputerGroup** 資料表中。  例如，下列查詢會從 Active Directory 傳回一份網域電腦群組中的電腦清單。 

  `ComputerGroup | where GroupSource == "ActiveDirectory" and Group == "Domain Computers" | distinct Computer`

下列查詢只會針對網域電腦中的電腦，傳回 UpdateSummary 記錄。

  ```
  let ADComputers = ComputerGroup | where GroupSource == "ActiveDirectory" and Group == "Domain Computers" | distinct Computer;
  UpdateSummary | where Computer in (ADComputers)
  ```




## <a name="computer-group-records"></a>電腦群組記錄
針對從 Active Directory 或 WSUS 建立每個電腦群組成員資格，Log Analytics 工作區中會建立一筆記錄。  這些記錄的類型為 **ComputerGroup**，且具有下表中的屬性。  如果電腦群組是根據記錄檔搜尋，則不會建立記錄。

| 屬性 | 說明 |
|:--- |:--- |
| 類型 |*ComputerGroup* |
| SourceSystem |*SourceSystem* |
| 電腦 |成員電腦的名稱。 |
| 群組 |群組的名稱。 |
| GroupFullName |群組的完整路徑，包括來源和來源名稱。 |
| GroupSource |群組的收集來源。 <br><br>ActiveDirectory<br>WSUS<br>WSUSClientTargeting |
| GroupSourceName |群組的收集來源名稱。  對於 Active Directory，這是網域名稱。 |
| ManagementGroupName |SCOM 代理程式的管理群組名稱。  若為其他代理程式，此為 AOI-\<工作區 ID\> |
| TimeGenerated |建立或更新電腦群組的日期和時間。 |

## <a name="next-steps"></a>後續步驟
* 了解 [記錄搜尋](log-analytics-log-searches.md) ，其可分析從資料來源和方案所收集的資料。  

