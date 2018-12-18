---
title: 管理 Azure Active Directory 中的自訂網域名稱 | Microsoft Docs
description: Azure Active Directory 中管理網域的管理概念和做法
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 11/14/2017
ms.author: curtand
ms.reviewer: elkuzmen
ms.openlocfilehash: b503ef4c5cd922d4f7b58940cfc98a83a78ae986
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37448866"
---
# <a name="managing-custom-domain-names-in-your-azure-active-directory"></a>管理 Azure Active Directory 中的自訂網域名稱
網域名稱是許多目錄資源識別項的重要部分：它可能是使用者的使用者名稱或電子郵件地址的一部分、群組位址的一部分，也可能是應用程式的應用程式識別碼 URI 的一部分。 Azure Active Directory (Azure AD) 中的資源可包含已驗證為目錄 (其中含有資源) 所擁有的網域名稱。 只有全域管理員可以在 Azure AD 中執行網域管理工作。

## <a name="set-the-primary-domain-name-for-your-azure-ad-directory"></a>設定 Azure AD 目錄的主要網域名稱
建立目錄時，初始網域名稱 (例如 ‘contoso.onmicrosoft.com’) 同時也是主要網域名稱。 當您建立新的使用者時，主要網域即為新使用者的預設網域名稱。 設定主要網域名稱可以簡化系統管理員在入口網站中建立新使用者的程序。 變更主要網域名稱：

1. 使用具備目錄全域管理員身分的帳戶來登入 [Azure 入口網站](https://portal.azure.com) 。
2. 選取 **Azure Active Directory**。
3. 選取 [自訂網域名稱]。
     
   ![開啟使用者管理](./media/domains-manage/add-custom-domain.png)
4. 選取您想要設為主要網域的網域名稱。
5. 選取 [設為主要] 命令。 出現提示時，請確認您的選擇。
   
   ![將網域名稱設為主要網域名稱](./media/domains-manage/make-primary-domain.png)

您可以將目錄的主要網域名稱變更為任何已驗證的非同盟自訂網域。 變更目錄的主要網域時，並不會變更任何現有使用者的使用者名稱。

## <a name="add-custom-domain-names-to-your-azure-ad-tenant"></a>為 Azure AD 租用戶新增自訂網域名稱
您最多可以新增 900 個受控網域名稱。 如果您要設定所有網域與內部部署 Active Directory 建立同盟，則最多可以在每個目錄中新增 450 個網域名稱。 如需詳細資訊，請參閱[同盟和受控網域名稱](https://docs.microsoft.com/azure/active-directory/active-directory-add-domain-concepts#federated-and-managed-domain-names)。

## <a name="add-subdomains-of-a-custom-domain"></a>新增自訂網域的子網域
如果您想要將第三層網域名稱新增至您的目錄，例如 'europe.contoso.com'，則應先新增並驗證第二層網域，例如 contoso.com。 Azure AD 會自動驗證子網域。 若要查看您剛才新增的子網域是否已通過驗證，請重新整理列出網域的瀏覽器頁面。

## <a name="what-to-do-if-you-change-the-dns-registrar-for-your-custom-domain-name"></a>如果您變更您的自訂網域名稱的 DNS 註冊機構，該怎麼辦
如果您變更自訂網域名稱的 DNS 註冊機構，仍可以繼續在 Azure AD 中使用自訂網域名稱而不會中斷，且不需要其他組態工作。 如果您的自訂網域名稱搭配使用 Office 365、Intune 或仰賴 Azure AD 自訂網域名稱的其他服務，則請參閱這些服務的文件。

## <a name="delete-a-custom-domain-name"></a>刪除自訂網域名稱
如果貴組織不再使用該網域名稱，或如果您需要在另一個 Azure AD 中使用該網域名稱，您可以從您的 Azure AD 刪除自訂網域名稱。

若要刪除自訂網域名稱，您必須先確定目錄中沒有任何資源仰賴該網域名稱。 如果有下列情況，即無法刪除目錄的網域名稱：

* 有任何使用者的使用者名稱、電子郵件地址或 Proxy 位址包含該網域名稱。
* 有任何群組的電子郵件地址或 Proxy 位址包含該網域名稱。
* Azure AD 中任何應用程式的應用程式識別碼 URI 包括該網域名稱。

您必須變更或刪除 Azure AD 目錄中任何這類資源，才能刪除自訂網域名稱。

## <a name="use-powershell-or-graph-api-to-manage-domain-names"></a>使用 PowerShell 或圖形 API 來管理網域名稱
在 Azure Active Directory 中，大部分的網域名稱管理工作也都可以使用 Microsoft PowerShell 來完成，或使用 Azure AD 圖形 API 以程式設計方式來完成。

* [使用 PowerShell 管理 Azure AD 中的網域名稱](https://msdn.microsoft.com/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains)
* [使用圖形 API 管理 Azure AD 中的網域名稱](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/domains-operations)

## <a name="next-steps"></a>後續步驟
* [新增自訂網域名稱](../fundamentals/add-custom-domain.md)

