---
title: 依 Azure 資源類型區分的移動作業支援 | Microsoft Docs
description: 列出可移至新資源群組或訂用帳戶的 Azure 資源類型。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/28/2018
ms.author: tomfitz
ms.openlocfilehash: 5bb820d816115dccf470b6c32d080862495e8310
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434971"
---
# <a name="move-operation-support-for-resources"></a>資源的移動作業支援

此文章列出 Azure 資源類型是否支援移動作業。 雖然資源類型支援作業，但可能會有導致無法移動資源的情況。 如需有關影響移動作業之情況的詳細資料，請參閱[將資源移動到新的資源群組或訂用帳戶](resource-group-move-resources.md)。

## <a name="find-resource-provider-and-resource-type"></a>尋找資源提供者和資源類型

若要判斷是否可以移動某個資源，您必須找出其資源提供者和資源類型。

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmResource -ResourceGroupName demogroup | Select Name, ResourceType | Format-table
```

對於 Azure CLI，請使用：

```azurecli-interactive
az resource list -g demogroup --query '[].{name:name, reourcetype:type}'
```

傳回資源類型時，會以 `<resource-provider>/<resource-type-name>` 格式傳回。 因此，`Microsoft.OperationalInsights/workspaces` 值的資源提供者為 **Microsoft.OperationalInsights**，而資源類型名稱則為 **workspaces**。

找出資源提供者和資源類型之後，請使用此文章中的表格來判斷該資源類型是否支援移動作業。

## <a name="microsoftaad"></a>Microsoft.AAD

| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | --------------- | ----------- |
| domainservices | 否 | 否 |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| servers | 是 | 是 |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | --------------- | ----------- |
| service | 是 | 是 |

## <a name="microsoftauthorization"></a>Microsoft.Authorization
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | --------------- | ----------- |
| policyassignments | 否 | 否 |

## <a name="microsoftautomation"></a>Microsoft.Automation
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| automationaccounts | 是 | 是 |
| automationaccounts/configurations | 是 | 是 |
| automationaccounts/runbooks | 是 | 是 |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| b2cdirectories | 是 | 是 |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| registrations | 是 | 是 |

## <a name="microsoftbackup"></a>Microsoft.Backup
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| backupvault | 否 | 否 |

## <a name="microsoftbatch"></a>Microsoft.Batch
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| batchaccounts | 是 | 是 |

## <a name="microsoftbatchai"></a>Microsoft.BatchAI
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| clusters | 否 | 否 |
| fileservers | 否 | 否 |
| jobs | 否 | 否 |
| workspaces | 否 | 否 |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| mapapis | 否 | 否 |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| biztalk | 是 | 是 |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| blueprintassignments | 否 | 否 |

## <a name="microsoftbotservice"></a>Microsoft.BotService
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| botservices | 是 | 是 |

## <a name="microsoftcache"></a>Microsoft.Cache
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| redis | 是 | 是 |

## <a name="microsoftcdn"></a>Microsoft.Cdn
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| 設定檔 | 是 | 是 |
| profiles/endpoints | 是 | 是 |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| certificateorders | 是 | 是 |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| domainnames | 是 | 否 |
| virtualmachines | 是 | 否 |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| networksecuritygroups | 否 | 否 |
| reservedips | 否 | 否 |
| virtualnetworks | 否 | 否 |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| storageaccounts | 是 | 否 |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |

## <a name="microsoftcompute"></a>Microsoft.Compute
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| availabilitysets | 是 | 是 |
| disks | 是 | 是 |
| galleries | 否 | 否 |
| galleries/images | 否 | 否 |
| galleries/images/versions | 否 | 否 |
| images | 是 | 是 |
| restorepointcollections | 否 | 否 |
| sharedvmimages | 否 | 否 |
| sharedvmimages/versions | 否 | 否 |
| snapshots | 是 | 是 |
| virtualmachines | 是 | 是 |
| virtualmachines/extensions | 是 | 是 |
| virtualmachinescalesets | 是 | 是 |

## <a name="microsoftcontainer"></a>Microsoft.Container
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| containergroups | 否 | 否 |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| containergroups | 否 | 否 |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| registries | 是 | 是 |
| registries/buildtasks | 是 | 是 |
| registries/replications | 否 | 否 |
| registries/tasks | 是 | 是 |
| registries/webhooks | 是 | 是 |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| containerservices | 否 | 否 |
| managedclusters | 否 | 否 |
| openshiftmanagedclusters | 否 | 否 |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| 應用程式所需 | 是 | 是 |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| 連接器 | 是 | 是 |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| hubs | 是 | 是 |

## <a name="microsoftdatabox"></a>Microsoft.DataBox
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| jobs | 否 | 否 |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| databoxedgedevices | 否 | 否 |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| workspaces | 否 | 否 |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| catalogs | 是 | 是 |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| datafactories | 是 | 是 |
| factories | 是 | 是 |

## <a name="microsoftdatalake"></a>Microsoft.DataLake
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| datalakeaccounts | 否 | 否 |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| 服務 | 否 | 否 |
| services/projects | 否 | 否 |
| slots | 否 | 否 |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| servers | 否 | 否 |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| servers | 是 | 是 |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| servergroups | 否 | 否 |
| servers | 是 | 是 |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| artifactsources | 否 | 否 |
| rollouts | 否 | 否 |
| servicetopologies | 否 | 否 |
| servicetopologies/services | 否 | 否 |
| servicetopologies/services/serviceunits | 否 | 否 |

## <a name="microsoftdevices"></a>Microsoft.Devices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| iothubs | 是 | 是 |
| provisioningservices | 是 | 是 |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| labcenters | 否 | 否 |
| labs | 是 | 否 |
| labs/servicerunners | 是 | 是 |
| labs/virtualmachines | 是 | 否 |
| schedules | 否 | 否 |

## <a name="microsoftdns"></a>microsoft.dns
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| dnszones | 否 | 否 |
| dnszones/a | 否 | 否 |
| dnszones/aaaa | 否 | 否 |
| dnszones/cname | 否 | 否 |
| dnszones/mx | 否 | 否 |
| dnszones/ptr | 否 | 否 |
| dnszones/srv | 否 | 否 |
| dnszones/txt | 否 | 否 |
| trafficmanagerprofiles | 否 | 否 |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| databaseaccounts | 是 | 是 |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| domains | 是 | 是 |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| topics | 是 | 是 |

## <a name="microsofteventhub"></a>Microsoft.EventHub
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| clusters | 是 | 是 |
| namespaces | 是 | 是 |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| hanainstances | 是 | 是 |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| dedicatedhsms | 否 | 否 |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| clusters | 是 | 是 |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| jobs | 是 | 是 |

## <a name="microsoftinsights"></a>microsoft.insights
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| actiongroups | 是 | 是 |
| activitylogalerts | 否 | 否 |
| alertrules | 是 | 是 |
| autoscalesettings | 是 | 是 |
| components | 是 | 是 |
| metricalerts | 否 | 否 |
| scheduledqueryrules | 是 | 是 |
| webtests | 是 | 是 |
| workbooks | 是 | 是 |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| iotapps | 是 | 是 |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| vaults | 是 | 是 |

## <a name="microsoftlabservices"></a>Microsoft.LabServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| labaccounts | 是 | 是 |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |

## <a name="microsoftlogic"></a>Microsoft.Logic
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| integrationaccounts | 是 | 是 |
| workflows | 是 | 是 |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| commitmentplans | 是 | 是 |
| webservices | 是 | 否 |
| workspaces | 是 | 是 |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| operationalizationclusters | 是 | 是 |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |
| accounts/workspaces | 是 | 是 |
| accounts/workspaces/projects | 是 | 是 |
| teamaccounts | 是 | 是 |
| teamaccounts/workspaces | 是 | 是 |
| teamaccounts/workspaces/projects | 是 | 是 |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| workspaces | 是 | 是 |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| userassignedidentities | 是 | 是 |

## <a name="microsoftmaps"></a>Microsoft.Maps
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| accounts | 是 | 是 |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| classicdevservices | 否 | 否 |

## <a name="microsoftmedia"></a>Microsoft.Media
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| mediaservices | 是 | 是 |
| mediaservices/liveevents | 是 | 是 |
| mediaservices/streamingendpoints | 是 | 是 |

## <a name="microsoftmigrate"></a>Microsoft.Migrate
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| projects | 否 | 否 |

## <a name="microsoftnetwork"></a>Microsoft.Network
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| applicationgateways | 否 | 否 |
| applicationsecuritygroups | 是 | 是 |
| azurefirewalls | 是 | 是 |
| connections | 是 | 是 |
| ddosprotectionplans | 否 | 否 |
| dnszones | 是 | 是 |
| expressroutecircuits | 否 | 否 |
| expressroutecrossconnections | 否 | 否 |
| expressroutegateways | 否 | 否 |
| expressrouteports | 否 | 否 |
| frontdoors | 是 | 是 |
| frontdoorwebapplicationfirewallpolicies | 是 | 是 |
| interfaceendpoints | 否 | 否 |
| loadbalancers | 是 | 是 |
| localnetworkgateways | 是 | 是 |
| networkintentpolicies | 是 | 是 |
| networkinterfaces | 是 | 是 |
| networkprofiles | 否 | 否 |
| networksecuritygroups | 是 | 是 |
| networkwatchers | 是 | 是 |
| networkwatchers/connectionmonitors | 是 | 是 |
| networkwatchers/lenses | 是 | 是 |
| networkwatchers/pingmeshes | 是 | 是 |
| publicipaddresses | 是 | 是 |
| publicipprefixes | 是 | 是 |
| routefilters | 否 | 否 |
| routetables | 是 | 是 |
| serviceendpointpolicies | 是 | 是 |
| trafficmanagerprofiles | 是 | 是 |
| virtualhubs | 是 | 是 |
| virtualnetworkgateways | 是 | 是 |
| virtualnetworks | 是 | 是 |
| virtualnetworktaps | 否 | 否 |
| virtualwans | 是 | 是 |
| vpngateways | 是 | 是 |
| vpnsites | 是 | 是 |
| webapplicationfirewallpolicies | 是 | 是 |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| namespaces | 是 | 是 |
| namespaces/notificationhubs | 是 | 是 |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| workspaces | 是 | 是 |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| managementconfigurations | 是 | 是 |
| solutions | 是 | 是 |
| 檢視 | 是 | 是 |

## <a name="microsoftportal"></a>Microsoft.Portal
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| dashboards | 是 | 是 |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| workspacecollections | 是 | 是 |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| capacities | 是 | 是 |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| vaults | 是 | 是 |

## <a name="microsoftrelay"></a>Microsoft.Relay
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| namespaces | 是 | 是 |

## <a name="microsoftsaas"></a>Microsoft.SaaS
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| 應用程式所需 | 是 | 否 |

## <a name="microsoftscheduler"></a>Microsoft.Scheduler
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| flows | 是 | 是 |
| jobcollections | 是 | 是 |

## <a name="microsoftsearch"></a>Microsoft.Search
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| searchservices | 是 | 是 |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| namespaces | 是 | 是 |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| clusters | 是 | 是 |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| 應用程式所需 | 是 | 是 |
| networks | 是 | 是 |
| 磁碟區 | 是 | 是 |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| signalr | 是 | 是 |

## <a name="microsoftsiterecovery"></a>Microsoft.SiteRecovery
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| siterecoveryvault | 否 | 否 |

## <a name="microsoftsolutions"></a>Microsoft.Solutions
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| appliancedefinitions | 否 | 否 |
| appliances | 否 | 否 |
| applicationdefinitions | 否 | 否 |
| 應用程式所需 | 否 | 否 |

## <a name="microsoftsql"></a>Microsoft.Sql
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| managedinstances | 是 | 是 |
| managedinstances/databases | 是 | 是 |
| servers | 是 | 是 |
| servers/databases | 是 | 是 |
| servers/elasticpools | 是 | 是 |
| virtualclusters | 是 | 是 |

## <a name="microsoftstorage"></a>Microsoft.Storage
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| storageaccounts | 是 | 是 |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| storagesyncservices | 是 | 是 |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| managers | 否 | 否 |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| streamingjobs | 是 | 是 |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| environments | 是 | 是 |
| environments/eventsources | 是 | 是 |
| environments/referencedatasets | 是 | 是 |

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| 帳戶 | 是 | 是 |
| account/extension | 是 | 是 |
| account/project | 是 | 是 |

## <a name="microsoftweb"></a>Microsoft.Web
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| certificates | 否 | 是 |
| classicmobileservices | 否 | 否 |
| connectiongateways | 是 | 是 |
| connections | 是 | 是 |
| customapis | 是 | 是 |
| hostingenvironments | 否 | 否 |
| serverfarms | 是 | 是 |
| sites | 是 | 是 |
| sites/premieraddons | 是 | 是 |
| sites/slots | 是 | 是 |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT
| 資源類型 | 資源群組 | 訂用帳戶 |
| ------------- | -------------- | ------------ |
| deviceservices | 是 | 是 |


## <a name="third-party-services"></a>協力廠商服務

協力廠商服務目前不支援移動作業。 這些資源提供者包括：

* 84codes.CloudAMQP
* AppDynamics.APM
* Aspera.Transfers
* Auth0.Cloud
* Citrix.Cloud
* Citrix.Services
* CloudSimple.PrivateCloudIaaS
* Cloudyn.Analytics
* Conexlink.MyCloudIT
* Crypteron.DataSecurity
* Dynatrace.DynatraceSaaS
* Dynatrace.Ruxit
* LiveArena.Broadcast
* Lombiq.DotNest
* Mailjet.Email
* Myget.PackageManagement
* NewRelic.APM
* nuubit.nextgencdn
* Paraleap.CloudMonix
* Pokitdok.Platform
* RavenHq.Db
* Raygun.CrashReporting
* RevAPM.MobileCDN
* Sendgrid.Email
* Sparkpost.Basic
* stackify.retrace
* SuccessBricks.ClearDB
* TrendMicro.DeepSecurity
* U2uconsult.TheIdentityHub


## <a name="next-steps"></a>後續步驟

* 如需用來移動資源的命令，請參閱[將資源移動到新的資源群組或訂用帳戶](resource-group-move-resources.md)。
