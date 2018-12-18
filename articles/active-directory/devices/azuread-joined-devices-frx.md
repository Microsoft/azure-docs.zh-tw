---
title: 在初次執行期間向 Azure AD 加入新的 Windows 10 裝置 | Microsoft Docs
description: 本主題說明使用者如何在初次執行體驗期間設定加入 Azure AD。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 06a149f7-4aa1-4fb9-a8ec-ac2633b031fb
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/25/2018
ms.author: markvi
ms.reviewer: jairoc
ms.openlocfilehash: eaf0b3e3b607145598660dbb64cadd5a277360cb
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046317"
---
# <a name="tutorial-join-a-new-windows-10-device-with-azure-ad-during-a-first-run"></a>教學課程：在初次執行期間向 Azure AD 加入新的 Windows 10 裝置

使用 Azure Active Directory (Azure AD) 中的裝置管理，您可以確保使用者會從符合安全性與合規性之標準的裝置來存取您的資源。 如需詳細資訊，請參閱 [Azure Active Directory 中的裝置管理簡介](overview.md)。

使用 Windows 10，您可以在初次執行體驗 (FRX) 期間將新的裝置加入 Azure AD。  
這可讓您將壓縮包裝的裝置散發給您的員工或學生。

如果您在裝置上安裝了 Windows 10 專業版或 Windows 10 企業版，體驗則預設為公司所擁有裝置的安裝程序。

在 Windows「全新體驗」中，不支援加入內部部署 Active Directory (AD) 網域的功能。 如果您打算要在安裝期間讓電腦加入 AD 網域，應該要選取 [使用本機帳戶設定 Windows] 連結。 然後，您就可以透過自己電腦上的設定來加入網域。
 
在本教學課程中，您將了解如何在 FRX 期間將裝置加入 Azure AD：
 > [!div class="checklist"]
> * 必要條件
> * 加入裝置
> * 驗證

## <a name="prerequisites"></a>必要條件

若要加入 Windows 10 裝置，必須設定 Device Registration Service，才能註冊裝置。 除了擁有在 Azure AD 租用戶中加入裝置的權限之外，您所擁有的已註冊裝置必須少於已設定的最大值。 如需詳細資訊，請參閱[設定裝置設定](device-management-azure-portal.md#configure-device-settings)。

此外，如果您的租用戶已同盟，則您的識別提供者「必須」支援 WS-Fed 和 WS-Trust 使用者名稱/密碼端點。 這可以是 1.3 或 2005 版。 將裝置加入 Azure AD 及使用密碼登入裝置時，都需要此通訊協定支援。

## <a name="joining-a-device"></a>加入裝置

**若要在 FRX 期間將 Windows 10 裝置加入 Azure AD：**


1. 當您開啟新裝置，並啟動安裝程序時，您應該會看到 [正在準備] 的訊息。 請依照提示來設定您的裝置。

2. 首先，自訂您的地區及語言， 然後接受 Microsoft 軟體授權條款。
 
    ![自訂您的地區](./media/azuread-joined-devices-frx/01.png)

3. 選取您要用來連線到網際網路的網路。

4. 按一下 [此裝置屬於我的組織]。 

    ![[誰是這部電腦的擁有者] 畫面](./media/azuread-joined-devices-frx/02.png)

5. 輸入組織提供給您的認證，然後按一下 [登入]。

    ![登入畫面](./media/azuread-joined-devices-frx/03.png)

6. 您的裝置會在 Azure AD 中找出相符的租用戶。 如果您位於同盟網域，系統會將您重新導向至內部部署安全性權杖服務 (STS) 伺服器，例如 Active Directory 同盟服務 (AD FS)。

7. 如果您是非同盟網域的使用者，請直接在裝載 Azure AD 的頁面上輸入您的認證。 

8. 系統會提示您進行 Multi-Factor Authentication 挑戰。 
 
9. Azure AD 會確認是否需要在行動裝置管理中註冊。

10. Windows 會在 Azure AD 中註冊組織目錄中的裝置，如果適用的話就在行動裝置管理中加以註冊。

11. 如果您是：
    - 受控使用者，Windows 會透過自動登入程序帶您前往桌面。

    - 同盟使用者，系統會將您導向至 Windows 登入畫面來輸入認證。

## <a name="verification"></a>驗證

若要確認裝置是否已加入您的 Azure AD，請在 Windows 裝置上檢閱 [存取公司或學校資源] 對話方塊。 此對話方塊應該指出您已連線至 Azure AD 目錄。

![存取公司或學校資源](./media/azuread-joined-devices-frx/13.png)


## <a name="next-steps"></a>後續步驟

- 如需詳細資訊，請參閱 [Azure Active Directory 中的裝置管理簡介](overview.md)。

- 如需在 Azure AD 入口網站中管理裝置的詳細資訊，請參閱[使用 Azure 入口網站管理裝置](device-management-azure-portal.md)。
