---
title: Azure 診斷記錄檔概觀
description: 認識 Azure 診斷記錄，並了解如何利用它們來了解 Azure 資源內發生的事件。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 06/07/2018
ms.author: johnkem
ms.component: logs
ms.openlocfilehash: 630c8e5f48ba475755c3c9ffe632fcba086437e4
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983104"
---
# <a name="collect-and-consume-log-data-from-your-azure-resources"></a>收集並取用來自 Azure 資源的記錄資料

## <a name="what-are-azure-monitor-diagnostic-logs"></a>什麼是 Azure 監視器診斷記錄

**Azure 監視器診斷記錄**是 Azure 服務所發出的記錄，會經常提供有關該服務的作業豐富資料。 Azure 監視器提供兩種類型的診斷記錄：
* **租用戶記錄** - 這些記錄來自租用戶層級服務，這些服務存在於 Azure 訂用帳戶外部，例如 Azure Active Directory 記錄。
* **資源記錄** - 這些記錄來自 Azure 服務，可部署 Azure 訂用帳戶內的資源，例如網路安全性群組或儲存體帳戶。

    ![資源診斷記錄與其他類型的記錄 ](./media/monitoring-overview-of-diagnostic-logs/Diagnostics_Logs_vs_other_logs_v5.png)

這些記錄的內容會依 Azure 服務和資源類型而有所不同。 例如，網路安全性群組規則計數器和 Key Vault 稽核是兩種診斷記錄的類型。

這些記錄與[活動記錄](monitoring-overview-activity-logs.md)不同。 活動記錄可讓您深入探索在訂用帳戶中的資源上使用 Resource Manager 所執行的作業，例如建立虛擬機器或刪除邏輯應用程式。 活動記錄是訂用帳戶層級記錄。 資源層級診斷記錄可讓您深入探索在該資源本身內所執行的作業，例如從 Key Vault 取得密碼。

這些記錄也與客體 OS 層級診斷記錄不同。 客體 OS 診斷記錄是由虛擬機器內執行的代理程式或其他支援的資源類型所收集的記錄。 資源層級診斷記錄不需要代理程式，且會從 Azure 平台本身擷取資源特定的資料，而客體 OS 層級診斷記錄會從虛擬機器上執行的作業系統和應用程式擷取資料。

並非所有資源皆支援此處所述的診斷記錄。 [本文包含一個章節，會列出哪些服務會支援診斷記錄](./monitoring-diagnostic-logs-schema.md)。

## <a name="what-you-can-do-with-diagnostic-logs"></a>診斷記錄的用途
以下是您可以利用診斷記錄進行的事：

![診斷記錄檔的邏輯位置](./media/monitoring-overview-of-diagnostic-logs/Diagnostics_Logs_Actions.png)

* 將診斷記錄檔儲存到[**儲存體帳戶**](monitoring-archive-diagnostic-logs.md)以利稽核或手動檢查。 您可以使用**資源診斷設定**指定保留時間 (以天為單位)。
* [將診斷記錄檔串流至**事件中樞**](monitoring-stream-diagnostic-logs-to-event-hubs.md)，以供第三方服務或自訂的分析解決方案 (如 PowerBI) 擷取。
* 使用 [Log Analytics](../log-analytics/log-analytics-azure-storage.md) 分析診斷記錄

您可以使用並非發出記錄的同一個訂用帳戶中的儲存體帳戶或事件中樞命名空間。 進行此設定的使用者必須具有這兩個訂用帳戶的適當 RBAC 存取權。

> [!NOTE]
>  您目前無法將網路流程記錄封存到位在安全虛擬網路後面的儲存體帳戶。

> [!WARNING]
> 2018 年 11 月 1 日起，儲存體帳戶中的記錄資料格式將變更為 JSON 資料行。 [請參閱本文以了解影響的描述，以及如何更新您的工具，來處理新的格式。](./monitor-diagnostic-logs-append-blobs.md) 
>
> 

## <a name="diagnostic-settings"></a>診斷設定

資源診斷記錄是使用資源診斷設定進行設定的。 租用戶診斷記錄是使用租用戶診斷設定進行設定的。 服務的**診斷設定**會控制：

* 診斷記錄和計量傳送至何處 (儲存體帳戶、事件中樞和/或 Log Analytics)。
* 傳送何種記錄類別，以及是否也會傳送計量資料。
* 每個記錄類別應該在儲存體帳戶中保留多久
    - 保留期為 0 天表示會永遠保留記錄。 否則，此值可以是 1 到 2147483647 之間的任意天數。
    - 如果已設定保留原則，但將儲存體帳戶的記錄儲存停用 (例如，若只選取事件中樞或 Log Analytics 選項)，保留原則不會有任何作用。
    - 保留原則是每天套用，因此在一天結束時 (UTC)，這一天超過保留原則的記錄會被刪除。 例如，如果您的保留原則為一天，在今天一開始，昨天之前的記錄檔會被刪除。 刪除程序會從 UTC 午夜開始，但是請注意，可能需要長達 24 小時的時間，記錄才會從您的儲存體帳戶中刪除。

透過入口網站中的診斷設定、透過 Azure PowerShell 和 CLI 命令、或是透過 [Azure 監視器 REST API](https://docs.microsoft.com/rest/api/monitor/)，可以輕鬆地進行這些設定。

> [!NOTE]
> 目前不支援透過診斷設定傳送多維度計量。 跨維度值所彙總的維度計量會匯出為扁平化單一維度計量。
>
> 「例如」：可以在每個佇列層級瀏覽並繪製事件中樞上的「內送郵件」計量。 不過，當您透過診斷設定匯出時，計量將會呈現為事件中樞內所有佇列的所有內送郵件。
>
>

## <a name="how-to-enable-collection-of-diagnostic-logs"></a>如何啟用診斷記錄的集合

您可以[在 Resource Manager 範本中建立資源時](./monitoring-enable-diagnostic-logs-using-template.md)啟用診斷記錄的集合，或是在建立資源之後，從入口網站中該資源的頁面啟用集合。 您也可以在任何時間點使用 Azure PowerShell 或 CLI 命令，或使用 Azure 監視器 REST API 啟用收集。

> [!TIP]
> 這些指示可能無法直接套用於每個資源。 請透過此頁面底部的結構描述連結，來了解適用於特定資源類型的特殊步驟。

### <a name="enable-collection-of-diagnostic-logs-in-the-portal"></a>在入口網站中啟用診斷記錄的集合

您可以在建立資源之後，於 Azure 入口網站中移至特定資源或巡覽至 Azure 監視器，來啟用資源診斷記錄的收集。 若要透過 Azure 監視器來啟用此作業：

1. 在 [Azure 入口網站](http://portal.azure.com)中，巡覽至 Azure 監視器，然後按一下 [診斷設定]

    ![Azure 監視器的監視區段](media/monitoring-overview-of-diagnostic-logs/diagnostic-settings-blade.png)

2. 選擇性地依資源群組或資源類型篩選清單，然後按一下您要設定診斷設定的資源。

3. 如果您選取的資源上沒有任何設定，系統會提示您建立設定。 按一下「開啟診斷」。

   ![新增診斷設定 - 無現有的設定](media/monitoring-overview-of-diagnostic-logs/diagnostic-settings-none.png)

   如果資源上已有設定，您將會看此資源上已設定的設定清單。 按一下「新增診斷設定」。

   ![新增診斷設定 - 現有的設定](media/monitoring-overview-of-diagnostic-logs/diagnostic-settings-multiple.png)

3. 為您的設定指定名稱，核取您要傳送資料之每個目的地的方塊，然後設定要用於每個目的地的資源。 使用 [保留 (天數)] 滑桿選擇性地設定這些記錄的保留天數 (僅適用於儲存體帳戶目的地)。 保留天數為 0 會無限期地儲存記錄。

   ![新增診斷設定 - 現有的設定](media/monitoring-overview-of-diagnostic-logs/diagnostic-settings-configure.png)

4. 按一下 [檔案] 。

過了幾分鐘之後，新的設定就會出現在此資源的設定清單中，而且每次產生新的事件資料，都會將診斷記錄傳送至指定的目的地。

租用戶診斷設定只能在租用戶服務的入口網站刀鋒視窗中進行設定 - [Azure 監視器診斷設定] 刀鋒視窗中看不到這些設定。 例如，您可以按一下 [稽核記錄] 刀鋒視窗中的 [資料匯出設定]，來設定 Azure Active Directory 稽核記錄。

![AAD 診斷設定](./media/monitoring-overview-of-diagnostic-logs/diagnostic-settings-aad.png)

### <a name="enable-collection-of-resource-diagnostic-logs-via-powershell"></a>透過 PowerShell 啟用資源診斷記錄的收集

若要透過 Azure PowerShell 啟用資源診斷記錄的收集，請使用下列命令：

若要啟用儲存體帳戶中診斷記錄的儲存體，請使用下列命令：

```powershell
Set-AzureRmDiagnosticSetting -ResourceId [your resource id] -StorageAccountId [your storage account id] -Enabled $true
```

儲存體帳戶識別碼是您要傳送記錄之目標儲存體帳戶的資源識別碼。

若要將診斷記錄串流至事件中樞，請使用下列命令：

```powershell
Set-AzureRmDiagnosticSetting -ResourceId [your resource id] -ServiceBusRuleId [your Service Bus rule id] -Enabled $true
```

服務匯流排規則識別碼是此格式的字串︰`{Service Bus resource ID}/authorizationrules/{key name}`。

若要將診斷記錄傳送到 Log Analytics 工作區，請使用下列命令：

```powershell
Set-AzureRmDiagnosticSetting -ResourceId [your resource id] -WorkspaceId [resource id of the log analytics workspace] -Enabled $true
```

您可以使用下列命令取得 Log Analytics 工作區的資源識別碼：

```powershell
(Get-AzureRmOperationalInsightsWorkspace).ResourceId
```

您可以結合這些參數讓多個輸出選項。

您目前無法使用 Azure PowerShell 設定租用戶診斷設定。

### <a name="enable-collection-of-resource-diagnostic-logs-via-the-azure-cli"></a>透過 Azure CLI 啟用資源診斷記錄的收集

若要透過 Azure CLI 啟用資源診斷記錄的收集，請使用 [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) 命令。

若要在儲存體帳戶中啟用診斷記錄的儲存：

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <name or ID of storage account> \
    --resource <target resource object ID> \
    --resource-group <storage account resource group> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

僅在 `--storage-account` 不是物件識別碼時才需要 `--resource-group` 引數。

若要將診斷記錄串流至事件中樞：

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --event-hub <event hub name> \
    --event-hub-rule <event hub rule ID> \
    --resource <target resource object ID> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true
    }
    ]'
```

規則識別碼是一個字串，格式為：`{Service Bus resource ID}/authorizationrules/{key name}`。

若要將診斷記錄傳送到 Log Analytics 工作區：

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --workspace <log analytics name or object ID> \
    --resource <target resource object ID> \
    --resource-group <log analytics workspace resource group> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true
    }
    ]'
```

僅在 `--workspace` 不是物件識別碼時才需要 `--resource-group` 引數

您可以使用任何命令將字典新增至傳遞作為 `--logs` 參數的 JSON 陣列，從而將其他類別新增至診斷記錄。 您可以結合 `--storage-account`、`--event-hub` 和 `--workspace` 參數來啟用多個輸出選項。

您目前無法使用 CLI 設定租用戶診斷設定。

### <a name="enable-collection-of-resource-diagnostic-logs-via-rest-api"></a>透過 REST API 啟用資源診斷記錄的收集

若要使用 Azure 監視器 REST API 變更診斷設定，請參閱[這份文件](https://docs.microsoft.com/rest/api/monitor/)。

您目前無法使用 Azure 監視器 REST API 設定租用戶診斷設定。

## <a name="manage-resource-diagnostic-settings-in-the-portal"></a>在入口網站中管理資源診斷設定

請確定所有資源皆已設定了診斷設定。 在入口網站中，巡覽至 [監視器]，然後開啟 [診斷設定]。

![入口網站中的診斷記錄檔刀鋒視窗](./media/monitoring-overview-of-diagnostic-logs/diagnostic-settings-nav.png)

您可能必須按一下 [所有服務] 才能找到 [監視器] 區段。

您可以在此檢視和篩選所有支援診斷設定的資源，以查看它們是否已啟用診斷。 您也可以向下切入，以查看資源上是否設定了多項設定，並確認資料將流向哪個儲存體帳戶、事件中樞命名空間及/或 Log Analytics 工作區。

![入口網站中的診斷記錄結果](./media/monitoring-overview-of-diagnostic-logs/diagnostic-settings-blade.png)

新增診斷設定會顯示 [診斷設定] 檢視，您可以在其中啟用、停用或修改所選取資源的診斷設定。

## <a name="supported-services-categories-and-schemas-for-diagnostic-logs"></a>診斷記錄支援的服務、類別和結構描述

如需支援的服務以及這些服務所使用之記錄類別和結構描述的完整清單，[請參閱這篇文章](monitoring-diagnostic-logs-schema.md)。

## <a name="next-steps"></a>後續步驟

* [將資源診斷記錄串流至**事件中樞**](monitoring-stream-diagnostic-logs-to-event-hubs.md)
* [使用 Azure 監視器 REST API 變更資源診斷設定](https://msdn.microsoft.com/library/azure/dn931931.aspx)
* [使用 Log Analytics 分析來自 Azure 儲存體的記錄](../log-analytics/log-analytics-azure-storage.md)
