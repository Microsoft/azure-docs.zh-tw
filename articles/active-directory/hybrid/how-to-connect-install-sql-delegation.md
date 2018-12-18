---
title: 使用 SQL 委派的管理員權限安裝 Azure AD Connect | Microsoft Docs
description: 本主題說明可使用僅有 SQL dbo 權限之帳戶來進行安裝的 Azure AD Connect 更新。
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.reviewer: jparsons
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/07/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: b951f541487d70bccb8a709f79a3674ef22eba3a
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309547"
---
# <a name="install-azure-ad-connect-using-sql-delegated-administrator-permissions"></a>使用 SQL 委派的管理員權限安裝 Azure AD Connect
最新的 Azure AD Connect 組建之前的版本在部署需要 SQL 的設定時不支援管理委派。  想要安裝 Azure AD Connect 的使用者必須在 SQL 伺服器上擁有伺服器管理員 (SA) 權限。

使用最新版的 Azure AD Connect 時，SQL 系統管理員即可執行頻外資料庫佈建，然後由具有資料庫擁有者權限的 Azure AD Connect 管理員進行安裝。

## <a name="before-you-begin"></a>開始之前
若要使用這項功能，您需要了解其中有數個移動組件，且每個組件都可能牽涉到組織中的不同系統管理員。  下表摘要說明在使用這項功能部署 Azure AD Connect 時的個別角色及其各自的責任。

|角色|說明|
|-----|-----|
|網域或樹系 AD 系統管理員|建立可供 Azure AD Connect 用來執行同步處理服務的網域層級服務帳戶。  如需服務帳戶的詳細資訊，請參閱[帳戶和權限](reference-connect-accounts-permissions.md)。
|SQL 系統管理員|建立 ADSync 資料庫，並對 Azure AD Connect 系統管理員以及網域/樹系系統管理員所建立的服務帳戶授與登入 + dbo 存取權。|
Azure AD Connect 系統管理員|安裝 Azure AD Connect，並在自訂安裝期間指定服務帳戶。

## <a name="steps-for-installing-azure-ad-connect-using-sql-delegated-permissions"></a>使用 SQL 委派的權限來安裝 Azure AD Connect 的步驟
若要在頻外佈建資料庫，並使用資料庫擁有者權限安裝 Azure AD Connect，請使用下列步驟。

>[!NOTE]
>雖然沒有必要，但**強烈建議**您在建立資料庫時選取 Latin1_General_CI_AS 定序。


1.  讓 SQL 系統管理員使用不區分大小寫的定序序列 **(Latin1_General_CI_AS)** 建立 ADSync 資料庫。  資料庫必須命名為 **ADSync**。  在安裝 Azure AD Connect 時，復原模式、相容性層級和內含項目類型會更新為正確的值。  不過，SQL 系統管理員必須正確設定定序序列，否則 Azure AD Connect 會阻止安裝。  若要復原 SA，就必須先刪除再重新建立資料庫。</br>
![定序](./media/how-to-connect-install-sql-delegation/sql4.png)
2.  對 Azure AD Connect 系統管理員和網域服務帳戶授與下列權限：
    - SQL 登入 
    - **資料庫擁有者 (dbo)** 權限。  </br>
![權限](./media/how-to-connect-install-sql-delegation/sql3a.png)
3.  傳送電子郵件給 Azure AD Connect 系統管理員，內容中要指出在安裝 Azure AD Connect 時需使用 SQL 伺服器和執行個體名稱。

## <a name="additional-information"></a>其他資訊
資料庫佈建好之後，Azure AD Connect 系統管理員就可以在他們方便的時候安裝及設定內部部署同步處理。  

使用預先建立的資料庫時，需要有 **/UseExistingDatabase** 旗標。  它不僅用於復原情況。

除了支援新的 Azure AD Connect 安裝外，這項功能也可讓您在任何與 **/UseExistingDatabase** 旗標相關的案例中執行委派。  如需使用現有資料庫來安裝 Azure AD Connect 的詳細資訊，請參閱[使用現有的 ADSync 資料庫安裝 Azure AD Connect](how-to-connect-install-existing-database.md)


## <a name="next-steps"></a>後續步驟
- [使用快速設定開始使用 Azure AD Connect](how-to-connect-install-express.md)
- [自訂 Azure AD Connect 安裝](how-to-connect-install-custom.md)
- [使用現有的 ADSync 資料庫安裝 Azure AD Connect](how-to-connect-install-existing-database.md)  
