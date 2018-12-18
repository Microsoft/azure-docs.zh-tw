---
title: Service Fabric 叢集安全性：用戶端角色 |Microsoft Docs
description: 本文說明兩個用戶端角色及提供給這些角色的權限。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: coreysa
editor: ''
ms.assetid: 7bc808d9-3609-46a1-ac12-b4f53bff98dd
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: 39b77813e3494e809b92738924544e196d89a166
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34204646"
---
# <a name="role-based-access-control-for-service-fabric-clients"></a>角色型存取控制 (適用於 Service Fabric 用戶端)
Azure Service Fabric 針對連線到 Service Fabric 叢集的用戶端，支援兩種不同的存取控制類型：系統管理員和使用者。 存取控制可讓叢集系統管理員針對不同的使用者群組限制特定叢集作業的存取權，讓叢集更加安全。  

**系統管理員**可以完整存取管理功能 (包括讀取/寫入功能)。 **使用者**預設只具有管理功能的讀取存取權 (例如查詢功能)，以及解析應用程式和服務的能力。

您可在建立叢集時為每個角色提供個別的憑證，以指定兩個用戶端角色 (系統管理員和用戶端)。 如需有關設定安全 Service Fabric 叢集的詳細資訊，請參閱 [Service Fabric 叢集安全性](service-fabric-cluster-security.md) 。

## <a name="default-access-control-settings"></a>預設存取控制設定
系統管理員存取控制類型可以完整存取所有的 FabricClient API。 它可以在 Service Fabric 叢集上執行任何讀取和寫入，包括下列作業：

### <a name="application-and-service-operations"></a>應用程式和服務作業
* **CreateService**：建立服務                             
* **CreateServiceFromTemplate**：從範本建立服務                             
* **UpdateService**：更新服務                             
* **DeleteService**：刪除服務                             
* **ProvisionApplicationType**：佈建應用程式類型                             
* **CreateApplication**：建立應用程式                               
* **DeleteApplication**：刪除應用程式                             
* **UpgradeApplication**：啟動或中斷應用程式升級                             
* **UnprovisionApplicationType**：解除應用程式類型佈建                             
* **MoveNextUpgradeDomain**：以明確的更新網域繼續進行應用程式升級                             
* **ReportUpgradeHealth**：以目前的升級進度繼續進行應用程式升級                             
* **ReportHealth**：回報健康狀態                             
* **PredeployPackageToNode**：預先佈署 API                            
* **CodePackageControl**：重新啟動程式碼封裝                             
* **RecoverPartition**：復原某個資料分割                             
* **RecoverPartitions**：復原多個資料分割                             
* **RecoverServicePartitions**：復原服務分割                             
* **RecoverSystemPartitions**：復原系統服務分割                             

### <a name="cluster-operations"></a>叢集作業
* **ProvisionFabric**：佈建 MSI 和/或叢集資訊清單                             
* **UpgradeFabric**：啟動叢集升級                             
* **UnprovisionFabric**：解除 MSI 和/或叢集資訊清單佈建                         
* **MoveNextFabricUpgradeDomain**：以明確的更新網域繼續進行叢集升級                             
* **ReportFabricUpgradeHealth**：以目前的升級進度繼續進行叢集升級                             
* **StartInfrastructureTask**：啟動基礎結構工作                             
* **FinishInfrastructureTask**：完成基礎結構工作                             
* **InvokeInfrastructureCommand**：基礎結構工作管理命令                              
* **ActivateNode**：啟用節點                             
* **DeactivateNode**：停用某個節點                             
* **DeactivateNodesBatch**：停用多個節點                             
* **RemoveNodeDeactivations**：還原多個節點上的停用                             
* **GetNodeDeactivationStatus**：檢查停用狀態                             
* **NodeStateRemoved**：回報已移除的節點狀態                             
* **ReportFault**：回報錯誤                             
* **FileContent**：映像存放區用戶端檔案傳輸 (叢集外部)                             
* **FileDownload**：起始映像存放區用戶端檔案下載 (叢集外部)                             
* **InternalList**：映像存放區用戶端檔案清單作業 (內部)                             
* **Delete**：映像存放區用戶端刪除作業                              
* **Upload**：映像存放區用戶端上傳作業                             
* **NodeControl**：啟動、停止及重新啟動節點                             
* **MoveReplicaControl**：將複本從一個節點移至另一個節點                             

### <a name="miscellaneous-operations"></a>其他作業
* **Ping**：用戶端 Ping                             
* **Query**：所有允許的查詢
* **NameExists**：檢查命名 URI 是否存在                             

使用者存取控制類型預設限制為下列作業： 

* **EnumerateSubnames**：列舉命名 URI                             
* **EnumerateProperties**：列舉命名屬性                             
* **PropertyReadBatch**：命名屬性讀取作業                             
* **GetServiceDescription**：長時間輪詢服務通知和讀取服務描述                             
* **ResolveService**：抱怨型服務解析                             
* **ResolveNameOwner**：解析命名 URI 擁有者                             
* **ResolvePartition**：解析系統服務                             
* **ServiceNotifications**：事件型服務通知                             
* **GetUpgradeStatus**：輪詢應用程式升級狀態                             
* **GetFabricUpgradeStatus**：輪詢叢集升級狀態                             
* **InvokeInfrastructureQuery**：查詢基礎結構工作                             
* **List**：映像存放區用戶端檔案清單作業                             
* **ResetPartitionLoad**：重設容錯移轉單元的載入                             
* **ToggleVerboseServicePlacementHealthReporting**：切換詳細服務位置健康情況報告                             

系統管理員存取控制也可存取上述作業。

## <a name="changing-default-settings-for-client-roles"></a>變更用戶端角色的預設設定
在叢集資訊清單檔案中，您可以視需要提供系統管理員功能給用戶端。 您可以在[建立叢集](service-fabric-cluster-creation-via-portal.md)時，前往 [Fabric 設定] 選項，並在 [名稱]、[系統管理員]、[使用者] 和 [值] 欄位中提供上述設定來變更預設值。

## <a name="next-steps"></a>後續步驟
[Service Fabric 叢集安全性](service-fabric-cluster-security.md)

[Service Fabric 叢集建立](service-fabric-cluster-creation-via-portal.md)

