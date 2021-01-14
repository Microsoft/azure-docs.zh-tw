---
title: 在 Azure 監視器 Log Analytics 中使用客戶管理的儲存體帳戶
description: 針對 Log Analytics 案例使用您自己的儲存體帳戶
ms.subservice: logs
ms.topic: conceptual
author: noakup
ms.author: noakuper
ms.date: 09/03/2020
ms.openlocfilehash: 706392d95e371fe303bb9f2c18f59e4a224d83c0
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98201054"
---
# <a name="using-customer-managed-storage-accounts-in-azure-monitor-log-analytics"></a>在 Azure 監視器 Log Analytics 中使用客戶管理的儲存體帳戶

Log Analytics 依賴各種案例中的 Azure 儲存體。 此使用通常會自動管理。 不過，在某些情況下，您需要提供和管理您自己的儲存體帳戶，也稱為客戶管理的儲存體帳戶。 本檔涵蓋將客戶管理的儲存體用於 WAD/LAD 記錄、Private Link 和客戶管理的金鑰 (CMK) 加密。 

> [!NOTE]
> 我們建議您不要相依于 Log Analytics 上傳至客戶管理的儲存體（假設格式設定和內容可能會變更）。

## <a name="ingesting-azure-diagnostics-extension-logs-wadlad"></a>擷取 Azure 診斷擴充功能記錄 (WAD/LAD) 
Azure 診斷擴充代理程式 (分別稱為 WAD 和 LAD，適用于 Windows 和 Linux 代理程式) 收集不同的作業系統記錄，並將它們儲存在客戶管理的儲存體帳戶。 然後，您可以將這些記錄內嵌至 Log Analytics，以檢查和分析這些記錄。
### <a name="how-to-collect-azure-diagnostics-extension-logs-from-your-storage-account"></a>如何從儲存體帳戶收集 Azure 診斷擴充功能記錄
使用 [Azure 入口網站](./diagnostics-extension-logs.md#collect-logs-from-azure-storage) 或藉由呼叫 [儲存體深入解析 API](/rest/api/loganalytics/storage%20insights/createorupdate)，將儲存體帳戶連接至 Log Analytics 工作區作為儲存體資料來源。

支援的資料類型：
* syslog
* Windows 事件
* Service Fabric
* ETW 事件
* IIS 記錄

## <a name="using-private-links"></a>使用私用連結
使用私用連結來連線 Azure 監視器資源時，會使用客戶管理的儲存體帳戶來內嵌自訂記錄檔或 IIS 記錄檔。 這些資料類型的內嵌程式會先將記錄上傳到中繼 Azure 儲存體帳戶，然後才內嵌至工作區。 

### <a name="using-a-customer-managed-storage-account-over-a-private-link"></a>在 Private Link 上使用客戶管理的儲存體帳戶
#### <a name="workspace-requirements"></a>工作區需求
透過私人連結連接到 Azure 監視器時，Log Analytics 代理程式只能將記錄傳送至可透過私人連結存取的工作區。 這項需求表示您應該：
* 設定 Azure 監視器 Private Link 範圍 (AMPLS) 物件
* 將其連線至您的工作區
* 透過私人連結將 AMPLS 連接到您的網路。 

如需有關 AMPLS 設定程式的詳細資訊，請參閱 [使用 Azure Private Link 將網路安全地連接到 Azure 監視器](./private-link-security.md)。 

#### <a name="storage-account-requirements"></a>儲存體帳戶的需求
若要讓儲存體帳戶成功連接到您的私人連結，必須：
* 位於您的 VNet 或對等互連網路上，並透過私人連結連接到您的 VNet。
* 位於與其連結的工作區相同的區域。
* 允許 Azure 監視器存取儲存體帳戶。 如果您選擇只允許選取的網路來存取儲存體帳戶，您應該選取例外狀況：「允許信任的 Microsoft 服務存取此儲存體帳戶」。
![儲存體帳戶信任 MS 服務映射](./media/private-storage/storage-trust.png)
* 如果您的工作區也會處理來自其他網路的流量，您應該將儲存體帳戶設定為允許來自相關網路/網際網路的連入流量。

### <a name="using-a-customer-managed-storage-account-for-cmk-data-encryption"></a>使用客戶管理的儲存體帳戶進行 CMK 資料加密
Azure 儲存體會加密儲存體帳戶中的所有待用資料。 根據預設，它會使用 Microsoft 管理的金鑰 (MMK) 來加密資料;不過，Azure 儲存體也可讓您使用來自 Azure Key vault 的 CMK 來加密儲存體資料。 您可以將自己的金鑰匯入 Azure Key Vault，也可以使用 Azure Key Vault Api 來產生金鑰。
#### <a name="cmk-scenarios-that-require-a-customer-managed-storage-account"></a>需要客戶管理的儲存體帳戶的 CMK 案例
* 使用 CMK 加密記錄-警示查詢
* 使用 CMK 加密已儲存的查詢

#### <a name="how-to-apply-cmk-to-customer-managed-storage-accounts"></a>如何將 CMK 套用至客戶管理的儲存體帳戶
##### <a name="storage-account-requirements"></a>儲存體帳戶的需求
儲存體帳戶與金鑰保存庫必須位於相同區域，但可位於不同的訂用帳戶中。 如需 Azure 儲存體加密與金鑰管理的詳細資訊，請參閱 [Azure 儲存體待用資料的加密](../../storage/common/storage-service-encryption.md)。

##### <a name="apply-cmk-to-your-storage-accounts"></a>將 CMK 套用至您的儲存體帳戶
若要設定您的 Azure 儲存體帳戶以搭配使用 CMK 與 Azure Key Vault，請使用 [Azure 入口網站](../../storage/common/customer-managed-keys-configure-key-vault.md?toc=%252fazure%252fstorage%252fblobs%252ftoc.json)、 [PowerShell](../../storage/common/customer-managed-keys-configure-key-vault.md?toc=%252fazure%252fstorage%252fblobs%252ftoc.json)或 [CLI](../../storage/common/customer-managed-keys-configure-key-vault.md?toc=%252fazure%252fstorage%252fblobs%252ftoc.json)。 

## <a name="link-storage-accounts-to-your-log-analytics-workspace"></a>將儲存體帳戶連結到您的 Log Analytics 工作區
### <a name="using-the-azure-portal"></a>使用 Azure 入口網站
在 Azure 入口網站上，開啟您的工作區功能表，然後選取 *連結的儲存體帳戶*。 分頁會開啟，並依上述的使用案例顯示連結的儲存體帳戶 (在 Private Link 上進行內嵌，將 CMK 套用至儲存的查詢或) 的警示。
![連結的儲存體帳戶分頁圖： ](./media/private-storage/all-linked-storage-accounts.png) 選取資料表上的專案將會開啟其儲存體帳戶詳細資料，您可以在其中設定或更新此類型的連結儲存體帳戶。 
![如果您想要的話，請連結儲存體帳戶的分頁影像 ](./media/private-storage/link-a-storage-account-blade.png) ，以便在不同的使用案例中使用相同的帳戶。

### <a name="using-the-azure-cli-or-rest-api"></a>使用 Azure CLI 或 REST API
您也可以透過 [Azure CLI](/cli/azure/monitor/log-analytics/workspace/linked-storage) 或 [REST API](/rest/api/loganalytics/linkedstorageaccounts)，將儲存體帳戶連結到您的工作區。

適用的 dataSourceType 值為：
* CustomLogs –用於自訂記錄和 IIS 記錄內嵌的儲存體帳戶
* 查詢-使用儲存體帳戶來儲存儲存的查詢 (CMK 加密所需) 
* 警示-使用儲存體帳戶來儲存以記錄為基礎的警示 (CMK 加密所需) 


## <a name="managing-linked-storage-accounts"></a>管理連結的儲存體帳戶

### <a name="create-or-modify-a-link"></a>建立或修改連結
當您將儲存體帳戶連結至工作區時，Log Analytics 將會開始使用它，而不是服務所擁有的儲存體帳戶。 您可以 
* 註冊多個儲存體帳戶，以分散它們之間的記錄負載
* 針對多個工作區重複使用相同的儲存體帳戶

### <a name="unlink-a-storage-account"></a>取消連結儲存體帳戶
若要停止使用儲存體帳戶，請取消連結工作區中的儲存體。 取消連結工作區中的所有儲存體帳戶表示 Log Analytics 會嘗試依賴服務管理的儲存體帳戶。 如果您的網路具有網際網路的有限存取權，則這些存放裝置可能無法使用，而且依賴儲存體的任何案例都將會失敗。

### <a name="replace-a-storage-account"></a>取代儲存體帳戶
若要取代用於內嵌的儲存體帳戶，
1.  **建立新儲存體帳戶的連結。** 記錄代理程式也會取得更新的設定，並開始將資料傳送到新的儲存體。 此程式可能需要幾分鐘的時間。
2.  **然後將舊的儲存體帳戶取消連結，讓代理程式停止寫入已移除的帳戶。** 擷取程序會繼續讀取此帳戶的資料，直到全部擷取完畢為止。 除非您看到所有記錄都已擷取，否則請勿刪除儲存體帳戶。

### <a name="maintaining-storage-accounts"></a>維護儲存體帳戶
#### <a name="manage-log-retention"></a>記錄管理保留
使用您自己的儲存體帳戶時，保留期由您負責。 Log Analytics 不會刪除儲存在您私人儲存體上的記錄。 相反地，您應該設定原則，根據您的喜好設定來處理負載。

#### <a name="consider-load"></a>考慮載入
儲存體帳戶可以處理讀取和寫入要求的特定負載，然後才開始節流要求 (如需詳細資訊，請參閱 [Blob 儲存體) 的擴充性和效能目標](../../storage/common/scalability-targets-standard-account.md) 。 節流會影響內嵌記錄所花的時間。 如果您的儲存體帳戶已超載，請註冊額外的儲存體帳戶，以分配它們之間的負載。 若要監視儲存體帳戶的容量和效能，請參閱 [Azure 入口網站中的見解]( https://docs.microsoft.com/azure/azure-monitor/insights/storage-insights-overview)。

### <a name="related-charges"></a>相關費用
儲存體帳戶會依據儲存的資料量、儲存體類型和冗余類型來收費。 如需詳細資訊，請參閱[區塊 Blob 定價](https://azure.microsoft.com/pricing/details/storage/blobs)和[表格儲存體定價](https://azure.microsoft.com/pricing/details/storage/tables)。


## <a name="next-steps"></a>後續步驟

- 瞭解如何 [使用 Azure Private Link 安全地將網路連線到 Azure 監視器](private-link-security.md)
- 瞭解 [Azure 監視器客戶管理的金鑰](customer-managed-keys.md)
