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
ms.openlocfilehash: 12d3b358be8bb90b63e5e7310123f8ae7093994c
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43190267"
---
# <a name="how-to-plan-your-hybrid-azure-active-directory-join-implementation"></a>如何規劃混合式 Azure Active Directory Join 實作

以類似的方式，裝置會變成您想要保護的另一個身分識別，也可用來隨時隨地保護您的資源。 您可以使用下列其中一種方法將裝置的身分識別導入 Azure AD 中，以達到此目標：

- Azure AD Join
- 混合式 Azure AD Join
- Azure AD 註冊

將您的裝置導入 Azure AD 中，您將可透過跨雲端和內部部署資源的單一登入 (SSO)，將使用者的生產力最大化。 在此同時，您可以利用[條件式存取](../active-directory-conditional-access-azure-portal.md)來保護對雲端和內部部署資源的存取。

如果您有內部部署 Active Directory 環境，而且您想要將加入網域的裝置加入 Azure AD，您可以藉由設定混合式 Azure AD 已加入裝置來完成。 本文提供在您的環境中實作混合式 Azure AD Join 的相關步驟。 


## <a name="prerequisites"></a>必要條件

本文假設您熟悉 [Azure Active Directory 中的裝置管理簡介](../device-management-introduction.md)。


## <a name="plan-your-implementation"></a>計劃您的實作

若要規劃您的混合式 Azure AD 實作，您應該熟悉：

|   |   |
|---|---|
|![勾選][1]|檢閱支援的裝置|
|![勾選][1]|檢閱您應該知道的事情|
|![勾選][1]|選取您的案例|


 


## <a name="review-supported-devices"></a>檢閱支援的裝置 

混合式 Azure AD Join 支援廣泛的 Windows 裝置。 因為執行舊版 Windows 的裝置設定需要其他或不同的步驟，所以支援的裝置會分為兩類：

**現行 Windows 裝置**

- Windows 10
    
- Windows Server 2016


對於執行 Windows 桌面作業系統的裝置，支援的版本為 Windows 10 年度更新版 (版本 1607) 或更新版本。 最佳做法是升級至最新版的 Windows 10。



 **舊版 Windows 裝置**

- Windows 8.1
 
- Windows 7

- Windows Server 2012 R2
 
- Windows Server 2012 
 
- Windows Server 2008 R2 


在第一個規劃步驟中，您應該檢閱您的環境，並判斷是否需要支援 Windows 舊版裝置。



## <a name="review-things-you-should-know"></a>檢閱您應該知道的事情

如果您的環境是由單一樹系組成，而該樹系已將身分識別資料同步處理到多個 Azure AD 租用戶，您就無法使用混合式 Azure AD Join。

如果您使用的是系統準備工具 (Sysprep)，請確定您用來建立映像的 Windows 安裝尚未針對混合式 Azure AD Join 進行設定。

如果您依賴虛擬機器 (VM) 快照集來建立其他 VM，請確定您使用的 VM 快照集尚未針對混合式 Azure AD Join 進行設定。

對於已針對使用者設定檔漫遊或認證漫遊設定的裝置，不支援註冊舊版 Windows 裝置。 如果您倚賴設定檔或設定的漫遊，請使用 Windows 10。

- 在非同盟環境中，**支援**透過無縫單一登入 [Azure Active Directory 無縫單一登入](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso-quick-start)來註冊舊版 Windows 裝置。 
 
- 使用沒有無縫單一登入的 Azure AD 傳遞驗證時，**不**支援舊版 Windows 裝置的註冊。

- 對於使用漫遊設定檔的裝置，**不支援**註冊舊版 Windows 裝置。 如果您倚賴設定檔或設定的漫遊，請使用 Windows 10。


不支援註冊執行網域控制站 (DC) 角色的 Windows Server。

如果您的組織需要透過已驗證的輸出 Proxy 存取網際網路，您就必須確定 Windows 10 電腦可以成功向輸出 Proxy 進行驗證。 Windows 10 電腦使用電腦內容來執行裝置註冊，所以必須使用電腦內容來設定輸出 Proxy 驗證。


混合式 Azure AD Join 程序可自動向 Azure AD 註冊已加入網域的內部部署裝置。 在有些情況下，您不想自動註冊所有裝置。 如果這適合您，請參閱[如何控制裝置的混合式 Azure AD Join](hybrid-azuread-join-control.md)。



## <a name="select-your-scenario"></a>選取您的案例

您可以針對下列案例設定混合式 Azure AD Join：

- 受控網域
- 同盟網域  



如果您的環境有受控網域，則混合式 Azure AD Join 支援：

- 採用無縫單一登入 (SSO) 的傳遞驗證 (PTA) 

- 採用無縫單一登入 (SSO) 的密碼雜湊同步 (PHS) 

從 1.1.819.0 版開始，Azure AD Connect 即為您提供用來設定混合式 Azure AD Join 的精靈。 此精靈可讓您大幅簡化設定程序。 如需詳細資訊，請參閱

- [設定適用於同盟網域的混合式 Azure Active Directory Join](hybrid-azuread-join-federated-domains.md)


- [設定適用於受控網域的混合式 Azure Active Directory Join](hybrid-azuread-join-managed-domains.md)


 如果安裝必要的 Azure AD Connect 版本不適合您，請參閱[如何手動設定裝置註冊](../device-management-hybrid-azuread-joined-devices-setup.md)。 






## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [設定適用於同盟網域的混合式 Azure Active Directory Join](hybrid-azuread-join-federated-domains.md)
> [設定適用於受控網域的混合式 Azure Active Directory Join](hybrid-azuread-join-managed-domains.md)




<!--Image references-->
[1]: ./media/hybrid-azuread-join-plan/12.png
