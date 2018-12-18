---
title: 使用現有的 ADSync 資料庫安裝 Azure AD Connect | Microsoft Docs
description: 本主題會說明如何使用現有的 ADSync 資料庫。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.reviewer: cychua
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/30/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 36db41308678f3f1bd713561f9a844288f5db401
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306297"
---
# <a name="install-azure-ad-connect-using-an-existing-adsync-database"></a>使用現有的 ADSync 資料庫安裝 Azure AD Connect
Azure AD Connect 需要 SQL Server 資料庫來儲存資料。 您可以使用 Azure AD Connect 安裝的預設 SQL Server 2012 Express LocalDB 或使用您自己的完整版 SQL。 在先前，當您安裝 Azure AD Connect 時，一律會建立名為 ADSync 的新資料庫。 使用 Azure AD Connect 1.1.613.0 版 (或更新版本)，您可以選擇指向現有的 ADSync 資料庫來安裝 Azure AD Connect。

## <a name="benefits-of-using-an-existing-adsync-database"></a>使用現有 ADSync 資料庫的優點
透過指向現有的 ADSync 資料庫：

- 除了認證資訊之外，儲存在 ADSync 資料庫 (包括自訂同步處理規則、連接器、篩選條件和選擇性功能組態) 的同步處理設定會自動復原，並在安裝期間加以使用。 Azure AD Connect 用來透過內部部署 AD 和 Azure AD 同步處理變更的認證會予以加密，並只能由先前的 Azure AD Connect 伺服器加以存取。
- 所有儲存在 ADSync 資料庫中的身分識別資料 (與連接器空間和 Metaverse 相關聯) 和同步處理 Cookie 也會加以回復。 最新安裝的 Azure AD Connect 伺服器可以從先前 Azure AD Connect 伺服器離開的位置繼續進行同步處理，而不需執行完整的同步處理。

## <a name="scenarios-where-using-an-existing-adsync-database-is-beneficial"></a>使用現有 ADSync 資料庫的情節會有所幫助
這些優點在下列情節中很有用：


- 您擁有現有的 Azure AD Connect 部署。 現有的 Azure AD Connect 伺服器不再運作，但包含 ADSync 資料庫的 SQL Server 仍會正常運作。 您可以安裝新的 Azure AD Connect 伺服器，並加以指向現有的 ADSync 資料庫。 
- 您擁有現有的 Azure AD Connect 部署。 包含 ADSync 資料庫的 SQL Server 不再正常運作。 不過，您有資料庫的最新備份。 您可以先將 ADSync 資料庫還原到新的 SQL Server。 在那之後，您可以安裝新的 Azure AD Connect 伺服器，並加以指向還原的 ADSync 資料庫。
- 您所擁有的現有 Azure AD Connect 部署是使用 LocalDB。 由於 LocalDB 所加諸的 10 GB 限制，您需要移轉至完整的 SQL。 您可以從 LocalDB 備份 ADSync 資料庫，然後將它還原到 SQL Server。 在那之後，您可以重新安裝新的 Azure AD Connect 伺服器，並加以指向還原的 ADSync 資料庫。
- 您正嘗試安裝暫存伺服器，而想要確定其設定與目前作用中伺服器的設定相符。 您可以備份 ADSync 資料庫，並將它還原到另一個 SQL Server。 在那之後，您可以重新安裝新的 Azure AD Connect 伺服器，並加以指向還原的 ADSync 資料庫。

## <a name="prerequisite-information"></a>必要條件資訊

在繼續之前，需要記下的重要注意事項：

- 請務必檢閱在硬體安裝 Azure AD Connect 的必要條件和必要條件，以及安裝 Azure AD Connect 所需的帳戶和權限。 透過「使用現有資料庫」模式安裝 Azure AD Connect 所需的權限與「自訂」安裝的權限相同。
- 只有使用完整 SQL 時才支援在現有的 ADSync 資料庫部署 Azure AD Connect。 使用 SQL Express LocalDB 時並不支援。 如果您在 LocalDB 中有想要使用的現有 ADSync 資料庫，就必須先備份該 ADSync 資料庫 (LocalDB)，然後將它還原到完整 SQL。 之後，您就可以使用此方法，在還原的資料庫部署 Azure AD Connect。
- 用於安裝 Azure AD Connect 的版本必須滿足下列條件：
    - 1.1.613.0 或以上版本，以及
    - 相同或高於上一次搭配 ADSync 資料庫使用的 Azure AD Connect 版本。 如果用於安裝的 Azure AD Connect 版本高於上一次搭配 ADSync 資料庫使用的版本，可能就需要完整的同步處理。  如果兩個版本之間有結構描述或同步處理規則上的變更，就必須進行完整同步處理。 
- 所使用的 ADSync 資料庫應該包含相對較新的同步處理狀態。 上次使用現有 ADSync 資料庫的同步處理活動應在過去三週內發生。
- 透過「使用現有資料庫」方法安裝 Azure AD Connect 時，不會保留上一個 Azure AD Connect 伺服器上設定的登入方法。 此外，您無法在安裝期間設定登入方法。 安裝完成之後，您才能設定登入方法。
- 無法讓多個 Azure AD Connect 伺服器共用相同的 ADSync 資料庫。 「使用現有資料庫」方法可讓您搭配新的 Azure AD Connect 伺服器來重複使用現有的 ADSync 資料庫。 它不支援共用。

## <a name="steps-to-install-azure-ad-connect-with-use-existing-database-mode"></a>透過「使用現有資料庫」模式安裝 Azure AD Connect 的步驟
1.  將 Azure AD Connect 安裝程式 (AzureADConnect.MSI) 下載到 Windows Server。 按兩下 Azure AD Connect 安裝程式可開始安裝 Azure AD Connect。
2.  一旦 MSI 安裝完成後，Azure AD Connect 精靈就會開始使用快速模式安裝。 按一下 [結束] 圖示將畫面關閉。
![歡迎使用](./media/how-to-connect-install-existing-database/db1.png)
3.  啟動新的命令提示字元或 PowerShell 工作階段。 瀏覽至 <drive>\program files\Microsoft Azure AD Connect 資料夾。 執行 .\AzureADConnect.exe /useexistingdatabase 命令，可在「使用現有資料庫」安裝模式中啟動 Azure AD Connect 精靈。
![PowerShell](./media/how-to-connect-install-existing-database/db2.png)
4.  [歡迎使用 Azure AD Connect] 畫面隨即迎接您。 一旦您同意授權條款及隱私權注意事項後，請按一下 [繼續]。
![歡迎使用](./media/how-to-connect-install-existing-database/db3.png)
5.  在 [安裝必要元件] 畫面上，會啟用 [使用現有的 SQL Server] 選項。 指定裝載 ADSync 資料庫的 SQL Server 名稱。 如果用來主控 ADSync 資料庫的 SQL 引擎執行個體不是 SQL Server 上的預設執行個體，您就必須指定 SQL 引擎執行個體名稱。 此外，如果未啟用 SQL 瀏覽，就必須指定 SQL 引擎執行個體的連接埠號碼。 例如︰         
![歡迎使用](./media/how-to-connect-install-existing-database/db4.png)           

6.  在 [連線到 Azure AD] 畫面上，您必須提供 Azure AD 目錄的全域管理員認證。 建議使用預設 onmicrosoft.com 網域中的帳戶。 此帳戶只會用來在 Azure AD 中建立服務帳戶，而且在精靈完成後便不會使用。
![連線](./media/how-to-connect-install-existing-database/db5.png)
 
7.  在 [連線您的目錄] 畫面上，會列出針對目錄同步作業設定的現有 AD 樹系，旁邊會有紅色十字圖示。 若要同步處理內部部署 AD 樹系的變更，需要 AD DS 帳戶。 Azure AD Connect 精靈無法擷取儲存在 ADSync 資料庫的 AD DS 帳戶認證，因為認證會予以加密，且只能由先前的 Azure AD Connect 伺服器進行解密。 按一下 [變更認證] 可指定 AD 樹系的 AD DS 帳戶。
![Directories](./media/how-to-connect-install-existing-database/db6.png)
 
 
8.  在快顯對話方塊中，您可以 (i) 提供企業管理員認證，並且讓 Azure AD Connect 為您建立 AD DS 帳戶，或 (ii) 自行建立 AD DS 帳戶，並且為 Azure AD Connect 提供其認證。 一旦您選取選項並提供必要的認證後，請按一下 [確定] 以關閉快顯對話方塊。
![歡迎使用](./media/how-to-connect-install-existing-database/db7.png)
 
 
9.  一旦提供認證後，紅色十字圖示會取代為綠色勾號圖示。 按 [下一步] 。
![歡迎使用](./media/how-to-connect-install-existing-database/db8.png)
 
 
10. 在 [準備好設定] 畫面中，按一下 [安裝]。
![歡迎使用](./media/how-to-connect-install-existing-database/db9.png)
 
 
11. 安裝完成之後，Azure AD Connect 伺服器會自動啟用暫存模式。 建議您先檢閱伺服器組態和擱置的未預期變更匯出，然後再停用暫存模式。 

## <a name="next-steps"></a>後續步驟

- 安裝了 Azure AD Connect 之後，您可以 [驗證安裝和指派授權](how-to-connect-post-installation.md)。
- 深入了解這些在安裝時啟用的功能︰[防止意外刪除](how-to-connect-sync-feature-prevent-accidental-deletes.md)和 [Azure AD Connect Health](how-to-connect-health-sync.md)。
- 深入了解這些常見主題︰[排程器和如何觸發同步處理](how-to-connect-sync-feature-scheduler.md)。
- 深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
