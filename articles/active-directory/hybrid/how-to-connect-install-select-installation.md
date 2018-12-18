---
title: Azure AD Connect：選取安裝類型 | Microsoft Docs
description: 本主題將逐步解說如何選取要用於 Azure AD Connect 的安裝類型
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: e4185b6b30bdd01492d9aae7ac38e18aaba7eef5
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306017"
---
# <a name="select-which-installation-type-to-use-for-azure-ad-connect"></a>選取要用於 Azure AD Connect 的安裝類型
Azure AD Connect 針對新安裝提供兩種安裝類型：快速和自訂。 本主題將協助您決定安裝時要使用的選項。

## <a name="express"></a>Express
[快速] 是最常見的選項，大約 90% 的全新安裝是使用此選項。 其設計目的是要提供適用於最常見客戶案例的組態。

此選項假設：

- 您有內部部署的單一 Active Directory 樹系。
- 您有可用來進行安裝的企業系統管理員帳戶。
- 您內部部署 Active Directory 中的物件少於 10 萬個。

您會獲得：

- 從內部部署到 Azure AD 的[密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md)，用於進行單一登入。
- 同步處理[使用者、群組、連絡人及 Windows 10 電腦](concept-azure-ad-connect-sync-default-configuration.md)的組態。
- 所有網域和所有 OU 中所有合格物件的同步處理。
- [自動升級](how-to-connect-install-automatic-upgrade.md)會啟用以確保您一律使用最新的可用版本。

在下列情況下，您仍然可以選擇使用 [快速]：

- 如果您不想要同步處理所有 OU，您仍然可以使用 [快速]，然後在最後一頁取消選取 [開始同步處理程序]。 接著，再次執行安裝精靈並變更[組態選項](how-to-connect-installation-wizard.md#customize-synchronization-options)中的 OU，然後啟用排定的同步處理。
- 您想要啟用 Azure AD Premium 的其中一個功能，例如密碼回寫。 請先透過快速安裝來完成初始安裝。 接著，再次執行安裝精靈並變更[組態選項](how-to-connect-installation-wizard.md#customize-synchronization-options)。

## <a name="custom"></a>自訂
自訂路徑所允許的選項比快速安裝多更多。 在上一節所述的快速組態無法代表您組織情況的所有案例中，都應該使用自訂路徑。

使用時機：

- 您無法存取 Active Directory 中的企業管理帳戶。
- 您有多個樹系，或是您未來打算同步處理多個樹系。
- 您樹系中有無法從 Connect 伺服器連線的網域。
- 您打算使用同盟或傳遞驗證來進行使用者登入。
- 您擁有的物件超過 10 萬個，並且需要使用完整的 SQL Server。
- 您打算使用群組型篩選，而不只是網域型或 OU 型篩選。

## <a name="upgrade-from-dirsync"></a>從 DirSync 升級
如果您目前使用的是 DirSync，則請依照[從 DirSync 升級](how-to-dirsync-upgrade-get-started.md)中的步驟來升級您現有的組態。 有兩個不同的升級選項：

- 就地升級：將 Connect 安裝在相同伺服器上。
- 平行部署：將 Connect 安裝在新伺服器上，但現有 DirSync 伺服器仍可運作。

## <a name="upgrade-from-azure-ad-sync"></a>從 Azure AD Sync 升級
如果您目前使用的是 Azure AD Sync，則可以依照從一個 Connect 版本升級到更新版本時的[相同步驟](how-to-upgrade-previous-version.md)操作。 有兩個不同的升級選項：

- 就地升級：將 Connect 安裝在相同伺服器上。
- 搖擺移轉：將 Connect 安裝在新伺服器上，但現有 Azure AD Sync 伺服器仍可運作。

## <a name="migrate-from-fim2010-or-mim2016"></a>從 FIM2010 或 MIM2016 移轉
如果您目前使用的是 Forefront Identity Manager 2010 或 Microsoft Identity Manager 2016 搭配 Azure AD Connector，則您的唯一選項是移轉。 請依照[搖擺移轉](how-to-upgrade-previous-version.md#swing-migration)所述的步驟操作。 在步驟中，請將所有提到 Azure AD Sync 的地方取代成 FIM2010/MIM2016。

## <a name="next-steps"></a>後續步驟
請依據您所選取要使用的選項，使用左邊的目錄來尋找含有詳細步驟的文章。
