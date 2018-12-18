---
title: Azure 監視器 CLI 快速入門範例
description: Azure 監視器功能的範例 CLI 命令。 Azure 監視器是一項 Microsoft Azure 服務，可讓您根據設定的遙測資料值、自動調整雲端服務、虛擬機器和 Web Apps，傳送警示通知或呼叫 Web URL。
author: rboucher
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: robb
ms.component: ''
ms.openlocfilehash: 1728511402d48904fa968790f790314fdf57767c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961513"
---
# <a name="azure-monitor-cli-quick-start-samples"></a>Azure 監視器 CLI 快速入門範例
本文說明可協助您存取 Azure 監視器 功能的命令列介面 (CLI) 命令範例。 Azure 監視器可讓您根據設定的遙測資料值、自動調整雲端服務、虛擬機器和 Web Apps，以及傳送警示通知，或呼叫 Web URL。

## <a name="prerequisites"></a>必要條件

如果您尚未安裝 Azure CLI，請遵循[安裝 Azure CLI](/cli/azure/install-azure-cli) 中的指示。 您也可以使用 [Azure Cloud Shell](/azure/cloud-shell) 執行 CLI，作為您瀏覽器中的互動式體驗。 請在 [Azure 監視器 CLI 參考](https://docs.microsoft.com/cli/azure/monitor?view=azure-cli-latest)中參閱所有可用命令的完整參考。 

## <a name="log-in-to-azure"></a>登入 Azure
第一步是登入您的 Azure 帳戶。

```azurecli
az login
```

執行此命令之後，您必須透過螢幕上的指示來登入。 所有命令都會在您的預設訂用帳戶內容中運作。

若要列出目前訂用帳戶的詳細資料，使用下列命令。

```azurecli
az account show
```

若要將使用中的內容變更為其他訂用帳戶，請使用下列命令。

```azurecli
az account set -s <Subscription ID or name>
```

若要檢視所有支援的 Azure 監視器命令清單，執行下列命令。

```azurecli
az monitor -h
```

## <a name="view-activity-log-for-a-subscription"></a>檢視訂用帳戶的活動記錄檔

若要檢視活動記錄檔事件的清單，請執行下列命令。

```azurecli
az monitor activity-log list
```

嘗試下列命令以檢視所有可用的選項。

```azurecli
az monitor activity-log list -h
```

以下是依 resourceGroup 列出記錄檔清單的範例

```azurecli
az monitor activity-log list --resource-group <group name>
```

依 caller 列出記錄檔的範例

```azurecli
az monitor activity-log list --caller myname@company.com
```

依 caller 列出日期範圍內在資源類型上的記錄之範例

```azurecli
az monitor activity-log list --resource-provider Microsoft.Web \
    --caller myname@company.com \
    --start-time 2016-03-08T00:00:00Z \
    --end-time 2016-03-16T00:00:00Z
```

## <a name="work-with-alerts"></a>使用警示 
> [!NOTE]
> 目前在 CLI 中僅支援警示 (傳統)。 

### <a name="get-alert-classic-rules-in-a-resource-group"></a>取得資源群組中的警示 (傳統) 規則

```azurecli
az monitor activity-log alert list --resource-group <group name>
az monitor activity-log alert show --resource-group <group name> --name <alert name>
```

### <a name="create-a-metric-alert-classic-rule"></a>建立計量警示 (傳統) 規則

```azurecli
az monitor alert create --name <alert name> --resource-group <group name> \
    --action email <email1 email2 ...> \
    --action webhook <URI> \
    --target <target object ID> \
    --condition "<METRIC> {>,>=,<,<=} <THRESHOLD> {avg,min,max,total,last} ##h##m##s"
```

### <a name="delete-an-alert-classic-rule"></a>刪除警示 (傳統) 規則

```azurecli
az monitor alert delete --name <alert name> --resource-group <group name>
```

## <a name="log-profiles"></a>記錄檔的設定檔

在此區段中運用資訊來使用記錄檔的設定檔。

### <a name="get-a-log-profile"></a>取得記錄檔設定檔

```azurecli
az monitor log-profiles list
az monitor log-profiles show --name <profile name>
```

### <a name="add-a-log-profile-with-retention"></a>新增有保留期的記錄檔設定檔

```azurecli
az monitor log-profiles create --name <profile name> --location <location of profile> \
    --locations <locations to monitor activity in: location1 location2 ...> \
    --categories <categoryName1 categoryName2 ...> \
    --days <# days to retain> \
    --enabled true \
    --storage-account-id <storage account ID to store the logs in>
```

### <a name="add-a-log-profile-with-retention-and-eventhub"></a>新增有保留期與 EventHub 的記錄檔設定檔

```azurecli
az monitor log-profiles create --name <profile name> --location <location of profile> \
    --locations <locations to monitor activity in: location1 location2 ...> \
    --categories <categoryName1 categoryName2 ...> \
    --days <# days to retain> \
    --enabled true
    --storage-account-id <storage account ID to store the logs in>
    --service-bus-rule-id <service bus rule ID to stream to>
```

### <a name="remove-a-log-profile"></a>移除記錄檔設定檔

```azurecli
az monitor log-profiles delete --name <profile name>
```

## <a name="diagnostics"></a>診斷

在此區段中運用資訊來使用診斷設定。

### <a name="get-a-diagnostic-setting"></a>取得診斷設定

```azurecli
az monitor diagnostic-settings list --resource <target resource ID>
```

### <a name="create-a-diagnostic-log-setting"></a>建立診斷記錄設定 

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <storage account ID> \
    --resource <target resource object ID> \
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

### <a name="delete-a-diagnostic-setting"></a>刪除診斷設定

```azurecli
az monitor diagnostic-settings delete --name <diagnostic name> \
    --resource <target resource ID>
```

## <a name="autoscale"></a>Autoscale

利用此區段中的資訊來使用自動調整設定。 您必須修改這些範例。

### <a name="get-autoscale-settings-for-a-resource-group"></a>取得資源群組的自動調整設定

```azurecli
az monitor autoscale list --resource-group <group name>
```

### <a name="get-autoscale-settings-by-name-in-a-resource-group"></a>依名稱取得資源群組中的自動調整設定

```azurecli
az monitor autoscale show --name <settings name> --resource-group <group name>
```

### <a name="set-autoscale-settings"></a>設定自動調整設定

```azurecli
az monitor autoscale create --name <settings name> --resource-group <group name> \
    --count <# instances> \
    --resource <target resource ID>
```
