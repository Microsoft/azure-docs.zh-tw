---
title: 在 Azure API 管理中使用群組管理開發人員帳戶 | Microsoft Docs
description: 了解如何在 Azure API 管理中使用群組來管理管理開發人員帳戶。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/13/2018
ms.author: apimpm
ms.openlocfilehash: 3986b07c3568c3dcbb4077361d38f74d658458cd
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38603059"
---
# <a name="how-to-create-and-use-groups-to-manage-developer-accounts-in-azure-api-management"></a>如何在 Azure API 管理中建立和使用群組來管理開發人員帳戶
在 API 管理中，群組的作用是管理產品對於開發人員的可見度。 產品是先設為可讓群組看見，然後群組中的開發人員就能檢視和訂閱與群組相關聯的產品。 

API 管理具有下列不可變的系統群組：

* **管理員** - Azure 訂用帳戶管理員是此群組的成員。 管理員可管理 API 管理服務執行個體、建立開發人員所使用的 API、作業和產品。
* **開發人員** - 已驗證開發人員入口網站使用者屬於此群組。 開發人員是使用您的 API 建置應用程式的客戶。 開發人員獲授與開發人員入口網站的存取權，並建置呼叫 API 作業的應用程式。
* **來賓** - 未經驗證的開發人員入口網站使用者 (例如，造訪 API 管理執行個體之開發人員入口網站的潛在客戶) 屬於此群組。 他們可獲得特定唯讀存取權限，例如他們可檢視 API 但無法進行呼叫。

除了這些系統群組以外，管理員還可以建立自訂群組，或使用 [使用 Azure Active Directory 相關租用戶中的外部群組][leverage external groups in associated Azure Active Directory tenants]。 自訂群組和外部群組可以與系統群組一起使用，提供開發人員 API 產品的能見度及存取權。 例如，您可以為與特定夥伴組織有關的開發人員建立一個自訂群組，並只允許他們存取來自含相關 API 之產品的 API。 使用者可以是多個群組的成員。

本指南說明 API 管理執行個體的管理員如何加入新的群組，並將這些群組與產品和開發人員建立關聯。

除了在發佈者入口網站中建立和管理群組，您還可以使用 API 管理 REST API [群組](https://docs.microsoft.com/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity) 實體，建立和管理您的群組。

## <a name="prerequisites"></a>先決條件

完成本文中的工作：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-group"> </a>建立群組

本節說明如何將新的群組新增至 API 管理帳戶。

1. 選取畫面左側的 [群組] 索引標籤。
2. 按一下 [+新增]。
3. 輸入群組的唯一名稱和選用的描述。
4. 按下 [建立] 。

    ![新增群組](./media/api-management-howto-create-groups/groups001.png)

一旦建立群組，它就會新增至 [群組] 清單。 <br/>若要編輯群組的 [名稱] 或 [說明]，請按一下群組名稱和 [設定]。<br/>若要刪除群組，按一下群組名稱，然後按下 [刪除]。

現在已建立群組，它可以與產品和開發人員建立關聯。

## <a name="associate-group-product"> </a>將群組與產品建立關聯

1. 選取左側的 [產品] 索引標籤。
2. 按一下所需產品的名稱。
3. 按下 [存取控制]。
4. 按一下 [+ 新增群組]。

    ![建立群組與產品的關聯](./media/api-management-howto-create-groups/groups002.png)
5. 選取您想要新增的群組。

    ![建立群組與產品的關聯](./media/api-management-howto-create-groups/groups003.png)

    若要從產品中移除群組，按一下 [刪除]。

    ![刪除群組](./media/api-management-howto-create-groups/groups004.png)

當產品與群組建立關聯之後，該群組中的開發人員就可以檢視和訂閱產品。

> [!NOTE]
> 若要加入 Azure Active Directory 群組，請參閱 [如何在 Azure API 管理中使用 Azure Active Directory 授權開發人員帳戶](api-management-howto-aad.md)。

## <a name="associate-group-developer"> </a>將群組與開發人員建立關聯

本節說明如何讓群組與成員產生關聯。

1. 選取畫面左側的 [群組] 索引標籤。
2. 選取 [成員]。

    ![新增成員](./media/api-management-howto-create-groups/groups005.png)
3. 按下 [+新增] 然後選取成員。

    ![新增成員](./media/api-management-howto-create-groups/groups006.png)
4. 按 [選取]。

在開發人員與群組之間加入關聯之後，您可以在 [ **使用者** ] 索引標籤中檢視此關聯。

## <a name="next-steps"> </a>後續步驟

* 當開發人員加入至群組之後，他們就可以檢視和訂閱與該群組相關聯的產品。 如需詳細資訊，請參閱 [如何在 Azure API 管理中建立和發佈產品][How create and publish a product in Azure API Management]。
* 除了在發佈者入口網站中建立和管理群組，您還可以使用 API 管理 REST API [群組](https://docs.microsoft.com/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity) 實體，建立和管理您的群組。

[Create a group]: #create-group
[Associate a group with a product]: #associate-group-product
[Associate groups with developers]: #associate-group-developer
[Next steps]: #next-steps

[How create and publish a product in Azure API Management]: api-management-howto-add-products.md

[Get started with Azure API Management]: get-started-create-service-instance.md
[Create an API Management service instance]: get-started-create-service-instance.md
[leverage external groups in associated Azure Active Directory tenants]: api-management-howto-aad.md
