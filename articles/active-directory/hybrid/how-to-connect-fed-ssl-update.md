---
title: Azure AD Connect - 更新 AD FS 伺服器陣列的 SSL 憑證 | Microsoft Docs
description: 本文件詳述使用 Azure AD Connect 更新 AD FS 伺服器陣列 SSL 憑證的步驟。
services: active-directory
manager: mtillman
editor: billmath
ms.assetid: 7c781f61-848a-48ad-9863-eb29da78f53c
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/09/2018
ms.component: hybrid
author: billmath
ms.custom: seohack1
ms.author: billmath
ms.openlocfilehash: 8320eae9e1c12123a4379887124dee810998c4e3
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309527"
---
# <a name="update-the-ssl-certificate-for-an-active-directory-federation-services-ad-fs-farm"></a>更新 Active Directory Federation Services (AD FS) 伺服器陣列的 SSL 憑證

## <a name="overview"></a>概觀
本文說明您如何使用 Azure AD Connect 來更新 Active Directory Federation Services (AD FS) 伺服器陣列的 SSL 憑證。 即使選取的使用者登入方法不是 AD FS，您也可以使用 Azure AD Connect 工具輕鬆地更新 AD FS 伺服器陣列的 SSL 憑證。

只要 3 個簡單步驟，即可跨所有同盟和 Web 應用程式 Proxy (WAP) 伺服器，執行 AD FS 之 SSL 憑證的完整更新作業︰

![三個步驟](./media/how-to-connect-fed-ssl-update/threesteps.png)


>[!NOTE]
>若要深入了解 AD FS 所使用的憑證，請參閱[了解 AD FS 所使用的憑證](https://technet.microsoft.com/library/cc730660.aspx)。

## <a name="prerequisites"></a>必要條件

* **AD FS 伺服器陣列**︰請確定您的 AD FS 伺服器陣列是 Windows Server 2012 R2 型或更新版本。
* **Azure AD Connect**︰請確定 Azure AD Connect 版本為 1.1.553.0 或更新版本。 您會使用工作「更新 AD FS SSL 憑證」。

![更新 SSL 工作](./media/how-to-connect-fed-ssl-update/updatessltask.png)

## <a name="step-1-provide-ad-fs-farm-information"></a>步驟 1︰提供 AD FS 伺服器陣列資訊

Azure AD Connect 會透過下列方式，嘗試自動取得 AD FS 伺服器陣列的相關資訊︰
1. 自 AD FS (Windows Server 2016 或更新版本) 查詢伺服器陣列資訊。
2. 參照來自前次執行的資訊 (使用 Azure AD Connect 儲存在本機)。

您可以修改所顯示之伺服器的清單，方法是新增或移除伺服器以反映 AD FS 伺服器陣列的目前組態。 只要提供了伺服器資訊，Azure AD Connect 會顯示連線和目前的 SSL 憑證狀態。

![AD FS 伺服器資訊](./media/how-to-connect-fed-ssl-update/adfsserverinfo.png)

如果清單包含不再屬於 AD FS 伺服器陣列的伺服器，則按一下 [移除] 可從 AD FS 伺服器陣列中的伺服器清單刪除伺服器。

![清單中的離線伺服器](./media/how-to-connect-fed-ssl-update/offlineserverlist.png)

>[!NOTE]
> 在 Azure AD Connect 中從 AD FS 伺服器陣列的伺服器清單中移除伺服器是本機作業，並且會更新 Azure AD Connect 在本機維護之 AD FS 伺服器陣列的資訊。 Azure AD Connect 不會修改 AD FS 上的組態以反映變更。    

## <a name="step-2-provide-a-new-ssl-certificate"></a>步驟 2︰提供新的 SSL 憑證

確認 AD FS 伺服器陣列伺服器的相關資訊之後，Azure AD Connect 會要求新的 SSL 憑證。 提供使用密碼保護的 PFX 憑證以繼續安裝。

![SSL 憑證](./media/how-to-connect-fed-ssl-update/certificate.png)

您提供憑證之後，Azure AD Connect 會經歷一系列的必要條件。 確認憑證以確定憑證對於 AD FS 伺服器陣列是正確的︰

-   憑證的主體名稱/替代主體名稱與同盟服務名稱相同，或者是萬用字元憑證。
-   憑證有效期限為 30 天以上。
-   憑證信任鏈結有效。
-   憑證使用密碼保護。

## <a name="step-3-select-servers-for-the-update"></a>步驟 3︰選取用於更新的伺服器

在下一個步驟中，選取 SSL 憑證需要更新的伺服器。 無法選取離線的伺服器進行更新。

![選取要更新的伺服器](./media/how-to-connect-fed-ssl-update/selectservers.png)

組態完成之後，Azure AD Connect 會顯示訊息，指出更新的狀態，並且會提供一個選項來驗證 AD FS 登入。

![組態完成](./media/how-to-connect-fed-ssl-update/configurecomplete.png)   

## <a name="faqs"></a>常見問題集

* **新 AD FS SSL 憑證之憑證的主體名稱應該是什麼？**

    Azure AD Connect 會檢查憑證的主體名稱/替代主體名稱是否包含同盟服務名稱。 例如，如果同盟服務名稱為 fs.contoso.com，則主體名稱/替代主體名稱必須是 fs.contoso.com。  也接受萬用字元憑證。

* **為什麼會要求我於 WAP 伺服器頁面上再次輸入認證？**

    如果提供用於連線至 AD FS 伺服器的認證也不具備管理 WAP 伺服器的權限，Azure AD Connect 會要求提供在 WAP 伺服器上具有系統管理權限的認證。

* **伺服器顯示為離線。我該怎麼辦？**

    如果伺服器已離線，Azure AD Connect 即無法執行任何作業。 如果伺服器是 AD FS 伺服器陣列的一部分，請檢查伺服器的連線。 在解決問題之後，按下 [重新整理] 圖示，以更新精靈中的狀態。 如果伺服器稍早是伺服器陣列的一部分，但現在已不存在，請按一下 [移除]，以將它從 Azure AD Connect 維護的伺服器清單中刪除。 從 Azure AD Connect 的清單中移除伺服器並不會改變 AD FS 組態本身。 如果您在 Windows Server 2016 或更新版本中使用 AD FS，則伺服器會保持在組態設定，並且會在下一次執行工作再次顯示。

* **可以使用新的 SSL 憑證更新我的伺服器陣列伺服器子集嗎？**

    是。 您永遠可以再次執行工作「更新 SSL 憑證」來更新其餘的伺服器。 在 [選取要進行 SSL 憑證更新的伺服器] 頁面上，您可以依「SSL 到期日」來排序伺服器清單，以便輕鬆地存取尚未更新的伺服器。

* **我在上一次執行中移除了伺服器，但它仍然顯示為離線，並且列在 [AD FS 伺服器] 頁面。為何即使在移除後，離線的伺服器還在？**

    從 Azure AD Connect 的清單中移除伺服器並不會在 AD FS 組態中將它移除。 Azure AD Connect 會參考 AD FS (Windows Server 2016 或更新版本) 以取得伺服器陣列的任何相關資訊。 如果伺服器仍然出現在 AD FS 組態中，它會列回到清單中。  

## <a name="next-steps"></a>後續步驟

- [Azure AD Connect 和同盟](how-to-connect-fed-whatis.md)
- [使用 Azure AD Connect 管理和自訂 Active Directory Federation Services](how-to-connect-fed-management.md)
