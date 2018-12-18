---
title: Azure AD 密碼保護 (預覽) 中的疑難排解和記錄
description: 了解 Azure AD 密碼保護 (預覽) 的記錄和一般疑難排解
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: jsimmons
ms.openlocfilehash: 1eea6380d4276644db0c7681f23a4b0c5e79ff09
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39187344"
---
# <a name="preview-azure-ad-password-protection-monitoring-reporting-and-troubleshooting"></a>預覽：Azure AD 密碼保護的監視、報告和疑難排解

|     |
| --- |
| Azure AD 密碼保護和自訂的禁用密碼清單是 Azure Active Directory 的公開預覽版功能。 如需預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)|
|     |

在部署 Azure AD 密碼保護之後，監視和報告將是必要工作。 本文將深入探討以協助您了解各項服務將資訊記錄於何處，以及如何報告 Azure AD 密碼保護的使用情形。

## <a name="on-premises-logs-and-events"></a>內部部署記錄和事件

### <a name="dc-agent-service"></a>DC 代理程式服務

在每個網域控制站上，DC 代理程式服務軟體會將其密碼驗證的結果 (和其他狀態) 寫入至本機事件記錄：\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Admin

事件會由各種 DC 代理程式元件使用下列範圍進行記錄：

|元件 |事件識別碼範圍|
| --- | --- |
|DC 代理程式密碼篩選 dll| 10000-19999|
|DC 應用程式服務裝載處理序| 20000-29999|
|DC 代理程式服務原則驗證邏輯| 30000-39999|

就成功的密碼驗證作業而言，通常會有一個從 DC 代理程式密碼篩選 dll 記錄的事件。 如果密碼驗證作業失敗，則通常會記錄兩個事件，一個來自 DC 代理程式服務，另一個來自 DC 代理程式密碼篩選 dll。

系統會根據下列因素，記錄會擷取這些情況的離散事件：

* 指定的密碼是否設定或變更。
* 指定密碼的驗證是通過還是失敗。
* 驗證是否因為 Microsoft 全域原則與組織原則有所衝突而失敗。
* 目前的密碼原則目前是否開啟僅限稽核模式。

主要的密碼驗證相關事件如下：

|   |密碼變更 |密碼設定|
| --- | :---: | :---: |
|通過 |10014 |10015|
|失敗 (未通過客戶密碼原則)| 10016、30002| 10017、30003|
|失敗 (未通過 Microsoft 密碼原則)| 10016、30004| 10017、30005|
|僅限稽核通過 (將不會通過客戶密碼原則)| 10024、30008| 10025、30007|
|僅限稽核通過 (將不會通過 Microsoft 密碼原則)| 10024、30010| 10025、30009|

> [!TIP]
> 傳入的密碼會先根據 Microsoft 全域密碼進行驗證，如果驗證失敗，則不會執行進一步的處理。 這和對 Azure 中的密碼變更所執行的是相同的行為。

#### <a name="sample-event-log-message-for-event-id-10014-successful-password-set"></a>事件識別碼 10014 成功密碼設定的範例事件記錄訊息

為指定的使用者變更的密碼經驗證符合目前的 Azure 密碼原則。

 UserName：BPL_02885102771 FullName：

#### <a name="sample-event-log-message-for-event-id-10017-and-30003-failed-password-set"></a>事件識別碼 10017 和 30003 失敗密碼設定的範例事件記錄訊息

10017：

為指定的使用者重設的密碼不符合目前的 Azure 密碼原則，因此遭到拒絕。 如需詳細資訊，請參閱相互關聯的事件記錄訊息。

 UserName：BPL_03283841185 FullName：

30003：

為指定的使用者重設的密碼符合目前 Azure 密碼原則的每一租用戶禁用密碼清單所列的至少一個權杖，因此遭到拒絕。

 UserName：BPL_03283841185 FullName：

其他應注意的主要事件記錄訊息包括：

#### <a name="sample-event-log-message-for-event-id-30001"></a>事件識別碼 30001 的範例事件記錄訊息

已接受指定使用者的密碼，因為尚未提供 Azure 密碼原則

UserName：<user> FullName：<user>

此情況可能是因為下列一或多個原因所造成：%n

1. 尚未對 Azure 註冊樹系。

   解決步驟：系統管理員必須使用 Register-AzureADPasswordProtectionForest Cmdlet 註冊樹系。

2. 目前的樹系中至少有一個機器還沒有可用的 Azure AD 密碼保護 Proxy。

   解決步驟：系統管理員必須使用 Register-AzureADPasswordProtectionProxy Cmdlet 安裝並註冊 Proxy。

3. 此 DC 沒有任何 Azure AD 密碼保護 Proxy 執行個體的網路連線。

   解決步驟：確定至少已有一個 Azure AD 密碼保護 Proxy 執行個體的網路連線。

4. 此 DC 並未連線至網域中的其他網域控制站。

   解決步驟：確定有網域的網路連線。

#### <a name="sample-event-log-message-for-event-id-30006"></a>事件識別碼 30006 的範例事件記錄訊息

服務現在會強制執行下列 Azure 密碼原則。

 AuditOnly：1

 全域原則日期：‎2018‎-‎05‎-‎15T00:00:00.000000000Z

 租用戶原則日期：‎2018‎-‎06‎-‎10T20:15:24.432457600Z

 強制執行租用戶原則：1

#### <a name="dc-agent-log-locations"></a>DC 代理程式記錄位置

DC 代理程式服務也會將操作相關事件記錄到下列記錄：\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Operational

DC 代理程式服務也可將詳細偵錯層級追蹤事件記錄到下列記錄：\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Trace

> [!WARNING]
> 依預設會停用追蹤記錄。 啟用時，此記錄會接收大量事件，而可能會影響網域控制站的效能。 因此，只有在問題需要更深入的調查時才應啟用此增強型記錄，且應盡可能縮短啟用時間。

### <a name="azure-ad-password-protection-proxy-service"></a>Azure AD 密碼保護 Proxy 服務

密碼保護 Proxy 服務也會將最基本的事件集發出到下列事件記錄：\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Operational

密碼保護 Proxy 服務也可將詳細偵錯層級追蹤事件記錄到下列記錄：\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Trace

> [!WARNING]
> 依預設會停用追蹤記錄。 啟用時，此記錄會接收大量事件，而可能會影響 Proxy 主機的效能。 因此，只有在問題需要更深入的調查時才應啟用此記錄，且應盡可能縮短啟用時間。

### <a name="dc-agent-discovery"></a>DC 代理程式探索

`Get-AzureADPasswordProtectionDCAgent` Cmdlet 可用來顯示與執行於網域或樹系中的各種 DC 代理程式有關的基本資訊。 您可以從執行中的 DC 代理程式服務所註冊的 serviceConnectionPoint 物件中擷取這項資訊。 此 Cmdlet 的範例輸出如下：

```
PS C:\> Get-AzureADPasswordProtectionDCAgent
ServerFQDN            : bplChildDC2.bplchild.bplRootDomain.com
Domain                : bplchild.bplRootDomain.com
Forest                : bplRootDomain.com
Heartbeat             : 2/16/2018 8:35:01 AM
```

每個 DC 代理程式服務大約每小時會更新一次各種屬性。 資料仍會受限於 Active Directory 複寫延遲。

此 Cmdlet 的查詢範圍可使用 –Forest 或 –Domain 參數來變更。

### <a name="emergency-remediation"></a>緊急補救

如果發生 DC 代理程式服務造成問題的不良狀況，DC 代理程式服務可以立即關閉。 DC 代理程式密碼篩選 dll 會嘗試呼叫非執行中服務，且會記錄警告事件 (10012、10013)，但在這段期間傳入的所有密碼都會被接受。 隨後如有需要，也可以透過 Windows 服務控制管理員，在啟動類型設為 [已停用] 的情況下設定 DC 代理程式服務。

### <a name="performance-monitoring"></a>效能監視

DC 代理程式服務軟體會安裝名為 **Azure AD 密碼保護**的效能計數器物件。 以下是目前可用的效能計數器：

|效能計數器名稱 | 說明|
| --- | --- |
|已處理的密碼 |此計數器會顯示自上次重新啟動後已處理 (已接受或拒絕) 的密碼總數。|
|已接受的密碼 |此計數器會顯示自上次重新啟動後已接受的密碼總數。|
|已拒絕的密碼 |此計數器會顯示自上次重新啟動後已拒絕的密碼總數。|
|進行中的密碼篩選要求 |此計數器會顯示目前正在進行中的密碼篩選要求數目。|
|尖峰密碼篩選要求 |此計數器會顯示自上次重新啟動後的並行密碼篩選要求的尖峰數目。|
|密碼篩選要求錯誤 |此計數器會顯示自上次重新啟動後因錯誤而失敗的密碼篩選要求總數。 當 Azure AD 密碼保護 DC 代理程式服務未執行時，就可能發生錯誤。|
|密碼篩選要求/秒 |此計數器會顯示密碼的處理速率。|
|密碼篩選要求處理時間 |此計數器會顯示處理密碼篩選要求所需的平均時間。|
|尖峰密碼篩選要求處理時間 |此計數器會顯示自上次重新啟動後的尖峰密碼篩選要求處理時間。|
|因稽核模式而接受的密碼 |此計數器會顯示自上次重新啟動後，在正常情況下會遭到拒絕、但因密碼原則設定為稽核模式而被接受的密碼總數。|

## <a name="directory-services-repair-mode"></a>目錄服務修復模式

如果網域控制站在開機後進入目錄服務修復模式，則 DC 代理程式服務會偵測到此狀況將導致所有密碼驗證或強制活動停用，無論目前作用中的原則組態為何。

## <a name="domain-controller-demotion"></a>網域控制站降階

目前支援將仍在執行 DC 代理程式軟體的網域控制站降階。 不過，系統管理員應了解 DC 代理程式軟體在降階程序中會繼續執行，並且繼續強制執行目前的密碼原則。 新的本機系統管理員帳戶密碼 (指定為降階作業的一部分) 會像任何其他密碼一樣受到驗證。 Microsoft 建議在 DC 降階程序中應為本機系統管理員帳戶選擇安全的密碼；但 DC 代理程式軟體對新的本機系統管理員帳戶密碼所做的驗證，可能會干擾到既有的降階作業程序。
在降階成功後，網域控制站即已重新啟動，並重新以一般成員伺服器的形式執行，且 DC 代理程式軟體會回復為以被動模式執行。 其後您可以隨時將其解除安裝。

## <a name="removal"></a>移除

如果決定要解除安裝公用預覽版軟體，並從網域和樹系清除所有相關的狀態，您可以使用下列步驟來完成這項工作：

> [!IMPORTANT]
> 這些步驟務必要依序執行。 如果有任何密碼保護 Proxy 服務的執行個體仍繼續執行，它將會定期重新建立其 serviceConnectionPoint 物件，並定期重新建立 sysvol 狀態。

1. 從所有機器解除安裝密碼保護 Proxy 軟體。 此步驟**不需要**重新開機。
2. 從所有網域控制站解除安裝 DC 代理程式軟體。 此步驟**需要**重新開機。
3. 手動移除每個網域命名內容中的所有 Proxy 服務連接點。 您可以使用下列 Active Directory Powershell 命令找出這些物件的位置：
   ```
   $scp = “serviceConnectionPoint”
   $keywords = “{EBEFB703-6113-413D-9167-9F8DD4D24468}*”
   Get-ADObject -SearchScope Subtree -Filter { objectClass -eq $scp -and keywords -like $keywords }
   ```

   請勿省略 $keywords 變數值結尾處的星號 ("*")。

   透過 `Get-ADObject` 命令找到的產生物件隨後可使用管線傳送到 `Remove-ADObject`，或以手動方式刪除。 

4. 手動移除在每個網域命名內容中所有的 DC 代理程式連接點。 樹系中的每個網域控制站可能各有一個此類物件，視先前部署公用預覽版軟體的範圍而定。 您可以使用下列 Active Directory Powershell 命令找出該物件的位置：

   ```
   $scp = “serviceConnectionPoint”
   $keywords = “{B11BB10A-3E7D-4D37-A4C3-51DE9D0F77C9}*”
   Get-ADObject -SearchScope Subtree -Filter { objectClass -eq $scp -and keywords -like $keywords }
   ```

   透過 `Get-ADObject` 命令找到的產生物件隨後可使用管線傳送到 `Remove-ADObject`，或以手動方式刪除。

5. 手動移除樹系層級的組態狀態。 樹系組態狀態會保存在 Active Directory 組態命名內容中的容器內。 您可以依照下列方式加以探索和刪除：

   ```
   $passwordProtectonConfigContainer = "CN=Azure AD password protection,CN=Services," + (Get-ADRootDSE).configurationNamingContext
   Remove-ADObject $passwordProtectonConfigContainer
   ```

6. 手動刪除下列資料夾及其所有內容，以手動移除所有 sysvol 相關狀態：

   `\\<domain>\sysvol\<domain fqdn>\Policies\{4A9AB66B-4365-4C2A-996C-58ED9927332D}`

   如有必要，也可在指定的網域控制站上從本機存取此路徑；預設位置會類似於下列路徑：

   `%windir%\sysvol\domain\Policies\{4A9AB66B-4365-4C2A-996C-58ED9927332D}`

   如果已將 sysvol 共用設定於非預設位置中，此路徑將會不同。

## <a name="next-steps"></a>後續步驟

如需關於全域和自訂禁用密碼清單的詳細資訊，請參閱[禁用不當密碼](concept-password-ban-bad.md)一文
