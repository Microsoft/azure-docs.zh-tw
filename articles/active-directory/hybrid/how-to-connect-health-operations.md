---
title: Azure Active Directory Connect Health 作業
description: 本文說明可以在您部署 Azure AD Connect Health 之後執行的其他作業。
services: active-directory
documentationcenter: ''
author: zhiweiw
manager: mtillman
ms.assetid: 86cc3840-60fb-43f9-8b2a-8598a9df5c94
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2017
ms.author: billmath
ms.openlocfilehash: 4fa19c7feac8aab62e7e0b768c193aec6846d4fa
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304647"
---
# <a name="azure-active-directory-connect-health-operations"></a>Azure Active Directory Connect Health 作業
本主題說明您可以使用 Azure Active Directory (Azure AD) Connect Health 來執行的各種作業。

## <a name="enable-email-notifications"></a>啟用電子郵件通知
您可以將 Azure AD Connect Health 服務設定為當警示表示您的身分識別基礎結構狀況不良時，傳送電子郵件通知。 產生警示和解決警示時會發生這種情況。

![Azure AD Connect Health 電子郵件通知設定的螢幕擷取畫面](./media/how-to-connect-health-operations/email_noti_discover.png)

> [!NOTE]
> 依預設會啟用電子郵件通知。
>
>

### <a name="to-enable-azure-ad-connect-health-email-notifications"></a>啟用 Azure AD Connect Health 電子郵件通知
1. 針對您想要接收電子郵件通知的服務，開啟 [警示] 刀鋒視窗。
2. 從動作列中，按一下 [通知設定]。
3. 在電子郵件通知開關上，選取 [開啟]。
4. 如果要讓所有全域管理員都接收電子郵件通知，請選取此核取方塊。
5. 如果想要用其他任何電子郵件地址來接收電子郵件通知，請在 [其他電子郵件收件者] 方塊中指定。 若要從這份清單中移除電子郵件地址，請以滑鼠右鍵按一下該項目，然後選取 [刪除]。
6. 若要完成變更，請按一下 [儲存]。 只有在您儲存之後，變更才會生效。

## <a name="delete-a-server-or-service-instance"></a>刪除伺服器或服務執行個體

>[!NOTE] 
> 需要 Azure AD Premium 授權，才能刪除步驟。

在某些情況下，您可能需要移除不要監視的伺服器。 以下是從 Azure AD Connect Health 服務移除伺服器時所需要知道的事項。

刪除伺服器時，請注意下列事項：

* 這個動作會停止從該伺服器收集任何進一步的資料。 此伺服器會從監視服務中移除。 執行此動作之後，您就無法檢視此伺服器的新警示、監視或使用情況分析資料。
* 這個動作不會從伺服器解除安裝健康情況代理程式。 如果您在執行此步驟之前未解除安裝健康情況代理程式，您可能會在伺服器上看到與健康情況代理程式相關的錯誤。
* 這個動作不會刪除已從這部伺服器收集的資料。 將會依照 Azure 資料保留原則來刪除該資料。
* 執行此動作之後，如果想要重新開始監視相同的伺服器，您必須在這部伺服器上解除安裝健康情況代理程式，再重新安裝。

### <a name="delete-a-server-from-the-azure-ad-connect-health-service"></a>從 Azure AD Connect Health 服務刪除伺服器

>[!NOTE] 
> 需要 Azure AD Premium 授權，才能刪除步驟。

適用於 Active Directory 同盟服務 (AD FS) 和 Azure AD Connect (Sync) 的 Azure AD Connect Health：

1. 從 [伺服器清單] 刀鋒視窗中選取要移除的伺服器名稱，以開啟 [伺服器] 刀鋒視窗。
2. 在 [伺服器] 刀鋒視窗中，從動作列中按一下 [刪除]。
3. 在確認方塊中輸入伺服器名稱以確認。
4. 按一下 [刪除] 。

Azure AD Connect Health for Azure Active Directory Domain Services：

1. 開啟 [網域控制站] 儀表板。
2. 選取要移除的網域控制站。
3. 從動作列中，按一下 [刪除已選取項目]。
4. 確認刪除伺服器的動作。
5. 按一下 [刪除] 。

### <a name="delete-a-service-instance-from-azure-ad-connect-health-service"></a>從 Azure AD Connect Health 服務刪除服務執行個體
在某些情況下，您可能需要移除服務執行個體。 以下是從 Azure AD Connect Health 服務移除服務執行個體時所需要知道的事項。

刪除服務執行個體時，請注意下列事項：

* 這個動作會從監視服務移除目前的服務執行個體。
* 這個動作不會從已在此服務執行個體中監視的任何伺服器解除安裝或移除健康情況代理程式。 如果您在執行此步驟之前未解除安裝健康情況代理程式，您可能會在伺服器上看到與健康情況代理程式相關的錯誤。
* 將會根據 Azure 資料保留原則來刪除此伺服器執行個體中的所有資料。
* 執行此動作之後，如果想要開始監視該服務，請在所有伺服器上解除安裝健康情況代理程式，再重新安裝。 執行此動作之後，如果想要重新開始監視相同的伺服器，請在該伺服器上解除安裝、重新安裝及註冊健康情況代理程式。

#### <a name="to-delete-a-service-instance-from-the-azure-ad-connect-health-service"></a>從 Azure AD Connect Health 服務刪除服務執行個體
1. 從 [服務清單] 刀鋒視窗中選取您想要移除的服務識別碼 (伺服陣列名稱)，以開啟 [服務] 刀鋒視窗。
2. 在 [伺服器] 刀鋒視窗中，從動作列中按一下 [刪除]。
3. 在確認方塊中輸入服務名稱以確認 (例如：sts.contoso.com)。
4. 按一下 [刪除] 。
   <br><br>

[//]: # (RBAC 小節的開頭)
## <a name="manage-access-with-role-based-access-control"></a>使用角色型存取控制來管理存取
Azure AD Connect Health 的[角色型存取控制 (RBAC)](../../role-based-access-control/role-assignments-portal.md) 提供存取權給全域管理員以外的使用者和群組。 RBAC 會指派角色給目標使用者和群組，並提供機制來限制您目錄內的全域管理員。

### <a name="roles"></a>角色
Azure AD Connect Health 支援下列內建角色：

| 角色 | 權限 |
| --- | --- |
| 擁有者 |擁有者可以在 Azure AD Connect Health 內「管理存取」(例如將角色指派給使用者或群組)、「檢視入口網站中的所有資訊」(例如檢視警示)，以及「變更設定」(例如電子郵件通知)。 <br>預設會指派此角色給 Azure AD 全域管理員，而且無法變更。 |
| 參與者 |參與者可以在 Azure AD Connect Health 內「檢視入口網站中的所有資訊」(例如檢視警示)，以及「變更設定」(例如電子郵件通知)。 |
| 讀取器 |讀者可以在 Azure AD Connect Health 內「檢視入口網站中的所有資訊」(例如檢視警示)。 |

其他所有角色 (例如「使用者存取系統管理員」或「DevTest Labs 使用者」) 即使出現在入口網站體驗中，也不影響 Azure AD Connect Health 內的存取。

### <a name="access-scope"></a>存取範圍
Azure AD Connect Health 支援在兩個層級上管理存取：

* **所有服務執行個體**：在大部分情況下，這是建議的途徑。 它可以在 Azure AD Connect Health 所監視的所有角色類型上，控制所有服務執行個體的存取 (例如 AD FS 伺服陣列)。
* **服務執行個體**：在某些情況下，您可能需要根據角色類型或服務執行個體來隔離存取。 在此情況下，您可以管理服務執行個體層級的存取。  

如果使用者具有目錄或服務執行個體層級的存取，則會授與權限。

### <a name="allow-users-or-groups-access-to-azure-ad-connect-health"></a>允許使用者或群組存取 Azure AD Connect Health
下列步驟示範如何允許存取。
#### <a name="step-1-select-the-appropriate-access-scope"></a>步驟 1：選取適當的存取範圍
若要允許 Azure AD Connect Health 內的*所有服務執行個體*層級使用者存取，請開啟 Azure AD Connect Health 中的主要刀鋒視窗。<br>

#### <a name="step-2-add-users-and-groups-and-assign-roles"></a>步驟 2：新增使用者和群組及指派角色
1. 從 [設定] 區段中，按一下 [使用者]。<br>
   ![Azure AD Connect Health 資源提要欄位的螢幕擷取畫面](./media/how-to-connect-health-operations/startRBAC.png)
2. 選取 [新增] 。
3. 在 [選取角色] 窗格中，選取角色 (例如，**擁有者**)。<br>
   ![Azure AD Connect Health RBAC [使用者] 視窗的螢幕擷取畫面](./media/how-to-connect-health-operations/RBAC_add.png)
4. 輸入目標使用者或群組的名稱或識別碼。 您可以同時選取一或多個使用者或群組。 按一下 [選取] 。
   ![Azure AD Connect Health RBAC [使用者] 視窗的螢幕擷取畫面](./media/how-to-connect-health-operations/RBAC_select_users.png)
5. 選取 [確定] 。<br>
6. 完成角色指派之後，使用者和群組就會出現在清單中。<br>
   ![Azure AD Connect Health RBAC [使用者] 視窗的螢幕擷取畫面，其中已醒目提示新使用者](./media/how-to-connect-health-operations/RBAC_user_list.png)

現在，列出的使用者和群組根據其獲指派的角色，已具有存取權。

> [!NOTE]
> * 全域管理員一律具有所有作業的完整存取，但全域管理員帳戶不會出現在上述清單中。
> * Azure AD Connect Health 內不支援「邀請使用者」功能。
>
>

#### <a name="step-3-share-the-blade-location-with-users-or-groups"></a>步驟 3：與使用者或群組共用刀鋒視窗位置
1. 指派權限之後，使用者就可以前往[這裡](https://aka.ms/aadconnecthealth)來存取 Azure AD Connect Health。
2. 在刀鋒視窗上，使用者可以將刀鋒視窗或其他組件釘選到儀表板。 只要按一下 [釘選到儀表板] 圖示即可。<br>
   ![Azure AD Connect Health RBAC 釘選刀鋒視窗的螢幕擷取畫面，其中已醒目提示釘選圖示](./media/how-to-connect-health-operations/RBAC_pin_blade.png)

> [!NOTE]
> 被指派「讀者」角色的使用者無法從 Azure Marketplace 取得 Azure AD Connect Health 擴充。 此使用者無法執行必要的「建立」作業來這麼做。 此使用者仍可前往上述連結以存取刀鋒視窗。 為方便之後使用，使用者可以將刀鋒視窗釘選到儀表板。
>
>

### <a name="remove-users-or-groups"></a>移除使用者或群組
您可以移除已新增至 Azure AD Connect Health RBAC 的使用者或群組。 只要以滑鼠右鍵按一下使用者或群組，然後選取 [移除] 即可。<br>
![Azure AD Connect Health RBAC [使用者] 視窗的螢幕擷取畫面，其中已醒目提示 [移除]](./media/how-to-connect-health-operations/RBAC_remove.png)

[//]: # (RBAC 小節的結尾)

## <a name="next-steps"></a>後續步驟
* [Azure AD Connect Health](whatis-hybrid-identity-health.md)
* [Azure AD Connect Health 代理程式安裝](how-to-connect-health-agent-install.md)
* [使用 Azure AD Connect Health 來搭配 AD FS](how-to-connect-health-adfs.md)
* [使用 Azure AD Connect Health 進行同步處理](how-to-connect-health-sync.md)
* [在 AD DS 使用 Azure AD Connect Health](how-to-connect-health-adds.md)
* [Azure AD Connect Health 常見問題集](reference-connect-health-faq.md)
* [Azure AD Connect Health 版本歷程記錄](reference-connect-health-version-history.md)
