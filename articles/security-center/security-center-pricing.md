---
title: 升級為 Azure 資訊安全中心標準層以增強安全性 | Microsoft Docs
description: 這篇文章提供 Azure 資訊安全中心的價格資訊。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: 4d1364cd-7847-425a-bb3a-722cb0779f78
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/06/2018
ms.author: rkarlin
ms.openlocfilehash: 824362c5dab9f226aff84e75b6620221d3a9147f
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528472"
---
# <a name="upgrade-to-security-centers-standard-tier-for-enhanced-security"></a>升級為 Azure 資訊安全中心標準層以增強安全性
Azure 資訊安全中心提供統一的安全性管理和進階的威脅防護，保護 Azure、內部部署和其他雲端中執行的工作負載。 其提供了針對混合式雲端工作負載的可見性和控制能力、可降低威脅曝露度的主動防禦措施、還有智慧型偵測可幫助您跟上網路攻擊快速發展的腳步。

## <a name="pricing-tiers"></a>定價層
資訊安全中心提供兩個層級：

- 所有 Azure 訂用帳戶都會自動啟用**免費**層，並提供安全性原則、持續的安全性評估，以及可操作的安全性建議，以幫助您保護 Azure 資源。
- **標準**層將免費層的功能擴展到在私人雲端和其他公用雲端中執行的工作負載，為您的混合式雲端工作負載提供統一的安全性管理和威脅防護。 標準層也加入了進階威脅偵測功能，可使用內建的行為分析和機器學習來識別攻擊和零時差惡意探索、存取和應用程式控制，以減少曝露在網路攻擊和惡意軟體之下的程度等。 標準層的前 60 天免費。

如需詳細資訊，請參閱資訊安全中心[價格頁面](https://azure.microsoft.com/pricing/details/security-center/)。

## <a name="try-standard-free-for-60-days"></a>免費試用「標準」60 天
標準層的前 60 天免費。 在 60 天結束時，如果您選擇繼續使用服務，我們將自動開始針對使用量計費。

您可以將整個 Azure 訂用帳戶升級至標準層，該層會繼承訂用帳戶中的所有資源。

若要取得標準層：

1. 在 [資訊安全中心] 主功能表上選取 [安全性原則]。
2. 選取您要升級至「標準」的訂用帳戶。
3. 在 [安全性原則] 刀鋒視窗中，選取 [定價層]。
4. 選取 [標準版] 以進行升級。
5. 按一下 [檔案] 。

![安全性事件][1]

> [!NOTE]
> 若要啟用資訊安全中心的所有功能，您必須將標準版定價層套用至包含適用虛擬機器的訂用帳戶或資源群組。 設定工作區的定價不會啟用 Just-In-Time VM 存取、自動調整應用程式控制項，以及 Azure 資源的網路偵測。
>
>

## <a name="why-upgrade-to-standard"></a>為何要升級至標準？
資訊安全中心為您的混合式雲端工作負載提供增強的安全性與威脅防護，包括：

- **混合式安全性** – 取得您所有內部部署和雲端工作負載中安全性的統一檢視。 套用安全性原則，並持續評估您混合式雲端工作負載的安全性，以確保符合安全性標準。 從各種來源 (包括防火牆和其他合作夥伴解決方案) 收集、搜尋及分析安全性資料。
- **進階威脅偵測** - 利用進階分析和 Microsoft Intelligent Security Graph 勝過不斷進化的網路攻擊。  有效率的調控內建行為分析和機器學習服務，以發現攻擊和零時差惡意探索。 監視網路、機器和雲端服務中是否有傳入攻擊和侵入後活動。 使用互動式工具和內容相關威脅情報來簡化調查。
- **存取與應用程式控制** - 藉由套用依特定工作負載調整並由機器學習服務提供的白名單建議，來封鎖惡意程式碼和其他不想要的應用程式。 利用即時控制存取減少網路攻擊面，以管理 Azure VM 上的連接埠，大幅減少暴力密碼破解攻擊和其他網路攻擊。


## <a name="next-steps"></a>後續步驟
在本文中，已向您介紹資訊安全中心的價格。 若要深入了解標準層的增強式安全性和進階威脅防護，請參閱：

- [進階威脅偵測](security-center-threat-report.md)
- [Just-In-Time VM 存取控制](security-center-just-in-time.md)

<!--Image references-->
[1]: ./media/security-center-pricing/get-standard.png
