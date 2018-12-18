---
title: 如何更新雲端服務 | Microsoft Docs
description: 了解如何在 Azure 中更新雲端服務。 了解如何繼續進行雲端服務更新來確保可用性。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: c6a8b5e6-5c99-454c-9911-5c7ae8d1af63
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/19/2017
ms.author: jeconnoc
ms.openlocfilehash: 2f5a82fac18ab34bfa9d6b46f553227ed44a994a
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39008088"
---
# <a name="how-to-update-a-cloud-service"></a>如何更新雲端服務

更新雲端服務 (包括其角色和客體 OS) 是一個包含三步驟的程序。 首先，必須上傳新雲端服務或作業系統版本的二進位檔和組態檔案。 接著，Azure 會根據新雲端服務版本的需求，保留雲端服務的計算和網路資源。 最後，Azure 會執行輪流升級，以累加方式將租用戶更新為新版本或客體 OS，同時保留您的可用性。 本文將討論最後一個步驟 (輪流升級) 的細節。

## <a name="update-an-azure-service"></a>更新 Azure 服務
Azure 會將您的角色執行個體組織成名為升級網域 (UD) 的邏輯群組。 升級網域 (UD) 是角色執行個體的邏輯集合，會以群組方式進行更新。  Azure 會一次更新一個 UD 的一個雲端服務，讓其他 UD 中的執行個體能夠繼續處理流量。

預設的升級網域數目為 5。 您可以在服務的定義檔 (.csdef) 中包含 upgradeDomainCount 屬性，以指定不同數目的升級網域。 如需有關 upgradeDomainCount 屬性的詳細資訊，請參閱 [WebRole 結構描述](https://msdn.microsoft.com/library/azure/gg557553.aspx)或 [WorkerRole 結構描述](https://msdn.microsoft.com/library/azure/gg557552.aspx)。

當您在服務中執行一或多個角色的就地更新時，Azure 會根據所屬的升級網域來更新角色執行個體集合。 Azure 會更新指定的升級網域中的所有執行個體 (予以停止、更新、重新上線)，然後移到下一個網域。 Azure 只會停止在目前升級網域中執行的執行個體，以確保更新儘可能對執行中的服務造成最小的影響。 如需詳細資訊，請參閱本文後面的 [如何繼續進行更新](#howanupgradeproceeds) 。

> [!NOTE]
> 雖然 Azure 內容中的**更新**和**升級**有稍微不同的含義，但是在本文件中，這兩個詞彙可交替用於處理序和功能說明中。
>
>

您的服務必須定義一個角色至少有兩個執行個體，以便該角色在不需停機的狀況下進行就地升級。 如果服務僅由一個角色的一個執行個體組成，您的服務要等到就地更新完成才可使用。

本主題涵蓋下列有關 Azure 更新的資訊：

* [在更新期間允許的服務變更](#AllowedChanges)
* [如何繼續進行升級](#howanupgradeproceeds)
* [復原更新](#RollbackofanUpdate)
* [在進行中的部署上起始多項變更作業](#multiplemutatingoperations)
* [將角色散發於升級網域](#distributiondfroles)

<a name="AllowedChanges"></a>

## <a name="allowed-service-changes-during-an-update"></a>在更新期間允許的服務變更
下表顯示在更新期間允許的服務變更：

| 允許對主機、服務和角色進行的變更 | 就地更新 | 預備 (VIP 交換) | 刪除並重新部署 |
| --- | --- | --- | --- |
| 作業系統版本 |是 |yes |是 |
| .NET 信任等級 |是 |yes |是 |
| 虛擬機器大小<sup>1</sup> |是<sup>2</sup> |是 |是 |
| 本機儲存體設定 |只會增加<sup>2</sup> |是 |是 |
| 在服務中新增或移除角色 |是 |yes |是 |
| 特定角色的執行個體數目 |是 |yes |是 |
| 服務端點的數目或類型 |是<sup>2</sup> |否 |是 |
| 組態設定的名稱和值 |是 |yes |是 |
| 組態設定的值 (而不是名稱) |是 |yes |是 |
| 加入新憑證 |是 |yes |是 |
| 變更現有的憑證 |是 |yes |是 |
| 部署新程式碼 |是 |yes |是 |

<sup>1</sup> 大小變更已限制為可供雲端服務使用的大小子集。

<sup>2</sup> 需要 Azure SDK 1.5 或更新版本。

> [!WARNING]
> 變更虛擬機器大小將會損毀本機資料。
>
>

在更新期間不支援下列項目：

* 變更角色的名稱。 移除角色，而後加入採用新名稱的角色。
* 變更升級網域計數。
* 減少本機資源的大小。

如果您對服務的定義進行其他更新 (例如降低本機資源的大小)，則必須改為執行 VIP 交換更新。 如需詳細資訊，請參閱 [交換部署](https://msdn.microsoft.com/library/azure/ee460814.aspx)。

<a name="howanupgradeproceeds"></a>

## <a name="how-an-upgrade-proceeds"></a>如何繼續進行升級
您可以決定要更新服務中的所有角色或更新服務中的單一角色。 在任一情況下，正在升級且屬於第一個升級網域的每個角色的所有執行個體都會停止、升級並重新上線。 一旦重新上線，第二個升級網域中的執行個體會停止、升級並重新上線。 雲端服務一次最多可有一個作用中的升級。 升級作業一律針對最新版本的雲端服務執行。

下圖說明如果您要升級服務中的所有角色，如何繼續進行升級：

![升級服務](media/cloud-services-update-azure-service/IC345879.png "升級服務")

下圖說明如果您只要升級單一角色，如何繼續進行更新：

![升級角色](media/cloud-services-update-azure-service/IC345880.png "升級角色")  

在自動更新期間，Azure 網狀架構控制器會定期評估雲端服務的健康狀態，以判斷何時可放心進行下一個 UD。 此健康狀態評估會根據每一個角色執行，而且只考慮最新版本中的執行個體 (也就是 UD 中已進行的執行個體)。 它會確認每個角色的最少角色執行個體是否已達到令人滿意的終止狀態。

### <a name="role-instance-start-timeout"></a>角色執行個體啟動逾時
網狀架構控制器會等待 30 分鐘，讓每個角色執行個體達到啟動狀態。 當逾時期間過去後，網狀架構控制器會繼續前進到下一個角色執行個體。

### <a name="impact-to-drive-data-during-cloud-service-upgrades"></a>雲端服務升級期間對磁碟機資料造成的影響

將服務從單一執行個體升級為多個執行個體時，由於 Azure 升級服務的方式，您的服務將會在執行升級時關閉。 保證服務可用性的服務等級協定僅適用於已部署多個執行個體的服務。 下列清單說明每個 Azure 服務升級案例如何影響每個磁碟機上的資料：

|案例|C 磁碟機|D 磁碟機|E 磁碟機|
|--------|-------|-------|-------|
|VM 重新啟動|保留|保留|保留|
|入口網站重新啟動|保留|保留|終結|
|入口網站重新安裝映像|保留|終結|終結|
|就地升級|保留|保留|終結|
|節點移轉|終結|終結|終結|

請注意，在上述清單中，E: 磁碟機代表角色的根磁碟機，不應使用硬式編碼。 請改用 **%RoleRoot%** 環境變數來代表此磁碟機。

若要將升級單一執行個體服務時的停機時間最小化，請將新的多執行個體服務部署至預備伺服器並執行 VIP 交換。

<a name="RollbackofanUpdate"></a>

## <a name="rollback-of-an-update"></a>復原更新
Azure 讓您在 Azure 網狀架構控制器接受初始更新要求後，於服務上起始其他作業，以提供在更新期間管理服務的彈性。 只有當部署上的更新 (組態變更) 或升級處於 **進行中** 狀態時，才能執行復原。 只要服務有至少一個執行個體尚未更新為新版本，更新或升級就會被視為進行中。 若要測試是否允許復原，請檢查[取得部署](https://msdn.microsoft.com/library/azure/ee460804.aspx)和[取得雲端服務屬性](https://msdn.microsoft.com/library/azure/ee460806.aspx)作業所傳回的 RollbackAllowed 旗標值是否設定為 true。

> [!NOTE]
> 只有對 **就地** 更新或升級呼叫 Rollback 才有意義，因為 VIP 交換升級牽涉到以另一個執行個體取代服務的一整個執行中執行個體。
>
>

復原進行中的更新會對部署造成下列影響：

* 不會更新或升級尚未更新或升級為新版本的所有角色執行個體，因為這些執行個體已在執行服務的目標版本。
* 已更新或升級為新版服務封裝 (\*.cspkg) 檔案或服務組態 (\*.cscfg) 檔案 (或這兩個檔案) 的所有角色執行個體，都會還原成這些檔案的升級前版本。

此作用是由下列功能提供：

* [復原更新或升級](https://msdn.microsoft.com/library/azure/hh403977.aspx)作業：只要服務中有至少一個執行個體尚未更新為新版本，即可對組態更新 (藉由呼叫[變更部署組態](https://msdn.microsoft.com/library/azure/ee460809.aspx)來觸發) 或升級 (藉由呼叫[升級部署](https://msdn.microsoft.com/library/azure/ee460793.aspx)來觸發) 呼叫此作業。
* Locked 項目和 RollbackAllowed 項目；這兩個項目可當作[取得部署](https://msdn.microsoft.com/library/azure/ee460804.aspx)和[取得雲端服務屬性](https://msdn.microsoft.com/library/azure/ee460806.aspx)作業的回應本文一部分傳回：

  1. Locked 元素可讓您偵測何時可對指定的部署叫用變更作業。
  2. RollbackAllowed 項目可讓您偵測何時可對指定的部署呼叫 [復原更新或升級](https://msdn.microsoft.com/library/azure/hh403977.aspx) 作業。

  若要執行復原，您不必同時檢查 Locked 和 RollbackAllowed 元素。 就足以確認 RollbackAllowed 已設定為 true。 只有在使用設定為 “x-ms-version: 2011-10-01” 或更新版本的要求標頭叫用這些方法時，才會傳回這些元素。 如需標頭版本控制的詳細資訊，請參閱 [服務管理版本控制](https://msdn.microsoft.com/library/azure/gg592580.aspx)。

有某些情況下，不支援復原更新或升級，如下所示：

* 減少本機資源 - 如果更新增加角色的本機資源，則 Azure 平台不會允許復原。
* 配額限制 - 如果更新是相應減少作業，您可能不再有足夠的計算配額來完成復原作業。 每個 Azure 訂用帳戶都有相關聯的配額，以指定屬於該訂用帳戶的所有託管服務可以取用的核心數目上限。 如果執行指定之更新的復原會讓您的訂用帳戶超出配額，則不會啟用復原。
* 競爭情形 - 如果已完成初始更新，則不可能進行復原。

如果您在手動模式中使用 [升級部署](https://msdn.microsoft.com/library/azure/ee460793.aspx) 作業來控制您 Azure 託管服務的主要就地升級推展速率，復原更新可能很實用。

在升級推展期間，您會在手動模式中呼叫 [升級部署](https://msdn.microsoft.com/library/azure/ee460793.aspx) 並開始處理升級網域。 如果在某個時間點，當您監視升級時，您注意到您檢查的第一個升級網域中有某些角色執行個體變得沒有回應，您可以在部署上呼叫 [復原更新或升級](https://msdn.microsoft.com/library/azure/hh403977.aspx) 作業，讓尚未升級的執行個體維持不變，而讓已升級的執行個體復原為先前的服務封裝和組態。

<a name="multiplemutatingoperations"></a>

## <a name="initiating-multiple-mutating-operations-on-an-ongoing-deployment"></a>在進行中的部署上起始多項變更作業
在某些情況下，您可能想要在進行中的部署上起始多項同時變更作業。 例如，您可能執行服務更新，而當該更新推展於您的服務時，您想要進行一些變更，例如復原更新、套用不同的更新，或甚至刪除部署。 如果服務升級包含會使已升級的角色執行個體反覆損毀的不良程式碼，則可能必須這麼做。 在此情況下，Azure 網狀架構控制器將無法繼續套用該升級，因為已升級網域中的執行個體數目不足屬於正常狀況。 此狀態稱為「停滯部署」 。 復原更新或在失敗的更新之上套用全新的更新，即可脫離停滯部署狀態。

一旦 Azure 網狀架構控制器收到更新或升級服務的初始要求，您就可以開始進行後續的變更作業。 也就是說，您不必等候初始作業完成，即可開始進行另一個變更作業。

在第一個更新正在進行時起始第二個更新作業，將類似於執行復原作業。 如果第二個更新處於自動模式，則第一個升級網域將會立即升級，這可能會導致多個升級網域中的執行個體在同一時間離線。

變更作業如下：[變更部署組態](https://msdn.microsoft.com/library/azure/ee460809.aspx)、[升級部署](https://msdn.microsoft.com/library/azure/ee460793.aspx)、[更新部署狀態](https://msdn.microsoft.com/library/azure/ee460808.aspx)、[刪除部署](https://msdn.microsoft.com/library/azure/ee460815.aspx)及[復原更新或升級](https://msdn.microsoft.com/library/azure/hh403977.aspx)。

[取得部署](https://msdn.microsoft.com/library/azure/ee460804.aspx)和[取得雲端服務屬性](https://msdn.microsoft.com/library/azure/ee460806.aspx)這兩項作業會傳回 Locked 旗標，檢查該旗標可以判斷是否可以在指定的部署上叫用變更作業。

若要呼叫可傳回 Locked 旗標的這些方法的版本，您必須將要求標頭設定為 “x-ms-version: 2011-10-01” 或更新版本。 如需標頭版本控制的詳細資訊，請參閱 [服務管理版本控制](https://msdn.microsoft.com/library/azure/gg592580.aspx)。

<a name="distributiondfroles"></a>

## <a name="distribution-of-roles-across-upgrade-domains"></a>將角色散發於升級網域
Azure 會將角色的執行個體平均分散於一組升級網域，而升級網域可設定為服務定義 (.csdef) 檔案的一部分。 升級網域數目上限為 20，其預設值為 5。 如需如何修改服務定義檔的詳細資訊，請參閱 [Azure 服務定義結構描述 (.csdef 檔)](cloud-services-model-and-package.md#csdef)。

例如，如果您的角色有 10 個執行個體，依預設每個升級網域包含 2 個執行個體。 如果您的角色有 14 個執行個體，則其中 4 個升級網域包含 3 個執行個體，而第 5 個網域包含 2 個執行個體。

升級網域會使用以零為基底的索引進行識別：第一個升級網域的識別碼為 0，而第二個升級網域的識別碼為 1，依此類推。

下圖說明當服務定義兩個升級網域時，如何散發包含兩個角色的服務。 此服務正在執行 Web 角色的 8 個執行個體以及背景工作角色的 9 個執行個體。

![升級網域的分配](media/cloud-services-update-azure-service/IC345533.png "升級網域的分配")

> [!NOTE]
> 請注意，Azure 會控制執行個體配置於升級網域的方式。 您無法指定哪些執行個體會配置給哪一個網域。
>
>

## <a name="next-steps"></a>後續步驟
[如何管理雲端服務](cloud-services-how-to-manage-portal.md)  
[如何監視雲端服務](cloud-services-how-to-monitor.md)  
[如何設定雲端服務](cloud-services-how-to-configure-portal.md)  
