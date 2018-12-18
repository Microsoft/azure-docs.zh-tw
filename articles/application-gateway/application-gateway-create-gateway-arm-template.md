---
title: 建立 Azure 應用程式閘道 - 範本 | Microsoft Docs
description: 本頁面提供使用 Azure 資源管理員範本，建立 Azure 應用程式閘道的指示。
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/31/2017
ms.author: victorh
ms.openlocfilehash: c749cdf133caebb2d1f061d53a1db38e9ec433bd
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32770769"
---
# <a name="create-an-application-gateway-by-using-the-azure-resource-manager-template"></a>使用 Azure 資源管理員範本建立應用程式閘道

> [!div class="op_single_selector"]
> * [Azure 入口網站](application-gateway-create-gateway-portal.md)
> * [Azure Resource Manager PowerShell](application-gateway-create-gateway-arm.md)
> * [Azure 傳統 PowerShell](application-gateway-create-gateway.md)
> * [Azure Resource Manager 範本](application-gateway-create-gateway-arm-template.md)
> * [Azure CLI](application-gateway-create-gateway-cli.md)

Azure 應用程式閘道是第 7 層負載平衡器。 不論是在雲端或內部部署環境中，此閘道均提供在不同伺服器之間進行容錯移轉及效能路由傳送 HTTP 要求。 應用程式閘道提供許多應用程式傳遞控制器 (ADC) 功能，包括 HTTP 負載平衡、以 Cookie 為基礎的工作階段同質性、安全通訊端層 (SSL) 卸載、自訂健康情況探查、多網站支援，以及許多其他功能。 若要尋找完整的支援功能清單，請瀏覽[應用程式閘道概觀](application-gateway-introduction.md)

本文會逐步引導您從 GitHub 下載和修改現有 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)，以及從 GitHub、PowerShell 和 Azure CLI 部署範本。

如果您只需要直接從 GitHub 部署範本而不做任何變更，請跳至從 GitHub 部署範本。

## <a name="scenario"></a>案例

在此案例中，您將會：

* 建立具有 Web 應用程式防火牆的應用程式閘道
* 建立名為 VirtualNetwork1 且含有 10.0.0.0/16 保留 CIDR 區塊的虛擬網路。
* 建立名為 Appgatewaysubnet 且使用 10.0.0.0/28 做為其 CIDR 區塊的子網路。
* 針對想要用來為流量進行負載平衡的 Web 伺服器，設定兩個先前所設定的後端 IP。 在此範本範例中，後端 IP 是 10.0.1.10 和 10.0.1.11。

> [!NOTE]
> 這些設定都是適用於此範本的參數。 若要自訂範本，您可以在 azuredeploy.json 檔案中變更規則、接聽程式、SSL 和其他選項。

![案例](./media/application-gateway-create-gateway-arm-template/scenario.png)

## <a name="download-and-understand-the-azure-resource-manager-template"></a>下載並了解 Azure 資源管理員範本

您可以下載現有 Azure 資源管理員範本，以透過 Github 建立虛擬網路和兩個子網路，然後進行任何需要的變更，並重複使用該範本。 若要這樣做，請使用下列步驟：

1. 瀏覽至[建立已啟用 Web 應用程式防火牆的應用程式閘道](https://github.com/Azure/azure-quickstart-templates/tree/master/101-application-gateway-waf)
1. 依序按一下 [azuredeploy.json] 和 [RAW]。
1. 將檔案儲存至您電腦上的本機資料夾。
1. 如果您熟悉 Azure 資源管理員範本的使用方式，請跳至步驟 7。
1. 開啟您儲存的檔案，查看 **parameters** 這行下方的內容。
1. Azure 資源管理員範本的參數提供值的預留位置，可以在部署期間填寫。

  | 參數 | 說明 |
  | --- | --- |
  | **subnetPrefix** |應用程式閘道子網路的 CIDR 區塊。 |
  | **applicationGatewaySize** | 應用程式閘道的大小。  WAF 只允許中型和大型。 |
  | **backendIpaddress1** |第一部 Web 伺服器的 IP 位址。 |
  | **backendIpaddress2** |第二部 Web 伺服器的 IP 位址。 |
  | **wafEnabled** | 用於判斷 WAF 是否已啟用的設定。|
  | **wafMode** | Web 應用程式防火牆的模式。  可用的選項為 **prevention** 或 **detection**。|
  | **wafRuleSetType** | WAF 的規則集類型。  目前，OWASP 是唯一支援的選項。 |
  | **wafRuleSetVersion** |規則集版本。 OWASP CRS 2.2.9 和 3.0 是目前支援的選項。 |

1. 檢查 **resources** 下方的內容，並注意下列屬性：

   * **type**。 範本所建立的資源類型。 在此案例中，類型是 `Microsoft.Network/applicationGateways`，代表應用程式閘道。
   * **名稱**。 資源的名稱。 請注意 `[parameters('applicationGatewayName')]` 的用法，這表示此名稱是在部署期間由您輸入的內容，或是由參數檔案所提供。
   * **屬性**。 資源屬性的清單。 此範本會在應用程式閘道建立期間，使用虛擬網路與公用 IP 位址。

1. 巡覽回 [https://github.com/Azure/azure-quickstart-templates/blob/master/101-application-gateway-waf/](https://github.com/Azure/azure-quickstart-templates/blob/master/101-application-gateway-waf)。
1. 按一下 [azuredeploy-parameters.json]，然後按一下 [RAW]。
1. 將檔案儲存至您電腦上的本機資料夾。
1. 開啟您儲存的檔案，以編輯參數的值。 使用下列值來部署本文案例所述的應用程式閘道。

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "addressPrefix": {
            "value": "10.0.0.0/16"
            },
            "subnetPrefix": {
            "value": "10.0.0.0/28"
            },
            "applicationGatewaySize": {
            "value": "WAF_Medium"
            },
            "capacity": {
            "value": 2
            },
            "backendIpAddress1": {
            "value": "10.0.1.10"
            },
            "backendIpAddress2": {
            "value": "10.0.1.11"
            },
            "wafEnabled": {
            "value": true
            },
            "wafMode": {
            "value": "Detection"
            },
            "wafRuleSetType": {
            "value": "OWASP"
            },
            "wafRuleSetVersion": {
            "value": "3.0"
            }
        }
    }
    ```

1. 儲存檔案。 您可以使用線上 JSON 驗證工具 (例如 [JSlint.com](http://www.jslint.com/))，來測試 JSON 範本和參數範本。

## <a name="deploy-the-azure-resource-manager-template-by-using-powershell"></a>使用 PowerShell 來部署 Azure 資源管理員範本

如果您從未用過 Azure PowerShell，請造訪：[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)，並遵循指示登入 Azure，然後選取您的訂用帳戶。

1. 登入 PowerShell

    ```powershell
    Connect-AzureRmAccount
    ```

1. 檢查帳戶的訂用帳戶。

    ```powershell
    Get-AzureRmSubscription
    ```

    系統會提示使用您的認證進行驗證。

1. 選擇其中一個要使用的 Azure 訂用帳戶。

    ```powershell
    Select-AzureRmSubscription -Subscriptionid "GUID of subscription"
    ```

1. 如有需要，請使用 **New-AzureResourceGroup** Cmdlet 建立資源群組。 在下列範例中，您會在美國東部位置建立名為 AppgatewayRG 的資源群組。

    ```powershell
    New-AzureRmResourceGroup -Name AppgatewayRG -Location "West US"
    ```

1. 執行 **New-AzureRmResourceGroupDeployment** Cmdlet，使用先前下載並修改的範本和參數檔案來部署新的虛擬網路。
    
    ```powershell
    New-AzureRmResourceGroupDeployment -Name TestAppgatewayDeployment -ResourceGroupName AppgatewayRG `
    -TemplateFile C:\ARM\azuredeploy.json -TemplateParameterFile C:\ARM\azuredeploy-parameters.json
    ```

## <a name="deploy-the-azure-resource-manager-template-by-using-the-azure-cli"></a>使用 Azure CLI 來部署 Azure 資源管理員範本

若要使用 Azure CLI 部署您下載的 Azure Resource Manager 範本，請依照下列步驟執行：

1. 如果您從未使用過 Azure CLI，請參閱 [安裝和設定 Azure CLI](/cli/azure/install-azure-cli) ，並依照指示進行，直到選取您的 Azure 帳戶和訂用帳戶。

1. 如有必要，請執行 `az group create` 命令建立新的資源群組，如下列程式碼片段所示。 請查看命令的輸出內容。 輸出後顯示的清單可說明所使用的參數。 如需資源群組的詳細資訊，請瀏覽 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。

    ```azurecli
    az group create --location westus --name appgatewayRG
    ```
    
    **-n (or --name)**。 新資源群組的名稱。 在本文案例中為「appgatewayRG」 。
    
    **-l (或 --location)**。 建立新資源群組的 Azure 區域。 在我們的案例中為 *westus*。

1. 執行 `az group deployment create` Cmdlet，使用在上一個步驟中下載並修改的範本和參數檔案來部署新的虛擬網路。 輸出後顯示的清單可說明所使用的參數。

    ```azurecli
    az group deployment create --resource-group appgatewayRG --name TestAppgatewayDeployment --template-file azuredeploy.json --parameters @azuredeploy-parameters.json
    ```

## <a name="deploy-the-azure-resource-manager-template-by-using-click-to-deploy"></a>使用「按一下即部署」來部署 Azure 資源管理員範本

「按一下即部署」是另一種使用 Azure 資源管理員範本的方式。 這是將範本與 Azure 入口網站搭配使用的簡便方法。

1. 移至[建立具有 Web 應用程式防火牆的應用程式閘道](https://azure.microsoft.com/documentation/templates/101-application-gateway-waf/)。

1. 按一下 [ **部署至 Azure**]。

    ![部署至 Azure](./media/application-gateway-create-gateway-arm-template/deploytoazure.png)
    
1. 在入口網站上填寫適用於部署範本的參數，然後按一下 [確定] 。

    ![參數](./media/application-gateway-create-gateway-arm-template/ibiza1.png)
    
1. 選取 [我同意上方所述的條款及條件]，按一下 [購買]。

1. 在 [自訂部署] 刀鋒視窗上，按一下 [建立] 。

## <a name="providing-certificate-data-to-resource-manager-templates"></a>提供 Resource Manager 範本的憑證資料

使用 SSL 搭配範本時，必須以 base64 字串形式提供憑證，而不是上傳憑證。 若要將 .pfx 或 .cer 轉換為 Base64 字串，請使用下列其中一個命令。 下列命令會將憑證轉換為可以提供給範本的 Base64 字串。 預期的輸出是可以儲存在變數並貼在範本中的字串。

### <a name="macos"></a>macOS
```bash
cert=$( base64 <certificate path and name>.pfx )
echo $cert
```

### <a name="windows"></a>Windows
```powershell
[System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("<certificate path and name>.pfx"))
```

## <a name="delete-all-resources"></a>刪除所有資源

若要刪除這篇文章中建立的所有資源，請完成下列其中一個步驟：

### <a name="powershell"></a>PowerShell

```powershell
Remove-AzureRmResourceGroup -Name appgatewayRG
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az group delete --name appgatewayRG
```

## <a name="next-steps"></a>後續步驟

如果您想要設定 SSL 卸載，請造訪：[設定應用程式閘道以進行 SSL 卸載](application-gateway-ssl.md)。

如果您想要將應用程式閘道設為與內部負載平衡器搭配使用，請造訪：[建立具有內部負載平衡器 (ILB) 的應用程式閘道](application-gateway-ilb.md)。

如果您想進一步了解一般負載平衡選項，請造訪：

* [Azure 負載平衡器](https://azure.microsoft.com/documentation/services/load-balancer/)
* [Azure 流量管理員](https://azure.microsoft.com/documentation/services/traffic-manager/)

