---
title: 使用 Azure 入口網站部署 Azure 資源 | Microsoft Docs
description: 使用 Azure 入口網站和 Azure Resource Manager 來部署資源。
services: azure-resource-manager,azure-portal
documentationcenter: ''
author: tfitzmac
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/03/2018
ms.author: tomfitz
ms.openlocfilehash: b8a6d58ad79b591548542d69e69b71ec61af79af
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220120"
---
# <a name="deploy-resources-with-resource-manager-templates-and-azure-portal"></a>使用 Resource Manager 範本與 Azure 入口網站來部署資源

此文章示範如何使用 [Azure 入口網站](https://portal.azure.com)搭配 [Azure Resource Manager](resource-group-overview.md) 來部署您的 Azure 資源。 若要了解如何管理您的資源，請參閱 [透過入口網站管理 Azure 資源](resource-group-portal.md)。

## <a name="create-resource-group"></a>建立資源群組

1. 若要建立空的資源群組，請選取 [資源群組]。

   ![選取資源群組](./media/resource-group-template-deploy-portal/select-resource-groups.png)

1. 在 [資源群組] 底下選取 [新增]。

   ![新增資源群組](./media/resource-group-template-deploy-portal/add-resource-group.png)

1. 提供其名稱和位置，再視需要選取訂用帳戶。 您需要提供資源群組的位置，因為資源群組會儲存資源的相關中繼資料。 為了符合法規，您可能會想要指定中繼資料的儲存位置。 一般情況下，我們建議您指定大部分資源所在的位置。 使用相同位置可簡化範本。

   ![設定群組值](./media/resource-group-template-deploy-portal/set-group-properties.png)

   當您完成設定屬性時，選取 [建立]。

1. 若要查看新的資源群組，請選取 [重新整理]。

   ![重新整理資源群組](./media/resource-group-template-deploy-portal/refresh-resource-groups.png)

## <a name="deploy-resources-from-marketplace"></a>從 Marketplace 部署資源

建立資源群組之後，您可以將資源從 Marketplace 部署至該群組。 Marketplace 針對常見的案例提供預先定義的解決方案。

1. 若要開始部署，請選取 [建立資源]。

   ![新增資源](./media/resource-group-template-deploy-portal/new-resources.png)

1. 尋找您想要部署的資源類型。

   ![選取 [資源類型]](./media/resource-group-template-deploy-portal/select-resource-type.png)

1. 如果沒有看到要部署的特定解決方案，您可以在 Marketplace 搜尋。 例如，若要尋找 Wordpress 解決方案，請先輸入 **Wordpress**，並選取您想要的選項。

   ![搜尋 Marketplace](./media/resource-group-template-deploy-portal/search-resource.png)

1. 根據所選資源的類型，您必須在部署前設定相關的屬性集合。 對於所有類型，您必須選取目的地資源群組。 下圖顯示如何建立 Web 應用程式並將其部署至您建立的資源群組。

   ![建立資源群組](./media/resource-group-template-deploy-portal/select-existing-group.png)

   或者，您也可以決定在部署資源時建立資源群組。 選取 [新建]  並提供資源群組的名稱。

   ![建立新的資源群組](./media/resource-group-template-deploy-portal/select-new-group.png)

1. 您的部署隨即開始。 部署可能需要幾分鐘的時間。 部署完成後，您就會看到通知。

   ![檢視通知](./media/resource-group-template-deploy-portal/view-notification.png)

1. 部署您的資源之後，您可以選取 [新增]，將更多資源新增至資源群組。

   ![新增資源](./media/resource-group-template-deploy-portal/add-resource.png)

## <a name="deploy-resources-from-custom-template"></a>從自訂範本部署資源

如果您想要執行部署，但不使用 Marketplace 中的任何範本，您可建立自訂範本以定義您的解決方案的基礎結構。 若要了解如何建立範本，請參閱[了解 Azure Resource Manager 範本的結構和語法](resource-group-authoring-templates.md)。

> [!NOTE]
> 入口網站介面不支援參考 [Key Vault 中的祕密](resource-manager-keyvault-parameter.md)。 請改用 [PowerShell](resource-group-template-deploy.md) 或 [Azure CLI](resource-group-template-deploy-cli.md)，在本機或從外部 URI 部署您的範本。

1. 若要透過入口網站來部署自訂的範本，請選取 [建立資源]，並搜尋 [範本部署]，直到您可以從選項中選取它為止。

   ![搜尋範本部署](./media/resource-group-template-deploy-portal/search-template.png)

1. 選取 [建立] 。

   ![選取 [建立]](./media/resource-group-template-deploy-portal/show-template-option.png)

1. 您會看到建立範本的幾個選項。 選取 [在編輯器中組建您自己的範本]。

   ![檢視選項](./media/resource-group-template-deploy-portal/see-options.png)

1. 有可供自訂的空白範本。

   ![建立範本](./media/resource-group-template-deploy-portal/blank-template.png)

1. 您可以手動編輯 JSON 語法，也可以從[快速入門範本資源庫](https://azure.microsoft.com/resources/templates/)選取預先建立的範本。 不過，此文章會使用 [新增資源] 選項。

   ![編輯範本](./media/resource-group-template-deploy-portal/select-add-resource.png)

1. 選取 [儲存體帳戶]，並提供名稱。 提供值完畢後，選取 [確定]。

   ![選取儲存體帳戶](./media/resource-group-template-deploy-portal/add-storage-account.png)

1. 編輯器會自動新增資源類型的 JSON。 請注意，其包含了用於定義儲存體帳戶類型的參數。 選取 [ **儲存**]。

   ![顯示範本](./media/resource-group-template-deploy-portal/show-json.png)

1. 現在，有選項可以用來部署範本中定義的資源。 若要部署，請同意條款與條件，並選取 [購買]。

   ![部署範本](./media/resource-group-template-deploy-portal/provide-custom-template-values.png)

## <a name="deploy-resources-from-a-template-saved-to-your-account"></a>從儲存至您帳戶的範本部署資源

入口網站可讓您將範本儲存至您的 Azure 帳戶，並於日後部署它。 如需範本的詳細資訊，請參閱[建立及部署第一個 Azure Resource Manager 範本](resource-manager-create-first-template.md)。

1. 若要尋找您已儲存的範本，請選取 [更多服務]。

   ![更多服務](./media/resource-group-template-deploy-portal/more-services.png)

1. 搜尋**範本**並選取該選項。

   ![搜尋範本](./media/resource-group-template-deploy-portal/find-templates.png)

1. 在儲存至您帳戶的範本清單中，選取您要使用的範本。

   ![已儲存的範本](./media/resource-group-template-deploy-portal/saved-templates.png)

1. 選取 [部署]  來重新部署這個已儲存的範本。

   ![部署已儲存的範本](./media/resource-group-template-deploy-portal/deploy-saved-template.png)

## <a name="next-steps"></a>後續步驟
* 若要檢視稽核記錄檔，請參閱 [使用 Resource Manager 來稽核作業](resource-group-audit.md)。
* 若要針對部署錯誤進行疑難排解，請參閱[檢視部署作業](resource-manager-deployment-operations.md)。
* 若要從部署或資源群組擷取範本，請參閱 [從現有資源匯出 Azure Resource Manager 範本](resource-manager-export-template.md)。
* 若要安全地在多個區域推出您的服務，請參閱 [Azure 部署管理員](deployment-manager-overview.md)。
