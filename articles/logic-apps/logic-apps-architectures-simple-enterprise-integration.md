---
title: 簡單的企業整合架構模式 - Azure Integration Services
description: 此架構參考示範如何使用 Azure Logic Apps 和 Azure API 管理來實作簡單的企業整合模式
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: mattfarm
ms.author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.date: 06/15/2018
ms.openlocfilehash: 5ed6fa9f514bae3ea651edba6702714e2680091f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46955940"
---
# <a name="simple-enterprise-integration-architecture"></a>簡單的企業整合架構

本文說明的企業整合架構會使用經過證實的做法，您可以在使用 Azure Integration Services 時將此做法套用至整合應用程式。 針對許多需要 HTTP API、工作流程及協調流程的不同應用程式模式，您可以使用此架構作為這些應用程式模式的基礎。

![架構圖 - 簡單的企業整合](./media/logic-apps-architectures-simple-enterprise-integration/simple_arch_diagram.png)

此系列說明可重複使用的元件組件，而這類元件組件可應用於建置一般性整合應用程式。 因為整合技術有許多可能的應用程式，範圍從簡單的點對點應用程式到完整的企業 Azure 服務匯流排應用程式，所以請考慮您需要針對應用程式和基礎結構實作哪些元件。

## <a name="architecture-components"></a>架構元件

此企業整合架構包含下列元件：

- **資源群組**：[資源群組](../azure-resource-manager/resource-group-overview.md)是 Azure 資源的邏輯容器。

- **Azure API 管理**：[API 管理](https://docs.microsoft.com/azure/api-management/)是一個用於發佈、保護及轉換 HTTP API 的完全受控平台。

- **Azure API 管理開發人員入口網站**：Azure API 管理的每個執行個體都提供[開發人員入口網站](../api-management/api-management-customize-styles.md)的存取權。 此入口網站可讓您存取文件和程式碼範例。 您也可以在開發人員入口網站中測試 API。

- **Azure Logic Apps**：[Logic Apps](../logic-apps/logic-apps-overview.md) 是一個無伺服器平台，用於建置企業工作流程和整合。

- **連接器**：Logic Apps 會使用[連接器](../connectors/apis-list.md)來連線到常用的服務。 Logic Apps 提供數百個連接器，但您也可以建立自訂連接器。

- **IP 位址**：Azure API 管理服務有固定的公用 [IP 位址](../virtual-network/virtual-network-ip-addresses-overview-arm.md)和網域名稱。 預設網域名稱是 azure-api.net 的子網域，例如 contoso.azure-api.net，但您也可以設定[自訂網域](../api-management/configure-custom-domain.md)。 Logic Apps 和服務匯流排也有公用 IP 位址。 不過，基於安全考量，此架構會將呼叫 Logic Apps 端點的存取權限制為僅限 API 管理的 IP 位址。 對服務匯流排的呼叫會受到共用存取簽章 (SAS) 保護。

- **Azure DNS**：[Azure DNS](https://docs.microsoft.com/azure/dns/) 是一個適用於 DNS 網域的主機服務。 Azure DNS 採用 Microsoft Azure 基礎結構來提供名稱解析。 只要將您的網域裝載於 Azure，就可以使用您用於其他 Azure 服務的相同認證、API、工具和計費方式來管理 DNS 記錄。 若要使用自訂網域名稱 (例如 contoso.com)，請建立將自訂網域名稱對應至 IP 位址的 DNS 記錄。 如需詳細資訊，請參閱[在 API 管理中設定自訂網域名稱](../api-management/configure-custom-domain.md)。

- **Azure Active Directory (Azure AD)**：使用 [Azure AD](https://docs.microsoft.com/azure/active-directory/) 或其他識別提供者來進行驗證。 Azure AD 藉由傳遞[適用於 API 管理的 JSON Web 權杖](../api-management/policies/authorize-request-based-on-jwt-claims.md)進行驗證，以提供用於存取 API 端點的驗證。 對於標準和進階層，Azure AD 可以保護 API 管理開發人員入口網站的存取。

## <a name="patterns"></a>模式 

此架構會使用一些可作為作業基礎的模式：

* 複合 API 的建置方式是使用邏輯應用程式，其可協調對下列三者的呼叫：軟體即服務 (SaaS) 系統、Azure 服務及任何發佈至 API 管理的 API 呼叫。 Logic Apps [也可透過 API 管理開發人員入口網站來發佈](../api-management/import-logic-app-as-api.md)。

* 應用程式會使用 Azure AD 來[取得 OAuth 2.0 安全性權杖](../api-management/api-management-howto-protect-backend-with-aad.md)，此為取得 API 存取權所需的權杖。

* Azure API 管理會[驗證安全性權杖](../api-management/api-management-howto-protect-backend-with-aad.md)，然後將要求傳遞給後端 API 或邏輯應用程式。

## <a name="recommendations"></a>建議

您的特定需求可能與本文所述的一般架構不同。 以本節的建議作為起點。

### <a name="azure-api-management-tier"></a>Azure API 管理層

使用 API 管理的基本、標準或進階層。 這些層會提供生產環境服務等級協定 (SLA)，並支援在 Azure 區域內相應放大規模。 各層的單位數有所不同。 進階層也支援跨多個 Azure 區域相應放大規模。 根據您的功能集和必要輸送量的層級，選擇您的階層。 如需詳細資訊，請參閱 [API 管理價格](https://azure.microsoft.com/pricing/details/api-management/)。

我們會向您收取所有執行中「API 管理」執行個體的費用。 如果您已相應增加規模，而且不需要一直保有該層級的效能，請考慮利用 API 管理的每小時計費方式並相應減少規模。

### <a name="logic-apps-pricing"></a>Logic Apps 價格

Logic Apps 會使用[無伺服器](../logic-apps/logic-apps-serverless-overview.md)模型。 計費的計算方式是以動作和連接器執行為基礎。 如需詳細資訊，請參閱 [Logic Apps 定價](https://azure.microsoft.com/pricing/details/logic-apps/)。 目前沒有任何針對 Logic Apps 的階層考量。

### <a name="logic-apps-for-asynchronous-api-calls"></a>非同步 API 呼叫的 Logic Apps

Logic Apps 在不需要低延遲的情況下運作得最好。 例如，Logic Apps 最適合非同步或執行時間稍長的 API 呼叫。 如果需要低延遲 (例如，會封鎖使用者介面的呼叫)，請使用不同的技術來實作您的 API 或作業。 例如，使用 Azure Functions，或者以 App Service 來部署的 Web API。 使用 API 管理讓您的 API 朝向您的 API 取用者。

### <a name="region"></a>區域

若要將網路延遲降到最低，請為 API 管理、Logic Apps 及服務匯流排選擇相同的區域。 通常會選擇最接近使用者的區域。

資源群組也有區域。 此區域會指定部署中繼資料的儲存位置，以及部署範本的執行位置。 若要改善部署期間的可用性，請將資源群組和資源放在相同區域。

## <a name="scalability"></a>延展性

若要在管理 API 管理服務時提升延展性，請視需要新增[快取原則](../api-management/api-management-howto-cache.md)。 快取也有助於降低後端服務的負載。

若要提供更大的容量，您可以在 Azure 區域中相應放大 Azure API 管理的基本、標準及進階層。 若要分析服務的使用情況，請在 [計量] 功能表上選取 [容量計量] 選項，然後視需要相應增加或相應減少規模。

調整「API 管理」服務規模的建議事項：

- 請在調整時考慮流量模式。 流量模式變動較大的客戶需要更多容量。

- 容量如果一直大於 66%，可能代表需要相應增加規模。

- 容量如果一直低於 20%，則可能代表需要相應減少規模。

- 在生產環境中啟用負載前，一律先以代表性負載進行 API 管理服務負載測試。

您可以將「進階」層服務相應放大成跨多個 Azure 區域。 如果藉由將服務擴展於多個 Azure 區域，您可以取得更高的 SLA (99.95%或 99.9%)，而且可以在多個區域中於使用者附近佈建服務。

Logic Apps 無伺服器模型代表系統管理員不需要針對服務延展性進行規劃。 服務會自動調整規模以滿足需求。

## <a name="availability"></a>可用性

* 對於基本、標準及進階層，適用於 Azure API 管理的服務等級協定 (SLA) 目前為 99.9%。 進階層組態使用於在兩個以上的區域中至少有 1 個單位的部署時，SLA 為 99.95%。

* 適用於 Azure Logic Apps 的 SLA 目前為 99.9%。

### <a name="backups"></a>備份

根據變更的規律性，[定期備份](../api-management/api-management-howto-disaster-recovery-backup-restore.md)您的 Azure API 管理組態。 將備份檔案儲存在不同於服務所在的位置或 Azure 區域中。 您可以接著針對災害復原策略，選擇下列兩個選項之一：

* 在災害復原事件中，佈建新的 API 管理執行個體、將備份還原至新的執行個體，以及重新指向 DNS 記錄。

* 將您服務的被動複本保存在另一個 Azure 區域中 (會產生額外費用)。 定期將備份還原到該複本。 若要在災害復原事件中還原服務，您只需重新指向 DNS 記錄。

您可以快速重建無伺服器的邏輯應用程式，因此藉由儲存相關 Azure Resource Manager 範本的複本來進行備份。 您可以將範本儲存在原始檔控制中，而且可以整合範本與您的持續整合/持續部署 (CI/CD) 程序。

如果您已透過 Azure API 管理發佈邏輯應用程式，而且該邏輯應用程式移到不同的資料中心，請更新應用程式的位置。 您可以使用基本 PowerShell 指令碼來更新您 API 的 [後端] 屬性。

## <a name="manageability"></a>可管理性

針對生產、部署和測試環境，各別建立資源群組。 個別的資源群組可讓您更輕鬆地管理部署、刪除測試部署及指派存取權限。

當您將資源指派給資源群組時，請考慮下列因素：

* **生命週期**：一般來說，會將具有相同生命週期的資源放在同一個資源群組中。

* **存取**：若要將存取原則套用至群組中的資源，您可以使用[角色型存取控制 (RBAC)](../role-based-access-control/overview.md)。

* **計費**：您可以檢視資源群組的彙總成本。

* **API 管理的定價層**：針對開發和測試環境，使用「開發人員」層。 若要將成本降到最低，請部署生產環境的複本、執行測試，然後關閉。

如需詳細資訊，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。

## <a name="deployment"></a>部署

* 若要部署 Azure API 管理和 Logic Apps，請使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)。 範本可讓您使用 PowerShell 或 Azure CLI，更輕鬆地自動執行部署。

* 將 Azure API 管理及任何個別的邏輯應用程式放在其自己的個別 Resource Manager 範本中。 使用不同的範本時，可以將資源儲存於原始檔控制系統中。 您接著可以一起部署這些範本，或者在持續整合/持續部署 (CI/CD) 流程中個別部署這些範本。

### <a name="versions"></a>版本

每次您變更邏輯應用程式的組態或透過 Resource Manager 範本部署更新時，Azure 都會保留該版本的複本，以方便您使用及保留所有具有執行歷程記錄的版本。 您可以使用這些版本來追蹤歷程記錄變更，或將某個版本升階成邏輯應用程式的目前組態。 例如，您可以有效地復原邏輯應用程式。

「Azure API 管理」有這些截然不同但互補的[版本設定概念](https://blogs.msdn.microsoft.com/apimanagement/2018/01/11/versions-revisions-general-availibility/)：

* 版本，可為 API 取用者提供根據其需求選擇 API 選項 (例如 v1、v2、搶鮮版 (Beta) 或生產環境版) 的能力

* 修訂，可讓 API 系統管理員安全地對 API 進行變更，然後將變更部署至使用者並視需要提供評論

在部署環境中，請考慮將 API 管理修訂視為一種方式，可以安全地進行變更、保留變更歷程記錄，以及讓 API 取用者知道有這些變更。 您可以使用 Resource Manager 範本，在開發環境中進行修訂，然後在其他環境中部署該變更。

雖然您可以先使用修訂來測試 API，再讓這些變更成為「目前」並可供使用者存取，但不建議將此方法用來進行負載或整合測試。 請改用個別的測試或生產階段前環境。

### <a name="configuration-and-sensitive-information"></a>組態和敏感性資訊

永遠不要將密碼、存取金鑰或連接字串簽入原始檔控制。 如果需要這些值，請使用適當的技術來保護和部署這些值。 

在 Logic Apps 中，如果邏輯應用程式需要您無法在連線內建立的任何敏感性值，請將這些值儲存在 Azure Key Vault 中，然後從 Resource Manager 範本參考這些值。 針對每個環境使用部署範本參數和參數檔。 如需詳細資訊，請參閱[保護工作流程中的參數和輸入](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-and-inputs-within-a-workflow)。

API 管理中會使用名為「具名值」或「屬性」的物件來管理祕密。 這些物件能安全地儲存可透過 API 管理原則存取的值。 如需詳細資訊，請參閱[管理 API 管理中的祕密](../api-management/api-management-howto-properties.md)。

## <a name="diagnostics-and-monitoring"></a>診斷和監控

您可以使用 [Azure 監視器](../azure-monitor/overview.md)在 [API 管理](../api-management/api-management-howto-use-azure-monitor.md)和 [Logic Apps](../logic-apps/logic-apps-monitor-your-logic-apps.md) 中進行作業監視。 Azure 監視器預設會啟用，並且會根據針對每個服務所設定的計量來提供資訊。

每個服務也具有下列選項：

* 如需進行更深入的分析並顯示在儀表板上，您可以將 Logic Apps 記錄傳送至 [Azure Log Analytics](logic-apps-monitor-your-logic-apps-oms.md)。

* 若要進行 DevOps 監視，您可以針對 API 管理設定 Azure Application Insights。

* API 管理支援[適用於自訂 API 分析的 Power BI 解決方案範本](http://aka.ms/apimpbi) \(英文\)。 您可以使用此解決方案範本來建立自己的分析解決方案。 若為商務使用者，Power BI 中有可用的報告。

## <a name="security"></a>安全性

雖然這份清單並未完整地說明所有的安全性最佳做法，以下安全性考量專門用於本文所述架構中部署的 Azure 服務：

* 若要確保使用者具有適當的存取層級，請使用角色型存取控制 (RBAC)。

* 使用 OAuth 或 Open ID Connect 來保護 API 管理中的公用 API 端點。 若要保護公用 API 端點，請設定識別提供者，並新增 JSON Web 權杖 (JWT) 驗證原則。

* 使用相互憑證，從 API 管理連線至後端服務。

* 藉由建立指向 API 管理 IP 位址的 IP 位址白名單，以保護 HTTP 觸發程序型的 Logic Apps。 列入白名單的 IP 位址可防止不先經過 API 管理就從公用網際網路呼叫邏輯應用程式的情況。

## <a name="next-steps"></a>後續步驟

* 了解[搭配佇列和事件的企業整合](../logic-apps/logic-apps-architectures-enterprise-integration-with-queues-events.md)
