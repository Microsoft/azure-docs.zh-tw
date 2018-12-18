---
title: Azure AD Connect - 使用 Azure AD Connect 管理與 Azure AD 的 AD FS 信任 | Microsoft Docs
description: 由 Azure AD 所處理之 Azure AD 信任的作業詳細資料。
keywords: AD FS, ADFS, AD FS 管理, AAD Connect, Connect, Azure AD, 信任, AAD, 宣告, 宣告規則, 發行, 轉換, 規則, 備份, 還原
services: active-directory
documentationcenter: ''
ms.reviewer: anandyadavmsft
manager: mtillman
ms.component: hybrid
ms.assetid: 2593b6c6-dc3f-46ef-8e02-a8e2dc4e9fb9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/28/2018
ms.author: billmath
author: billmath
ms.custom: ''
ms.openlocfilehash: 5ac69c53a6b6c1e4695b88e5806f8e883cd52c66
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432047"
---
# <a name="manage-ad-fs-trust-with-azure-ad-using-azure-ad-connect"></a>使用 Azure AD Connect 管理與 Azure AD 的 AD FS 信任

## <a name="overview"></a>概觀

Azure AD Connect 可以管理內部部署 Active Directory Federation Service (AD FS) 與 Azure AD 之間的同盟。 本文提供下列內容的概觀：

* 由 Azure AD Connect 在信任上所設定的各種設定
* 由 Azure AD Connect 所設定的發行轉換規則 (宣告規則)
* 如何在升級和設定更新之間備份和還原您的宣告規則。 

## <a name="settings-controlled-by-azure-ad-connect"></a>由 Azure AD Connect 所控制的設定

Azure AD Connect **只**會管理與 Azure AD 信任相關的設定。 Azure AD Connect 不會在 AD FS 中修改有關其他信賴憑證者信任的任何設定。 下表指出由 Azure AD Connect 所控制的設定。

| 設定 | 說明 |
| :--- | :--- |
| 權杖簽署憑證 | Azure AD Connect 可以用來重設並重新建立與 Azure AD 的信任。 Azure AD Connect 會針對 AD FS 進行權杖簽署憑證的單次立即變換，並更新 Azure AD 網域同盟設定。|
| 權杖簽署演算法 | Microsoft 建議使用 SHA-256 作為權杖簽署演算法。 Azure AD Connect 可以偵測是否將權杖簽署演算法設定為比 SHA-256 更不安全的值。 它將在下一個可能的設定作業中將設定更新為 SHA-256。 必須更新其他信賴憑證者信任，才能使用新的權杖簽署憑證。 |
| Azure AD 信任識別碼 | Azure AD Connect 會針對 Azure AD 信任設定正確的識別碼值。 AD FS 可以使用識別碼值來唯一識別 Azure AD 信任。 |
| Azure AD 端點 | Azure AD Connect 確保針對 Azure AD 信任所設定的端點一律會根據每個最新的建議值來取得彈性與效能。 |
| 發行轉換規則 | 有數個可在同盟設定中用來取得 Azure AD 功能最佳效能所需的宣告規則。 Azure AD Connect 確保一律會使用一組正確的建議宣告規則來設定 Azure AD 信任。 |
| 替代識別碼 | 如果將同步處理設定為使用替代識別碼，Azure AD Connect 就會將 AD FS 設定為使用替代識別碼來執行驗證。 |
| 自動中繼資料更新 | 與 Azure AD 的信任已設定來自動更新中繼資料。 AD FS 會定期檢查 Azure AD 信任的中繼資料，並使其在 Azure AD 端變更時保持最新狀態。 |
| 整合式 Windows 驗證 (IWA) | 在加入混合式 Azure AD 作業期間，啟用 IWA 進行裝置註冊，使下層裝置更容易加入混合式 Azure AD |

## <a name="execution-flows-and-federation-settings-configured-by-azure-ad-connect"></a>由 Azure AD Connect 所設定的執行流程和同盟設定

Azure AD Connect 不會在設定流程期間更新 Azure AD 信任的所有設定。 所修改的設定取決於正在執行的工作或執行流程。 下表列出不同執行流程中受影響的設定。

| 執行流程 | 受影響的設定 |
| :--- | :--- |
| 首次安裝 (快速) | None |
| 首次安裝 (新的 AD FS 伺服器陣列) | 建立新的 AD FS 伺服器陣列，並從頭開始建立與 Azure AD 的信任。 |
| 首次安裝 (現有的 AD FS 伺服器陣列，現有的 Azure AD 信任) | Azure AD 信任識別碼、發行轉換規則、Azure AD 端點、替代識別碼 (如有必要)、自動中繼資料更新 |
| 重設 Azure AD 信任 | 權杖簽署憑證、權杖簽署演算法、Azure AD 信任識別碼、發行轉換規則、Azure AD 端點、替代識別碼 (如有必要)、自動中繼資料更新 |
| 新增同盟伺服器 | None |
| 新增 WAP 伺服器 | None |
| 裝置選項 | 發行轉換規則、適用於裝置註冊的 IWA |
| 新增同盟網域 | 如果網域是第一次新增，亦即，安裝程式正從單一網域同盟變更為多網域同盟，Azure AD Connect 將從頭開始重新建立信任。 如果已針對多個網域設定與 Azure AD 的信任，則只會修改發行轉換規則 |
| 更新 SSL | None |

在所有作業期間 (可在其中修改任何設定)，Azure AD Connect 會在 **%ProgramData%\AADConnect\ADFS** 上建立目前信任設定的備份

![顯示現有 Azure AD 信任備份相關訊息的 Azure AD Connect 頁面](./media/how-to-connect-azure-ad-trust/backup2.png)

> [!NOTE]
> 在 1.1.873.0 版之前，備份只包含發行轉換規則，而且已將它們備份於精靈追蹤記錄檔中。

## <a name="issuance-transform-rules-set-by-azure-ad-connect"></a>由 Azure AD Connect 所設定的發行轉換規則

Azure AD Connect 確保一律會使用一組正確的建議宣告規則來設定 Azure AD 信任。 Microsoft 建議使用 Azure AD Connect 來管理您的 Azure AD 信任。 本節會列出發行轉換規則集及其描述。

| 規則名稱 | 說明 |
| --- | --- |
| 發行 UPN | 此規則會針對 userprincipalname 從同步處理設定中設定的屬性，查詢 userprincipalname 的值。|
| 針對自訂的 ImmutableId 宣告，查詢 objectguid 和 msdsconsistencyguid | 此規則會在管線中，針對 objectguid 和 msdsconsistencyguid 值 (如果有的話) 新增暫存值 |
| 檢查 msdsconsistencyguid 是否存在 | 依據 msdsconsistencyguid 的值是否存在，我們會設定暫存旗標來指示要用來作為 ImmutableId 的項目 |
| 發行 msdsconsistencyguid 作為固定 ID (如果有的話) | 發行 msdsconsistencyguid 作為 ImmutableId (如果值存在) |
| 如果 msdsConsistencyGuid 規則不存在，請發行 objectGuidRule | 如果 msdsconsistencyguid 的值不存在，將發行 objectguid 的值作為 ImmutableId |
| 發行 nameidentifier | 此規則會發行 nameidentifier 宣告的值。|
| 針對已加入網域的電腦發行 accounttype | 如果要驗證的實體為已加入網域的裝置，此規則就會發行帳戶類型作為表示已加入網域之裝置的 DJ |
| 如果它不是電腦帳戶，即會發行值為「使用者」的 AccountType | 如果要驗證的實體為使用者，此規則就會將帳戶類型發行為「使用者」 |
| 如果它不是電腦帳戶，即會發行 issuerid | 當驗證實體不是裝置時，此規則就會發行 issuerId 值。 值會透過 regex 來建立，其是由 Azure AD Connect 設定的。 將使用 Azure AD Connect 進行同盟的所有網域納入考量之後，即會建立 regex。 |
| 發行 issuerid 以進行 DJ 電腦驗證 | 當驗證實體為裝置時，此規則就會發行 issuerId 值 |
| 針對已加入網域的電腦發行 onpremobjectguid | 如果要驗證的實體為已加入網域的裝置，此規則會針對該裝置發行內部部署的 objectguid |
| 傳遞主要 SID | 此規則會發行驗證實體的主要 SID |
| 傳遞宣告：insideCorporateNetwork | 此規則會發行一個宣告，以協助 Azure AD 了解驗證是來自公司網路內部或外部 |
| 傳遞宣告：Psso |   |
| 發行密碼到期宣告 | 此規則會針對密碼到期時間、要驗證之實體的密碼到期天數，以及要進行路由傳送以變更密碼的 URL 來發行三個宣告。|
| 傳遞宣告：authnmethodsreferences | 在此規則下所發行之宣告中的值表示已針對實體執行的驗證類型 |
| 傳遞宣告：multifactorauthenticationinstant | 此宣告的值會指定使用者上次執行多重要素驗證的時間 (UTC 格式)。 |
| 傳遞宣告：AlternateLoginID | 如果驗證是使用替代登入識別碼來執行的，此規則就會發行 AlternateLoginID 宣告。 |

> [!NOTE]
> 如果您在 Azure AD Connect 設定期間使用非預設的選項，則適用於發行 UPN 和 ImmutableId 的宣告規則將有所不同

## <a name="restore-issuance-transform-rules"></a>還原發行轉換規則

每次更新 Azure AD 信任設定時，Azure AD Connect 1.1.873.0 版或更新版本就會建立 Azure AD 信任設定的備份。 在 **%ProgramData%\AADConnect\ADFS** 上備份 Azure AD 信任設定。 檔案名稱格式如下：AadTrust-&lt;date&gt;-&lt;time&gt;.txt，例如，AadTrust-20180710-150216.txt

![Azure AD 信任範例備份的螢幕擷取畫面](./media/how-to-connect-azure-ad-trust/backup.png)

您可以使用下列建議步驟來還原發行轉換規則

1. 在 [伺服器管理員] 中開啟 AD FS 管理 UI
2. 前往 [AD FS]**&gt; [信賴憑證者信任] &gt; [Microsoft Office 365 識別身分平台] &gt; [編輯宣告發行原則]**  來開啟 Azure AD 信任內容
3. 按一下 [新增規則]
4. 在宣告規則範本中，選取 [使用自訂規則傳送宣告]，然後按一下 [下一步]
5. 從備份檔案複製宣告規則的名稱，然後將它貼到 [宣告規則名稱] 欄位中
6. 將備份檔案中的宣告規則複製到 [自訂規則] 的文字欄位，然後按一下 [完成]

> [!NOTE]
> 請確定您的其他規則不會與 Azure AD Connect 所設定的規則產生衝突。

## <a name="next-steps"></a>後續步驟
* [使用 Azure AD Connect 管理和自訂 Active Directory 同盟服務](how-to-connect-fed-management.md)
