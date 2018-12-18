---
title: Azure AD 啟用密碼回寫
description: 在本教學課程中，您會在 Azure AD Connect 中啟用密碼回寫，將雲端起始的密碼變更傳回至內部部署 AD。
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: tutorial
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: e91ca2ff438e3ae9e9b3f46e5a8db89f17fb1c8e
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295116"
---
# <a name="tutorial-enabling-password-writeback"></a>教學課程：啟用密碼回寫

在本教學課程中，您將為混合式環境啟用密碼回寫。 密碼回寫可用來將 Azure Active Directory (Azure AD) 中的密碼變更同步處理回您的內部部署 Active Directory Domain Services (AD DS) 環境。 密碼回寫會在 Azure AD Connect 中啟用，以提供將 Azure AD 中的密碼變更傳回至現有內部部署目錄的安全機制。 您可以在[什麼是密碼回寫](concept-sspr-writeback.md)一文中進一步了解密碼回寫的內部運作方式。

> [!div class="checklist"]
> * 在 Azure AD Connect 中啟用密碼回寫選項
> * 在自助式密碼重設 (SSPR) 中啟用密碼回寫選項

## <a name="prerequisites"></a>必要條件

* 能夠存取可運作且至少已被指派試用版授權的 Azure AD 租用戶。
* 在 Azure AD 租用戶中具有全域系統管理員權限的帳戶。
* 設定為執行最新版 [Azure AD Connect](../hybrid/how-to-connect-install-express.md) 的現有伺服器。
* 已完成先前的自助式密碼重設 (SSPR) 教學課程。

## <a name="enable-password-writeback-option-in-azure-ad-connect"></a>在 Azure AD Connect 中啟用密碼回寫選項

若要啟用密碼回寫，我們必須先從 Azure AD Connect 安裝所在的伺服器啟用此功能。

1. 若要設定和啟用密碼回寫，請登入 Azure AD Connect 伺服器，然後啟動 **Azure AD Connect** 設定精靈。
2. 在 [歡迎] 頁面上，選取 [設定]。
3. 在 [其他工作] 頁面上，選取 [自訂同步處理選項]，然後選取 [下一步]。
4. 在 [連線到 Azure AD] 頁面上，輸入全域管理員認證，然後選取 [下一步]。
5. 在 [連線目錄] 和 [網域/OU] 篩選頁面上，選取 [下一步]。
6. 在 [選用功能] 頁面上，選取 [密碼回寫] 旁邊的方塊，然後選取 [下一步]。
7. 在 [準備好設定] 頁面上，選取 [設定]，然後等待程序完成。
8. 看到設定完成時，選取 [結束]。

## <a name="enable-password-writeback-option-in-sspr"></a>在 SSPR 中啟用密碼回寫選項

在 Azure AD Connect 中啟用密碼回寫功能，只完成了必要程序的一半。 允許 SSPR 使用密碼回寫，才能完成據以讓變更或重設其密碼的使用者在內部部署設定該密碼的迴圈。

1. 使用全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至 **Azure Active Directory**，按一下 [密碼重設]，然後選擇 [內部部署整合]。
3. 將 [將密碼寫回至內部部署目錄] 的選項設定為 [是]。
4. 將 [允許使用者在不重設密碼的情況下解除鎖定帳戶] 的選項設定為 [是]。
5. 按一下 [儲存] 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已針對自助式密碼重設啟用密碼回寫。 請將 Azure 入口網站視窗保持開啟，並繼續進行下一個教學課程，以設定自助式密碼重設的其他相關設定，再推出試驗的解決方案。

> [!div class="nextstepaction"]
> [在 Windows 登入畫面中啟用 SSPR](tutorial-sspr-windows.md)
