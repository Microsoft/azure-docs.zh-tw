---
title: 設定執行個體和驗證 (入口網站)
titleSuffix: Azure Digital Twins
description: 瞭解如何使用 Azure 入口網站設定 Azure 數位 Twins 服務的實例
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: contperf-fy21q2
ms.openlocfilehash: 61b396cbcc8c91c75c961f702de7ed6a33e676e4
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98947012"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-portal"></a>設定 Azure 數位 Twins 實例和驗證 (入口網站) 

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

本文涵蓋 **設定新 Azure 數位 Twins 實例** 的步驟，包括建立實例和設定驗證。 完成本文之後，您將會有可開始進行程式設計的 Azure 數位 Twins 實例。

這一版的文章會使用 Azure 入口網站手動逐一完成這些步驟。 Azure 入口網站是統一的 Web 式主控台，提供有別於命令列工具的替代方案。
* 若要使用 CLI 手動完成這些步驟，請參閱這篇文章的 CLI 版本： [*如何：設定 (cli) 的實例和驗證*](how-to-set-up-instance-cli.md)。
* 若要使用部署腳本範例來執行自動安裝，請參閱本文的腳本版本： [*如何：設定實例和驗證 (腳本)*](how-to-set-up-instance-scripted.md)。

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

## <a name="create-the-azure-digital-twins-instance"></a>建立 Azure 數位 Twins 實例

在本節中，您將使用 [Azure 入口網站](https://ms.portal.azure.com/)**建立 Azure 數位 Twins 的新實例**。 流覽至入口網站，並使用您的認證登入。

在入口網站中，從選取 [Azure 服務] 首頁功能表中的 [ _建立資源_ ] 開始。

:::image type="content" source= "media/how-to-set-up-instance/portal/create-resource.png" alt-text="從 Azure 入口網站的首頁中選取 [建立資源]":::

在搜尋方塊中搜尋 *Azure 數位 Twins* ，然後從結果中選擇 **azure 數位 Twins** 服務。 選取 [ _建立_ ] 按鈕，以建立服務的新實例。

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins.png" alt-text="從 Azure 數位 Twins 服務頁面選取 [建立]":::

在 [ **建立資源** ] 頁面上，填入下列值：
* **訂** 用帳戶：您所使用的 Azure 訂用帳戶
  - **資源群組**：用來部署實例的資源群組。 如果您還沒有現有的資源群組，您可以選取 [ *建立新* 的連結]，並輸入新資源群組的名稱，在這裡建立一個資源群組。
* **位置**：適用于部署的 Azure 數位啟用 Twins 區域。 如需區域支援的詳細資訊，請造訪 [*依區域提供的 azure 產品 (Azure 數位 Twins)*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins)。
* **資源名稱**： Azure 數位 Twins 實例的名稱。 如果您的訂用帳戶在已使用指定名稱的區域中有另一個 Azure 數位 Twins 實例，系統會要求您挑選不同的名稱。

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins-2.png" alt-text="填寫描述的值以建立 Azure 數位 Twins 資源":::

當您完成時，如果您不想為您的實例設定其他設定，可以選取 [ **審核 + 建立** ]。 這會將您帶到摘要頁面，您可以在其中查看您所輸入的實例詳細資料，並完成 **建立**。 

如果您想要為您的實例設定更多詳細資料，下一節將說明其餘的安裝程式索引標籤。

### <a name="additional-setup-options"></a>其他設定選項

以下是您可以在安裝期間設定的其他選項，方法是使用 [ **建立資源** ] 進程中的其他索引標籤。

* **網路** 功能：在此索引標籤中，您可以使用 [Azure Private Link](../private-link/private-link-overview.md) 來啟用私人端點，以消除實例的公用網路公開。 如需相關指示，請參閱作法 [*：使用 Private Link 啟用私用存取*](how-to-enable-private-link.md#add-a-private-endpoint-during-instance-creation)。
* **Advanced**：在此索引標籤中，您可以為您的實例啟用 [系統管理](../active-directory/managed-identities-azure-resources/overview.md) 的身分識別，以便在將事件轉送至 [端點](concepts-route-events.md)時使用。 如需相關指示，請參閱作法 [*：啟用路由事件的受控*](how-to-enable-managed-identities.md)識別。
* **標記**：在此索引標籤中，您可以將標記新增至您的實例，以協助您在 Azure 資源之間進行組織。 如需 Azure 資源標記的詳細資訊，請參閱 [*為邏輯組織標記資源、資源群組和*](../azure-resource-manager/management/tag-resources.md)訂用帳戶。

### <a name="verify-success-and-collect-important-values"></a>驗證成功並收集重要值

選取 [ **建立**] 完成您的實例設定之後，您可以透過入口網站圖示列，在您的 Azure 通知中查看實例部署的狀態。 通知將會指出部署成功的時間，您將能夠選取 [ _移至資源_ ] 按鈕來查看您建立的實例。

:::image type="content" source="media/how-to-set-up-instance/portal/notifications-deployment.png" alt-text="顯示成功部署並反白顯示 [移至資源] 按鈕的 Azure 通知的查看":::

或者，如果部署失敗，通知將會指出原因。 觀察錯誤訊息的建議，然後重試建立實例。

>[!TIP]
>建立實例之後，您可以隨時在 Azure 入口網站搜尋列中搜尋實例的名稱，以返回其頁面。

從實例的 *[總覽* ] 頁面中，記下其 *名稱*、 *資源群組* 和 *主機名稱*。 當您繼續使用 Azure 數位 Twins 實例時，這些都是您可能需要的重要值。 如果其他使用者將針對該實例進行程式設計，您應該與這些使用者共用這些值。

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="從實例的總覽頁面中反白顯示重要值":::

您現在已準備好開始使用 Azure 數位 Twins 實例。 接下來，您將授與適當的 Azure 使用者權限來管理它。

## <a name="set-up-user-access-permissions"></a>設定使用者存取權限

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

首先，在 Azure 入口網站中開啟 Azure 數位 Twins 實例的頁面。 從實例的功能表中，選取 [ *存取控制] (IAM)*。 選取 [  **+ 新增** ] 按鈕以新增角色指派。

:::image type="content" source="media/how-to-set-up-instance/portal/add-role-assignment-1.png" alt-text="從 [存取控制 (IAM) ] 頁面中選取要新增角色指派":::

在 [ *新增角色指派* ] 頁面上，填入 [Azure 訂用帳戶) 中具有 [足夠許可權](#prerequisites-permission-requirements) 的使用者必須完成的值 (：
* **角色**：從下拉式功能表中選取 *Azure 數位 Twins 資料擁有* 者
* **指派存取權給**：使用 *使用者、群組或服務主體*
* **選取**：搜尋要指派之使用者的名稱或電子郵件地址。 當您選取結果時，使用者將會顯示在 [ *選取的成員* ] 區段中。

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-set-up-instance/portal/add-role-assignment-2.png" alt-text="將列出的欄位填入 [新增角色指派] 對話方塊":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

當您完成輸入詳細資料時，請按 [ *儲存* ] 按鈕。

### <a name="verify-success"></a>確認是否成功

您可以在 [ *存取控制] (IAM) > 角色指派* 下，查看您所設定的角色指派。 使用者應該會顯示在清單中，且具有 *Azure 數位 Twins 資料擁有* 者的角色。 

:::image type="content" source="media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="在 Azure 入口網站中查看 Azure 數位 Twins 實例的角色指派":::

您現在已準備好開始使用 Azure 數位 Twins 實例，並已獲指派管理該實例的許可權。

## <a name="next-steps"></a>後續步驟

使用 Azure 數位 Twins CLI 命令，在您的實例上測試個別 REST API 呼叫： 
* [az dt 參考](/cli/azure/ext/azure-iot/dt?preserve-view=true&view=azure-cli-latest)
* [操作說明：*使用 Azure Digital Twins CLI*](how-to-use-cli.md)

或者，請參閱如何使用驗證碼將用戶端應用程式連接到您的實例：
* [*How to：撰寫應用程式驗證碼*](how-to-authenticate-client.md)