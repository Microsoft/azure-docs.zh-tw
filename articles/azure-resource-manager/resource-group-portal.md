---
title: 使用 Azure 入口網站管理 Azure 資源 | Microsoft Docs
description: 使用 Azure 入口網站和 Azure Resource Manager 來管理資源。 示範如何使用儀表板來監視資源。
services: azure-resource-manager,azure-portal
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 0725bbf2-5913-4c07-af6e-24e11d957fbc
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/15/2016
ms.author: tomfitz
ms.openlocfilehash: 7398e01a46b5d296f26905e2063acdb98383f567
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38600356"
---
# <a name="manage-azure-resources-through-portal"></a>透過入口網站管理 Azure 資源

本文示範如何使用 [Azure 入口網站](https://portal.azure.com)搭配 [Azure Resource Manager](resource-group-overview.md) 來管理您的 Azure 資源。 若要了解如何透過入口網站部署資源，請參閱 [使用 Resource Manager 範本與 Azure 入口網站來部署資源](resource-group-template-deploy-portal.md)。

[!INCLUDE [Handle personal data](../../includes/gdpr-intro-sentence.md)]

## <a name="manage-resource-groups"></a>管理資源群組

資源群組是存放 Azure 方案相關資源的容器。 資源群組可以包含方案的所有資源，或只包含您要以群組方式管理的資源。 您可決定如何根據對組織最有利的方式，將資源配置到資源群組。 一般而言，會新增共用相同生命週期的資源到相同資源群組，因此您可以以群組為單位輕鬆地部署、更新、刪除它們。 

資源群組會儲存資源相關中繼資料。 因此，當您指定資源群組的位置時，您便是指定中繼資料的儲存位置。 基於相容性理由，您可能需要確保您的資料存放在特定區域中。

1. 若要查看訂用帳戶中的所有資源群組，請選取 [資源群組] 。
   
    ![瀏覽資源群組](./media/resource-group-portal/browse-groups.png)
2. 若要建立空的資源群組，請選取 [新增] 。
   
    ![新增資源群組](./media/resource-group-portal/add-resource-group.png)
3. 提供新資源群組的名稱和位置。 選取 [建立] 。
   
    ![建立資源群組](./media/resource-group-portal/create-empty-group.png)
4. 您可能必須選取 [重新整理]  才能查看最近建立的資源群組。
   
    ![重新整理資源群組](./media/resource-group-portal/refresh-resource-groups.png)
5. 若要自訂針對資源群組顯示的資訊，請選取 [資料行] 。
   
    ![自訂資料行](./media/resource-group-portal/select-columns.png)
6. 選取要新增的資料行，然後選取 [更新] 。
   
    ![新增資料行](./media/resource-group-portal/add-columns.png)
7. 若要了解如何將資源部署至新的資源群組，請參閱 [使用 Resource Manager 範本與 Azure 入口網站來部署資源](resource-group-template-deploy-portal.md)。
8. 若要快速存取資源群組，您可以將資源群組釘選到您的儀表板。
   
    ![釘選資源群組](./media/resource-group-portal/pin-group.png)
9. 儀表板會顯示資源群組及其資源。 您可以選取資源群組或其任意資源，以瀏覽至這些項目。
   
    ![釘選資源群組](./media/resource-group-portal/show-resource-group-dashboard.png)

## <a name="tag-resources"></a>標記資源
您可以將標籤套用至資源群組和資源，以便以邏輯方式組織您的資產。 如需使用標籤的相關資訊，請參閱 [使用標籤來組織您的 Azure 資源](resource-group-using-tags.md)。

[!INCLUDE [resource-manager-tag-resource](../../includes/resource-manager-tag-resources.md)]

## <a name="monitor-resources"></a>監視資源
當您選取資源時，入口網站會顯示預設圖形和資料表，以便監視該資源類型。

1. 選取資源並注意 [監視]  區段。 它包含與該資源類型相關的圖形。 下圖顯示儲存體帳戶的預設監視資料。
   
    ![顯示監視](./media/resource-group-portal/show-monitoring.png)
2. 您可以選取區段上方的省略符號 (...)，將區段釘選到您的儀表板。 您也可以自訂區段大小或完全予以移除。 下圖顯示如何釘選、自訂或移除 [CPU 和記憶體] 區段。
   
    ![釘選區段](./media/resource-group-portal/pin-cpu-section.png)
3. 將此區段釘選到儀表板之後，您會在儀表板上看見摘要。 而且，立即加以選取，您即可看到更多有關資料的詳細資訊。
   
    ![檢視儀表板](./media/resource-group-portal/view-startboard.png)
4. 若要完全自訂透過入口網站監視的資料，瀏覽至預設儀表板，然後選取 [新增儀表板] 。
   
    ![儀表板](./media/resource-group-portal/dashboard.png)
5. 為新儀表板命名，並將圖格拖曳到儀表板。 圖格會由不同選項篩選。
   
    ![儀表板](./media/resource-group-portal/create-dashboard.png)
   
     若要了解如何使用儀表板，請參閱 [Creating and sharing dashboards in the Azure portal (在 Azure 入口網站建立和共用儀表板)](../azure-portal/azure-portal-dashboards.md)。

## <a name="manage-resources"></a>管理資源
在入口網站中檢視資源時，您會看到用於管理特定資源的選項。

![管理資源](./media/resource-group-portal/manage-resources.png)

您可以使用這些選項執行作業，例如啟動和停止虛擬機器，或重新設定虛擬機器的屬性。

## <a name="move-resources"></a>移動資源
如果您需要將資源移至其他資源群組或其他訂用帳戶，請參閱 [將資源移到新的資源群組或訂用帳戶](resource-group-move-resources.md)。

## <a name="lock-resources"></a>鎖定資源
您可以鎖定訂用帳戶、資源群組或資源，以防止組織中的其他使用者不小心刪除或修改重要資源。 如需詳細資訊，請參閱[使用 Azure Resource Manager 來鎖定資源](resource-group-lock-resources.md)。

[!INCLUDE [resource-manager-lock-resources](../../includes/resource-manager-lock-resources.md)]

## <a name="view-your-subscription-and-costs"></a>檢視訂用帳戶和成本
您可以檢視訂用帳戶的相關資訊和所有資源的彙總成本。 選取 [訂用帳戶]  及您想要查看的訂用帳戶。 您可能只有一個訂用帳戶可選取。

![訂用帳戶](./media/resource-group-portal/select-subscription.png)

您會看到完工速率。

![完工速率](./media/resource-group-portal/burn-rate.png)

還有依資源類型的成本分析。

![資源成本](./media/resource-group-portal/cost-by-resource.png)

## <a name="export-template"></a>匯出範本
設定資源群組之後，您可以檢視此資源群組的資源管理員範本。 匯出此範本有兩個優點︰

1. 因為此範本包含所有完整的基礎結構，所以您可以輕鬆地自動進行解決方案的未來部署。
2. 您可以查看代表您的解決方案的 JavaScript 物件標記法 (JSON)，藉此熟悉範本語法。

如需逐步指引，請參閱 [從現有資源匯出 Azure Resource Manager 範本](resource-manager-export-template.md)。

## <a name="delete-resource-group-or-resources"></a>刪除資源群組或資源
刪除資源群組會刪除其內含的所有資源。 您也可以刪除資源群組內的個別資源。 刪除資源群組時請務必小心。 資源群組中包含的資源，可能是其他資源群組中資源的依存項目。

![刪除群組](./media/resource-group-portal/delete-group.png)

## <a name="next-steps"></a>後續步驟
* 若要檢視活動記錄檔，請參閱[使用 Resource Manager 來稽核作業](resource-group-audit.md)。
* 若要檢視有關部署的詳細資料，請參閱[檢視部署作業](resource-manager-deployment-operations.md)。
* 若要透過入口網站部署資源，請參閱 [使用 Resource Manager 範本與 Azure 入口網站來部署資源](resource-group-template-deploy-portal.md)。
* 若要管理資源的存取權，請參閱 [使用角色指派來管理 Azure 訂用帳戶資源的存取權](../role-based-access-control/role-assignments-portal.md)。
* 如需關於企業如何使用 Resource Manager 有效地管理訂用帳戶的指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)。

