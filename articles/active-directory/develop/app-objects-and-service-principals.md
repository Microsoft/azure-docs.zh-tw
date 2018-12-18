---
title: Azure Active Directory 中的應用程式和服務主體物件
description: 了解 Azure Active Directory 中應用程式物件與服務主體物件之間的關聯性。
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
services: active-directory
editor: ''
ms.assetid: adfc0569-dc91-48fe-92c3-b5b4833703de
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.custom: aaddev
ms.reviewer: sureshja
ms.openlocfilehash: f73c4f7f606f264f899aeb6405ac7bfae71e518d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948044"
---
# <a name="application-and-service-principal-objects-in-azure-active-directory"></a>Azure Active Directory 中的應用程式和服務主體物件

在 Azure Active Directory (Azure AD) 的內容中使用「應用程式」這個詞彙時，有時會誤解其意義。 本文釐清 Azure AD 應用程式整合的概念和具體層面，並舉例說明如何註冊和同意[多租用戶應用程式](developer-glossary.md#multi-tenant-application)，提供更清楚的說明。

## <a name="overview"></a>概觀

已與 Azure AD 整合的應用程式，其含意已超出軟體層面。 「應用程式」經常當作概念詞彙，不僅指應用程式軟體，在執行階段的驗證/授權「對話」中，也是指其 Azure AD 註冊和角色。

根據定義，應用程式可以這些角色運作：

- [用戶端](developer-glossary.md#client-application)角色 (取用資源)
- [資源伺服器](developer-glossary.md#resource-server)角色 (對用戶端公開 API)
- 用戶端角色和資源伺服器角色

[OAuth 2.0 授權授與流程](developer-glossary.md#authorization-grant)定義對話通訊協定，可讓用戶端/資源各自存取/保護資源的資料。

在下列各節中，您會看到 Azure AD 應用程式模型在設計階段和執行階段是如何代表應用程式。

## <a name="application-registration"></a>應用程式註冊

當您在 [Azure 入口網站][AZURE-Portal]註冊 Azure AD 應用程式時，Azure AD 租用戶中會建立兩個物件︰

- 應用程式物件，以及
- 服務主體物件

### <a name="application-object"></a>應用程式物件

Azure AD 應用程式是由其唯一一個應用程式物件所定義，該物件位於應用程式註冊所在的 Azure AD 租用戶，也稱為應用程式的「主要」租用戶。 Azure AD Graph [Application 實體][AAD-Graph-App-Entity]會定義應用程式物件屬性的結構描述。

### <a name="service-principal-object"></a>服務主體物件

若要存取受到 Azure AD 租用戶保護的資源，需要存取權的實體必須以安全性主體呈現。 這同時適用於使用者 (使用者主體) 和應用程式 (服務主體)。

安全性主體會定義 Azure AD 租用戶中使用者/應用程式的存取原則和權限。 此動作可啟用核心功能，例如登入期間的使用者/應用程式驗證，以及資源存取期間的授權。

當應用程式擁有權限可存取租用戶中的資源時 (通過註冊時或[同意](developer-glossary.md#consent))，服務主體物件就會隨即建立。 Azure AD Graph [ServicePrincipal 實體][AAD-Graph-Sp-Entity]會定義服務主體物件屬性的結構描述。

### <a name="application-and-service-principal-relationship"></a>應用程式和服務主體關聯性

將應用程式物件視為應用程式的「全域」代表 (用於所有租用戶)，而將服務主體看做是「本機」代表 (用於特定租用戶)。

應用程式物件就像範本，可從中「衍生」通用和預設屬性，用以建立相對應的服務主體物件。 因此，應用程式物件與軟體應用程式之間存在 1:1 關聯性，而與其對應的服務主體物件之間存在一對多關聯性。

每一個會用到應用程式的租用戶中必須建立服務主體，才能讓它建立身分識別來登入及/或存取租用戶所保護的資源。 單一租用戶的應用程式只能有一個服務主體 (在其主租用戶中)，並在應用程式註冊期間建立和同意使用。 如果是多租用戶 Web 應用程式/API，則在使用者已同意使用它的每個租用戶中，還會建立服務主體。 

> [!NOTE]
> 您對應用程式物件所做的任何變更也只會反映於它在應用程式的主要租用戶 (其註冊所在租用戶) 中的服務主體物件。 就多租用戶應用程式而言，對應用程式物件所做的變更，必須等到透過[應用程式存取面板](https://myapps.microsoft.com)移除存取權再重新授權之後，才會反映在任何取用者租用戶的服務主體物件中。
>
> 也請注意，依預設，原生應用程式會註冊為多租用戶。

## <a name="example"></a>範例

下圖說明應用程式的應用程式物件與對應的服務主體物件之間的關係，是以一個稱為「HR 應用程式」 的範例多租用戶應用程式為背景。 此範例案例中有三個 Azure AD 租用戶︰

- **Adatum** - 開發 **HR 應用程式**的公司所使用的租用戶
- **Contoso** - Contoso 組織所使用的租用戶，其為 **HR 應用程式**的取用者
- **Fabrikam** - Fabrikam 組織所使用的租用戶，其亦會取用 **HR 應用程式**

![應用程式物件和服務主體物件之間的關聯性](./media/app-objects-and-service-principals/application-objects-relationship.png)

在此範例案例中：

| 步驟 | 說明 |
|------|-------------|
| 1    | 在應用程式的主要租用戶中建立應用程式和服務主體物件的程序。 |
| 2    | 當 Contoso 和 Fabrikam 的系統管理員完成同意，系統就會在其公司的 Azure AD 租用戶中建立服務主體物件，並指派系統管理員所授與的權限。 也請注意，HR 應用程式可能會設定/設計為允許由使用者同意以進行個人使用。 |
| 3    | HR 應用程式的取用者租用戶 (Contoso 和 Fabrikam) 都分別擁有自己的服務主體物件。 每個均代表他們在執行階段的應用程式執行個體使用，其中皆受到個別系統管理員所同意的權限控管。 |

## <a name="next-steps"></a>後續步驟

- 您可以使用 [Azure AD Graph 總管](https://graphexplorer.azurewebsites.net/)來查詢應用程式和服務主體物件。
- 您可以使用 Azure AD Graph API、[Azure 入口網站][AZURE-Portal]的應用程式資訊清單編輯器，或 [Azure AD PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview?view=azureadps-2.0) 來存取應用程式的應用程式物件 (如其 OData 的 [Application 實體][AAD-Graph-App-Entity]所代表)。
- 您可以透過 Azure AD Graph API 或 [Azure AD PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview?view=azureadps-2.0) 來存取應用程式的服務主體物件 (如其 OData 的 [ServicePrincipal 實體][AAD-Graph-Sp-Entity] 所代表)。

<!--Image references-->

<!--Reference style links -->
[AAD-Graph-App-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity
[AAD-Graph-Sp-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity
[AZURE-Portal]: https://portal.azure.com
