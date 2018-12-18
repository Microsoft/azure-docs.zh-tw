---
title: Azure AD Synchronization Service Manager UI 中的連接器 | Microsoft Docs'
description: 了解 Azure AD Connect 的 Synchronization Service Manager 中的 [連接器] 索引標籤。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 60f1d979-8e6d-4460-aaab-747fffedfc1e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 592fcd80b80ecbd101dd559aa8baf78a60aaa517
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306847"
---
# <a name="using-connectors-with-the-azure-ad-connect-sync-service-manager"></a>使用連接器搭配 Auzre AD Connect Sync Service Manager

![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/connectors.png)

[連接器] 索引標籤可用來管理同步處理引擎連接的所有系統。

## <a name="connector-actions"></a>連接器動作
| 動作 | 註解 |
| --- | --- |
| 建立 |請勿使用。 若要連接到其他的 AD 樹系，請使用安裝精靈。 |
| properties |用於網域和 OU 篩選。 |
| [刪除](#delete) |用來刪除連接器空間中的資料或刪除與樹系的連接。 |
| [更新執行設定檔](#configure-run-profiles) |除了網域篩選以外，不會在此處進行任何設定。 您可以使用此動作來查看已設定的執行設定檔。 |
| 執行 |用來啟動設定檔的一次性執行。 |
| Stop |停止目前執行設定檔的連接器。 |
| 匯出連接器 |請勿使用。 |
| 匯入連接器 |請勿使用。 |
| 更新連接器 |請勿使用。 |
| 重新整理結構描述 |重新整理快取的結構描述。 最好是改為在安裝精靈中使用此選項，因為其也會更新同步處理規則。 |
| [搜尋連接器空間](#search-connector-space) |用來尋找物件，以及 [在整個系統中追隨物件及其資料](#follow-an-object-and-its-data-through-the-system)。 |

### <a name="delete"></a>刪除
刪除動作適用於兩個不同的用途。  
![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/connectordelete.png)

[僅刪除連接器空間]  選項會移除所有資料，但保留組態。

[刪除連接器和連接器空間]  選項會移除資料及組態。 當您不想再連接到樹系時，可使用此選項。

這兩個選項會同步處理所有物件，並更新 metaverse 物件。 此動作是長時間執行的作業。

### <a name="configure-run-profiles"></a>更新執行設定檔
此選項可讓您查看連接器所設定的執行設定檔。

![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/configurerunprofiles.png)

### <a name="search-connector-space"></a>搜尋連接器空間
尋找物件和疑難排解資料問題時，搜尋連接器空間動作非常有用。

![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/cssearch.png)

先選取一個 [範圍] 。 您可以依據資料 (RDN、DN、錨點、子樹狀目錄) 或物件狀態 (所有其他選項) 進行搜尋。  
![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/cssearchscope.png)  
例如，如果您進行樹狀子目錄的搜尋，將會取得某一個 OU 中的所有物件。  
![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/cssearchsubtree.png)  
您可以從此格線選取物件、選取「屬性」，然後[跟隨物件](tshoot-connect-object-not-syncing.md)，從來源連接器空間、通過 Metaverse，再到目標連接器空間。

### <a name="changing-the-ad-ds-account-password"></a>變更 AD DS 帳戶密碼
如果您變更帳戶密碼，同步處理服務就再也無法匯入/匯出內部部署 AD 的變更。   這時您會看到下列情形：

- AD 連接器的匯入/匯出步驟失敗，並出現「no-start-credentials」錯誤。
- Windows 事件檢視器底下的應用程式事件記錄會包含事件識別碼為 6000 的錯誤與「管理代理程式 "contoso.com" 無法執行，因為認證無效」的訊息。

若要解決此問題，請使用下列程序來更新 AD DS 使用者帳戶︰


1. 啟動同步處理服務管理員 ([開始] → [同步處理服務])。
</br>![Sync Service Manager](./media/how-to-connect-sync-service-manager-ui-connectors/startmenu.png)
2. 移至 [連接器] 索引標籤。
3. 選取設定為使用 AD DS 帳戶的 AD 連接器。
4. 選取 [動作] 下方的 [屬性]。
5. 在快顯對話方塊中，選取 [連線至 Active Directory 樹系]：
6. 樹系名稱會指出對應的內部部署 AD。
7. 使用者名稱會指出用於同步處理服務的 AD DS 帳戶。
8. 在 [密碼] 文字方塊中輸入新的 AD DS 帳戶密碼 ![Azure AD Connect 同步處理加密金鑰公用程式](./media/how-to-connect-sync-service-manager-ui-connectors/key6.png)
9. 按一下 [確定] 以儲存新密碼，然後重新啟動同步處理服務，以從記憶體快取中移除舊密碼。



## <a name="next-steps"></a>後續步驟
深入了解 [Azure AD Connect 同步](how-to-connect-sync-whatis.md) 組態。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
