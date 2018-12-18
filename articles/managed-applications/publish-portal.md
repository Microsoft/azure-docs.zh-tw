---
title: 透過入口網站發佈 Azure 受控應用程式 | Microsoft Docs
description: 示範如何使用 Azure 入口網站建立 Azure 受控應用程式，以供組織成員使用。
services: managed-applications
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 11/02/2017
ms.author: tomfitz
ms.openlocfilehash: e52acd8587203c4729ac2bcd6e4bbc09620ead86
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35766700"
---
# <a name="publish-a-service-catalog-application-through-azure-portal"></a>透過 Azure 入口網站發佈服務類別目錄應用程式

您可以使用 Azure 入口網站來發佈[受控應用程式](overview.md)，以供組織成員使用。 例如，IT 部門可以發佈受控應用程式，以確保符合組織標準。 這些受控應用程式可透過服務類別目錄取得，而非 Azure Marketplace。

## <a name="prerequisites"></a>必要條件

發佈受控應用程式時，您會指定管理資源的身分識別。 建議您指定 Azure Active Directory 使用者群組。 若要建立 Azure Active Directory 使用者群組，請參閱[在 Azure Active Directory 中建立群組和新增成員](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)。 

包含受控應用程式定義的 .zip 檔案必須可以透過 URI 取得。 建議您將 .zip 檔案上傳至儲存體 blob。 

## <a name="create-managed-application-with-portal"></a>使用入口網站建立受控應用程式

1. 在左上角，選取 [新增]。

   ![新增服務](./media/publish-portal/new.png)

1. 搜尋**服務類別目錄**。

1. 在結果中捲動，直到您找到**服務類別目錄受控應用程式定義**。 請選取它。

   ![搜尋受控應用程式定義](./media/publish-portal/select-managed-apps-definition.png)

1. 選取 [建立] 以開始建立受控應用程式定義的程序。

   ![建立受控應用程式定義](./media/publish-portal/create-definition.png)

1. 提供名稱、顯示名稱、描述、位置、訂用帳戶與資源群組的值。 針對套件檔案 URI，提供您建立之 zip 檔案的路徑。

   ![提供值](./media/publish-portal/fill-application-values.png)

1. 當您來到 [驗證和鎖定層級] 區段時，選取 [新增授權]。

   ![新增授權](./media/publish-portal/add-authorization.png)

1. 選取 Azure Active Directory 群組來管理資源，然後選取 [確定]。

   ![新增授權群組](./media/publish-portal/add-auth-group.png)

1. 當您已提供所有值時，選取 [建立]。

   ![建立受控應用程式](./media/publish-portal/create-app.png)

## <a name="next-steps"></a>後續步驟

* 如需受控應用程式的簡介，請參閱[受控應用程式概觀](overview.md)。
* 如需受控應用程式範例，請參閱[適用於 Azure 受控應用程式之範例專案](sample-projects.md)。
* 若要了解如何建立受控應用程式的 UI 定義檔案，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。