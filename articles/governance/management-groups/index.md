---
title: 使用 Azure 管理群組來組織資源
description: 了解管理群組及如何使用它們。
author: rthorn17
manager: rithorn
ms.assetid: 482191ac-147e-4eb6-9655-c40c13846672
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 9/28/2018
ms.author: rithorn
ms.openlocfilehash: 6b369c8209e62ff3c98b3fdf78378b403b0a0d2d
ms.sourcegitcommit: 7bc4a872c170e3416052c87287391bc7adbf84ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48017648"
---
# <a name="organize-your-resources-with-azure-management-groups"></a>使用 Azure 管理群組來組織資源

如果貴組織有多個訂用帳戶，您可能需要一個方法來有效率地管理這些訂用帳戶的存取、原則和相容性。 Azure 管理群組可以在訂用帳戶之上提供範圍層級。 您要將訂用帳戶整理到稱為「管理群組」的容器中，並將治理條件套用至管理群組。 管理群組內的所有訂用帳戶都會自動繼承套用到管理群組的條件。 無論具有何種類型的訂用帳戶，管理群組都可為您提供企業級的大規模管理功能。

例如，您可以將原則套用至管理群組，以限制可建立虛擬機器 (VM) 的區域。 此原則會套用至所有的管理群組、訂用帳戶以及該管理群組下的資源，而僅允許在該區域中建立的 VM。

## <a name="hierarchy-of-management-groups-and-subscriptions"></a>管理群組和訂用帳戶的階層

您可以建置管理群組和訂用帳戶的彈性結構，將資源組織到一個階層中，以便執行統一原則與存取管理。 下圖顯示使用管理群組建立治理階層的範例。

![樹狀](./media/MG_overview.png)

透過建立如同此範例的階層，您可以將原則 (例如將 VM 位置限制在美國西部區域) 套用至 "Infrastructure Team Management Group" (基礎結構小組管理群組) 群組，以啟用內部合規性和安全性原則。 此原則將會同時繼承至管理群組底下的兩個 "EA Subscription" (EA 訂用帳戶)，並會套用至那些訂用帳戶底下的所有 VM。 由於此原則是從管理群組繼承至訂用帳戶，此安全性原則將無法由資源或訂用帳戶擁有者改變，並能進一步提升治理能力。

另一個案例是使用管理群組來向使用者提供多個訂用帳戶的存取權。 透過將多個訂用帳戶移至該管理群組底下，您將能在管理群組上建立[角色型存取控制](../../role-based-access-control/overview.md) (RBAC) 指派，這會將該存取權繼承至所有的訂用帳戶。
您不需要透過指令碼將 RBAC 指派給多個訂用帳戶，而只需要單獨對管理群組進行指派，使用者便能存取其所需要的所有內容。

### <a name="important-facts-about-management-groups"></a>關於管理群組的重要事實

- 單一目錄中可支援 10,000 個管理群組。
- 管理群組樹狀結構最多可支援六個層級的深度。
  - 此限制不包含根層級或訂用帳戶層級。
- 每個管理群組和訂用帳戶只能支援一個父系。
- 一個管理群組可以有多個子系。
- 所有訂用帳戶和管理群組都包含在每個目錄的單一階層中。 如需預覽期間的例外狀況，請參閱[關於根管理群組的重要事實](#important-facts-about-the-root-management-group)。

## <a name="root-management-group-for-each-directory"></a>每個目錄的根管理群組

每個目錄會都會有一個最上層管理群組，名為「根」管理群組。
這個根管理群組會建置於階層內，讓所有的管理群組和訂用帳戶摺疊於其中。 這個根管理群組可讓全域原則和 RBAC 指派在目錄層級套用。 [目錄管理員必須提高本身的權限](../../role-based-access-control/elevate-access-global-admin.md)，以成為此根群組最初的擁有者。 管理員一旦成為群組的擁有者，即可將任何 RBAC 角色指派給其他目錄使用者或群組，以管理階層。

### <a name="important-facts-about-the-root-management-group"></a>關於根管理群組的重要事實

- 依預設會提供根管理群組的名稱和識別碼。 顯示名稱可隨時更新，而在 Azure 入口網站中呈現為不同名稱。
  - 名稱會是 "Tenant root group" (租用戶根群組)。
  - 識別碼則是 Azure Active Directory 識別碼。
- 根管理群組不同於其他管理群組，無法移動或刪除。  
- 所有的訂用帳戶和管理群組可摺疊到目錄內的一個根管理群組中。
  - 目錄中的所有資源可摺疊到根管理群組，以便進行全域管理。
  - 建立新的訂用帳戶時，會自動預設為根管理群組。
- 所有 Azure 客戶都可以看到根管理群組，但並非所有的客戶都有權管理該根管理群組。
  - 有訂用帳戶存取權的所有人，都可以查看訂用帳戶在階層中的上下內容。  
  - 根管理群組的預設存取權不會授予任何人。 目錄全域系統管理員是唯一可以提升自己以取得存取權的使用者。  目錄系統管理員一旦有存取權之後，就可以將任何 RBAC 角色指派給其他使用者以進行管理。  

> [!IMPORTANT]
> 根管理群組上的任何使用者存取權指派或原則指派，都會**套用至目錄中的所有資源**。
> 因此，所有客戶都應評估是否需要在此範圍內定義項目。
> 在這個範圍內，只應指派「必要」的使用者存取權和原則。  

## <a name="initial-setup-of-management-groups"></a>管理群組的初始設定

當任何使用者開始使用管理群組時，會發生初始的設定程序。 第一個步驟是在目錄中建立根管理群組。 建立此群組後，所有存在於目錄中的現有訂用帳戶都將成為根管理群組的子系。 此程序的原因是要確定目錄中只有一個管理群組階層。 目錄內的單一階層可讓系統管理客戶得以套用全域存取權和原則，讓目錄內的其他客戶無法略過。 當目錄中只有一個階層時，在根目錄上指派的任何項目將套用至目錄中所有的管理群組、訂用帳戶、資源群組和資源。

## <a name="trouble-seeing-all-subscriptions"></a>看不到所有訂用帳戶

若目錄在 (2018 年 6 月 25 日) 之前已在早期預覽版本中開始使用管理群組，則您可能會發現並非所有訂用帳戶均強制排定至階層。  這是因為系統會在目錄中的根管理群組中將角色或原則指派完成之後，才會實作強制將訂用帳戶排定至階層的流程。

### <a name="how-to-resolve-the-issue"></a>如何解決此問題

有兩種自助式選項可以解決此問題。

1. 從根管理群組中移除所有角色和原則指派
    1. 從根管理群組中移除所有原則和角色指派之後，服務會將所有訂用帳戶回填至下一個隔日工作週期階層。  進行檢查的原因，是為了確保並未意外地將存取權或原則指派提供給所有租用戶訂用帳戶。
    1. 要在不影響您服務的情況下進行此流程，最好的方法是將角色或原則指派套用至比根管理群組低一階的層級。 如此一來，您就可以從根範圍中移除所有指派。
1. 直接呼叫 API 以啟動回填流程
    1. 目錄中的所有已授權客戶都可以呼叫 *TenantBackfillStatusRequest* 或 *StartTenantBackfillRequest* API。 呼叫了 StartTenantBackfillRequest API 之後，系統就會開始初步的設定流程，將所有訂用帳戶移動至該階層。 此流程也會強制將所有新的訂用帳戶均設為根管理群組的子系。 若您允許根上的所有原則或存取指派都可以套用至所有訂用帳戶，則此流程就能在不變更根層級上任何指派的情況下完成。

若您對於此回填流程有任何問題，請連絡：managementgroups@microsoft.com  
  
## <a name="management-group-access"></a>管理群組存取

Azure 管理群組支援對所有的資源存取和角色定義使用 [Azure 角色型存取控制 (RBAC)](../../role-based-access-control/overview.md)。
這些權限會被存在於階層中的子資源繼承。 可以將任何內建的 RBAC 角色指派至繼承階層直到資源的管理群組。
例如，可以將 RBAC 角色 VM 參與者指派至管理群組。 此角色對管理群組沒有任何作用，但會繼承該管理群組下的所有 VM。

下圖顯示了角色清單與管理群組上支援的動作。

| RBAC 角色名稱             | 建立 | 重新命名 | 移動 | 刪除 | 指派存取權 | 指派原則 | 讀取  |
|:-------------------------- |:------:|:------:|:----:|:------:|:-------------:| :------------:|:-----:|
|擁有者                       | X      | X      | X    | X      | X             | X             | X     |
|參與者                 | X      | X      | X    | X      |               |               | X     |
|MG 參與者*             | X      | X      | X    | X      |               |               | X     |
|讀取者                      |        |        |      |        |               |               | X     |
|MG 讀取者*                  |        |        |      |        |               |               | X     |
|資源原則參與者 |        |        |      |        |               | X             |       |
|使用者存取系統管理員   |        |        |      |        | X             |               |       |

*：MG 參與者和 MG 讀取者僅允許使用者執行管理群組範圍的動作。  

### <a name="custom-rbac-role-definition-and-assignment"></a>自訂的 RBAC 角色定義和指派

管理群組目前不支援自訂的 RBAC 角色。 若要檢視這個項目的狀態，請參閱[管理群組意見反應論壇](https://aka.ms/mgfeedback)。

## <a name="next-steps"></a>後續步驟

若要深入了解管理群組，請參閱：

- [建立管理群組以組織 Azure 資源](create.md)
- [如何變更、刪除或管理您的管理群組](manage.md)
- [安裝 Azure PowerShell 模組](https://www.powershellgallery.com/packages/AzureRM.ManagementGroups)
- [檢閱 REST API 規格](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/managementgroups/resource-manager/Microsoft.Management/preview)
- [安裝 Azure CLI 擴充功能](/cli/azure/extension?view=azure-cli-latest#az-extension-list-available)