---
title: Azure AD Connect：開始使用快速設定 | Microsoft Docs
description: 了解如何下載、安裝和執行 Azure AD Connect 的安裝精靈。
services: active-directory
author: billmath
manager: mtillman
editor: curtand
ms.assetid: b6ce45fd-554d-4f4d-95d1-47996d561c9f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 03/27/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 0e31f311c123fa8ace523cfde21ebdf2f707fce5
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46303871"
---
# <a name="getting-started-with-azure-ad-connect-using-express-settings"></a>使用快速設定開始使用 Azure AD Connect
當您有單一樹系拓撲和用於驗證的**密碼雜湊同步處理**時，便可使用 Azure AD Connect [快速設定](how-to-connect-password-hash-synchronization.md)。 **快速設定** 是預設選項，並且會用在最常部署的案例。 只要簡短地按幾下即可將內部部署目錄擴充至雲端。

在開始安裝 Azure AD Connect 之前，請務必要[下載 Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771) 並完成 [Azure AD Connect：硬體和必要條件](how-to-connect-install-prerequisites.md)中的必要條件步驟。

如果快速設定不符合拓撲，請參閱 [相關文件](#related-documentation) 中的其他案例。

## <a name="express-installation-of-azure-ad-connect"></a>快速安裝 Azure AD Connect
您可以在 [影片](#videos) 一節看到這些步驟的執行示範。

1. 以本機系統管理員身分登入您想要安裝 Azure AD Connect 的伺服器。 請在您想要做為同步處理伺服器的伺服器上進行此步驟。
2. 瀏覽並按兩下 **AzureADConnect.msi**。
3. 在 [歡迎] 畫面上，選取同意授權條款的方塊，然後按一下 [繼續] 。  
4. 在 [快速設定] 畫面上，按一下 [ **使用快速設定**]。  
   ![歡迎使用 Azure AD Connect](./media/how-to-connect-install-express/express.png)
5. 在 [連接到 Azure AD] 畫面上，輸入您的 Azure AD 的全域系統管理員使用者名稱和密碼。 按 [下一步] 。  
   ![連接至 Azure AD](./media/how-to-connect-install-express/connectaad.png)  
   如果您收到錯誤訊息，而且有連線問題，請參閱[針對連線問題進行疑難排解](tshoot-connect-connectivity.md)。
6. 在 [連接到 AD DS] 畫面上輸入企業系統管理員帳戶的使用者名稱和密碼。 您可以用 NetBios 或 FQDN 格式輸入網域部分，也就是 FABRIKAM\administrator 或 fabrikam.com\administrator。 按 [下一步] 。  
   ![連線到 AD DS](./media/how-to-connect-install-express/connectad.png)
7. 只有在未完成[必要條件](how-to-connect-install-prerequisites.md)中的[驗證網域](../active-directory-domains-add-azure-portal.md)時，才會顯示 [[**Azure AD 登入組態**](plan-connect-user-signin.md#azure-ad-sign-in-configuration)] 頁面。
   ![未驗證的網域](./media/how-to-connect-install-express/unverifieddomain.png)  
   如果您看到此頁面，請檢閱每一個標示為**未新增**和**未驗證**的網域。 確定您所使用的網域皆已在 Azure AD 中完成驗證。 驗證好網域時，按一下 [重新整理] 符號。
8. 在 [準備好設定] 畫面中，按一下 [安裝] 。
   * 在 [準備設定] 頁面上，您可以取消選取 [設定一完成，即開始同步處理程序]  核取方塊。 如果您想要進行其他設定 (例如[篩選](how-to-connect-sync-configure-filtering.md))，應該取消選取此核取方塊。 若您取消選取此選項，精靈會設定同步處理，但是會讓排程器保持停用。 直到您[重新執行安裝精靈](how-to-connect-installation-wizard.md)以手動方式加以啟用時，才會執行排程器。
   * 啟用 [設定一完成，即開始同步處理程序] 核取方塊，將會立即觸發完整同步處理，將使用者、群組和連絡人同步至 Azure AD。 
   * 如果您的內部部署 Active Directory 中有 Exchange，則您也可以選擇啟用 [**Exchange 混合式部署**](https://technet.microsoft.com/library/jj200581.aspx)。 如果您打算在雲端和內部部署均設置 Exchange 信箱，請啟用此選項。
     ![準備設定 Azure AD Connect](./media/how-to-connect-install-express/readytoconfigure.png)
9. 當安裝完成時，按一下 [結束] 。
10. 安裝完成之後，請先登出再重新登入，才能使用 Synchronization Service Manager 或同步處理規則編輯器。

## <a name="videos"></a>影片
如需使用快速安裝的影片，請參閱：

> [!VIDEO https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Azure-Active-Directory-Connect-Express-Settings/player]
>
>

## <a name="next-steps"></a>後續步驟
安裝了 Azure AD Connect 之後，您可以 [驗證安裝和指派授權](how-to-connect-post-installation.md)。

深入了解這些在安裝時啟用的功能︰[自動升級](how-to-connect-install-automatic-upgrade.md)、[防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md)和 [Azure AD Connect Health](how-to-connect-health-sync.md)。

深入了解這些常見主題︰[排程器和如何觸發同步處理](how-to-connect-sync-feature-scheduler.md)。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。

## <a name="related-documentation"></a>相關文件

| 話題 | 連結 |
| --- | --- |
| Azure AD Connect 概觀 | [整合您的內部部署目錄與 Azure Active Directory](whatis-hybrid-identity.md)
| 使用自訂設定進行安裝 | [自訂 Azure AD Connect 安裝](how-to-connect-install-custom.md) |
| 從 DirSync 升級 | [從 Azure AD 同步作業工具 (DirSync) 升級](how-to-dirsync-upgrade-get-started.md)|
| 用於安裝的帳戶 | [進一步了解 Azure AD Connect 認證和權限](reference-connect-accounts-permissions.md) |
