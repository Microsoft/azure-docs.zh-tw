---
title: Azure 藍圖概觀
description: 「Azure 藍圖」是 Azure 中的服務，可讓您用來建立、定義及在您的 Azure 環境中部署成品。
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: overview
ms.service: blueprints
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: c7ffbe86407bc776870890e5b4151556f572832e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957500"
---
# <a name="what-is-azure-blueprints"></a>什麼是 Azure 藍圖？

正如「藍圖」可讓工程師或架構設計人員為專案勾勒出設計參數一樣，Azure 藍圖可讓雲端架構人員和中央 IT 人員定義一組可重複使用的 Azure 資源，其中實作並遵循組織的標準、模式和需求。 Azure 藍圖可讓開發小組在知道他們是以符合組織合規性進行建置的情況下，快速地佈建及建立新環境，並包含一組內建元件 (例如網路) 以加速開發和交貨。

藍圖是以宣告方式來協調部署多個資源範本與其他成品，例如：

- 角色指派
- 原則指派
- Azure 資源管理員範本
- 資源群組

## <a name="how-it-is-different-from-resource-manager-templates"></a>它與 Resource Manager 範本有何不同

藍圖的設計目的是協助進行「環境設定」，通常包含一組資源群組、原則和角色指派，而不只是 Resource Manager 範本部署。 藍圖是一種套件，可讓這些「成品」類型集合在一起，並可讓您撰寫及控制該套件的版本，包括透過 CI/CD 管線。 最終，每個成品都可在單一可稽核且可追蹤的作業中指派給訂用帳戶。

你想要在藍圖中為部署所納入的一切，幾乎都可使用 Resource Manager 範本來完成。 不過，Resource Manager 範本並非原生存在於 Azure 中的文件，而是分別儲存在本機或原始檔控制中。 範本可用來部署一或多個 Azure 資源，但是一旦部署那些資源之後，所使用之範本的聯繫和關聯性就會遺失。

有了藍圖之後，藍圖定義 (「應該被」部署的) 和藍圖指派 (「已被」部署的) 之間的關聯性就得以保留。 此聯繫可改良對部署的追蹤和稽核，讓同一藍圖所控管的多個訂用帳戶一次升級等等。

您不需要在 Resource Manager 範本和藍圖之間做出取捨。 每個藍圖可以包含多個 Resource Manager 範本「成品」，或是不包含。 這表示先前開發及維護 Resource Manager 範本程式庫所投入的心力，也可用在藍圖中。

## <a name="how-it-is-different-from-azure-policy"></a>它與 Azure 原則有何不同

藍圖是一種套件或容器，用來組成專注特定方面的標準、模式、以及與 Azure 雲端服務實作相關聯的要求組合，並且設計為可重複使用以確保一致性和合規性。

[原則](../policy/overview.md)是預設允許和明確拒絕的系統，在部署期間專注在資源屬性，並適用於既有資源。 它可透過確保訂用帳戶內的資源遵循需求和標準，來支援 IT 治理。

在藍圖中納入原則，不僅可在藍圖指派期間建立正確的模式或設計，也能確保環境只會有已核准或預期的變更，藉此確保持續符合藍圖意圖的規範。

原則可以作為許多「成品」之一，包含在藍圖定義中。 藍圖也支援使用參數搭配原則和方案。

## <a name="blueprint-definition"></a>藍圖定義

藍圖是由「成品」所組成。 藍圖目前支援下列資源作為成品：

|資源  | 階層選項| 說明  |
|---------|---------|---------|
|資源群組     | 訂用帳戶 | 建立新的資源群組以供藍圖內的其他成品使用。  這些預留位置資源群組可讓您確切地以自己想要的方式組織資源，並針對包含的原則和角色指派成品以及 Azure Resource Manager 範本提供範圍限制器。         |
|Azure Resource Manager 範本      | 資源群組 | 這些範本可用來撰寫複雜的環境，例如 SharePoint 伺服器陣列、Azure Automation State Configuration，或是 Log Analytics 工作區。 |
|原則指派     | 訂用帳戶、資源群組 | 可讓原則或方案指派到藍圖所指派到的管理群組或訂用帳戶。 原則或方案必須位在藍圖的範圍內 (在藍圖管理群組中或其下)。 如果原則或方案具有參數，您可以在藍圖建立或是藍圖指派期間指派這些參數。       |
|角色指派   | 訂用帳戶、資源群組 | 新增現有使用者或群組到內建角色，以確保適當人員一律擁有資源的正確存取權。 角色指派可針對整個訂用帳戶進行定義，或是以巢狀方式針對包含在藍圖內的特定資源群組定義。 |

### <a name="blueprints-and-management-groups"></a>藍圖和管理群組

建立藍圖定義時，您將定義藍圖的儲存位置。 目前，藍圖只能儲存到有**參與者**權限即可存取的[管理群組](../management-groups/overview.md)。 接著，藍圖便可指派給該管理群組的任一子系 (管理群組或訂用帳戶)。

> [!IMPORTANT]
> 如果您不具有任何管理群組或任何已設定之管理群組的存取權，則載入藍圖定義清單時不會顯示任何可用的藍圖定義，且按一下 [範圍] 將會開啟有關擷取管理群組警告的視窗。 若要解決此問題，請確定您的訂用帳戶有適當的存取權限，且屬於[管理群組](../management-groups/overview.md)。

### <a name="blueprint-parameters"></a>藍圖參數

藍圖可以傳遞參數給原則/方案，或是 Azure Resource Manager 範本。
當任一「成品」新增到藍圖時，建立者可決定要為每個藍圖指派已定義的值，或是允許每個藍圖指派在指派時間提供值。 此彈性可提供選擇是要為藍圖的所有使用者定義預先決定的值，或是在指派時再做出決定。

> [!NOTE]
> 藍圖可以有自己專屬的參數，但目前只有在藍圖是從 REST API 而非透過入口網站建立時才能建立這些參數。

如需詳細資訊，請參閱[藍圖參數](./concepts/parameters.md)。

### <a name="blueprint-publishing"></a>藍圖發佈

藍圖最初建立時，它被視為處於**草稿**模式。 當它準備好指派時，必須是**已發佈**。 發佈需要定義**版本**字串 (字母、數字和連字號，長度上限為 20 個字元)，以及選擇性的**變更附註**。
**版本**可區別相同藍圖日後的變更，並允許個別指派每個版本。 這也表示相同藍圖的不同**版本**可指派給相同的訂用帳戶。 當對藍圖進行其他變更時，**已發佈** **版本**仍會存在 (除了**未發佈的變更**)。 一旦變更完成，更新過的藍圖會是**已發佈**、具有新的唯一**版本**，並且可立即指派。

## <a name="blueprint-assignment"></a>藍圖指派

藍圖的每個**已發佈** **版本**都可指派給現有的訂用帳戶。 在入口網站中，藍圖的預設**版本**會設定為最近**已發佈**的版本。 如果有成品參數 (或是藍圖參數)，則參數將會在指派過程中定義。

## <a name="permissions-in-azure-blueprints"></a>Azure 藍圖中的權限

若要使用藍圖，您必須透過[角色型存取控制](../../role-based-access-control/overview.md) (RBAC) 獲得授權。 若要建立藍圖，您的帳戶需要下列權限：

- `Microsoft.Blueprint/blueprints/write` - 建立藍圖定義
- `Microsoft.Blueprint/blueprints/artifacts/write` - 在藍圖定義上建立成品
- `Microsoft.Blueprint/blueprints/versions/write` - 發佈藍圖

若要刪除藍圖，您的帳戶需要下列權限：

- `Microsoft.Blueprint/blueprints/delete`
- `Microsoft.Blueprint/blueprints/artifacts/delete`
- `Microsoft.Blueprint/blueprints/versions/delete`

> [!NOTE]
> 因為藍圖定義是以管理群組建立，所以藍圖定義權限必須在管理群組範圍上授與，或是在管理群組範圍上繼承。

若要指派或取消指派藍圖，您的帳戶需要下列權限：

- `Microsoft.Blueprint/blueprintAssignments/write` - 指派藍圖
- `Microsoft.Blueprint/blueprintAssignments/delete` - 取消指派藍圖

> [!NOTE]
> 因為藍圖指派是在訂用帳戶上建立，所以藍圖指派和取消指派權限必須在訂用帳戶範圍上授與，或是在訂用帳戶範圍上繼承。

這些權限包含在**擁有者**角色中，且除了藍圖指派的權限之外，也包含在**參與者**角色中。 如果這些內建角色不符合您的安全性需求，請考慮建立[自訂角色](../../role-based-access-control/custom-roles.md)。

> [!NOTE]
> Azure 藍圖的服務主體在指派的訂用帳戶上需要**擁有者**角色才能進行部署。 如果使用入口網站，會針對部署自動授與此角色並撤銷。 如果使用 REST API，必須手動授與此角色，但在部署完成之後仍會自動撤銷。

## <a name="next-steps"></a>後續步驟

- [建立藍圖 - 入口網站](create-blueprint-portal.md)
- [建立藍圖 - REST API](create-blueprint-rest-api.md)