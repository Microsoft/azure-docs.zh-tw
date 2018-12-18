---
title: 在 Azure 中的 Service Fabric 上監視和診斷 Windows 容器 | Microsoft Docs
description: 在本教學課程中，您會設定 Log Analytics，以便監視和診斷 Azure Service Fabric 上的 Windows 容器。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/08/2018
ms.author: dekapur
ms.custom: mvc
ms.openlocfilehash: 24b4cbe0eb13a41ccd6ae98db90d4bf0cf7d22a7
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49311213"
---
# <a name="tutorial-monitor-windows-containers-on-service-fabric-using-log-analytics"></a>教學課程：使用 Log Analytics 監視 Service Fabric 上的 Windows 容器

此為教學課程的第三部分，將逐步引導您設定 Log Analytics 來監視 Service Fabric 上協調的 Windows 容器。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 設定適用於 Service Fabric 叢集的 Log Analytics
> * 使用 Log Analytics 工作區，檢視和查詢來自您容器和節點的記錄
> * 設定 Log Analytics 代理程式以挑選容器和節點計量

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前，您必須：

* 具有 Azure 上的叢集，或[使用本教學課程建立一個](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* [將容器化應用程式部署至該叢集](service-fabric-host-app-in-a-container.md)

## <a name="setting-up-log-analytics-with-your-cluster-in-the-resource-manager-template"></a>在 Resource Manager 範本中，使用您的叢集設定 Log Analytics

如果您使用本教學課程第一部分中[提供的範本](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-OMS-UnSecure)，它應該會包含下列已新增至一般 Service Fabric Azure Resource Manager 範本的內容。 假如您有自己的叢集，而您想要設定該叢集以使用 Log Analytics 來監視容器：

* 對您的 Resource Manager 範本進行下列變更。
* 藉由[部署範本](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)，使用 PowerShell 部署它以升級您的叢集。 Azure Resource Manager 會解析該資源存在，因此將推出它作為升級。

### <a name="adding-log-analytics-to-your-cluster-template"></a>將 Log Analytics 新增至您的叢集範本

在您的 *template.json* 中進行下列變更：

1. 將 Log Analytics 工作區位置和名稱新增至您的 parameters 區段：

    ```json
    "omsWorkspacename": {
      "type": "string",
      "defaultValue": "[toLower(concat('sf',uniqueString(resourceGroup().id)))]",
      "metadata": {
        "description": "Name of your Log Analytics Workspace"
      }
    },
    "omsRegion": {
      "type": "string",
      "defaultValue": "East US",
      "allowedValues": [
        "West Europe",
        "East US",
        "Southeast Asia"
      ],
      "metadata": {
        "description": "Specify the Azure Region for your Log Analytics workspace"
      }
    }
    ```

    變更用來將相同參數新增至 template.parameters.json 的值，以及變更該處所使用的值。

2. 將解決方案名稱和解決方案新增至您的 variables：

    ```json
    "omsSolutionName": "[Concat('ServiceFabric', '(', parameters('omsWorkspacename'), ')')]",
    "omsSolution": "ServiceFabric"
    ```

3. 新增 Microsoft Monitoring Agent 作為虛擬機器擴充功能。 尋找虛擬機器擴展集資源：*resources* > *"apiVersion": "[variables('vmssApiVersion')]"*。 在 *properties* > *virtualMachineProfile* > *extensionProfile* > *extension* 下方，於 *ServiceFabricNode* 擴充功能下方新增下列擴充功能描述： 
    
    ```json
    {
        "name": "[concat(variables('vmNodeType0Name'),'OMS')]",
        "properties": {
            "publisher": "Microsoft.EnterpriseCloud.Monitoring",
            "type": "MicrosoftMonitoringAgent",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
            },
            "protectedSettings": {
                "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
            }
        }
    },
    ```

4. 新增 Log Analytics 工作區作為個別資源。 在 resources 中的虛擬機器擴展集資源之後，新增下列內容：

    ```json
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[parameters('omsWorkspacename')]",
        "type": "Microsoft.OperationalInsights/workspaces",
        "properties": {
            "sku": {
                "name": "Free"
            }
        },
        "resources": [
            {
                "apiVersion": "2015-11-01-preview",
                "name": "[concat(variables('applicationDiagnosticsStorageAccountName'),parameters('omsWorkspacename'))]",
                "type": "storageinsightconfigs",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]",
                    "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
                ],
                "properties": {
                    "containers": [ ],
                    "tables": [
                        "WADServiceFabric*EventTable",
                        "WADWindowsEventLogsTable",
                        "WADETWEventTable"
                    ],
                    "storageAccount": {
                        "id": "[resourceId('Microsoft.Storage/storageaccounts/', variables('applicationDiagnosticsStorageAccountName'))]",
                        "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-06-15').key1]"
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "name": "System",
                "type": "datasources",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsEvent",
                "properties": {
                    "eventLogName": "System",
                    "eventTypes": [
                        {
                            "eventType": "Error"
                        },
                        {
                            "eventType": "Warning"
                        },
                        {
                            "eventType": "Information"
                        }
                    ]
                }
            }
        ]
    },
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[variables('omsSolutionName')]",
        "type": "Microsoft.OperationsManagement/solutions",
        "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('OMSWorkspacename'))]"
        ],
        "properties": {
            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        },
        "plan": {
            "name": "[variables('omsSolutionName')]",
            "publisher": "Microsoft",
            "product": "[Concat('OMSGallery/', variables('omsSolution'))]",
            "promotionCode": ""
        }
    },
    ```

[這裡](https://github.com/ChackDan/Service-Fabric/blob/master/ARM%20Templates/Tutorial/azuredeploy.json)是一個範例範本 (用於本教學課程的第一部分)，其中包含所有您可視需要參考的變更。 這些變更會將 Log Analytics 工作區新增至您的資源群組。 工作區將設定為從使用 [Windows Azure 診斷](service-fabric-diagnostics-event-aggregation-wad.md)代理程式設定的儲存體資料表挑選 Service Fabric 平台事件。 Log Analytics 代理程式 (Microsoft Monitoring Agent) 也已新增至您叢集中的每個節點作為虛擬機器擴充功能；這表示當您調整叢集時，會在每部機器上自動設定此代理程式，並連接相同的工作區。

使用您的新變更來部署範本，以升級目前的叢集。 一旦完成此作業後，您應該會在資源群組中看到 Log Analytics 資源。 當叢集已就緒時，將您的容器化應用程式部署到其中。 在下一個步驟中，我們將設定監視容器。

## <a name="add-the-container-monitoring-solution-to-your-log-analytics-workspace"></a>將容器監視解決方案新增至您的 Log Analytics 工作區

若要在您的工作區中設定容器解決方案，請搜尋「容器監視解決方案」並建立容器資源 (在 [監視 + 管理] 類別下方)。

![新增容器解決方案](./media/service-fabric-tutorial-monitoring-wincontainers/containers-solution.png)

當系統提示您提供 Log Analytics 工作區時，選取已在您資源群組中建立的工作區，然後按一下 [建立]。 這樣即會在您的工作區中新增一個「容器監視解決方案」，將自動導致範本所部署的 Log Analytics 代理程式開始收集 Docker 記錄和統計資料。 

瀏覽回到您的「資源群組」，您現在應該會在其中看到剛新增的監視解決方案。 如果您按一下該解決方案，登陸頁面應該會顯示已在執行中的容器映像數目。

請注意，我已從教學課程的[第二部分](service-fabric-host-app-in-a-container.md)，針對我的 fabrikam 容器執行了 5 個執行個體

![容器解決方案登陸頁面](./media/service-fabric-tutorial-monitoring-wincontainers/solution-landing.png)

按一下 [容器監視解決方案] 將帶您前往更詳細的儀表板，讓您能夠在多個面板中捲動，以及在 Log Analytics 中執行查詢。

請注意，截至 2017 年 9 月，此解決方案正在進行部分更新，當我們正致力於將多個 Orchestrator 整合到相同的解決方案時，請忽略您可能收到有關 Kubernetes 事件的任何錯誤。

由於代理程式會挑選 Docker 記錄，因此預設會顯示 *stdout* 和 *stderr*。 如果您向右捲動，您將看到容器映像詳細目錄、狀態、計量，以及您可執行來取得更有用資料的範例查詢。

![容器解決方案儀表板](./media/service-fabric-tutorial-monitoring-wincontainers/container-metrics.png)

按一下這其中任一個面板，將帶您前往 Log Analytics 查詢，此查詢會產生顯示的值。 將查詢變更為 *\**，以查看所挑選記錄的所有不同種類。 您可以從這裡查詢或篩選容器效能、記錄，或是查看 Service Fabric 平台事件。 您的代理程式也會經常從每個節點發出活動訊號，如果您的叢集設定變更，您可以查看以確定仍會從您的所有機器收集資料。

![容器查詢](./media/service-fabric-tutorial-monitoring-wincontainers/query-sample.png)

## <a name="configure-log-analytics-agent-to-pick-up-performance-counters"></a>設定 Log Analytics 代理程式以挑選效能計數器

使用 Log Analytics 代理程式的另一個優點是能夠變更您想要透過 Log Analytics UI 體驗挑選的效能計數器，而不必每次設定 Azure 診斷代理程式並執行以 Resource Manager 範本為基礎的升級。 若要這樣做，在您容器監視 (或 Service Fabric) 解決方案的登陸頁面上按一下 [OMS 工作區]。

這將帶您前往 Log Analytics 工作區，您可以在其中檢視解決方案、建立自訂儀表板，以及設定 Log Analytics 代理程式。 
* 若要 [開啟進階設定] 功能表，請按一下 [進階設定] 連結。
* 按一下 [連接的來源] > [Windows 伺服器]，以確認您擁有「5 部已連接的 Windows 電腦」。
* 按一下 [資料] > [Windows 效能計數器]，以搜尋並新增效能計數器。 您將在這裡看到可收集的效能計數器建議清單 (來自 Log Analytics)，以及用於搜尋其他計數器的選項。 請確認您已收集 **Processor(_Total)\% Processor Time** 和 **Memory(*)\Available MBytes** 計數器。

請在數分鐘後**重新整理**您的容器監視解決方案，而您應該開始查看傳入的「電腦效能」資料。 這將有助於您了解資源的使用方式。 您也可以使用這些計量作出調整叢集的適當決策，或確認叢集是否正如預期般平衡您的負載。

注意：請確定已正確設定時間篩選，讓您能夠取用這些計量。

![效能計數器 2](./media/service-fabric-tutorial-monitoring-wincontainers/perf-counters2.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定適用於 Service Fabric 叢集的 Log Analytics
> * 使用 Log Analytics 工作區，檢視和查詢來自您容器和節點的記錄
> * 設定 Log Analytics 代理程式以挑選容器和節點計量

現在您已設定監視您的容器化應用程式，請嘗試進行下列動作：

* 設定適用於 Linux 叢集的 Log Analytics，請遵循類似上述的步驟。 參考[此範本](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/SF%20OMS%20Samples/Linux) \(英文\)，在 Resource Manager 範本中進行變更。
* 設定 Log Analytics 以設定[自動化警示](../log-analytics/log-analytics-alerts.md)，來協助偵測與診斷。
* 瀏覽 Service Fabric [建議的效能計數器](service-fabric-diagnostics-event-generation-perf.md)清單，以針對您的叢集進行設定。
* 熟悉 Log Analytics 的[記錄搜尋和查詢](../log-analytics/log-analytics-log-searches.md)功能。