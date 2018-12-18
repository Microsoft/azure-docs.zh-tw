---
title: ClearDB MySQL 資料庫搭配 Azure App Service 的常見問題集 | Microsoft Docs
description: 使用 ClearDB MySQL 資料庫搭配 Azure App Service 常見問題的解答。
documentationcenter: php
services: mysql
author: sunbuild
manager: yochayk
tags: mysql
ms.service: multiple
ms.workload: data-management
ms.topic: article
ms.date: 10/27/2016
ms.author: sumuth
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: f145e879dab4484d647cf8ea21582958df0f0cd4
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665717"
---
# <a name="faq-for-cleardb-mysql-databases-with-azure-app-service"></a>ClearDB MySQL 資料庫搭配 Azure App Service 的常見問題集
此常見問題集可回答為 Azure Web Apps 使用及購買 ClearDB MySQL 資料庫的常見問題。

> [!IMPORTANT]
> 自 2018 年 6 月 13 日起，ClearDB 將目前由 Microsoft 收費的 Azure 客戶轉換為使用 ClearDB 的直接計費模式。 本文中的資訊已過時。 您將無法再建立或升級在 Azure 中建立的 ClearDB 資料庫。
>
> 如需詳細資訊和後續步驟，請參閱 [ClearDB 服務方案之變更](http://w2.cleardb.net/important-change-of-billing-notice-for-all-azure-cleardb-service-plans/) (英文)。


## <a name="what-options-do-i-have-for-mysql-on-azure"></a>在 Azure 上的 MySQL 有哪些選項？
您有幾種選項：

* [ClearDB Shared MySQL 資料庫](/marketplace/partners/cleardb/databases/)
* [ClearDB MySQL Premium 叢集](/marketplace/partners/cleardb-clusters/cluster/)
* [Azure VM 上執行的 MySQL 叢集](https://github.com/azure/azure-quickstart-templates/tree/master/mysql-replication)
* [Azure VM 上執行的 MySQL 單一執行個體](virtual-machines/windows/classic/mysql-2008r2.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)

ClearDB 是 MySQL 代管服務，會為您管理 MySQL 基礎結構。 在 Azure 虛擬機器上執行自己的 MySQL 叢集或資料庫時，您必須設定 MySQL 伺服器，並使用修補程式讓它保持更新。

## <a name="do-i-need-a-credit-card-for-the-web-app--mysql-template-in-the-azure-marketplace"></a>要取得 Azure Marketplace 中的 Web 應用程式與 MySQL 範本需要使用信用卡嗎？
這取決於您使用的訂用帳戶類型。 以下是一些常用的訂用帳戶類型：

* [預付型](https://azure.microsoft.com/offers/ms-azr-0003p/)：需要信用卡，購買付費的 MySQL 資料庫時，將向您的信用卡收費。
* [免費試用](https://azure.microsoft.com/pricing/free-trial/)：包括可用於 Microsoft Azure 服務的信用額度，但不允許購買第三方資源。 若要購買第三方服務或付費的 MySQL 資料庫，您需要使用啟用信用卡的訂用帳戶。 針對 Web Apps，您可以建立免費的 ClearDB MySQL 資料庫。
* [MSDN 訂用帳戶](https://azure.microsoft.com/pricing/member-offers/msdn-benefits/) 和 **MSDN 開發人員測試預付型**：類似於免費試用，MSDN 訂用帳戶要求您使用信用卡向 ClearDB 購買付費的 MySQL 解決方案。
* [Enterprise 合約 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)：我們會每季以單一彙總的發票就 EA 向 EA 客戶的所有 Azure Marketplace (第三方) 購買項目收費。 對於任何 Marketplace 購買項目，將向您就財務承諾以外收費。 請注意，目前 Azure 市集無法供在亞塞拜然、克羅埃西亞、挪威和波多黎各註冊的客戶使用。 

## <a name="why-was-i-charged-350-for-a-web-app--mysql-from-the-azure-marketplace"></a>為什麼透過 Azure Marketplace 向我收取 Web 應用程式 + MySQL 3.50 元美金的費用？
預設資料庫選項是 Titan，也就是 3.50 元美金。 建立資料庫時我們不會顯示成本，您可能會錯誤地購買您不想要的資料庫。 我們正在設法改善此情況，但在那之前，您必須檢查對 Web 應用程式和資料庫所選取的定價層，之後才按一下 [建立]  並開始部署資源。

## <a name="i-am-running-mysql-on-my-own-azure-virtual-machine-can-i-connect-my-azure-web-app-to-my-database"></a>我在自己的 Azure 虛擬機器上執行 MySQL。 可以將我的 Azure Web 應用程式連接到我的資料庫嗎？
是。 只要您的 Azure VM 已將遠端存取權提供給 Web 應用程式，Web 應用程式就能連線到您的資料庫。 如需詳細資訊，請參閱 [在虛擬機器上安裝 MySQL](virtual-machines/windows/classic/mysql-2008r2.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

## <a name="in-which-countries-are-cleardb-premium-mysql-clusters-supported"></a>支援 ClearDB Premium MySQL 叢集的國家 (地區)有哪些？
[ClearDB Premium MySQL 叢集](/marketplace/partners/cleardb-clusters/cluster/) 。

## <a name="can-i-create-a-new-cluster-prior-to-creating-a-database-with-cleardb-premium-cluster-solution"></a>可以於使用 ClearDB Premium 叢集解決方案建立資料庫之前建立新的叢集嗎？
否，不支援建立空白的 ClearDB 叢集。 Azure 入口網站可讓您在叢集中建立資料庫，這可能會同時建立新的叢集。

## <a name="will-i-be-warned-if-i-try-to-delete-a-cleardb-mysql-database-that-is-in-use-by-one-of-my-applications"></a>如果我嘗試刪除正由其中一個應用程式使用的 ClearDB MySQL 資料庫，會收到警告嗎？
否，如果您刪除應用程式所相依的 Marketplace 購買項目，Azure 將不會警告您。

## <a name="which-regions-can-i-create-cleardb-databases-in"></a>可以在哪些區域中建立 ClearDB 資料庫？
Azure Marketplace 無法供在亞塞拜然、克羅埃西亞、挪威或波多黎各註冊的客戶使用。 這些區域中無法使用 ClearDB。

## <a name="what-pricing-tier-should-i-choose-for-a-production-web-app-and-database"></a>針對生產 Web 應用程式和資料庫，應該選擇哪個定價層？
對 Web Apps 使用「基本」或更高的定價層。 針對 ClearDB，我們建議 Saturn 或 Jupiter 計劃。 請檢閱 [Web Apps](https://azure.microsoft.com/pricing/details/app-service/) 和 [ClearDB MySQL 資料庫](/marketplace/partners/cleardb/databases/)每個定價層的功能和限制，來選擇符合您需求的定價層。

## <a name="how-do-i-upgrade-my-cleardb-database-from-one-plan-to-another"></a>如何將我的 ClearDB 資料庫從一個計劃升級到另一個？
在 [Azure 入口網站](https://portal.azure.com)，您可以相應增加 ClearDB 共用裝載資料庫。 閱讀[本文](https://blogs.msdn.microsoft.com/appserviceteam/2016/10/06/upgrade-your-cleardb-mysql-database-in-azure-portal/)以深入了解。 我們目前不支援 Azure 入口網站中的 ClearDB 進階叢集升級。

## <a name="i-cant-see-my-cleardb-database-in-azure-portal"></a>我在 Azure 入口網站中看不到我的 ClearDB 資料庫？
如果在傳統入口網站中建立了 ClearDB 資料庫，您在 [Azure 入口網站](https://portal.azure.com)中就看不到您的資料庫。 這個問題目前無法解決。

## <a name="who-do-i-contact-for-support-when-my-database-is-down"></a>我的資料庫關閉時應向誰連絡尋求支援？
如有任何資料庫相關的問題，請連絡 [ClearDB 支援](https://www.cleardb.com/developers/help/support) 。 準備好提供問題資訊與您的 Azure 訂用帳戶資訊。

## <a name="can-i-create-additional-users-for-my-cleardb-mysql-database-cluster-solution"></a>我可以為自己的 ClearDB MySQL 資料庫叢集解決方案建立其他的使用者嗎？
不行。 您無法建立其他的使用者，但可以在自己的 ClearDB 資料庫叢集上建立其他的資料庫。  

## <a name="can-basicpro-series-databases-be-upgraded-in-place-similar-to-planetary-plans-today-on-cleardb-portal"></a>與 ClearDB 入口網站上目前的 Planetary 計劃類似，是否可以就地升級基本/專業系列資料庫嗎？
是，可以就地升級基本系列資料庫 (基本 60 到基本 500)。 可以就地升級專業系列 (專業 125 到專業 1000)，但專業 60 除外。 我們目前不支援升級專業 60 資料庫。 

## <a name="when-i-migrate-my-resources-from-one-subscription-to-another-does-my-cleardb-mysql-database-get-migrated-as-well"></a>當我把資源從某個訂用帳戶移轉到另一個訂用帳戶時，我的 ClearDB MySQL 資料庫也會跟著移轉嗎？
當跨訂用帳戶執行資源移轉時，適用某些 [限制](azure-resource-manager/resource-group-move-resources.md#app-service-limitations) 。 ClearDB MySQL 資料庫是第三方服務，因此在 Azure 訂用帳戶移轉期間是不會移轉的。 如果您在移轉 Azure 資源之前，沒有管理自己 MySQL 資料庫的移轉作業，您的 ClearDB MySQL 資料庫可能會遭到停用。 請先手動移轉資料庫，然後再為您的 Web 應用程式執行 Azure 訂用帳戶移轉作業。 

## <a name="i-hit-the-spending-limit-on-my-subscription-i-removed-the-limit-and-my-app-service-is-online-however-the-database-is-not-accessible-how-do-i-re-enable-the-cleardb-database"></a>我達到訂用帳戶的消費限制。 我移除了限制，且 App Service 也上線，但仍無法存取資料庫。 我如何重新啟用 ClearDB 資料庫？
連絡 [ClearDB 支援](https://www.cleardb.com/developers/help/support)，以重新啟用資料庫。 提供您的 Azure 訂用帳戶資訊和資料庫名稱。

## <a name="can-i-transfer-a-cleardb-database-from-a-credit-card-subscription-to-an-ea-subscription"></a>我可以將 ClearDB 資料庫從信用卡訂用帳戶移轉給 EA 訂用帳戶嗎？
現有的 ClearDB 資料庫會使用與現有的訂用帳戶相關聯的信用卡。 若要使用 EA 訂用帳戶，您需要將資料移轉到新的資料庫：

* 使用您的 EA 訂用帳戶購買新 ClearDB 資料庫。
* 將資料移轉至新的資料庫。
* 更新應用程式以使用新的資料庫。
* 刪除舊的 ClearDB 資料庫。

使用 MySQL (ClearDB) 建立新的 Web 應用程式，或建立 MySQL 資料庫 (ClearDB) 時，您選擇的訂用帳戶會決定您將支付服務的方式。 使用 EA 訂用帳戶，對於在 Azure 入口網站中採購第三方服務，例如 ClearDB，我們不會進行封鎖。 將每季並以後付方式向 EA 訂用帳戶就財務承諾以外收費。 EA 客戶必須設定付款方式，例如信用卡，才能支付任何第三方 Marketplace 服務。

## <a name="where-can-i-see-the-charges-for-cleardb-resources-in-an-ea-subscription"></a>在哪裡可以查看 EA 訂用帳戶中 ClearDB 資源的費用？
針對直接 EA 客戶，可以在企業入口網站上看見 Azure Marketplace 費用。 請注意，所有 Marketplace 購買項目和使用均會每季並以後付方式就財務承諾以外收費。 EA 客戶需要直接付費給第三方服務提供者，並且能夠藉由啟用其 EA 帳戶的付款方式 (例如：信用卡) 來完成。

間接的 EA 客戶可以在企業版入口網站的 [管理訂用帳戶]  頁面上找到自己的 Azure Marketplace 訂用帳戶，但價格是隱藏的。 客戶應該連絡其 LSP 以了解相關的 Marketplace 費用資訊。

您的 EA Azure 註冊管理員可以管理對 Azure Marketplace 第三方服務的存取權。 他們可以透過市集的 [管理帳戶] 和企業入口網站中 [帳戶] 區段下的訂用帳戶，來停用或重新啟用對第三方購買項目的存取。

## <a name="who-do-i-contact-for-questions-about-my-bill-for-cleardb-services-in-my-ea-subscription"></a>若對我的 EA 訂用帳戶中 ClearDB 服務的帳單有問題，應該連絡誰？
如有任何關於自己 EA 註冊的計費問題，請連絡 [企業版客戶支援](http://aka.ms/AzureEntSupport) 。 EA 入口網站支援小組會回答您的問題或協助解決您的問題。

## <a name="more-information"></a>詳細資訊
[Azure Marketplace 常見問題集](https://azure.microsoft.com/marketplace/faq/)

