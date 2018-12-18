---
title: 企業整合架構模式 - Azure Integration Services
description: 此架構參考說明如何使用 Azure Logic Apps、Azure API 管理、Azure 服務匯流排和 Azure 事件方格來實作企業整合模式
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: mattfarm
ms.author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.date: 06/15/2018
ms.openlocfilehash: 8fbc84b4016659b6d0d6ce9ec47c05a0b241c3d9
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48855369"
---
# <a name="enterprise-integration-architecture-with-queues-and-events"></a>使用佇列和事件的企業整合架構

本文說明的企業整合架構會使用經過證實的做法，您可以在使用 Azure 整合服務時將此做法套用至整合應用程式。 針對許多需要 HTTP API、工作流程及協調流程的不同應用程式模式，您可以使用此架構作為這些應用程式模式的基礎。

![架構圖 - 搭配佇列和事件的企業整合](media/logic-apps-architectures-enterprise-integration-with-queues-events/integr_queues_events_arch_diagram.png)

此系列說明可重複使用的元件組件，而這類元件組件可應用於建置一般性整合應用程式。 因為整合技術有許多可能的應用程式，範圍從簡單的點對點應用程式到完整的企業 Azure 服務匯流排應用程式，所以請考慮您需要針對應用程式和基礎結構實作哪些元件。

## <a name="architecture-components"></a>架構元件

此架構的建置基礎為[架構參考：簡單的企業整合](../logic-apps/logic-apps-architectures-simple-enterprise-integration.md)一文中說明的架構。 該架構的[建議](../logic-apps/logic-apps-architectures-simple-enterprise-integration.md#recommendations)也適用於此處，但為求簡潔，本文會在[建議](#recommendations)一節中省略這些建議。 此企業整合架構包含下列元件：

- **資源群組**：[資源群組](../azure-resource-manager/resource-group-overview.md)是 Azure 資源的邏輯容器。

- **Azure API 管理**：[API 管理](https://docs.microsoft.com/azure/api-management/)是一個用於發佈、保護及轉換 HTTP API 的完全受控平台。

- **Azure API 管理開發人員入口網站**：Azure API 管理的每個執行個體都提供[開發人員入口網站](../api-management/api-management-customize-styles.md)的存取權。 此入口網站可讓您存取文件和程式碼範例。 您也可以在開發人員入口網站中測試 API。

- **Azure Logic Apps**：[Logic Apps](../logic-apps/logic-apps-overview.md) 是一個無伺服器平台，用於建置企業工作流程和整合。

- **連接器**：Logic Apps 會使用[連接器](../connectors/apis-list.md)來連線到常用的服務。 Logic Apps 提供數百個連接器，但您也可以建立自訂連接器。

- **服務匯流排**：[服務匯流排](../service-bus-messaging/service-bus-messaging-overview.md)可提供安全可靠的傳訊。 您可以將傳訊用來分離應用程式，然後與其他訊息型系統整合。

- **Azure 事件方格**：[事件方格](../event-grid/overview.md)是一個用來發佈和傳遞應用程式事件的無伺服器平台。

- **IP 位址**：Azure API 管理服務有固定的公用 [IP 位址](../virtual-network/virtual-network-ip-addresses-overview-arm.md)和網域名稱。 預設網域名稱是 azure-api.net 的子網域，例如 contoso.azure-api.net，但您也可以設定[自訂網域](../api-management/configure-custom-domain.md)。 Logic Apps 和服務匯流排也有公用 IP 位址。 不過，基於安全考量，此架構會將呼叫 Logic Apps 端點的存取權限制為僅限 API 管理的 IP 位址。 對服務匯流排的呼叫會受到共用存取簽章 (SAS) 保護。

- **Azure DNS**：[Azure DNS](https://docs.microsoft.com/azure/dns/) 是一個適用於 DNS 網域的主機服務。 Azure DNS 採用 Microsoft Azure 基礎結構來提供名稱解析。 只要將您的網域裝載於 Azure，就可以使用您用於其他 Azure 服務的相同認證、API、工具和計費方式來管理 DNS 記錄。 若要使用自訂網域名稱 (例如 contoso.com)，請建立將自訂網域名稱對應至 IP 位址的 DNS 記錄。 如需詳細資訊，請參閱[在 API 管理中設定自訂網域名稱](../api-management/configure-custom-domain.md)。

- **Azure Active Directory (Azure AD)**：使用 [Azure AD](https://docs.microsoft.com/azure/active-directory/) 或其他識別提供者來進行驗證。 Azure AD 藉由傳遞[適用於 API 管理的 JSON Web 權杖](../api-management/policies/authorize-request-based-on-jwt-claims.md)進行驗證，以提供用於存取 API 端點的驗證。 對於標準和進階層，Azure AD 可以保護 API 管理開發人員入口網站的存取。

## <a name="patterns"></a>模式 

此架構會使用一些可作為作業基礎的模式：

* 現有的後端 HTTP API 是透過 API 管理開發人員入口網站來發佈的。 在入口網站中，組織內部和 (或) 外部的開發人員  
可以將對這些 API 的呼叫整合到應用程式。

* 複合 API 的建置方式是使用邏輯應用程式，其可協調對下列三者的呼叫：軟體即服務 (SaaS) 系統、Azure 服務及任何發佈至 API 管理的 API 呼叫。 Logic Apps [也可透過 API 管理開發人員入口網站來發佈](../api-management/import-logic-app-as-api.md)。

* 應用程式會使用 Azure AD 來[取得 OAuth 2.0 安全性權杖](../api-management/api-management-howto-protect-backend-with-aad.md)，此為取得 API 存取權所需的權杖。

* Azure API 管理會[驗證安全性權杖](../api-management/api-management-howto-protect-backend-with-aad.md)，然後將要求傳遞給後端 API 或邏輯應用程式。

* 「Azure 服務匯流排」佇列可用來將應用程式活動[分離](../service-bus-messaging/service-bus-messaging-overview.md)，並[緩和負載中的尖峰](https://docs.microsoft.com/azure/architecture/patterns/queue-based-load-leveling)。 訊息會由 Logic Apps、第三方應用程式新增至佇列，或透過 API 管理將佇列發佈為 HTTP API 來新增至佇列 (未顯示)。

* 將訊息新增至服務匯流排佇列時，就會觸發事件。 事件會觸發邏輯應用程式，由它來處理訊息。

* 其他 Azure 服務 (例如 Azure Blob 儲存體和 Azure 事件中樞) 也會將事件發佈至事件方格。 這些服務會觸發 Logic Apps 來接收事件，然後執行後續動作。

## <a name="recommendations"></a>建議

您的特定需求可能與本文所述的一般架構不同。 以本節的建議作為起點。

### <a name="service-bus-tier"></a>服務匯流排層

請使用支援事件方格通知的服務匯流排進階層。 如需詳細資訊，請參閱[服務匯流排價格](https://azure.microsoft.com/pricing/details/service-bus/)。

### <a name="event-grid-pricing"></a>事件格線價格

事件方格會使用無伺服器模型。 計費的計算方式以作業 (事件執行) 的數目為基礎。 如需詳細資訊，請參閱[事件方格價格](https://azure.microsoft.com/pricing/details/event-grid/)。 目前沒有任何針對事件方格的階層考量。

### <a name="use-peeklock-to-consume-service-bus-messages"></a>使用 PeekLock 來取用服務匯流排訊息

當您建立邏輯應用程式以取用服務匯流排訊息時，請讓邏輯應用程式使用 [PeekLock](../service-bus-messaging/service-bus-messaging-overview.md#queues) 來存取訊息群組。 當您使用 PeekLock 時，邏輯應用程式可以先執行步驟來驗證每個訊息，然後再完成或放棄訊息。 此方法可防止意外的訊息遺失。

### <a name="check-for-multiple-objects-when-an-event-grid-trigger-fires"></a>引發事件格線觸發程序時，請檢查是否有多個物件

事件方格觸發程序引發時，此動作只是表示「至少發生了其中一個事項。」 例如，當事件方格針對出現在服務匯流排佇列中的訊息觸發邏輯應用程式時，該邏輯應用程式應該一律假設有一或多個訊息可供處理。

### <a name="region"></a>區域

若要將網路延遲降到最低，請為 API 管理、Logic Apps 及服務匯流排選擇相同的區域。 通常會選擇最接近使用者的區域。

資源群組也有區域。 此區域會指定部署中繼資料的儲存位置，以及部署範本的執行位置。 若要改善部署期間的可用性，請將資源群組和資源放在相同區域。

## <a name="scalability"></a>延展性

若要達到更高的延展性，服務匯流排進階層可以將傳訊單位數目相應放大。 進階層設定可以有一個、兩個或四個傳訊單位。 如需調整服務匯流排規模的詳細資訊，請參閱[使用服務匯流排傳訊的效能改進最佳作法](../service-bus-messaging/service-bus-performance-improvements.md)。

## <a name="availability"></a>可用性

* 對於基本、標準及進階層，適用於 Azure API 管理的服務等級協定 (SLA) 目前為 99.9%。 進階層組態使用於在兩個以上的區域中至少有 1 個單位的部署時，SLA 為 99.95%。

* 適用於 Azure Logic Apps 的 SLA 目前為 99.9%。

### <a name="disaster-recovery"></a>災害復原

若要能夠在發生嚴重服務中斷時進行容錯移轉，請考慮在服務匯流排進階層中實作異地災害復原。 如需詳細資訊，請參閱 [Azure 服務匯流排異地災害復原](../service-bus-messaging/service-bus-geo-dr.md)。

## <a name="manageability"></a>可管理性

針對生產、部署和測試環境，各別建立資源群組。 個別的資源群組可讓您更輕鬆地管理部署、刪除測試部署及指派存取權限。

當您將資源指派給資源群組時，請考慮下列因素：

* **生命週期**：一般來說，會將具有相同生命週期的資源放在同一個資源群組中。

* **存取**：若要將存取原則套用至群組中的資源，您可以使用[角色型存取控制 (RBAC)](../role-based-access-control/overview.md)。

* **計費**：您可以檢視資源群組的彙總成本。

* **API 管理的定價層**：針對開發和測試環境，使用「開發人員」層。 若要將成本降到最低，請部署生產環境的複本、執行測試，然後關閉。

如需詳細資訊，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。

## <a name="deployment"></a>部署

* 若要部署 API 管理、Logic Apps、事件方格和服務匯流排，請使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-authoring-templates.md)。 範本可讓您使用 PowerShell 或 Azure CLI，更輕鬆地自動執行部署。

* 請將 API 管理、任何個別的 Logic Apps、事件方格主題及服務匯流排命名空間放在其本身的個別 Resource Manager 範本中。 使用不同的範本時，可以將資源儲存於原始檔控制系統中。 您接著可以一起部署這些範本，或者在持續整合/持續部署 (CI/CD) 流程中個別部署這些範本。

## <a name="diagnostics-and-monitoring"></a>診斷和監控

就像 API 管理和 Logic Apps 一樣，您可以使用預設為啟用的 Azure 監視器來監視服務匯流排。 Azure 監視器會根據針對每個服務所設定的計量來提供資訊。 

## <a name="security"></a>安全性

若要保護服務匯流排，請使用共用存取簽章 (SAS)。 例如，您可以使用 [SAS 驗證](../service-bus-messaging/service-bus-sas.md)，授與使用者具特定權限的服務匯流排資源存取權。 如需詳細資訊，請參閱[服務匯流排驗證和授權](../service-bus-messaging/service-bus-authentication-and-authorization.md)。

如果您需要將服務匯流排佇列以 HTTP 端點的形式 (舉例而言) 公開，以發佈新訊息，請使用 API 管理讓該端點成為前端，以保護佇列。 接著，您可以視情況使用憑證或 OAuth 驗證來保護該端點。 要保護端點，最簡單的方法是使用邏輯應用程式與 HTTP 要求/回應觸發程序作為媒介。

事件方格服務會透過驗證碼來保護事件傳遞。 如果您使用 Logic Apps 來取用事件，將會自動執行驗證。 如需詳細資訊，請參閱 [Event Grid 安全性和驗證](../event-grid/security-authentication.md)。

## <a name="next-steps"></a>後續步驟

* 了解[簡單的企業整合](logic-apps-architectures-simple-enterprise-integration.md)
