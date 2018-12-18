---
title: 如何設定混合式 Azure Active Directory 已加入的裝置 | Microsoft Docs
description: 了解如何設定混合式 Azure Active Directory 已加入的裝置。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/31/2018
ms.author: markvi
ms.reviewer: sandeo
ms.openlocfilehash: 9ffc84009adfca60e9ae6b188b65b15e874e7d9c
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39622165"
---
# <a name="how-to-control-the-hybrid-azure-ad-join-of-your-devices"></a>如何控制裝置的混合式 Azure AD Join

混合式 Azure AD Join 程序可自動向 Azure AD 註冊已加入網域的內部部署裝置。 在某些情況下，您會不想自動註冊所有裝置。 舉例來說，在首次推出期間要確認一切都正常運作時，即屬此種情況。

本文將為您提供相關指引，說明如何控制裝置的混合式 Azure AD Join。 


## <a name="prerequisites"></a>必要條件

本文假設您已熟悉以下各項：

-  [Azure Active Directory 中的裝置管理簡介](../device-management-introduction.md)
 
-  [如何規劃混合式 Azure Active Directory Join 實作](hybrid-azuread-join-plan.md)

-  設定適用於[受控網域](hybrid-azuread-join-managed-domains.md)或[同盟網域](hybrid-azuread-join-federated-domains.md)的混合式 Azure Active Directory Join



## <a name="control-windows-current-devices"></a>控制現行 Windows 裝置

對於執行 Windows 桌面作業系統的裝置，支援的版本為 Windows 10 年度更新版 (版本 1607) 或更新版本。 最佳做法是升級至最新版的 Windows 10。

所有的現行 Windows 裝置都會在裝置啟動或使用者登入時自動向 Azure AD 進行註冊。 您可以使用群組原則物件 (GPO) 或 System Center Configuration Manager 來控制此行為。

若要控制現行 Windows 裝置，您必須： 

1.  **對所有裝置**：停用自動註冊裝置。
2.  **對選取的裝置**：啟用自動註冊裝置。

如果您已確認一切都正常運作，即可重新為所有裝置啟用自動裝置註冊。



## <a name="group-policy-object"></a>群組原則物件 

您可以藉由部署下列 GPO 來控制裝置的裝置註冊行為：**將已加入網域的電腦註冊為裝置**

**若要設定 GPO**：

1.  開啟 [伺服器管理員]，然後移至 [工具] \> [群組原則管理]。

2.  移至要停用/啟用自動註冊的網域所對應的網域節點。

3.  在 [群組原則物件] 上按一下滑鼠右鍵，然後選取 [新增]。

4.  輸入群組原則物件的名稱 (例如*混合式 Azure AD Join*)。 

5.  按一下 [確定]。

6.  以滑鼠右鍵按一下新的 GPO，然後選取 [編輯]。

7.  移至 [電腦設定] \> [原則] \> [系統管理範本] \> [Windows 元件] \> [裝置註冊]。 

8.  以滑鼠右鍵按一下 [將已加入網域的電腦註冊為裝置]，然後選取 [編輯]。

    > [!NOTE] 
    > 此群組原則範本已從舊版的 [群組原則管理] 主控台重新命名。 如果您使用舊版的主控台，請移至 [電腦設定] \> [原則] \> [系統管理範本] \> [Windows 元件] \> [加入工作場所] \> [自動將用戶端電腦加入工作場所]。 

9.  選取下列其中一個設定，然後按一下 [套用]：

    - **已停用** - 防止自動裝置註冊
    - **已啟用** - 啟用自動裝置註冊

10. 按一下 [確定]。

您必須將 GPO 連結至您選擇的位置。 若要為組織中所有已加入網域的現行裝置設定此原則，請將 GPO 連結至網域。 若要執行受控制的部署，請為屬於組織單位或安全性群組、且已加入網域的現行 Windows 裝置設定此原則。

### <a name="configuration-manager-controlled-deployment"></a>組態管理員控制的部署 

您可以藉由設定下列用戶端設定來控制現行裝置的裝置註冊行為：**自動向 Azure Active Directory 註冊已加入網域的新 Windows 10 裝置**


**若要設定用戶端設定**：

1.  開啟 [組態管理員]，然後移至 [雲端服務]。

2.  在 [裝置設定] 下方，針對 [自動向 Azure Active Directory 註冊已加入網域的新 Windows 10 裝置] 選取下列其中一個設定：

    - **否** - 防止自動裝置註冊
    - **是** - 啟用自動裝置註冊


3.  按一下 [確定]。
    

您必須將此用戶端設定連結至您選擇的位置。 例如，若要為組織中所有的現行 Windows 裝置設定此用戶端設定，請將用戶端設定連結至網域。 若要執行受控制的部署，您可以為屬於組織單位或安全性群組、且已加入網域的現行 Windows 裝置設定此用戶端設定。

> [!Important]
> 雖然上述組態會處理已加入網域的現有 Windows 10 裝置，但由於加入網域的新 Windows 10 裝置上的群組原則或 Configuration Manager 設定的實際應用程式可能會發生延遲，加入網域的新裝置仍有可能嘗試完整的混合式 Azure AD 加入。 若要避免這個問題，建議您從以前從未加入混合式 Azure AD，且已套用上述群組原則設定或 Configuration Manager 用戶端設定的裝置建立新的 sysprep 映像 (作為佈建方法的範例)。 您還必須使用新的映像來佈建加入組織網域的新電腦。 

## <a name="control-windows-down-level-devices"></a>控制舊版 Windows 裝置

若要註冊舊版 Windows 電腦，您必須從下載中心的[適用於非 Windows 10 電腦的 Microsoft Workplace Join](https://www.microsoft.com/en-us/download/details.aspx?id=53554) 頁面下載並安裝 Windows Installer 套件 (.msi)。

您可以使用軟體發佈系統 (例如 System Center Configuration Manager) 來部署此套件。 此套件支援使用 quiet 參數的標準無訊息安裝選項。 [System Center Configuration Manager](https://www.microsoft.com/cloud-platform/system-center-configuration-manager) Current Branch 提供額外的舊版權益，例如能夠追蹤已完成的註冊。

安裝程式會在系統上建立排定的工作，此工作是在使用者的內容中執行。 此工作會在使用者登入 Windows 時觸發。 此工作會在向 Azure AD 進行驗證後，透過使用者認證以無訊息方式向 Azure AD 加入裝置。


若要控制裝置註冊，您應將 Windows Installer 套件僅部署至一組選取的舊版 Windows 裝置。 如果您已確認一切都正常運作，即可對所有舊版裝置推出套件。


## <a name="next-steps"></a>後續步驟

* [Azure Active Directory 中的裝置管理簡介](../device-management-introduction.md)



