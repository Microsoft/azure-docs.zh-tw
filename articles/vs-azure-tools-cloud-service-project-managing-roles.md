---
title: 使用 Visual Studio 在 Azure 雲端服務中管理角色 | Microsoft Docs
description: 了解如何使用 Visual Studio，在 Azure 雲端服務中新增及移除角色。
services: visual-studio-online
author: ghogen
manager: douge
assetId: 5ec9ae2e-8579-4e5d-999e-8ae05b629bd1
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/21/2017
ms.author: ghogen
ms.openlocfilehash: 023d70fd1f1ae2f79483f44a84cfedd5d1e39f3f
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141707"
---
# <a name="managing-roles-in-azure-cloud-services-with-visual-studio"></a>使用 Visual Studio 在 Azure 雲端服務中管理角色
當您建立 Azure 雲端服務之後，您可以在該服務中加入角色或從中移除現有角色。 您也可以匯入現有的專案，並將它轉換成角色。 例如，您可以匯入 ASP.NET Web 應用程式，並將它指定為 Web 角色。

## <a name="adding-a-role-to-an-azure-cloud-service"></a>將角色加入至 Azure 雲端服務
下列步驟會逐步引導您完成將 Web 或背景工作角色加入至 Visual Studio 中的 Azure 雲端服務專案。

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，展開專案資料夾。

1. 以滑鼠右鍵按一下 [角色] 節點，以顯示操作功能表。 從操作功能表中，選取 [新增]，然後選取現有的 Web 角色或背景工作角色，或是建立 Web 或背景工作角色專案。 您可以選取適當的專案 (例如 ASP.NET Web 應用程式專案)，並將它與角色專案產生關聯。

    ![將角色加入至 Azure 雲端服務專案的功能表選項](media/vs-azure-tools-cloud-service-project-managing-roles/add-role.png)

## <a name="removing-a-role-from-an-azure-cloud-service"></a>從 Azure 雲端服務移除角色
下列步驟會逐步引導您完成從 Visual Studio 中的 Azure 雲端服務專案移除 Web 或背景工作角色。

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，展開專案資料夾。

1. 展開 [角色] 節點。

1. 以滑鼠右鍵按一下您要移除的節點，然後從操作功能表中選取 [移除]。 

    ![將角色加入至 Azure 雲端服務的功能表選項](media/vs-azure-tools-cloud-service-project-managing-roles/remove-role.png)

## <a name="readding-a-role-to-an-azure-cloud-service-project"></a>將角色重新加入至 Azure 雲端服務專案
如果您從雲端服務專案中移除角色，但稍後決定將該角色重新加入至專案，則只有角色宣告和基本屬性 (例如端點和診斷資訊) 會被加入專案。 不會將任何其他資源或參考加入至 `ServiceDefinition.csdef` 檔案或 `ServiceConfiguration.cscfg` 檔案。 如果您想要加入此資訊，就必須手動將它重新加回這些檔案。

例如，您可能移除了 Web 服務角色，但稍後決定將這個角色重新加回方案。 如果您這樣做，將會發生錯誤。 為了避免這個錯誤，您必須將下列 XML 顯示的 `<LocalResources>` 元素重新加回 `ServiceDefinition.csdef` 檔案。 使用您重新加回專案的 Web 服務角色名稱作為 **<LocalStorage>** 項目的部分名稱屬性。 在此範例中，此 Web 服務角色的名稱是 **WCFServiceWebRole1**。

    <WebRole name="WCFServiceWebRole1">
        <Sites>
          <Site name="Web">
            <Bindings>
              <Binding name="Endpoint1" endpointName="Endpoint1" />
            </Bindings>
          </Site>
        </Sites>
        <Endpoints>
          <InputEndpoint name="Endpoint1" protocol="http" port="80" />
        </Endpoints>
        <Imports>
          <Import moduleName="Diagnostics" />
        </Imports>
       <LocalResources>
          <LocalStorage name="WCFServiceWebRole1.svclog" sizeInMB="1000" cleanOnRoleRecycle="false" />
       </LocalResources>
    </WebRole>

## <a name="next-steps"></a>後續步驟
- [使用 Visual Studio 設定 Azure 雲端服務的角色](vs-azure-tools-configure-roles-for-cloud-service.md)
