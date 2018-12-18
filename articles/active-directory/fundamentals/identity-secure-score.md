---
title: 什麼是 Azure AD 中的身分識別安全分數？ - 預覽 | Microsoft Docs
description: 了解如何使用身分識別安全分數來改善 Azure AD 租用戶的安全性狀態。
services: active-directory
keywords: 身分識別安全分數, Azure AD, 對公司資源的安全存取
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.component: fundamentals
ms.topic: overview
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/19/2018
ms.author: markvi
ms.reviewer: nigu
ms.openlocfilehash: a9bb22cae2e9fcd607a81d442f4fb51f12c43454
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224132"
---
# <a name="what-is-the-identity-secure-score-in-azure-ad---preview"></a>什麼是 Azure AD 中的身分識別安全分數？ - 預覽

您的 Azure AD 租用戶有多安全？ 如果您不知道如何回答這個問題，請閱讀本文以了解身分識別安全分數如何協助您監視和改善身分識別安全性狀態。 

## <a name="what-is-an-identity-secure-score"></a>什麼是身分識別安全分數？

身分識別安全分數是介於 1 和 248 之間的數字，可作為指標來指出您有多符合 Microsoft 對於安全性的最佳做法建議。


![安全分數](./media/identity-secure-score/01.png)



此分數可協助您：

- 客觀地測量身分識別安全性狀態

- 規劃如何改善身分識別安全性

- 檢閱改善是否成功 


您可以在身分識別安全分數儀表板上存取分數和相關資訊。 在此儀表板上，您可以找到：

- 分數

    ![安全分數](./media/identity-secure-score/02.png)

- 比較圖表

    ![安全分數](./media/identity-secure-score/03.png)

- 趨勢圖表

    ![安全分數](./media/identity-secure-score/04.png)

- 身分識別安全性最佳做法的清單。 

    ![安全分數](./media/identity-secure-score/05.png)


藉由遵循改善動作，您可以：

- 改善安全性狀態和分數。
 
- 善用 Microsoft 的身分識別功能。 



## <a name="how-do-i-get-my-secure-score"></a>如何取得安全分數？

所有版本的 Azure AD 都有提供身分識別安全分數。

若要存取分數，請移至 [Azure AD 概觀儀表板](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/IdentitySecureScore)。



## <a name="how-does-it-work"></a>運作方式

每隔 48 小時，Azure 就會查看安全性組態，並將您的設定與建議的最佳做法進行比較。 根據這項評估的結果，計算出租用戶的新分數。 這表示最久可能需要 48 小時後，分數中才會反映出您所進行的組態變更。 

系統會根據 Azure AD 組態測量每項建議。 如果您使用第三方產品來啟用最佳做法建議，則可在改善動作的設定中指出這一點。

![安全分數](./media/identity-secure-score/07.png)


此外，您也可以選擇將不適用於環境的建議設定為加以忽略。 遭到忽略的建議便不會列入分數的計算中。 
 
![安全分數](./media/identity-secure-score/06.png)



## <a name="how-does-it-help-me"></a>其如何提供協助？

此安全分數可協助您：

- 客觀地測量身分識別安全性狀態

- 規劃如何改善身分識別安全性

- 檢閱改善是否成功



## <a name="what-you-should-know"></a>您應該知道的事情

### <a name="who-can-use-the-identity-secure-score"></a>誰可以使用身分識別安全分數？

身分識別安全分數可供下列角色使用：

- 全域管理員
- 安全性系統管理員 
- 安全性讀取者 

### <a name="what-does-not-scored-mean"></a>[不計分] 是什麼意思？

標示為 [不計分] 的動作是可在組織中執行，但由於未連接到工具而不會列入計分的動作。 因此，您仍可改善安全性，但目前不會因為這些動作而獲得分數。

### <a name="how-often-is-my-score-updated"></a>分數多久會更新一次？

分數會每天計算一次 (大約在太平洋標準時間上午 1:00)。 如果您變更了測量的動作，分數會在隔天自動更新。 最久需要 48 小時的時間，分數中才會反映出該項變更。


### <a name="my-score-changed-how-do-i-figure-out-why"></a>分數變了。 要如何找出原因？

在[安全分數入口網站](https://securescore.microsoft.com/#!/score)上的 [分數分析器] 頁面上，按一下特定日期的某個資料點，然後向下捲動即可查看當天已完成和未完成的動作，以找出變更之處。

### <a name="does-the-secure-score-measure-my-risk-of-getting-breached"></a>安全分數是否會測量遭到入侵的風險？

不會。 安全分數不表示您一定有多大的可能性會遭到入侵。 它只表示您所採用的功能有多大程度可以抵銷遭到入侵的風險。 沒有服務可以保證您一定不會遭到入侵，請絕對不要將安全分數當作保證。

### <a name="how-should-i-interpret-my-score"></a>如何解讀分數？

設定建議的安全性功能或執行安全性相關工作 (例如，讀取報告) 就會獲得分數。 某些動作就算完成一部分也會獲得分數，例如為使用者啟用 Multi-Factor Authentication (MFA)。 安全分數會直接代表所使用的 Microsoft 安全性服務。 記得，請務必在安全性與易用性之間取得平衡。 所有安全性控制都會對使用者造成影響。 對使用者影響較低的控制應該就不會對使用者的日常作業造成什麼影響。

若要查看過去的分數，請移至[安全分數入口網站](https://securescore.microsoft.com/#!/score)上的 [分數分析器] 頁面。 選擇特定日期，即可查看當天啟用了哪些控制，以及這些控制各自獲得的分數。


### <a name="how-does-the-identity-secure-score-relate-to-the-office-365-secure-score"></a>身分識別安全分數與 Office 365 安全分數有何關聯？ 

[Office 365 安全分數](https://docs.microsoft.com/office365/securitycompliance/office-365-secure-score)即將遷移至具有五個不同分數的彙總內：

- 身分識別

- 資料

- 裝置

- 基礎結構

- 應用程式

身分識別安全分數代表 Office 365 安全分數的身分識別部分。 這表示對於身分識別安全分數與 Office 365 身分識別分數所提出的建議會相同。 


## <a name="next-steps"></a>後續步驟

如果您想要觀看有關 Office 365 安全分數的影片，請按一下[這裡](https://www.youtube.com/watch?v=jzfpDJ9Kg-A)。
 
