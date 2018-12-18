---
title: 管理 Azure Stack 中的更新概觀 | Microsoft Docs
description: 深入了解 Azure Stack 整合系統的更新管理。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 9b0781f4-2cd5-4619-a9b1-59182b4a6e43
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2018
ms.author: mabrigg
ms.openlocfilehash: 67f363d14489340755251369b422475032d1e671
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222491"
---
# <a name="manage-updates-in-azure-stack-overview"></a>在 Azure Stack 中管理更新概觀

「適用於：Azure Stack 整合系統」

適用於 Azure Stack 整合式系統的 Microsoft 更新套件，通常大約會在每個月的第四個星期二發行。 請向您的 OEM 洽詢其特定通知程序，以確保更新通知觸達貴組織。 如需尚在有效支援期間之版本的相關資訊，您也可以在 [概觀] > [版本資訊] 底下查看此文件庫。 

每次發行的 Microsoft 軟體更新均以單一更新封裝的形式提供。 身為 Azure Stack 操作員，您可以從管理員入口網站匯入及安裝這些更新套件，並監視這些更新套件的安裝進度。 

原始設備製造商 (OEM) 硬體廠商也會發行更新，例如驅動程式和韌體更新。 雖然您的 OEM 硬體廠商會以個別套件的形式提供提供這些更新，但它們會使用與 Microsoft 更新套件中更新套件相同的匯入、安裝及管理方式來匯入、安裝及管理。

為了讓系統獲得支援，您必須將 Azure Stack 更新為特定版本等級。 務必檢閱 [Azure Stack 服務原則](azure-stack-servicing-policy.md)。

> [!NOTE]
> 您無法將 Azure Stack 更新封裝套用於 Azure Stack 開發套件。 更新封裝專為整合系統所設計。 如需資訊，請參閱[重新部署 ASDK](https://docs.microsoft.com/azure/azure-stack/asdk)。

## <a name="the-update-resource-provider"></a>更新資源提供者

Azure Stack 包含協調 Microsoft 軟體應用程式更新的更新資源提供者。 對於有實體主機、Service Fabric 應用程式和執行階段，以及所有基礎結構虛擬機器和其相關聯服務，這個資源提供者可確保均套用更新。

在更新安裝中，更新程序以 Azure Stack 中的各種子系統 (例如，實體主機和基礎結構虛擬機器) 為目標時，您可以檢視高階狀態。

## <a name="plan-for-updates"></a>規劃更新

強烈建議您向使用者通知任何維護工作，並且盡可能將一般的維護期間安排在非上班時間。 維護作業可能會影響租用戶工作負載和入口網站作業。


- 開始安裝此更新之前，請先執行 [Test-AzureStack](azure-stack-diagnostic-test.md) 與下列參數來驗證 Azure Stack 的狀態，並解決所發現的一切運作問題，包括所有警告和失敗。 也請檢閱作用中警示，並將所有需要採取動作的警示解決。  

  ```PowerShell
  Test-AzureStack -Include AzsControlPlane, AzsDefenderSummary, AzsHostingInfraSummary, AzsHostingInfraUtilization, AzsInfraCapacity, AzsInfraRoleSummary, AzsPortalAPISummary, AzsSFRoleSummary, AzsStampBMCSummary
  ``` 

## <a name="using-the-update-tile-to-manage-updates"></a>[更新] 圖格可用來管理更新
您要從系統管理員入口網站管理更新。 身為 Azure Stack 操作員，您可以使用儀表板中的 [更新] 圖格進行下列操作：

- 檢視重要資訊，例如目前的版本。
- 安裝更新，並監視進度。
- 檢閱先前安裝之更新本身的更新歷程記錄。
 
## <a name="determine-the-current-version"></a>判斷目前版本

[更新] 圖格會顯示目前的 Azure Stack 版本。 您可以在系統管理員入口網站中使用下列其中一種方法移至 [更新] 圖格：

- 在儀表板上，在 [更新] 圖格檢視目前的版本。
 
   ![預設儀表板上的 [更新] 圖格](./media/azure-stack-updates/image1.png)
 
- 在 [區域管理] 圖格上，按一下區域名稱。 在 [更新] 圖格中檢視目前的版本。

## <a name="next-steps"></a>後續步驟

- [Azure Stack 服務原則](azure-stack-servicing-policy.md) 
- [Azure Stack 中的區域管理](azure-stack-region-management.md)     


