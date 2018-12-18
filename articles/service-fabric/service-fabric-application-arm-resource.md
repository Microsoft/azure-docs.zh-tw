---
title: 利用 Azure Resource Manager 部署及升級應用程式和服務 | Microsoft 文件
description: 了解如何使用 Azure Resource Manager 範本將應用程式和服務部署到 Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/06/2017
ms.author: dekapur
ms.openlocfilehash: feb9d0a01cbba75fc9868f5a603d494c5c09ae2e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386292"
---
# <a name="manage-applications-and-services-as-azure-resource-manager-resources"></a>將應用程式和服務視為 Azure Resource Manager 進行管理

您可以透過 Azure Resource Manager 將應用程式和服務部署到 Service Fabric 叢集。 這意味著與其等待叢集準備就緒後透過 PowerShell 或 CLI 部署及管理應用程式，您現在可以利用 JSON 來表示應用程式和服務，然後在與叢集相同的 Resource Manager 範本中加以部署。 應用程式註冊、佈建和部署等程序全部都發生在一個步驟中。

這是部署叢集所需之任何設定、治理或叢集管理應用程式的建議方法。 包括[修補程式協調流程應用程式](service-fabric-patch-orchestration-application.md)、監視程式，或是任何需要在叢集中執行其他應用程式或服務才能部署的應用程式。 

當可行時，將應用程式視為 Resource Manager 資源來管理有助於改善：
* 稽核記錄：Resource Manager 會稽核每一項作業並保留詳細的*活動記錄*，協助您追蹤這些應用程式和叢集的變更。
* 角色型存取控制 (RBAC)：您可以透過同樣的 Resource Manager 範本來管理叢集和叢集上部署之應用程式的存取權限。
* Azure Resource Manager (透過 Azure 入口網站) 能一次滿足您管理叢集和重要應用程式部署的需求。

下列程式碼片段展示可透過範本管理的各種資源：

```json
{
    "apiVersion": "2017-07-01-preview",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2017-07-01-preview",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2017-07-01-preview",
    "type": "Microsoft.ServiceFabric/clusters/applications",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2017-07-01-preview",
    "type": "Microsoft.ServiceFabric/clusters/applications/services",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
    "location": "[variables('clusterLocation')]"
}
```


## <a name="add-a-new-application-to-your-resource-manager-template"></a>將應用程式新增到 Resource Manager 範本

1. 準備好叢集的 Resource Manager 範本，以供部署之用。 如需詳細資訊，請參閱[使用 Azure Resource Manager 來建立 Service Fabric 叢集](service-fabric-cluster-creation-via-arm.md)。
2. 思考幾個您計劃部署在叢集中的應用程式。 是否有任何應用程式將隨時保持執行狀態，讓其他應用程式得以依賴？ 您是否有部署任何叢集治理或設定應用程式的計劃？ 這類應用程式最適合透過 Resource Manager 範本管理，如同前文所討論。 
3. 一旦想好要利用這種方法部署哪些應用程式後，您必須封裝、壓縮應用程式，再將它們放在檔案共用上。 共用必須可透過 REST 端點存取，Azure Resource Manager 才能在部署期間取用。
4. 在 Resource Manager 範本的叢集宣告下方，描述每個應用程式的屬性。 這些屬性包括複本或執行個體計數，以及資源 (其他應用程式或服務) 之間的任何相依性鏈結。 如需完整的屬性清單，請參閱 [REST API Swagger 規格](https://aka.ms/sfrpswaggerspec)。請注意，這不會取代應用程式或服務資訊清單，而是在叢集的 Resource Manager 範本中描述它們的一些功能。 以下是範例範本，其中包括部署 *Application1*無狀態服務 *Service1* 和具狀態服務 *Service2*：

  ```json
  {
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "clusterName": {
        "type": "string",
        "defaultValue": "Cluster",
        "metadata": {
          "description": "Name of your cluster - Between 3 and 23 characters. Letters and numbers only."
        }
      },
      "applicationTypeName": {
        "type": "string",
        "defaultValue": "ApplicationType",
        "metadata": {
          "description": "The application type name."
        }
      },
      "applicationTypeVersion": {
        "type": "string",
        "defaultValue": "1",
        "metadata": {
          "description": "The application type version."
        }
      },
      "appPackageUrl": {
        "type": "string",
        "metadata": {
          "description": "The URL to the application package sfpkg file."
        }
      },
      "applicationName": {
        "type": "string",
        "defaultValue": "Application1",
        "metadata": {
          "description": "The name of the application resource."
        }
      },
      "serviceName": {
        "type": "string",
        "defaultValue": "Application1~Service1",
        "metadata": {
          "description": "The name of the service resource in the format of {applicationName}~{serviceName}."
        }
      },
      "serviceTypeName": {
        "type": "string",
        "defaultValue": "Service1Type",
        "metadata": {
          "description": "The name of the service type."
        }
      },
      "serviceName2": {
        "type": "string",
        "defaultValue": "Application1~Service2",
        "metadata": {
          "description": "The name of the service resource in the format of {applicationName}~{serviceName}."
        }
      },
      "serviceTypeName2": {
        "type": "string",
        "defaultValue": "Service2Type",
        "metadata": {
          "description": "The name of the service type."
        }
      }
    },
    "variables": {
      "clusterLocation": "[resourcegroup().location]"
    },
    "resources": [
      {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [],
        "properties": {
          "provisioningState": "Default"
        }
      },
      {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "appPackageUrl": "[parameters('appPackageUrl')]"
        }
      },
      {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters/applications",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'), '/versions/', parameters('applicationTypeVersion'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "typeName": "[parameters('applicationTypeName')]",
          "typeVersion": "[parameters('applicationTypeVersion')]",
          "parameters": {},
          "upgradePolicy": {
            "upgradeReplicaSetCheckTimeout": "01:00:00.0",
            "forceRestart": "false",
            "rollingUpgradeMonitoringPolicy": {
              "healthCheckWaitDuration": "00:02:00.0",
              "healthCheckStableDuration": "00:05:00.0",
              "healthCheckRetryTimeout": "00:10:00.0",
              "upgradeTimeout": "01:00:00.0",
              "upgradeDomainTimeout": "00:20:00.0"
            },
            "applicationHealthPolicy": {
              "considerWarningAsError": "false",
              "maxPercentUnhealthyDeployedApplications": "50",
              "defaultServiceTypeHealthPolicy": {
                "maxPercentUnhealthyServices": "50",
                "maxPercentUnhealthyPartitionsPerService": "50",
                "maxPercentUnhealthyReplicasPerPartition": "50"
              }
            }
          }
        }
      },
      {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters/applications/services",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applications/', parameters('applicationName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "serviceKind": "Stateless",
          "serviceTypeName": "[parameters('serviceTypeName')]",
          "instanceCount": "-1",
          "partitionDescription": {
            "partitionScheme": "Singleton"
          },
          "correlationScheme": [],
          "serviceLoadMetrics": [],
          "servicePlacementPolicies": []
        }
      },
      {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters/applications/services",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName2'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applications/', parameters('applicationName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "serviceKind": "Stateful",
          "serviceTypeName": "[parameters('serviceTypeName2')]",
          "targetReplicaSetSize": "3",
          "minReplicaSetSize": "2",
          "replicaRestartWaitDuration": "00:01:00.0",
          "quorumLossWaitDuration": "00:02:00.0",
          "standByReplicaKeepDuration": "00:00:30.0",
          "partitionDescription": {
            "partitionScheme": "UniformInt64Range",
            "count": "5",
            "lowKey": "1",
            "highKey": "5"
          },
          "hasPersistedState": "true",
          "correlationScheme": [],
          "serviceLoadMetrics": [],
          "servicePlacementPolicies": [],
          "defaultMoveCost": "Low"
        }
      }
    ]
  }
  ```

  > [!NOTE] 
  > *apiVersion* 必須設定為 `"2017-07-01-preview"`。 這個範本也能與叢集分開部署，只要叢集已部署完成即可。

5. 部署！ 

## <a name="manage-an-existing-application-via-resource-manager"></a>透過 Resource Manager 管理現有應用程式

如果您的叢集已啟動，而且您已經將幾個要視為 Resource Manager 資源管理的應用程式部署在叢集之上，與其移除應用程式再重新部署，不如透過使用相同 API 的 PUT 呼叫讓應用程式獲得 Resource Manager 資源的認可。 

> [!NOTE]
> 若要讓叢集升級忽略狀況不良的應用程式，客戶可以在 “upgradeDescription/healthPolicy” 區段指定 “maxPercentUnhealthyApplications: 100”；所有設定的詳細說明位於 [Service Fabric REST API 叢集升級原則文件](https://docs.microsoft.com/rest/api/servicefabric/sfrp-model-clusterupgradepolicy)中。

## <a name="next-steps"></a>後續步驟

* 使用 [Service Fabric CLI](service-fabric-cli.md) 或 [PowerShell](service-fabric-deploy-remove-applications.md) 將其他應用程式部署到叢集。 
* [升級 Service Fabric 叢集](service-fabric-cluster-upgrade.md)

