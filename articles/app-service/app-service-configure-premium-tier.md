---
title: 設定 Azure App Service 的 PremiumV2 層 | Microsoft Docs
description: 了解如何藉由調整為新的 PremiumV2 定價層，讓 Azure App Service 中的 Web 應用程式、行動應用程式和 API 應用程式獲得更好的效能。
keywords: App Service, Azure App Service, 級別, 可調整, App Service方案, App Service 成本
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: ff00902b-9858-4bee-ab95-d3406018c688
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/25/2018
ms.author: cephalin
ms.openlocfilehash: 59e5825554843ef7265cbccdd9a63db0426891fa
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226304"
---
# <a name="configure-premiumv2-tier-for-azure-app-service"></a>設定 Azure App Service 的 PremiumV2 層

新 **PremiumV2** 定價層提供更快速的處理器、SSD 儲存體，而且記憶體與核心的比例是現有定價層的兩倍。 有此效能優勢，您可在較少的執行個體上執行應用程式來節省成本。 在本文中，您將了解如何在 **PremiumV2** 層建立應用程式，或將應用程式相應增加為 **PremiumV2** 層。

## <a name="prerequisites"></a>必要條件

若要將 Web 應用程式相應增加為 **PremiumV2**，您必須讓 Azure App Service 中的 Web 應用程式在低於 **PremiumV2** 的定價層中執行，而且 Web 應用程式必須在支援 PremiumV2 的 App Service 部署中執行。

<a name="availability"></a>

## <a name="premiumv2-availability"></a>PremiumV2 可用性

**PremiumV2** 層可提供給 _Windows_ 和 _Linux_ 上的 App Service 使用。

在大部分 Azure 區域中都可使用 **PremiumV2**。 若要查看您的區域是否可以使用，請在 [Azure Cloud Shell](../cloud-shell/overview.md) 中執行下列 Azure CLI 命令：

```azurecli-interactive
az appservice list-locations --sku P1V2
```

<a name="create"></a>

## <a name="create-an-app-in-premiumv2-tier"></a>在 PremiumV2 層中建立應用程式

App Service 應用程式的定價層會定義在其執行所在的 [App Service 方案](azure-web-sites-web-hosting-plans-in-depth-overview.md)中。 您可以獨自建立 App Service 方案，或在建立 Web 應用程式時順便建立。

在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>中設定 App Service 方案時，請選取 [定價層]。 

選取 [生產]，然後選取 [P1V2]、[P2V2] 或 [P3V2]，然後按一下 [套用]。

![](media/app-service-configure-premium-tier/scale-up-tier-select.png)

> [!IMPORTANT] 
> 如果您沒有看到 **P1V2**、**P2V2** 和 **P3V2** 選項，或是選項呈現灰色，則 **PremiumV2** 可能無法在包含 App Service 方案的基礎 App Service 部署中使用。 如需詳細資訊，請參閱[從不支援的資源群組與區域組合中相應增加](#unsupported)。

## <a name="scale-up-an-existing-app-to-premiumv2-tier"></a>將現有應用程式相應增加為 PremiumV2 層

在將現有應用程式相應增加為 **PremiumV2** 層之前，請確定 **PremiumV2** 可供使用。 如需資訊，請參閱 [PremiumV2 可用性](#availability)。 如果無法使用，請參閱[從不支援的資源群組與區域組合中相應增加](#unsupported)。

視裝載環境而定，相應增加可能需要執行額外的步驟。 

在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>中，開啟您的 App Service 應用程式頁面。

在 App Service 應用程式頁面的左側導覽中，選取 [相應增加 (App Service 方案)]。

![](media/app-service-configure-premium-tier/scale-up-tier-portal.png)

選取 [生產]，然後選取 [P1V2]、[P2V2] 或 [P3V2]，然後按一下 [套用]。

![](media/app-service-configure-premium-tier/scale-up-tier-select.png)

如果作業成功完成，應用程式的概觀頁面會顯示其已進入 **PremiumV2** 層。

![](media/app-service-configure-premium-tier/finished.png)

### <a name="if-you-get-an-error"></a>如果您收到錯誤

如果基礎 App Service 部署不支援 PremiumV2，則某些 App Service 方案會無法相應增加為 PremiumV2 層。  如需詳細資訊，請參閱[從不支援的資源群組與區域組合中相應增加](#unsupported)。

<a name="unsupported"></a>

## <a name="scale-up-from-an-unsupported-resource-group-and-region-combination"></a>從不支援的資源群組與區域組合中相應增加

如果您的應用程式在無法使用 **PremiumV2** 的 App Service 部署中執行；或是，如果您的應用程式在目前不支援 **PremiumV2** 的區 域中執行，則您必須重新部署應用程式，才能利用 **PremiumV2** 的優勢。  您有兩個選擇：

- 建立**新的**資源群組，然後在**新的**資源群組中建立**新的** Web 應用程式和 App Service 方案，並且在建立程序期間選擇需要的 Azure 區域。  您**必須**在建立新的 App Service 方案時選取 **PremiumV2** 方案。  這可確保資源群組、App Service 方案及 Azure 區域組合產生的 App Service 方案會建立在支援 **PremiumV2** 的 App Service 部署中。  然後將應用程式的程式碼部署到新建立的應用程式和 App Service 方案中。 如有需要，您之後可以從 **PremiumV2** 相應縮小 App Service 方案來節省成本，而未來仍可再使用 **PremiumV2** 成功地進行相應增加。
- 如果應用程式已在現有的**進階**層執行，則可以將應用程式與所有應用程式設定、連接字串和部署組態複製到使用 **PremiumV2** 的新 App Service 方案中。

    ![](media/app-service-configure-premium-tier/clone-app.png)

    在 [複製應用程式] 頁面上，您可以使用 **PremiumV2** 在您想要的區域中建立 App Service 方案，並指定您要複製的應用程式設定和組態。

## <a name="automate-with-scripts"></a>使用指令碼進行自動化

您可以使用 [Azure CLI](/cli/azure/install-azure-cli) 或 [Azure PowerShell](/powershell/azure/overview)，透過指令碼在 **PremiumV2** 層自動建立應用程式。

### <a name="azure-cli"></a>Azure CLI

下列命令會在 P1V2 建立 App Service 方案。 您可以在 Cloud Shell 中執行該命令。 `--sku` 選項包括 P1V2、P2V2 和 P3V2。

```azurecli-interactive
az appservice plan create \
    --resource-group <resource_group_name> \
    --name <app_service_plan_name> \
    --sku P1V2
```

### <a name="azure-powershell"></a>Azure PowerShell

下列命令會在 P1V2 建立 App Service 方案。 `-WorkerSize` 選項包括 [小型]、[中型] 和 [大型]。

```PowerShell
New-AzureRmAppServicePlan -ResourceGroupName <resource_group_name> `
    -Name <app_service_plan_name> `
    -Location <region_name> `
    -Tier "PremiumV2" `
    -WorkerSize "Small"
```
## <a name="more-resources"></a>其他資源

[在 Azure 中相應增加應用程式的規模](web-sites-scale.md)  
[手動或自動調整執行個體計數](../monitoring-and-diagnostics/insights-how-to-scale.md)