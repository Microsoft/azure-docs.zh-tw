---
title: 搭配服務匯流排之 Azure 資源的受控識別
description: 本文說明如何使用受控識別來存取 Azure 服務匯流排實體 (佇列、主題和訂用帳戶) 。
ms.topic: article
ms.date: 01/21/2021
ms.openlocfilehash: dac12820adf863a7e51457d34f9180543089fc7b
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98881562"
---
# <a name="authenticate-a-managed-identity-with-azure-active-directory-to-access-azure-service-bus-resources"></a>使用 Azure Active Directory 來驗證受控識別，以存取 Azure 服務匯流排資源
[Azure 資源的受控識別](../active-directory/managed-identities-azure-resources/overview.md)是一個跨 Azure 功能，可讓您建立與應用程式程式碼執行所在之部署相關聯的安全識別。 然後您可以將該識別與存取控制角色產生關連，該角色會授與用來存取應用程式所需之特定 Azure 資源的自訂權限。

使用受控識別，Azure 平台就能管理此執行階段識別。 您不需要為了識別本身或您需要存取的資源，在應用程式程式碼或設定中儲存及保護存取金鑰。 若服務匯流排用戶端應用程式在 Azure App Service 應用程式中執行，或在已啟用 Azure 資源的受控識別支援的虛擬機器中執行，則不需處理 SAS 規則和金鑰，或任何其他存取權杖。 用戶端應用程式只需要服務匯流排傳訊命名空間的端點位址。 當應用程式連線時，服務匯流排會將受控實體的內容繫結至作業 (在此文章稍後的範例顯示) 中的用戶端。 一旦它與受控識別相關聯，您的服務匯流排用戶端就能執行所有授權的作業。 授權是藉由將受控實體與服務匯流排角色相關聯來授與。 

## <a name="overview"></a>概觀
當安全性主體 (使用者、群組或應用程式) 嘗試存取服務匯流排實體時，要求必須獲得授權。 使用 Azure AD，對資源的存取是兩個步驟的程序。 

 1. 首先，會驗證安全性主體的身分識別，並傳回 OAuth 2.0 權杖。 要求權杖的資源名稱是 `https://servicebus.azure.net` 。
 1. 接下來，權杖會作為要求的一部分傳遞至服務匯流排服務，以授權存取指定的資源。

驗證步驟要求應用程式要求在執行階段包含 OAuth 2.0 存取權杖。 如果應用程式是在 azure 實體中執行，例如 Azure VM、虛擬機器擴展集或 Azure 函式應用程式，則可以使用受控識別來存取資源。 

授權步驟需要將一或多個 Azure 角色指派給安全性主體。 Azure 服務匯流排提供的 Azure 角色包含服務匯流排資源的一組許可權。 指派給安全性主體的角色會決定主體將擁有的許可權。 若要深入瞭解如何將 Azure 角色指派給 Azure 服務匯流排，請參閱 [適用于 Azure 服務匯流排的 azure 內建角色](#azure-built-in-roles-for-azure-service-bus)。 

對服務匯流排提出要求的原生應用程式和 web 應用程式，也可以使用 Azure AD 來授權。 本文說明如何要求存取權杖，並使用它來授權服務匯流排資源的要求。 


## <a name="assigning-azure-roles-for-access-rights"></a>指派存取權限的 Azure 角色
Azure Active Directory (Azure AD) 透過 [AZURE RBAC (的 azure 角色型存取控制 ](../role-based-access-control/overview.md)，來授與受保護資源的存取權限。 Azure 服務匯流排定義一組 Azure 內建角色，其中包含用來存取服務匯流排實體的一般許可權集，您也可以定義自訂角色來存取資料。

當 Azure 角色指派給 Azure AD 安全性主體時，Azure 會為該安全性主體授與這些資源的存取權。 存取權的範圍可以是訂用帳戶、資源群組或服務匯流排命名空間的等級。 Azure AD 的安全性主體可能是使用者、群組、應用程式服務主體，或適用于 Azure 資源的受控識別。

## <a name="azure-built-in-roles-for-azure-service-bus"></a>適用于 Azure 服務匯流排的 Azure 內建角色
針對 Azure 服務匯流排，透過 Azure 入口網站和 Azure 資源管理 API 來管理命名空間和所有相關資源的功能，已使用 Azure RBAC 模型來保護。 Azure 提供下列 Azure 內建角色，以授權存取服務匯流排命名空間：

- [Azure 服務匯流排資料擁有](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner)者：允許對服務匯流排命名空間及其實體的資料存取 (佇列、主題、訂用帳戶和篩選) 
- [Azure 服務匯流排資料](../role-based-access-control/built-in-roles.md#azure-service-bus-data-sender)傳送者：使用此角色來授與服務匯流排命名空間及其實體的傳送存取權。
- [Azure 服務匯流排資料接收者](../role-based-access-control/built-in-roles.md#azure-service-bus-data-receiver)：使用此角色來授與接收服務匯流排命名空間及其實體的存取權。 

## <a name="resource-scope"></a>資源範圍 
將 Azure 角色指派給安全性主體之前，請先決定安全性主體應該具備的存取範圍。 最佳做法的要求是，最好只授與最小的可能範圍。

下列清單描述從最小範圍開始，您可以範圍存取服務匯流排資源的層級：

- **佇列**、 **主題** 或 **訂** 用帳戶：角色指派會套用至特定的服務匯流排實體。 目前，Azure 入口網站不支援將使用者/群組/受控識別指派給訂用帳戶層級的服務匯流排 Azure 角色。 以下是使用 Azure CLI 命令的範例： [az-角色-指派-建立](/cli/azure/role/assignment?#az-role-assignment-create) 以將身分識別指派給服務匯流排 Azure 角色： 

    ```azurecli
    az role assignment create \
        --role $service_bus_role \
        --assignee $assignee_id \
        --scope /subscriptions/$subscription_id/resourceGroups/$resource_group/providers/Microsoft.ServiceBus/namespaces/$service_bus_namespace/topics/$service_bus_topic/subscriptions/$service_bus_subscription
    ```
- **服務匯流排命名空間**：角色指派橫跨命名空間下的整個服務匯流排拓撲，以及與其相關聯的取用者群組。
- **資源群組**：角色指派會套用至資源群組下的所有服務匯流排資源。
- **訂** 用帳戶：角色指派會套用至訂用帳戶中所有資源群組中的所有服務匯流排資源。

> [!NOTE]
> 請記住，Azure 角色指派最多可能需要五分鐘的時間來傳播。 

如需如何定義內建角色的詳細資訊，請參閱 [瞭解角色定義](../role-based-access-control/role-definitions.md#management-and-data-operations)。 如需建立 Azure 自訂角色的詳細資訊，請參閱 [azure 自訂角色](../role-based-access-control/custom-roles.md)。

## <a name="enable-managed-identities-on-a-vm"></a>在 VM 上啟用受控識別
您必須先在 VM 上啟用 Azure 資源的受控識別，才能使用 Azure 資源的受控識別來授權 VM 的服務匯流排資源。 若要了解如何啟用 Azure 資源的受控識別，請參閱下列其中一篇文章：

- [Azure 入口網站](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Azure Resource Manager 範本](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Azure Resource Manager 用戶端程式庫](../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-a-managed-identity-in-azure-ad"></a>在 Azure AD 中將許可權授與受控識別
若要從您應用程式中的受控識別授權服務匯流排服務的要求，請先為該受控識別設定 Azure 角色型存取控制 (Azure RBAC) 設定。 Azure 服務匯流排定義的 Azure 角色包含從服務匯流排傳送和讀取的許可權。 當 Azure 角色指派給受控識別時，受控識別會獲得適當範圍內服務匯流排實體的存取權。

如需指派 Azure 角色的詳細資訊，請參閱 [使用 Azure Active Directory 進行驗證和授權，以存取服務匯流排資源](authenticate-application.md#azure-built-in-roles-for-azure-service-bus)。

## <a name="use-service-bus-with-managed-identities-for-azure-resources"></a>搭配使用服務匯流排和 Azure 資源的受控識別
若要搭配使用服務匯流排與受控識別，您必須將角色和適當的範圍指派給身分識別。 本節中的程式會使用在受控識別下執行的簡單應用程式，並存取服務匯流排資源。

在這裡，我們會使用 [Azure App Service](https://azure.microsoft.com/services/app-service/)中裝載的範例 web 應用程式。 如需建立 web 應用程式的逐步指示，請參閱[在 Azure 中建立 ASP.NET Core web 應用](../app-service/quickstart-dotnetcore.md)程式

建立應用程式之後，請遵循下列步驟： 

1. 移至 [ **設定** ]，然後選取 [身分 **識別**]。 
1. 選取要 **開啟** 的 **狀態**。 
1. 選取 [儲存]  以儲存設定。 

    ![Web 應用程式的受控識別](./media/service-bus-managed-service-identity/identity-web-app.png)

一旦您啟用此設定，就會在 Azure Active Directory (Azure AD) 中建立新的服務識別，並將其設定為 App Service 主機。

> [!NOTE]
> 當您使用受控識別時，連接字串的格式應為： `Endpoint=sb://<NAMESPACE NAME>.servicebus.windows.net/;Authentication=Managed Identity` 。

現在，將此服務識別指派給您服務匯流排資源中所需範圍內的角色。

### <a name="to-assign-azure-roles-using-the-azure-portal"></a>使用 Azure 入口網站指派 Azure 角色
若要將角色指派給服務匯流排命名空間，請流覽至 Azure 入口網站中的命名空間。 顯示資源的存取控制 (IAM) 設定，並遵循下列指示來管理角色指派：

> [!NOTE]
> 下列步驟會將服務身分識別角色指派給您的服務匯流排命名空間。 您可以遵循相同的步驟，在其他支援的範圍 (資源群組和訂用帳戶) 指派角色。 
> 
> [建立服務匯流排訊息命名空間](service-bus-create-namespace-portal.md) （如果您沒有的話）。 

1. 在 Azure 入口網站中，流覽至您的服務匯流排命名空間，並顯示命名空間的 **總覽** 。 
1. 選取左側功能表上的 [ **存取控制] (IAM)** ，以顯示服務匯流排命名空間的存取控制設定。
1.  選取 [角色指派] 索引標籤，以查看角色指派的清單。
3.  選取 [ **新增**]，然後選取 [ **新增角色指派**]。
4.  在 [ **新增角色指派** ] 頁面上，依照下列步驟執行：
    1. 針對 [ **角色**]，選取您要指派的服務匯流排角色。 在此範例中，它是 **Azure 服務匯流排資料擁有** 者。
    1. 在 [**將存取權指派給**] 欄位中，選取 [**系統指派的受控識別**] 下的 **App Service** 。 
    1. 選取 web 應用程式的受控識別建立所在的 **訂** 用帳戶。
    1. 選取您所建立 web 應用程式的 **受控識別** 。 身分識別的預設名稱與 web 應用程式的名稱相同。 
    1. 然後選取 [儲存]。
        
        ![[新增角色指派] 頁面](./media/service-bus-managed-service-identity/add-role-assignment-page.png)

    一旦您指派角色之後，web 應用程式就可以存取定義範圍下的服務匯流排實體。 

    > [!NOTE]
    > 如需支援受控識別的服務清單，請參閱 [支援 Azure 資源受控](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)識別的服務。

### <a name="run-the-app"></a>執行應用程式
現在修改您建立之 ASP.NET 應用程式的預設分頁。 您可以使用來自[這個 GitHub 存放庫](https://github.com/Azure-Samples/app-service-msi-servicebus-dotnet)的 Web 應用程式程式碼。  

Default.aspx 頁面是您的登陸頁面。 您可以在 Default.aspx.cs 檔案中找到程式碼。 結果是最小的 Web 應用程式，具有數個項目欄位，以及具有連線到服務匯流排的 [傳送] 和 [接收] 按鈕，以傳送或接收訊息。

請注意 [MessagingFactory](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 物件如何初始化。 程式碼不會使用共用存取權杖 (SAS) 權杖提供者，而會改用 `var msiTokenProvider = TokenProvider.CreateManagedIdentityTokenProvider();` 呼叫來建立受控識別的權杖提供者。 因此，不會保留及使用密碼。 受控識別內容到服務匯流排和授權交握的流程，都是由權杖提供者自動處理。 它是比使用 SAS 更簡單的模型。

當您進行這些變更之後，請發行並執行應用程式。 您只要下載發行設定檔然後在 Visual Studio 中匯入，就能取得正確的發行資料：

![取得發行設定檔](./media/service-bus-managed-service-identity/msi3.png)
 
若要傳送或接收訊息，請輸入命名空間名稱和您所建立之實體的名稱。 然後，按一下 [傳送] 或 [接收]。


> [!NOTE]
> - 受控識別只能在 Azure 環境中、在應用程式服務、Azure VM 和擴展集上運作。 對於 .NET 應用程式，Microsoft.Azure.Services.AppAuthentication 程式庫 (由服務匯流排 NuGet 套件使用) 提供讓應用程式透過此通訊協定進行提取的功能，也能支援本機部署經驗。 該程式庫還能讓您使用來自 Visual Studio、Azure CLI 2.0 或 Active Directory 整合式驗證的使用者帳戶，在部署機器上以本機方式測試程式碼。 如需使用此程式庫的本機開發選項詳細資訊，請參閱[使用 .NET 對 Azure Key Vault 進行服務對服務驗證](/dotnet/api/overview/azure/service-to-service-authentication)。  
> 
> - 受控識別目前不會使用 App Service 部署位置。

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)