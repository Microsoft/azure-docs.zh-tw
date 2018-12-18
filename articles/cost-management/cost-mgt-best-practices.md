---
title: 透過 Azure 成本管理將雲端投資最佳化 | Microsoft Docs
description: 本文有助於從雲端投資中獲得最大價值、降低成本並評估您的資金支出。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/10/2018
ms.topic: conceptual
ms.service: cost-management
manager: dougeby
ms.custom: ''
ms.openlocfilehash: f52a849fee4005628fb6a8ad567d89f5046d4765
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968063"
---
# <a name="how-to-optimize-your-cloud-investment-with-azure-cost-management"></a>如何透過 Azure 成本管理將雲端投資最佳化

Azure 成本管理可提供您規劃、分析以及減少費用，以便將雲端投資發揮到極致的工具。 本文件為您提供了一種有條理的成本管理方法，並強調在您解決組織的成本挑戰時可以使用的工具。 Azure 可讓您輕鬆建置及部署雲端解決方案。 但是，這些解決方案必須經過最佳化，才能將組織的成本降到最低。 依照本文件中所述的原則，並使用我們的工具將有助於確保貴組織做好成功的準備。

## <a name="methodology"></a>方法

成本管理是組織的問題，而且應該是在您投資雲端資源之前就開始持續進行的工作。 若要成功實施成本管理，並將成本最佳化，貴組織必須：

- 備妥可讓您邁向成功的適當工具
- 對成本負責
- 採取適當的動作，將費用最佳化

貴組織必須符合以下所列的三個關鍵群組，才能確保您成功管理成本。

- **財務** - 負責根據雲端費用預測，核准整個組織預算申請的人員。 他們會支付相對應的帳單，並將成本指派給各個小組以劃分權責。
- **經理** - 組織中的業務決策者，必須了解雲端費用，才能找出最佳費用結果。
- **應用程式小組** - 管理雲端資源的工程師會以每天為基礎，開發服務以符合組織的需求。 這些小組需要在其定義的預算中提供最高價值的彈性。

### <a name="key-principles"></a>主要原則

使用以下所述的原則，為貴組織在雲端成本管理方面的成就定位。

#### <a name="planning"></a>規劃

全方位的前期規劃可讓您根據特定的業務需求，量身打造雲端使用情況。 自問：

- 我要解決哪些業務問題？
- 我預期資源有哪些使用模式？

您的答案將協助您選擇最適合您的供應項目。 它們會決定要使用的基礎架構以及如何用於將 Azure 的效率發揮到極致。

#### <a name="visibility"></a>可見性

如果結構良好，成本管理可協助您告知人員所負責的 Azure 成本或所花的錢。 Azure 的服務專門針對深入了解錢花在*哪裡*而設計。 利用這些工具。 這些工具有助於找出未充分使用的資源、消除浪費，並將節省成本的機會發揮到極致。

#### <a name="accountability"></a>責任

歸納組織中的成本，以確保負責的人員對其小組的費用負責。 若要充分了解貴組織的 Azure 費用，您應該整理資源並仔細歸納成本。 良好的組織有助於管理並降低成本，並使人員對組織中的有效費用負責。

#### <a name="optimization"></a>最佳化

採取行動以減少費用。 根據透過規劃和增加成本可見度而蒐集到的結果充分發揮。 您可以考慮購買及授權最佳化，以及本文件稍後將詳細討論的基礎架構部署變更。

#### <a name="iteration"></a>反覆運算

組織中的每個人都必須參與成本管理生命週期。 他們必須持續參與，以便將成本最佳化。 嚴格遵循此反覆運算過程，使其成為貴組織中負責雲端治理的重要租用戶。

![主要原則](./media/cost-mgt-best-practices/principles.png)

## <a name="plan-with-cost-in-mind"></a>使用成本用心規劃

部署雲端資源之前，請評估下列項目：

- 最符合您需求的 Azure 供應項目
- 您打算使用的資源
- 這些資源可能的成本

Azure 會提供工具來協助您評估程序。 這些工具可讓您了解啟用工作負載所需的投資。 然後，您可以為您的情況選擇最佳設定。

### <a name="azure-onboarding-options"></a>Azure 上架選項

將成本管理中的經驗發揮到極致的第一個步驟是調查並決定最適合您的 Azure 供應項目。 請思考您打算在未來如何使用 Azure。 也請考慮您要如何設定計費模式。 做決定時，請考慮下列問題：

- 我打算使用 Azure 多久？ 我是要測試還是打算建置更長期的基礎架構？
- 我想要如何支付 Azure 費用？ 我應該預付較低的價格還是在月底開發票？

若要深入了解各種選項，請造訪[如何購買 Azure](https://azure.microsoft.com/pricing/purchase-options/)。 以下是幾個最常見的計費模式。

#### <a name="freehttpsazuremicrosoftcomfree"></a>[免費](https://azure.microsoft.com/free/)

- 12 個月的熱門免費服務
- 使用 200 美元的信用額度探索服務 30 天
- 25 項以上的服務一律免費

#### <a name="pay-as-you-gohttpsazuremicrosoftcomoffersms-azr-0003p"></a>[隨用隨付](https://azure.microsoft.com/offers/ms-azr-0003p)

- 沒有最低金額或承諾用量
- 具有競爭力的價格
- 用多少付多少
- 可隨時取消

#### <a name="enterprise-agreementhttpsazuremicrosoftcompricingenterprise-agreement"></a>[Enterprise 合約](https://azure.microsoft.com/pricing/enterprise-agreement/)

- 預付承諾用量的選項
- 取得較低的 Azure 定價

## <a name="estimate-the-cost-of-your-solution"></a>預估解決方案的成本

部署任何基礎架構之前，請評估解決方案的費用。 評估將有助於為貴組織前期的工作負載建立預算。 接著，您可以使用預算一段時間，來衡量您初始估計有效性的基準。 此外，您可以將其與已部署之解決方案的實際成本相比較。

### <a name="azure-pricing-calculator"></a>Azure 定價計算機

Azure 定價計算機可讓您混合及比對不同的 Azure 服務組合，以查看成本估計值。 您可以在 Azure 中使用不同的方式實作解決方案，每個方式都可能會影響您的整體費用。 及早思考雲端部署的所有基礎架構需求可協助您最有效地使用此工具。 此工具可以協助您在 Azure 中準確估算預估的費用。

如需詳細資訊，請參閱 [Azure 定價計算機](https://azure.microsoft.com/pricing/calculator)。

### <a name="azure-migrate"></a>Azure Migrate

Azure Migrate 是一種服務，可評估貴組織在內部部署資料中心內的目前工作負載。 它可讓您深入了解您需要從 Azure 替代方案獲得的服務。 首先，Migrate 會分析您的內部部署電腦，以判斷移轉是否可行。 接著，它會建議 Azure 中的 VM 大小，以便將效能發揮到極致。 最後，它也會建立以 Azure 為基礎的解決方案成本預估。

如需詳細資訊，請參閱 [Azure Migrate](../migrate/migrate-overview.md)。

## <a name="analyze-and-manage-your-costs"></a>分析和管理成本

隨時了解貴組織成本如何變化。 使用下列技術正確了解並管理您的費用。

### <a name="organize-and-tag-your-resources"></a>組織和標記資源

使用成本用心組織您的資源。 當您建立訂用帳戶和資源群組時，請思考負責相關聯成本的小組。 請確定您的報告會牢記您的組織。 訂用帳戶和資源群組提供良好的值區，以整理並歸納整個組織的費用。 標記提供良好的歸納成本方式。 您可以使用標記作為篩選條件。 此外，當您分析資料以及調查成本時，您可以使用標記來分組。 Enterprise 合約客戶也可以建立部門，並將訂用帳戶放在其下。 在 Azure 中，以成本為基礎的組織有助於將負責減少其小組費用的相關人員保留在您組織中。

### <a name="use-cost-analysis"></a>使用成本分析

成本分析可讓您使用標準資源屬性細分您的成本，以深入分析您組織的成本。 請考慮下列常見的問題作為分析的指南。 定期回答這些問題將協助您掌握更多資訊，並做出更具成本意識的決策。

- **當月預估成本** – 這個月到目前為止我產生了多少費用？ 我將維持在預算之內嗎？
- **調查異常** – 進行例行檢查，以確定成本維持在合理的正常使用範圍內。 趨勢為何？ 是否有任何異常值？
- **發票對帳** - 我最近的發票費用是否超過上個月？ 每月消費習慣如何變化？
- **內部退款** - 既然我知道自己收取了多少費用，那麼如何為我的組織將這些費用分類？

如需詳細資訊，請參閱[成本分析](quick-acm-cost-analysis.md)。

### <a name="export-billing-data-on-a-schedule"></a>排程匯出帳單資料

您需要將帳單資料匯入外部系統，例如儀表板或財務系統嗎？ 您可以排程每日自動報告，以避免每個月手動下載檔案。 此外，您可以將帳單資料匯出至 Azure 儲存體帳戶，並使用[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)取得通知。 接著，您可以使用 Azure 資料結合您可以在自己的系統中使用的自訂資料。

如需有關匯出帳單資料的詳細資訊，請參閱[建立和管理匯出的資料](tutorial-export-acm-data.md)。

### <a name="create-budgets"></a>建立預算

在您找出並分析您的消費模式之後，務必開始為自己和您的小組設定限制。 Azure 預算讓您能夠設定以成本或使用量為基礎的預算，且其中包含多個臨界值與警示。 請務必審核您定期建立的預算，以查看您的預算燃盡進度，並視需要進行變更。 Azure 預算也可讓您設定達到指定預算臨界值時的自動化觸發程序，。 例如，您可以設定您的服務關閉 VM。 或者，您可以將基礎架構移至不同的定價層，以回應預算觸發程序。

如需詳細資訊，請參閱 [Azure 預算](tutorial-acm-create-budgets.md)。

如需有關以預算為基礎之自動化的詳細資訊，請參閱[以預算為基礎的自動化](../billing/billing-cost-management-budget-scenario.md)。

## <a name="act-to-optimize"></a>採取最佳化行動
使用下列方式將費用最佳化。

### <a name="cut-out-waste"></a>減少浪費

在 Azure 中部署基礎架構之後，務必確定正在使用它。 立即開始儲存最簡單的方式是檢閱您的資源，並移除未在使用的所有資源。 您應該從該處判斷盡可能有效率地使用您的資源。

#### <a name="azure-advisor"></a>Azure Advisor

Azure Advisor 是一種服務，除此之外，可從 CPU 或網路使用方式觀點，識別低使用率的虛擬機器。 您可以從該處根據預估的成本，決定要關閉機器還是調整其大小，以繼續運轉機器。 Advisor 也會提供保留的執行個體購買建議。 這些建議是以您過去 30 天內的虛擬機器使用情況為基礎。 採取行動時，建議有助於減少費用。

如需詳細資訊，請參閱 [Azure Advisor](../advisor/advisor-overview.md)。

### <a name="size-your-vms-properly"></a>適當地調整 VM 的大小

VM 大小對於整體 Azure 費用會造成重大的影響。 Azure 中需要的 VM 數目可能不等於目前在內部部署資料中心內部署的 VM 數目。 請務必為您打算執行的工作負載，選擇適當的大小。

如需詳細資訊，請參閱 [Azure IaaS: proper sizing and cost](https://azure.microsoft.com/resources/videos/azurecon-2015-azure-iaas-proper-sizing-and-cost/)。

### <a name="use-purchase-discounts"></a>使用購買折扣

Azure 有許多組織可以利用的折扣以節省成本。

#### <a name="azure-reservations"></a>Azure 保留

Azure 保留可讓您預付一年或三年期的虛擬機器或 SQL Database 計算容量。 預付費用可讓您在所使用的資源上取得折扣。 Azure 保留可以大幅降低虛擬機器或 SQL Database 的計算成本。透過預付一年或三年期的承諾用量費用，即可節省高達預付型方案價格的 72%。 保留會提供計費折扣，且不會影響虛擬機器或 SQL Database 的執行階段狀態。

如需詳細資訊，請參閱[什麼是 Azure 保留？](../billing/billing-save-compute-costs-reservations.md)。

#### <a name="use-azure-hybrid-benefit"></a>使用 Azure Hybrid Benefit

如果您在內部部署中已經有 Windows Server 或 SQL Server 授權，您可以使用 Azure Hybrid Benefit 程式儲存在 Azure 中。 利用 Windows Server 權益，每個授權都涵蓋 OS (最多兩部虛擬機器上) 的費用，而您只需支付基礎計算費用。 您可以使用現有的 SQL Server 授權，最多節省 55% 的 vCore 型 SQL Database 選項。 選項包括 Azure 虛擬機器和 SQL Server Integration Services 中的 SQL Server。

如需詳細資訊，請參閱 [Azure Hybrid Benefit 節費計算機](https://azure.microsoft.com/pricing/hybrid-benefit/)。

#### <a name="monetary-commitment"></a>承諾用量

簽署 Enterprise 合約的客戶可以建立 Microsoft 的預付承諾用量。 除了提供簡單的方式來配置雲端資源的預算，承諾用量可讓您降低 Azure 的費率。 降低費率可為您的組織節省大量的成本。

### <a name="other-resources"></a>其他資源

Azure 還提供一項服務，可讓您建置能夠利用 Azure 中的剩餘容量來降低費率的服務。 如需詳細資訊，請參閱[使用低優先順序的 VM 搭配 Batch](../batch/batch-low-pri-vms.md)。

## <a name="next-steps"></a>後續步驟
- 如果您不熟悉成本管理，請閱讀[什麼是 Azure 成本管理？](overview-cost-mgt.md)來了解它如何協助您監視及控制 Azure 費用，以便將資源使用狀況最佳化。
