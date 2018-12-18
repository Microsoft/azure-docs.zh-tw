---
title: Azure 原則的概觀
description: Azure 原則是 Azure 中的一個服務，您可以在 Azure 環境中用來建立、指派和管理原則定義。
services: azure-policy
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: overview
ms.service: azure-policy
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: dbdffc7a6f77f3f34ce7937c60eb7a53e5f72590
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961275"
---
# <a name="what-is-azure-policy"></a>什麼是 Azure 原則？

進行 IT 控管，可確保貴組織能夠以符合效率及效益的方式來達成目標。 做法是在您的商務目標與 IT 專案之間建立明確見地。

您公司是否遇到似乎永遠無法解決的大量 IT 問題？
良好的 IT 控管包括規劃您的方案，以及在策略層級上設定優先順序，以協助管理及避免問題。 這是 Azure 原則存在的理由。

Azure 原則是 Azure 中的一個服務，您可以用來建立、指派和管理原則。 這些原則會對您的資源強制執行不同的規則和效果，讓這些資源能符合公司標準和服務等級協定的規範。 Azure 原則的運作方式是對資源執行評估，並掃描不符合所建立原則的規範。 例如，您可以設定原則，在環境中只允許特定 SKU 大小的虛擬機器。 一旦已實作此原則後，便會在建立和更新資源時再加以評估，並對您現有的資源進行評估。 稍後在本文件中，我們將介紹更多關於如何使用 Azure 原則來建立及實作原則的詳細資料。

> [!IMPORTANT]
> 無論定價層為何，所有指派現在都會提供 Azure 原則的合規性評估。 如果指派未顯示合規性資料，請確定已向 Microsoft.PolicyInsights 資源提供者註冊訂用帳戶。

## <a name="how-is-it-different-from-rbac"></a>它和 RBAC 有什麼不同？

原則和角色型存取控制 (RBAC) 之間有幾個主要差異。 RBAC 著重於不同範圍的使用者動作。 例如，您可能被加入所需範圍之資源群組的參與者角色中。 該角色可讓您對該資源群組進行變更。
原則著重於部署期間及既有資源的資源屬性。 例如，您能夠透過原則控制可以佈建的資源類型。 或者，您可以限制資源可以佈建的位置。 不同於 RBAC，原則是個預設允許和明確拒絕的系統。

### <a name="rbac-permissions-in-azure-policy"></a>Azure 原則中的 RBAC 權限

Azure 原則用權限代表兩個不同資源提供者中的作業：

- [Microsoft.Authorization](../../role-based-access-control/resource-provider-operations.md#microsoftauthorization)
- [Microsoft.PolicyInsight](../../role-based-access-control/resource-provider-operations.md#microsoftpolicyinsights)

有數個內建角色對 Azure 原則資源具有各種層級的權限，例如 [安全性系統管理員]，可以管理原則指派和定義，但無法檢視合規性資訊；以及 [讀取者]，可以讀取有關原則指派和定義的詳細資料，但無法進行變更或檢視合規性資訊。 而 [擁有者] 具有完整權限，但 [參與者] 沒有任何 Azure 原則權限。 若要授與可檢視原則合規性詳細資訊的權限，請建立[自訂角色](../../role-based-access-control/custom-roles.md)。

## <a name="policy-definition"></a>原則定義

建立及實作 Azure 原則中的原則始於建立原則定義。 每個原則定義都有其強制執行條件。 此外，還有符合條件時會進行的伴隨效果。

在 Azure 原則中，我們預設提供一些可供您使用的內建原則。 例如︰

- **需要 SQL Server 12.0**：此原則定義有條件/規則，以確保所有 SQL Server 都使用 12.0 版。 其效果是拒絕不符合這些準則的所有伺服器。
- **允許的儲存體帳戶 SKU**：此原則定義有一組條件/規則，判斷正在部署的儲存體帳戶是否在一組 SKU 大小內。 其效果是拒絕不符合已定義 SKU 大小集合的所有儲存體帳戶。
- **允許的資源類型**：此原則定義有一組條件/規則，指定您的組織可以部署的資源類型。 其效果是拒絕不屬於此定義清單的所有資源。
- **允許的位置**：此原則可讓您限制您組織在部署資源時可指定的位置。 其效果可用來強制執行您的地理合規性需求。
- **允許的虛擬機器 SKU**：此原則可讓您指定您組織可部署的一組虛擬機器 SKU。
- **套用標籤及其預設值**：此原則會套用必要的標籤及其預設值 (如果使用者未指定的話)。
- **強制執行標籤及其值**：此原則會對資源強制執行必要的標籤及其值。
- **不允許的資源類型**：此原則可讓您指定您組織不可以部署的資源類型。

若要實作這些原則定義 (內建和自訂定義)，您必須指派它們。 您可以透過 Azure 入口網站、PowerShell 或 Azure CLI 來指派任何這些原則。

請記住，大約一小時會進行一次原則重新評估；這表示，如果您在實作原則 (建立原則指派) 之後對原則定義進行變更，系統會在一小時內對您的資源進行重新評估原則。

若要深入了解原則定義的結構，請檢閱[原則定義結構](./concepts/definition-structure.md)。

## <a name="policy-assignment"></a>原則指派

原則指派是已指派在特定範圍內發生的原則定義。 此範圍可從[管理群組](../management-groups/overview.md)到資源群組。 「範圍」一詞係指作為原則定義指派對象的所有資源群組、訂用帳戶或管理群組。 原則指派會由所有子資源繼承。 這表示，如果將原則套用至某個資源群組，就會套用至該資源群組中的所有資源。 不過，您可以從原則指派中排除某個子範圍。

例如，在訂用帳戶範圍，您可以指派一個防止建立網路資源的原則。 不過，您將訂用帳戶內一個要用於網路基礎結構的資源群組排除。 您需將此網路資源群組的存取權授與您所信任來建立網路資源的使用者。

在另一個範例中，建議您在管理群組層級指派資源類型白名單原則。 然後，在子管理群組或甚至直接在訂用帳戶上指派更寬鬆的原則 (允許更多資源類型)。 不過，因為原則是明確的拒絕系統，此範例沒有作用。 相反地，您需要從管理群組層級的原則指派中排除子管理群組或訂用帳戶。 然後，對子管理群組或訂用帳戶層級指派更寬鬆的原則。 總結來說，如果有任何原則導致系統拒絕資源，則唯一能允許資源的方法是修改拒絕原則。

如需透過入口網站設定原則定義和指派的詳細資訊，請參閱[建立原則指派，以識別 Azure 環境中不符合規範的資源](assign-policy-portal.md)。 此外，還會提供適用於 [PowerShell](assign-policy-powershell.md) 和 [Azure CLI](assign-policy-azurecli.md) 的步驟。

## <a name="policy-parameters"></a>原則參數

原則參數減少您必須建立的原則定義數量，有助於簡化原則管理。 在建立更通用的原則定義時，您可以定義參數。 接著，您便可以針對不同的案例，重複使用該原則定義。 若要這麼做，只要在指派原則定義時傳入不同的值即可。 例如，為訂用帳戶指定一組位置。

參數是在建立原則定義時定義/建立。 定義參數時，需要指定名稱並選擇性地指定值。 例如，您可以為原則定義一個標題為 *location* 的參數。 接著，您可以在指派原則時給予它不同的值，例如 *EastUS* 或 *WestUS*。

如需原則參數的詳細資訊，請參閱[資源原則概觀 - 參數](./concepts/definition-structure.md#parameters)。

## <a name="initiative-definition"></a>計畫定義

計畫定義是針對達到單一主要目標而量身打造的原則定義集合。 計畫定義可讓管理及指派原則定義更為簡單。 其簡化方式是將一組原則組成單一項目。 例如，您可以建立一個標題為**在 Azure 資訊安全中心啟用監視**的計畫，目標是要在您的 Azure 資訊安全中心監視所有可用的安全性建議。

在這項計畫之下，您可能會有如下的原則定義：

- **在資訊安全中心監視未加密的 SQL Database** – 適用於監視未加密的 SQL 資料庫和伺服器。
- **在資訊安全中心監視作業系統弱點**– 適用於監視不符合設定基準的伺服器。
- **在資訊安全中心監視遺失的 Endpoint Protection** – 適用於監視沒有安裝 Endpoint Protection 代理程式的伺服器。

## <a name="initiative-assignment"></a>計畫指派

類似原則指派，計畫指派是指派至特定範圍的計畫定義。 計畫指派會減少為每個範圍進行多個計畫定義的需求。 此範圍也可從管理群組到資源群組。

從上述範例中，**在 Azure 資訊安全中心啟用監視**計畫可以指派至不同的範圍。 例如，一個指派項目可以指派給**subscriptionA**。
另一個可以指派給 **subscriptionB**。

## <a name="initiative-parameters"></a>計畫參數

類似原則參數，計畫參數減少重複性，有助於簡化計畫管理。 計畫參數基本上是計畫內原則定義所要使用的參數清單。

例如，以具有計畫定義的案例 - **initiativeC** 來說，其中包含原則定義 **policyA** 和 **policyB**，每個都必須有不同類型的參數：

| 原則 | 參數名稱 |參數類型  |附註 |
|---|---|---|---|
| policyA | allowedLocations | array  |因為參數類型已定義為陣列，所以此參數應該要有一個字串清單作為值 |
| policyB | allowedSingleLocation |字串 |因為參數類型已定義為字串，所以此參數應該要有一個字組作為值 |

在此案例中，定義 **initiativeC** 的計畫參數時，有三個選項：

- 使用此計畫內原則定義的參數：在此範例中，*allowedLocations* 和 *allowedSingleLocation* 會變成 **initiativeC** 的計畫參數。
- 提供值給此計畫定義內原則定義的參數。 在此範例中，您可以提供位置清單給 **policyA 的參數 – allowedLocations** 和 **policyB 的參數 – allowedSingleLocation**。 您也可以在指派此計畫時提供值。
- 提供指派此計畫時可以使用的*值*選項清單。 當您指派此計畫時，從計畫內原則定義繼承的參數，只能有來自此提供清單中的值。

例如，您可能會在計畫定義中建立一個包含 *EastUS*、*WestUS*、*CentralUS* 及 *WestEurope* 的值選項清單。 如果是這樣，在進行計畫指派時，您就無法輸入其他的值，例如 *Southeast Asia*，因為此值並不包含在該清單中。

## <a name="maximum-count-of-policy-objects"></a>原則物件的最大計數

[!INCLUDE [policy-limits](../../../includes/azure-policy-limits.md)]

## <a name="recommendations-for-managing-policies"></a>管理原則的建議

建立和管理原則定義和指派時，我們建議您遵循以下幾個指標，並將幾個秘訣牢記在心：

- 如果您要在環境中建立原則定義，建議您從稽核效果而不是從拒絕效果開始著手，以記錄原則定義對環境中資源的影響。 如果您已經有可自動調整應用程式的指令碼，設定拒絕效果可能會妨礙您既有的這些自動化工作。
- 在建立定義和指派時，請務必記住組織階層。 我們建議在較高的層級 (例如在管理群組或訂用帳戶層級) 建立定義，然後在下一個子層級指派。 例如，如果您在管理群組層級建立原則定義，該定義的原則指派範圍便可向下延伸到該管理群組內的訂用帳戶層級。
- 我們建議一律使用計畫定義而不是原則定義，即使您只要一個原則亦同。 例如，如果您有原則定義 - *policyDefA* ，而且將它建立在計畫定義 - *initiativeDefC* 底下，當您稍後決定為 *policyDefB* 建立另一個目標與 *policyDefA* 目標類似的原則定義時，您就可以在 *initiativeDefC* 底下新增它，然後以此方式更有效地追蹤這些定義。
- 請記住，在您從計畫定義建立了計畫指派之後，任何新增到計畫定義中的新原則定義，都會自動彙總在該計畫定義下的計畫指派底下。
- 一旦觸發計畫指派時，計畫中的所有原則也都將會觸發。 不過，如果您必須個別執行原則，最好不要包含在計畫中。

## <a name="video-overview"></a>影片概觀

以下 Azure 原則的概觀是從組建 2018 開始。 如需投影片或影片的下載，請瀏覽 Channel 9 上的[透過 Azure 原則控管您的 Azure 環境](https://channel9.msdn.com/events/Build/2018/THR2030)。

> [!VIDEO https://www.youtube.com/embed/dxMaYF2GB7o]

## <a name="next-steps"></a>後續步驟

現在您已大概了解 Azure 原則，以及一些重要概念，以下是建議的後續步驟：

- [使用入口網站指派原則定義](assign-policy-portal.md)
- [使用 Azure CLI 指派原則定義](assign-policy-azurecli.md)
- [使用 PowerShell 指派原則定義](assign-policy-powershell.md)
- 檢閱[使用 Azure 管理群組來組織資源](..//management-groups/overview.md)，以了解何謂管理群組
- 檢視 Channel 9 上的[透過 Azure 原則控管您的 Azure 環境](https://channel9.msdn.com/events/Build/2018/THR2030)