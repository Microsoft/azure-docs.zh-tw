---
title: 了解 Azure 虛擬機器使用情況 | Microsoft Docs
description: 了解虛擬機器使用情況詳細資料
services: virtual-machines
documentationcenter: ''
author: mmccrory
manager: jeconnoc
editor: ''
tags: azure-virtual-machine
ms.assetid: ''
ms.service: ''
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: vm
ms.workload: infrastructure-services
ms.date: 12/04/2017
ms.author: memccror
ms.openlocfilehash: 9980650008e2d2c2f768dcb3256ea5344119fa13
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31425175"
---
# <a name="understanding-azure-virtual-machine-usage"></a>了解 Azure 虛擬機器使用情況
分析您的 Azure 使用情況資料，可以深入了解使用量，以在整個組織內實現更妥善的成本管理與配置。 本文件會針對您的 Azure 計算使用量詳細資料，提供深入探討。 如需有關一般 Azure 使用情況的詳細資訊，請瀏覽至[了解您的帳單](../../billing/billing-understand-your-bill.md)。

## <a name="download-your-usage-details"></a>下載使用情況詳細資料
若要開始，請[下載使用情況詳細資料](../../billing/billing-download-azure-invoice-daily-usage-date.md)。 下表針對透過 Azure Resource Manager 部署的虛擬機器，提供其使用情況的定義和範例值。 本文件不包含透過傳統模型部署之 VM 的詳細資訊。


| 欄位             | 意義                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | 範例值                                                                                                                                                                                                                                                                                                                                                   |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 使用日期         | 使用資源的日期。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |  “2017/11/23”                                                                                                                                                                                                                                                                                                                                                     |
| 計量識別碼           | 識別這個使用量所屬的最上層服務。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | “虛擬機器”                                                                                                                                                                                                                                                                                                                                               |
| 計量子類別 | 計費的計量識別碼。 <ul><li>針對「計算時數」使用情況，每個 VM 大小 + 作業系統 (Windows、非 Windows) + 區域都會有一個計量。</li><li>針對進階軟體使用情況，則每個軟體類型都有一個計量。 大部分的進階軟體映像都會有適用於每個核心大小的不同計量。 如需詳細資訊，請瀏覽[計算定價頁面](https://azure.microsoft.com/pricing/details/virtual-machines/)。</li></ul>                                                                                                                                                                                                                                                                                                                                         | “2005544f-659d-49c9-9094-8e0aea1be3a5”                                                                                                                                                                                                                                                                                                                           |
| 計量名稱         | 這對於 Azure 中的每個服務都是特定的。 對於計算，它一律是 “計算時數”。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | “計算時數”                                                                                                                                                                                                                                                                                                                                                  |
| 計量區域       | 針對根據資料中心位置定價的某些服務，識別資料中心的位置。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |  “日本東部”                                                                                                                                                                                                                                                                                                                                                       |
| 單位               | 識別服務的計費單位。 計算資源是以每小時計費。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | “小時 ”                                                                                                                                                                                                                                                                                                                                                          |
| 已耗用           | 當日已耗用的資源量。 對於計算，我們會針對 VM 在指定時間內運作的每一分鐘計費 (精確度高達 6 個小數位數)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |    “1”, “0.5”                                                                                                                                                                                                                                                                                                                                                    |
| 資源位置  | 識別正在執行資源的資料中心。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | “日本東部”                                                                                                                                                                                                                                                                                                                                                        |
| 已耗用的服務   | 您所使用的 Azure 平台服務。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | "Microsoft.Compute"                                                                                                                                                                                                                                                                                                                                              |
| 資源群組     | 部署的資源正在其中執行的資源群組。 如需詳細資訊，請參閱 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |    "MyRG"                                                                                                                                                                                                                                                                                                                                                        |
| 執行個體識別碼        | 資源的識別碼。 識別碼包含在建立時為資源指定的名稱。 針對 VM，執行個體識別碼將會包含 SubscriptionId、ResourceGroupName 和 VMName (或用於擴展集的擴展集名稱)。                                                                                                                                                                                                                                                                                                                                                                                                                    | "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM1”<br><br>或<br><br>"/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachineScaleSets/MyVMSS1”                                                                                           |
| 標記               | 您指派給資源的標記。 使用標記為帳單記錄分組。 深入了解如何[標記虛擬機器](tag.md)。 這僅適用於 Resource Manager VM。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | "{"myDepartment":"RD","myUser":"myName"}"                                                                                                                                                                                                                                                                                                                        |
| 其他資訊    | 服務專屬的中繼資料。 針對 VM，我們會在其他資訊欄位中填入下列資訊： <ul><li>您所執行之映像類型專屬的映像。 在 [映像類型] 底下，尋找以下支援字串的完整清單。</li><li>服務類型：您部署的大小。</li><li>VMName：您 VM 的名稱。 這只會針對擴展集 VM 填入。 如果您需要您擴展集 VM 的 VM 名稱，可以在上面的執行個體識別碼字串中找到。</li><li>UsageType：這會指定此項目代表的使用情況類型。<ul><li>ComputeHR 是底層 VM 的「計算時數」使用情況，例如 Standard_D1_v2。</li><li>如果 VM 使用的是進階軟體 (如 Microsoft R Server)，則 ComputeHR_SW 將是進階軟體費用。</li></ul></li></ul>    | 虛擬機器 {"ImageType":"Canonical","ServiceType":"Standard_DS1_v2","VMName":"", "UsageType":"ComputeHR"}<br><br>虛擬機器擴展集 {"ImageType":"Canonical","ServiceType":"Standard_DS1_v2","VMName":"myVM1", "UsageType":"ComputeHR"}<br><br>進階軟體 {"ImageType":"","ServiceType":"Standard_DS1_v2","VMName":"", "UsageType":"ComputeHR_SW"} |

## <a name="image-type"></a>映像類型
對於 Azure 資源庫中的部分映像，系統會在 [其他資訊] 欄位中填入映像類型。 這可讓使用者了解並追蹤他們在虛擬機器上部署的類型。 以您所部署的映像作為基礎，填入此欄位的值如下所示：
  - BitRock 
  - Canonical 
  - FreeBSD 
  - Open Logic 
  - Oracle 
  - 適用於 SAP 的 SLES 
  - Windows Server 2012 R2 Preview 上的 SQL Server 14 Preview 
  - SUSE
  - SUSE Premium
  - StorSimple 雲端設備 
  - Red Hat
  - 適用於 SAP 商務應用程式的 Red Hat     
  - 適用於 SAP HANA 的 Red Hat 
  - Windows 用戶端 BYOL 
  - Windows Server BYOL 
  - Windows Server Preview 

## <a name="service-type"></a>服務類型
[其他資訊] 欄位中的 [服務類型] 欄位會對應至您所部署的確切 VM 大小。 進階儲存體 VM (SSD 型) 和非進階儲存體 VM (HDD 型) 的價格相同。 如果您部署 SSD 型的大小 (例如 Standard\_DS2\_v2)，則會在 [計量子類別] 欄中看到非 SSD 大小 (‘Standard\_D2\_v2 VM’)，並在 [其他資訊] 欄位中看到 SSD 大小 (‘Standard\_DS2\_v2’)。

## <a name="region-names"></a>區域名稱
在使用情況詳細資料中，[資源位置] 欄位內填入的區域名稱不同於 Azure Resource Manager 中使用的區域名稱。 以下是區域值之間的對應：

|    **Resource Manager 區域名稱**       |    **使用情況詳細資料中的資源位置**    |
|--------------------------|------------------------------------------|
|    australiaeast         |    澳大利亞東部                               |
|    australiasoutheast    |    澳大利亞東南部                          |
|    brazilsouth           |    巴西南部                              |
|    CanadaCentral         |    加拿大中部                            |
|    CanadaEast            |    加拿大東部                               |
|    CentralIndia          |    印度中部                            |
|    centralus             |    美國中部                            |
|    chinaeast             |    中國東部                            |
|    chinanorth            |    中國北部                           |
|    eastasia              |    東亞                             |
|    eastus                |    美國東部                               |
|    eastus2               |    美國東部 2                             |
|    GermanyCentral        |    德國中部                            |
|    GermanyNortheast      |    德國東北部                          |
|    japaneast             |    日本東部                               |
|    japanwest             |    日本西部                               |
|    KoreaCentral          |    韓國中部                            |
|    KoreaSouth            |    韓國南部                              |
|    northcentralus        |    美國中北部                      |
|    northeurope           |    北歐                          |
|    southcentralus        |    美國中南部                      |
|    southeastasia         |    東南亞                        |
|    SouthIndia            |    印度南部                              |
|    UKNorth               |    英國北部                              |
|    uksouth               |    英國南部                              |
|    UKSouth2              |    英國南部 2                            |
|    ukwest                |    英國西部                               |
|    USDoDCentral          |    美國國防部中央                        |
|    USDoDEast             |    美國 DoD 東部                           |
|    USGovArizona          |    美國政府亞利桑那州                         |
|    usgoviowa             |    美國政府愛荷華州                            |
|    USGovTexas            |    美國政府德克薩斯州                           |
|    usgovvirginia         |    美國政府維吉尼亞州                        |
|    westcentralus         |    美國中西部                       |
|    westeurope            |    西歐                           |
|    WestIndia             |    印度西部                               |
|    westus                |    美國西部                               |
|    westus2               |    美國西部 2                             |


## <a name="virtual-machine-usage-faq"></a>虛擬機器使用情況常見問題集
### <a name="what-resources-are-charged-when-deploying-a-vm"></a>部署 VM 時，會針對哪些資源收費？    
VM 取得 VM 本身、在 VM 上執行的任何進階軟體、與 VM 相關聯的儲存體帳戶\受控磁碟，以及從 VM 的網路頻寬傳輸等費用。
### <a name="how-can-i-tell-if-a-vm-is-using-azure-hybrid-benefit-in-the-usage-csv"></a>如何判斷 VM 是否使用使用情況 CSV 中的 Azure Hybrid Benefit？
如果您使用 [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) 進行部署，系統會以非 Windows VM 費率向您收取費用，因為您會將自己的授權帶至雲端。 在您的帳單中，您可以分辨哪些 Resource Manager VM 執行的是 Azure Hybrid Benefit，因為它們在 ImageType 欄中會標示「Windows\_Server BYOL」或「Windows\_用戶端 BYOL」。
### <a name="how-are-basic-vs-standard-vm-types-differentiated-in-the-usage-csv"></a>在使用情況 CSV 中，如何區分基本和標準 VM 類型？
系統會同時提供基本 VM 和標準 A 系列 VM。 如果您部署的是基本 VM，則在 [計量表子類別] 中會有「基本」字串。 如果您部署的是標準 A 系列 VM，則 VM 大小會顯示為 “A1 VM”，因為「標準」是預設值。 若要深入了解基本和標準之間的差異，請參閱[定價頁面](https://azure.microsoft.com/pricing/details/virtual-machines/)。
### <a name="what-are-extrasmall-small-medium-large-and-extralarge-sizes"></a>什麼是 ExtraSmall、Small、Medium、Large 和 ExtraLarge 大小？
ExtraSmall - ExtraLarge 是標準\_A0 – 標準\_A4 的舊版名稱。 在傳統 VM 使用情況記錄中，如果您已部署這些大小，可能就會看到這個慣例的使用。
### <a name="what-is-the-difference-between-meter-region-and-resource-location"></a>計量區域與資源位置之間的差異為何？
計量區域與計量相關聯。 對於在所有區域使用一個價格的某些 Azure 服務，[計量區域] 欄位可能是空白的。 不過，因為 VM 對於每個區域有專用的虛擬機器價格，因此會填入此欄位。 同樣地，虛擬機器的資源位置是 VM 部署所在的位置。 這兩個欄位中的 Azure 區域都相同，但是它們對於區域名稱可能會有不同的字串慣例。
### <a name="why-is-the-imagetype-value-blank-in-the-additional-info-field"></a>為什麼 [其他資訊] 欄位中的 ImageType 值是空白的？
ImageType 欄位只會針對映像子集填入。 如果您未部署上述其中一個映像，則 ImageType 是空白的。
### <a name="why-is-the-vmname-blank-in-the-additional-info"></a>為什麼 [其他資訊] 中的 VMName 是空白的？
VMName 只針對擴展集中的 VM，填入 [其他資訊] 欄位中。 InstanceID 欄位包含非擴展集 VM 的 VM 名稱。
### <a name="what-does-computehr-mean-in-the-usagetype-field-in-the-additional-info"></a>ComputeHR 在 [其他資訊] 的 UsageType 欄位中代表什麼意思？
ComputeHR 代表「計算時數」，代表底層基礎結構成本的使用事件。 如果 UsageType 為 ComputeHR\_SW，則使用事件代表 VM 的進階軟體費用。
### <a name="how-do-i-know-if-i-am-charged-for-premium-software"></a>如何知道我是否要支付進階軟體的費用？
探索最適合您需求的 VM 映像時，請務必簽出 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute)。 該映像所使用的是軟體方案費率。 如果您看到費率為「免費」，就是沒有額外的軟體成本。 
### <a name="what-is-the-difference-between-microsoftclassiccompute-and-microsoftcompute-in-the-consumed-service"></a>Microsoft.ClassicCompute 和 Microsoft.Compute 在已使用的服務中有哪些差異？
Microsoft.ClassicCompute 代表透過 Azure Service Manager 部署的傳統資源。 如果您透過 Resource Manager 進行部署，則 Microsoft.Compute 會填入已使用的服務中。 深入了解 [Azure 部署模型](../../azure-resource-manager/resource-manager-deployment-model.md)。
### <a name="why-is-the-instanceid-field-blank-for-my-virtual-machine-usage"></a>為什麼我的虛擬機器使用情況的 InstanceID 欄位是空白的？
如果您透過傳統部署模型進行部署，就無法使用 InstanceID 字串。
### <a name="why-are-the-tags-for-my-vms-not-flowing-to-the-usage-details"></a>為什麼我的 VM 的標記不會傳送至使用情況詳細資料？
標記只會將 Resource Manager VM 的使用情況 CSV 傳送給您。 傳統資源標記不適用於使用情況詳細資料。
### <a name="how-can-the-consumed-quantity-be-more-than-24-hours-one-day"></a>已取用的數量如何超過一天 24 小時？
在傳統模型中，資源的計費是在雲端服務層級進行彙總。 如果雲端服務中有多個 VM 使用相同的計費計量，就會將您的使用情況彙總在一起。 透過 Resource Manager 部署的 VM 會在 VM 層級計費，因此這個彙總將不適用。
### <a name="why-is-pricing-not-available-for-dsfsgsls-sizes-on-the-pricing-page"></a>為什麼定價不適用於定價頁面上的 DS/FS/GS/LS 大小？
支援進階儲存體的 VM 與不支援進階儲存體的 VM 都是以相同的費率計費。 只有儲存體成本有所差異。 如需詳細資訊，請瀏覽[儲存體定價頁面](https://azure.microsoft.com/pricing/details/storage/unmanaged-disks/)。

## <a name="next-steps"></a>後續步驟
若要深入了解使用情況詳細資料，請參閱[了解 Microsoft Azure 的計費](../../billing/billing-understand-your-bill.md)。

