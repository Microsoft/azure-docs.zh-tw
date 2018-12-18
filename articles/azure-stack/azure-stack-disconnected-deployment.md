---
title: 適用於 Azure Stack 整合式系統的 Azure 中斷連線部署決策 | Microsoft Docs
description: 針對多節點 Azure Stack 的 Azure 連線部署，決定部署規劃決策。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/01/2018
ms.author: jeffgilb
ms.reviewer: wfayed
ms.openlocfilehash: 4574b140e2e17462a5ff696b913bb4ef7bcb0ad0
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39412751"
---
# <a name="azure-disconnected-deployment-planning-decisions-for-azure-stack-integrated-systems"></a>適用於 Azure Stack 整合式系統的 Azure 中斷連線部署規劃決策
在您決定[如何將 Azure Stack 整合到您的混合式雲端環境](azure-stack-connection-models.md)之後，即可接著完成您的 Azure Stack 部署決策。

使用與 Azure 中斷連線的部署選項，您可以在沒有網際網路連線的情況下，部署和使用 Azure Stack。 不過，若使用已中斷連線的部署，您會受限於 AD FS 身分識別儲存和以容量為基礎的計費模型。 

選擇此選項的前提是您：
- 具有安全性或其他限制，而這些限制讓您必須在未連線到網際網路的環境中部署 Azure Stack。
- 想要讓資料 (包括使用量資料) 無法傳送到 Azure。
- 想要純粹使用 Azure Stack 作為部署到您公司內部網路的私人雲端解決方案，而且您對於混合式案例沒有興趣。

> [!TIP]
> 有時候，這種環境類型也稱為「潛水艇案例」。

已中斷連線的部署不完全表示您稍後無法在混合式租用戶 VM 案例中，將 Azure Stack 執行個體連線到 Azure。 這表示您無法在部署期間連線至 Azure，或您不想使用 Azure Active Directory 作為您的身分識別儲存。

## <a name="features-that-are-impaired-or-unavailable-in-disconnected-deployments"></a>中斷連線部署中受損或無法使用的功能 
依照設計，Azure Stack 在連線至 Azure 時的效能最佳，所以請注意，有一些特性和功能在中斷連線模式下會受損或完全無法使用。 

|功能|中斷連線模式的影響|
|-----|-----|
|使用 DSC 擴充功能設定 VM 後續部署的 VM 部署|受損 - DSC 擴充功能會依賴網際網路取得最新 WMF。|
|使用 Docker 擴充功能執行 Docker 命令的 VM 部署|受損 – Docker 會檢查網際網路中的最新版本，而且這項檢查會失敗。|
|Azure Stack 入口網站中的文件連結|無法使用 – 使用網際網路 URL 的連結 (例如「提供意見反應」、「說明」、「快速入門」等) 將無法運作。|
|參考線上補救指南的警示補救/緩和方法|無法使用 – 任何使用網際網路 URL 的警示補救連結將無法運作。|
|Marketplace – 能夠直接從 Azure Marketplace 選取及新增資源庫套件|受損 – 當您以中斷連線模式 (沒有任何網際網路連線) 中部署 Azure Stack 時，您無法透過 Azure Stack 入口網站下載 Marketplace 項目。 不過，您可使用 [Marketplace 摘要整合工具](https://docs.microsoft.com/azure/azure-stack/azure-stack-download-azure-marketplace-item#download-marketplace-items-in-a-disconnected-or-a-partially-connected-scenario-with-limited-internet-connectivity)將 Marketplace 項目下載至具有網際網路連線的電腦，再將其傳輸至 Azure Stack 環境。|
|使用 Azure Active Directory 同盟帳戶來管理 Azure Stack 部署|無法使用 – 這項功能需要連線到 Azure。 必須改為使用具有本機 Active Directory 執行個體的 AD FS。|
|應用程式服務|受損 - WebApps 可能需要存取網際網路以取得更新的內容。|
|命令列介面 (CLI)|受損 – CLI 降低了服務準則在驗證和佈建方面的功能。|
|Visual Studio – Cloud Discovery|受損 – Cloud Discovery 會探索不同的雲端，或完全無法運作。|
|Visual Studio – AD FS|受損 – 只有 Visual Studio Enterprise 支援 AD FS。
遙測|無法使用 – Azure Stack 的遙測資料，以及相依於遙測資料的任何第三方資源庫套件。|
|憑證|無法使用 – 在 HTTPS 的內容中，憑證撤銷清單 (CRL) 和線上憑證狀態通訊協定 (OSCP) 服務都需要網際網路連線能力。|
|金鑰保存庫|受損 – Key Vault 的常見使用案例就是讓應用程式在執行階段讀取祕密。 為此目的，應用程式在目錄中需要一個服務主體。 在 Azure Active Directory 中，預設允許一般使用者 (非系統管理員) 新增服務主體。 在 AD (使用 ADFS) 中，則不允許一般使用者這麼做。 這會在端對端體驗中造成困難，因為使用者一律必須透過目錄管理員新增任何應用程式。| 

## <a name="learn-more"></a>深入了解
- 如需使用案例、購買、合作夥伴和 OEM 硬體廠商的詳細資訊，請參閱 [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) 產品頁面。
- 如需 Azure Stack 整合系統的藍圖和地理可用性的詳細資訊，請參閱白皮書：[Azure Stack：Azure 的延伸模組](https://azure.microsoft.com/resources/azure-stack-an-extension-of-azure/)。 
- 若要深入了解 Microsoft Azure Stack 套件和定價，請[下載此 .pdf](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf)。 

## <a name="next-steps"></a>後續步驟
[資料中心網路整合](azure-stack-network.md)