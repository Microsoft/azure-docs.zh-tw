---
title: 從 DirSync 和 Azure AD Sync 升級 |Microsoft Docs
description: 描述如何從 DirSync 和 Azure AD Sync 升級至 Azure AD Connect。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: bd68fb88-110b-4d76-978a-233e15590803
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7f4bf3e68c64281cb875c063597ab01eb53bbf78
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310297"
---
# <a name="upgrade-windows-azure-active-directory-sync-and-azure-active-directory-sync"></a>升級 Windows Azure Active Directory Sync 和 Azure Active Directory Sync
Azure AD Connect 是連接內部部署目錄與 Azure AD 和 Office 365 的最佳方式。 這是將 Azure AD Connect 從 Windows Azure Active Directory Sync (DirSync) 或 Azure AD Sync 升級的最佳時機，因為這些工具現在已淘汰，而且從 2017 年 4 月 13 日起不再提供支援。

已淘汰的兩個身分識別同步處理工具會提供給單一樹系客戶 (DirSync)，以及多樹系和其他進階客戶 (Azure AD Sync)。 這些較舊的工具已經取代為適用於所有案例的單一解決方案︰Azure AD Connect。 它提供新的功能、增強功能和新案例的支援。 若要能夠繼續同步處理內部部署身分識別資料至 Azure AD 和 Office 365，強烈建議您升級至 Azure AD Connect。 Microsoft 不保證舊版本在 2017 年 12 月 31 日之後能夠正常運作。

最新版的 DirSync 發行於 2014 年 7 月，而最新版的 Azure AD Sync 發行於 2015 年 5 月。

## <a name="what-is-azure-ad-connect"></a>何謂 Azure AD Connect
Azure AD Connect 是 DirSync 和 Azure AD Sync 的後續產品。它結合了兩者支援的所有案例。 您可以在 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)中進一步了解。

## <a name="deprecation-schedule"></a>淘汰排程
| 日期 | 註解 |
| --- | --- |
| 2016 年 4 月 13 日 |Windows Azure Active Directory Sync (“DirSync”) 和 Microsoft Azure Active Directory Sync (“Azure AD Sync”) 已宣布淘汰。 |
| 2017 年 4 月 13 日 |支援結束。 客戶必須先升級至 Azure AD Connect 才能開啟支援案例。 |
|2017 年 12 月 31 日|Azure AD 已不再接受來自 Windows Azure Active Directory Sync ("DirSync") 和 Microsoft Azure Active Directory Sync ("Azure AD Sync") 的通訊。

## <a name="how-to-transition-to-azure-ad-connect"></a>如何轉換為 Azure AD Connect
如果您正在執行 DirSync，有兩種方式可以升級︰就地升級和平行部署。 對大多數的客戶，以及如果您擁有最新的作業系統和少於 50,000 個物件，建議採用就地升級。 在其他情況下，建議執行平行部署，DirSync 組態會移至執行 Azure AD Connect 的新伺服器。

| 解決方法 | 案例 |
| --- | --- |
| [從 DirSync 升級](how-to-dirsync-upgrade-get-started.md) |<li>如果您有已在執行中的現有 DirSync 伺服器。</li> |
| [從 Azure AD Sync 升級](how-to-upgrade-previous-version.md) |<li>如果您要從 Azure AD Sync 移動。</li> |

如果您想要了解如何執行就地升級以從 DirSync 升級到 Azure AD Connect，請觀看此第 9 頻道影片︰

> [!VIDEO https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Azure-Active-Directory-Connect-in-place-upgrade-from-legacy-tools/player]
>
>

## <a name="faq"></a>常見問題集
**問：我是使用 Connect，卻收到來自 Azure 團隊及/或來自 Office 365 郵件中心的訊息。**  
這項通知也會傳送給使用 Azure AD Connect 組建編號 1.0.\*.0 (使用 1.1 發行前版本) 的客戶。 Microsoft 建議客戶使用最新的 Azure AD Connect 版本。 1.1 版引進的[自動升級](how-to-connect-install-automatic-upgrade.md)功能可讓您持續輕鬆擁有最新的 Azure AD Connect 安裝版本。

**問︰DirSync/Azure AD Sync 會在 2017 年 4 月 13 日停止運作嗎？**  
DirSync/Azure AD Sync 在 2017 年 4 月 13 日會繼續運作。  不過，Azure AD 在 2017 年 12 月 31 日之後，將不再接受來自 DirSync/Azure AD Sync 的通訊。

**問：哪些 DirSync 版本可以進行升級？**  
目前使用的任何 DirSync 版本皆支援升級。 

**問：什麼是 FIM/MIM 的 Azure AD 連接器？**  
FIM/MIM 的 Azure AD 連接器**尚未**宣布淘汰。 它目前在 **功能凍結**狀態；不會新增任何功能，而且不會收到任何錯誤修正。 Microsoft 建議使用它的客戶規劃從它移至 Azure AD Connect。 強烈建議不要使用它啟動任何新的部署。 此連接器會在未來宣布淘汰。

## <a name="additional-resources"></a>其他資源
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
