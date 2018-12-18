---
title: Azure 資源健康狀態常見問題集 | Microsoft Docs
description: Azure 資源健康狀態的概觀
services: Resource health
documentationcenter: dev-center-name
author: BernardoAMunoz
manager: ''
editor: ''
ms.assetid: 85cc88a4-80fd-4b9b-a30a-34ff3782855f
ms.service: service-health
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Supportability
ms.date: 03/27/2018
ms.author: BernardoAMunoz
ms.openlocfilehash: 1e22c62c291909bb4079ebaf389cfbd15ae9d8d5
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42140613"
---
# <a name="azure-resource-health-faq"></a>Azure 資源健康狀態常見問題集
了解 Azure 資源健康狀態相關常見問題的答案。

## <a name="what-is-azure-resource-health"></a>何謂 Azure 資源健康狀態？
當 Azure 問題影響到您的資源健康狀態時，協助進行診斷並取得支援。 它會通知您資源的目前及過去的健康狀態，並協助您解決問題。 資源健康狀態會在您需要解決 Azure 服務問題時提供技術支援。  

## <a name="what-is-the-resource-health-intended-for"></a>資源健康狀態的目的為何？
一旦偵測到資源的問題之後，資源健康狀態便可協助您診斷根本原因。 它會協助減輕問題，並在您需要 Azure 服務問題的詳細說明時提供技術支援。

## <a name="what-health-checks-are-performed-by-resource-health"></a>資源健康狀態會執行哪些健康狀態檢查？
資源健康狀態會根據[資源類型](resource-health-checks-resource-types.md)執行各種檢查。 這些檢查旨在實作三種類型的問題︰ 
- 未規劃的事件，例如未預期的主機重新開機
- 規劃的事件，例如排程的主機作業系統更新
- 使用者動作觸發的事件，例如使用者重新啟動虛擬機器

## <a name="what-does-each-of-the-health-status-mean"></a>每個健全狀態所代表的意義為何？
有三個不同的健全狀態︰
- 可用︰Azure 平台沒有任何可能會影響此資源的已知問題
- 無法使用︰資源健康狀態偵測到會影響資源的問題
- 未知︰資源健康狀態無法判斷資源的健康情況，因為它已停止接收相關的資訊。 

## <a name="what-does-the-unknown-status-mean-is-something-wrong-with-my-resource"></a>不明狀態的意義為何？ 我的資源有什麼問題嗎？
當資源健康狀態停止接收特定資源的相關資訊時，就會將健康狀態設為未知。 儘管此狀態不是資源狀態的明確指示，如果您遇到問題，則可能表示 Azure 發生問題。

## <a name="how-can-i-get-help-for-a-resource-that-is-unavailable"></a>如何取得無法使用的資源說明？
您可以從 [資源健康狀態] 刀鋒視窗提交支援要求。 當資源因為平台事件而無法使用時，您無需 Microsoft 的支援合約即可將要求開啟。

## <a name="does-resource-health-differentiate-between-unavailability-cased-by-platform-problems-versus-something-i-did"></a>資源健康狀態是否會區分依平台問題歸類為無法使用的案例與我所做的事情？
是，當資源無法使用時，資源健康狀態會在下列其中一種類別內識別出根本原因： 
-   使用者啟動的動作
-   規劃的事件 
-   未規劃的事件

在入口網站中，會使用藍色通知圖示來顯示使用者啟動的動作，而規劃與未規劃的事件則會使用紅色警告圖示來顯示。 [資源健康狀態概觀](Resource-health-overview.md)中會提供更多詳細資料。  

## <a name="can-i-integrate-resource-health-with-my-monitoring-tools"></a>可以將資源健康狀態與我的監視工具進行整合嗎？
資源健康狀態是專為協助您診斷和解決會影響您資源的 Azure 服務問題所設計的服務。 雖然您可以使用資源健康狀態 API 以程式設計方式取得健康狀態，但我們建議您使用計量來監視您的資源。 一旦偵測到問題後，資源健康狀態可協助您判斷根本原因，並引導您完成動作來解決這些問題。 若要深入了解如何使用計量來檢查您的資源，請瀏覽 [Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)。

## <a name="where-do-i-find-resource-health"></a>哪裡可以找到資源健康狀態？
登入 Azure 入口網站之後，您可以使用多種方式來存取資源健康狀態：
- 瀏覽至您的資源。 在左側導覽中，選取 [資源健康狀態]。
- 移至 [Azure 服務健康狀態] 刀鋒視窗。  在左側導覽中，選取 [資源健康狀態]。
- 按一下入口網站右上角的問號，然後選取 [說明 + 支援]，即可開啟 [說明 + 支援] 刀鋒視窗。 當刀鋒視窗開啟之後，選取 [資源健康狀態]。

您也可以使用資源健康狀態 API，來取得資源的健康狀態相關資訊。

## <a name="is-resource-health-available-for-all-resource-types"></a>資源健康狀態是否適用於所有的資源類型？
您可以在[這裡](resource-health-checks-resource-types.md)找到透過資源健康狀態支援的健康情況檢查和資源類型清單。

## <a name="what-should-i-do-if-my-resource-is-showing-available-but-i-believe-it-is-not"></a>如果我的資源顯示無法使用，但我認為它可以使用，該怎麼辦？
檢查資源的健康情況時，您可以在健全狀態下面按一下 [報告不正確的健全狀態]。 提交報表之前，您可以選擇提供為何您認為目前健全狀態不正確的其他相關詳細資料。

## <a name="is-resource-health-available-for-all-azure-regions"></a>資源健康狀態是否適用於所有的 Azure 區域？ 
資源健康狀態可在所有 Azure 地區使用，除了下列區域︰
- 美國政府維吉尼亞州
- US Gov 愛荷華州
- 美國 DoD 東部
- 美國國防部中央
- 德國中部
- 德國東北部

## <a name="how-is-resource-health-different-from-azure-status-or-the-service-health-dashboard"></a>資源健康狀態和 Azure 狀態或服務健康狀態儀表板有何不同？
資源健康狀態可提供比 Azure 狀態或服務健康狀態儀表板更明確的資訊。

[Azure 狀態](https://status.azure.com)和服務健康狀態儀表板會通知您影響大規模客戶 (例如 Azure 區域) 的服務問題，而資源健康狀態會公開更多僅與特定資源相關的細微事件。 例如，如果主機意外重新啟動，資源健康狀態只會警示虛擬機器在該主機上執行的客戶。

請務必注意，為了讓您可完整看到影響資源的事件，資源健康狀態也會呈現諸如服務健康狀態儀表板中發佈的事件。

## <a name="do-i-need-to-activate-resource-health-for-each-resource"></a>是否需要啟動每個資源的資源健康狀態？
否，健康情況資訊適用於可透過資源健康狀態提供的所有資源類型。 

## <a name="do-we-need-to-enable-resource-health-for-my-organization"></a>我們是否需要為我的組織啟用資源健康狀態？
否。  不需進行任何安裝，即可在 Azure 入口網站內存取 Azure 資源健康狀態。

## <a name="is-resource-health-available-free-of-charge"></a>是否可免費使用資源健康狀態？
是。  Azure 資源健康狀態是免費的。

## <a name="what-are-the-recommendations-that-resource-health-provides"></a>資源健康狀態所提供的建議有哪些？
根據健康狀態，資源健康狀態會為您提供建議，旨在讓您減少疑難排解所花費的時間。 針對可用的資源，建議著重於如何解決客戶會發生的最常見問題。 如果資源是因為 Azure 未規劃事件而無法使用，則會著重於在復原程序期間以及之後協助您。 

## <a name="next-steps"></a>後續步驟

深入了解資源健康狀態：
-  [Azure 資源健康狀態概觀](Resource-health-overview.md)
-  [可透過 Azure 資源健康狀態使用的資源類型和健康檢查](resource-health-checks-resource-types.md)
