---
title: Azure 服務匯流排角色型存取控制 (RBAC) 預覽 | Microsoft Docs
description: Azure 服務匯流排角色型存取控制
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/19/2018
ms.author: spelluru
ms.openlocfilehash: 0da4fdf5afc0cbe76e19260b486aa9112d0d74c8
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854315"
---
# <a name="active-directory-role-based-access-control-preview"></a>Active Directory 角色型存取控制 (預覽)

Microsoft Azure 針對以 Azure Active Directory (Azure AD) 為基礎的資源和應用程式提供了整合式的存取控制管理功能。 使用 Azure AD，您即可管理使用者帳戶和應用程式 (具體而言是您的 Azure 型應用程式)，也可以讓您現有的 Active Directory 基礎結構與 Azure AD 同盟，以便提供也會跨越 Azure 資源和 Azure 託管應用程式的全公司單一登入功能。 然後，您可以將這些 Azure AD 使用者和應用程式身分識別指派給全域和服務特有的角色，以便授與 Azure 資源的存取權。

對於 Azure 服務匯流排來說，透過 Azure 入口網站和 Azure 資源管理 API 來的管理命名空間和所有相關資源的作業，已使用「角色型存取控制 (RBAC)」模型來加以保護。 執行階段作業的 RBAC 目前是處於公開預覽狀態的功能。 

使用 Azure AD RBAC 的應用程式不需要處理專屬於服務匯流排的 SAS 規則和金鑰或任何其他存取權杖。 用戶端應用程式會與 Azure AD 互動以建立驗證內容，並取得服務匯流排的存取權杖。 在使用需要互動式登入的網域使用者帳戶時，應用程式永遠不會直接處理任何認證。

## <a name="service-bus-roles-and-permissions"></a>服務匯流排角色和權限

在初始公開預覽中，您只能將 Azure AD 帳戶和服務主體新增至服務匯流排傳訊命名空間的「擁有者」或「參與者」角色。 此作業會對身分識別授與命名空間中所有實體的完全控制權。 會變更命名空間拓撲的管理作業一開始只能透過 Azure 資源管理來獲得支援，而無法透過原生服務匯流排 REST 管理介面來獲得支援。 這項支援也表示 .NET Framework 用戶端 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 物件無法與 Azure AD 帳戶搭配使用。  

## <a name="use-service-bus-with-an-azure-ad-domain-user-account"></a>搭配使用服務匯流排與 Azure AD 網域使用者帳戶

下列章節會說明要建立及執行應用程式範例 (會提示互動式 Azure AD 使用者進行登入) 所需進行的步驟，如何對服務匯流排授與該使用者帳戶的存取權，以及如何使用該身分識別來存取事件中樞。 

此簡介會說明簡單的主控台應用程式，[其程式碼位於 Github 上](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/RoleBasedAccessControl)。

### <a name="create-an-active-directory-user-account"></a>建立 Active Directory 使用者帳戶

第一個步驟是選擇性的。 每個 Azure 訂用帳戶都會自動與 Azure Active Directory 租用戶配對，如果您可以存取 Azure 訂用帳戶，則您的使用者帳戶已完成註冊。 這表示您可以直接使用您的帳戶。 

如果您仍想建立專供此案例使用的帳戶，請[遵循這些步驟](../automation/automation-create-aduser-account.md)。 您必須擁有在 Azure Active Directory 租用戶中建立帳戶的權限，如果是大型企業的案例，則需要擁有的就不一定是此權限。

### <a name="create-a-service-bus-namespace"></a>建立服務匯流排命名空間

接下來，在具有 RBAC 之預覽支援的其中一個 Azure 區域中[建立服務匯流排傳訊命名空間](service-bus-create-namespace-portal.md)：**美國東部**、**美國東部 2** 或**西歐**。 

命名空間建立好之後，瀏覽至其位於入口網站上的 [存取控制 (IAM)] 頁面，然後按一下 [新增] 以將 Azure AD 使用者帳戶新增至「擁有者」角色。 如果您使用自己的使用者帳戶，而且您已建立命名空間，則您已加入「擁有者」角色。 若要在角色中新增不同帳戶，請在 [新增權限] 面板的 [選取] 欄位中搜尋 Web 應用程式的名稱，然後按一下該項目。 然後按一下 [儲存] 。

![](./media/service-bus-role-based-access-control/rbac1.PNG)

使用者帳戶現在會具有服務匯流排命名空間的存取權，以及您先前建立之佇列的存取權。
 
### <a name="register-the-application"></a>註冊應用程式

若要能夠執行應用程式範例，請先讓其在 Azure AD 中進行註冊，並核准同意提示以便允許應用程式代表其存取 Azure 服務匯流排。 

由於應用程式範例是一個主控台應用程式，您必須註冊原生應用程式，並將 **Microsoft.ServiceBus** 的 API 權限新增至「所需的權限」集合。 原生應用程式也需要 Azure AD 中的 **redirect-URI** 以作為識別碼；此 URI 不必是網路目的地。 在此範例中使用 `http://servicebus.microsoft.com`，因為程式碼範例已使用該 URI。

[這個教學課程](../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md)會有詳細註冊步驟的說明。 請遵循這些步驟來註冊**原生**應用程式，然後遵循更新指示將 **Microsoft.ServiceBus** API 新增至所需的權限。 在遵循這些步驟時，請記下 **TenantId** 和 **ApplicationId**，因為您需要有這些值才能執行應用程式。

### <a name="run-the-app"></a>執行應用程式

若要能夠執行範例，您必須先編輯 App.config 檔案，然後根據您的案例設定下列值：

- `tenantId`：設定為 **TenantId** 值。
- `clientId`：設定為 **ApplicationId** 值。 
- `clientSecret`：如果您要使用用戶端密碼來登入，請在 Azure AD 中建立該用戶端密碼。 另外，請使用 Web 應用程式或 API 而非使用原生應用程式。 還有，請將應用程式新增到您先前所建立之命名空間中的 [存取控制 (IAM)] 底下。
- `serviceBusNamespaceFQDN`：設定為新建立之服務匯流排命名空間的完整 DNS 名稱；例如 `example.servicebus.windows.net`。
- `queueName`：設定為您所建立之佇列的名稱。
- 您在前面步驟的應用程式中所指定的重新導向 URI。
 
當您執行主控台應用程式時，系統會提示您選取案例；請藉由輸入其號碼並按 ENTER 鍵來按一下 [互動使用者登入]。 應用程式隨即會顯示登入視窗，要求您同意存取服務匯流排，然後使用該服務並利用登入身分識別來執行傳送/接收案例。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排訊息，請參閱下列主題。

* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)