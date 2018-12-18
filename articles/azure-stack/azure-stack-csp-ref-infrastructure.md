---
title: Azure Stack 雲端服務提供者的使用量報告基礎結構 | Microsoft Docs
description: 當雲端服務提供者 (CSP) 服務的租用戶產生使用量時，Azure Stack 中的基礎結構可用來追蹤此用量並轉送至 Azure。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2018
ms.author: sethm
ms.reviewer: alfredo
ms.openlocfilehash: 9526385eaea8a88f0c22e6420ba39a33f7166f96
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45633719"
---
## <a name="usage-reporting-infrastructure-for-cloud-service-providers"></a>適月於雲端服務提供者的使用方式報告基礎結構

Azure Stack 包含使用量發生時可加以追蹤並轉送至 Azure 所需的基礎結構。 在 Azure 中，Azure Commerce 會處理使用量資料，並向適當的 Azure 訂用帳戶收取使用量費用。 此作業的運作方式與在全域 Azure 雲端中監視使用量追蹤一樣。

您應該注意到全域 Azure 和 Azure Stack 兩者之間有某些概念是一致的。 Azure Stack 具有本機訂用帳戶，可履行與 Azure 訂用帳戶類似的角色。 本機訂用帳戶僅在本機上有效。 當使用量轉送到 Azure 時，本機訂用帳戶會對應至 Azure 訂用帳戶。

Azure Stack 具有本機使用量計量。 本機使用量會對應到 Azure Commerce 中使用的計量。 不過，計量識別碼並不相同。 相較於 Microsoft 用來計費的計量，本機可提供更多計量。

Azure Stack 和 Azure 的服務計價方式有一些差異。 例如，不同於 Azure，在 Azure Stack 中，VM 僅根據 vCore/小時來計費，所有 VM 系列的費率均相同。 原因在於全域 Azure 中的不同價格會反映不同的硬體。 在 Azure Stack 中，客戶會提供硬體，因此沒有理由針對不同的 VM 類別收取不同的費率。

您可以使用與 Azure 服務相同的方式，在合作夥伴中心內了解 Commerce 中使用的 Azure Stack 計量及其價格：

1. 在合作夥伴中心內，移至 [儀表板功能表] > [定價和供應項目]。
2. 在 [依使用量計費的服務] 下，選取 [目前]。
3. 開啟 [全域 CSP 中的 Azure 價目表] 試算表。
4. 篩選 [區域 = Azure Stack]。

## <a name="usage-and-billing-error-codes"></a>使用量與帳單錯誤碼

將租用戶新增至註冊時，可能會遇到下列錯誤訊息。

| Error                           | 詳細資料                                                                                                                                                                                                                                                                                                                           | 註解                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RegistrationNotFound            | 找不到所提供的註冊。 請確定已正確提供下列資訊：<br>1.訂用帳戶識別碼 (提供的值：訂用帳戶識別碼)，<br>2.資源群組 (提供的值：資源群組)，<br>3.註冊名稱 (提供的值：註冊名稱)。                             | 指向初始註冊的資訊不正確時，通常會發生這個錯誤。 如果您需要確認註冊的資源群組與名稱，可以在 Azure 入口網站中找到，方法是列出所有的資源。 如果您找到多個註冊資源，請在屬性中查看 CloudDeploymentID，然後選取其 CloudDeploymentID 與您雲端中 CloudDeploymentID 相符的註冊。 若要尋找 CloudDeploymentID，您可以在 Azure Stack 上使用此 PowerShell：<br>`$azureStackStampInfo = Invoke-Command -Session $session -ScriptBlock { Get-AzureStackStampInformation }` |
| BadCustomerSubscriptionId       | 所提供的「客戶訂用帳戶識別碼」和「註冊名稱」訂用帳戶識別碼並非由相同的 Microsoft 雲端服務提供者所擁有。 請確認客戶訂用帳戶識別碼是否正確。 若問題持續發生，請連絡支援服務。 | 當客戶訂用帳戶是 CSP 訂用帳戶，但它所積存到的 CSP 夥伴與初始註冊中所用訂用帳戶所積存到的 CSP 夥伴不同時，就會發生此錯誤。 進行這項檢查的目的，是為了避免造成向並非負責所用之 Azure Stack 的 CSP 夥伴收費的情況。                                                                                                                                                                                                                                                                          |
| InvalidCustomerSubscriptionId   | 「客戶訂用帳戶識別碼」無效。 請確定所提供的 Azure 訂用帳戶有效。                                                                                                                                                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| CustomerSubscriptionNotFound    | _registration 名稱下找不到「客戶訂用帳戶識別碼」。 請確定所使用的是有效的 Azure 訂用帳戶，且是使用 PUT 作業將訂用帳戶識別碼新增至註冊。                                                   | 當您嘗試確認租用戶已新增至訂用帳戶，但找不到與註冊相關聯的客戶訂用帳戶時，就會發生此錯誤。 客戶尚未新增至註冊，或寫入的訂用帳戶識別碼不正確。                                                                                                                                                                                                                                                                                                                                |
| UnauthorizedCspRegistration     | 所提供的「註冊名稱」並未核准使用多重租用。 傳送電子郵件給 azstCSP@microsoft.com，並包含您註冊使用的註冊名稱、資源群組和訂用帳戶識別碼。                                                                                    | 註冊必須先獲得 Microsoft 核准使用多重租用後，您才可以開始將租用戶新增至註冊。 請參閱本文件中的＜註冊租用戶＞一節，以取得進一步說明。                                                                                                                                                                                                                                                                                                                                                                                                             |
| CustomerSubscriptionsNotAllowed | 已中斷連線的客戶不支援客戶訂用帳戶作業。 若要使用這項功能，請使用「隨用即付」授權重新註冊。                                                                                                                                                                    | 您嘗試新增租用戶的註冊為「容量註冊」，也就是當您建立註冊時，會使用 BillingModel Capacity 參數。 只有「隨用即付」註冊允許新增租用戶。 您必須使用 BillingModel PayAsYouUse 參數來重新註冊。                                                                                                                                                                                                                                                                                          |
| InvalidCSPSubscription          | 所提供的「客戶訂用帳戶識別碼」並非有效的 CSP 訂用帳戶。 請確定所提供的 Azure 訂用帳戶有效。                                                                                                                                                        | 這很有可能是因為客戶訂用帳戶輸入錯誤。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| MetadataResolverBadGatewayError | 其中一個上游伺服器傳回未預期的錯誤。 請稍後再試。 若問題持續發生，請連絡支援服務。                                                                                                                                                                                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

## <a name="terms-used-for-billing-and-usage"></a>用於帳單及使用量的字詞

下列字詞和概念會用於 Azure Stack 中的使用量和計費：

| 詞彙 | 定義 |
| --- | --- |
| 直接 CSP 夥伴 | 直接的「雲端解決方案提供者 (CSP)」夥伴會直接收到來自 Microsoft 的 Azure 和 Azure Stack 使用量發票，並直接向客戶收費。 |
| 間接 CSP | 間接轉銷商會與間接提供者 (也稱為「散發者」) 合作。 轉銷商招募終端客戶；間接提供者則會保有與 Microsoft 的帳務關係、管理客戶帳單，並提供其他服務 (例如產品支援)。 |
| 終端客戶 | 終端客戶是企業和政府機關，擁有 Azure Stack 上執行的應用程式和其他工作負載。 |

## <a name="next-steps"></a>後續步驟

 - 若要深入了解 CSP 方案，請參閱[雲端解決方案提供者方案](https://partner.microsoft.com/solutions/microsoft-cloud-solutions)。
 - 若要深入了解如何取出 Azure Stack 的資源使用量資訊，請參閱 [Azure Stack 中的使用量與帳單](azure-stack-billing-and-chargeback.md)。
