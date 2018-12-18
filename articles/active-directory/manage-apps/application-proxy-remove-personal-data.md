---
title: 移除個人資料 - Azure Active Directory 應用程式 Proxy | Microsoft Docs
description: 從安裝在裝置上的連接器移除 Azure Active Directory 應用程式 Proxy 的個人資料。
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/21/2018
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: bccda196be82808e7dc369de3f517490f410e26e
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39366037"
---
# <a name="remove-personal-data-for-azure-active-directory-application-proxy"></a>移除 Azure Active Directory 應用程式 Proxy 的個人資料  

Azure Active Directory 應用程式 Proxy 需要您在裝置上安裝連接器，這表示您的裝置上可能會有個人資料。 本文提供如何刪除該個人資料來改善隱私權的步驟。 


## <a name="where-is-the-personal-data"></a>個人資料在哪裡？
應用程式 Proxy 可能將個人資料寫入下列記錄類型：

- 連接器事件記錄檔
- Windows 事件記錄檔

## <a name="remove-personal-data-from-windows-event-logs"></a>從 Windows 事件記錄檔移除個人資料

如需如何設定 Windows 事件記錄檔之資料保留的資訊，請參閱 [Settings for event logs](https://technet.microsoft.com/library/cc952132.aspx) (事件記錄檔的設定)。 若要深入了解 Windows 事件記錄檔，請參閱 [Using Windows Event Log](https://msdn.microsoft.com/library/windows/desktop/aa385772.aspx) (使用 Windows 事件記錄檔)。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-hybrid-note.md)]

## <a name="remove-personal-data-from-connector-event-logs"></a>從連接器事件記錄檔移除個人資料

若要確保應用程式 Proxy 記錄不會有個人資料，您可以：

- 視需要刪除或檢視資料，或
- 關閉記錄

使用下列各節從連接器事件記錄檔移除個人資料。 您必須在安裝連接器的所有裝置上完成移除程序。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

### <a name="view-or-export-specific-data"></a>檢視或匯出特定資料

若要檢視或匯出特定資料，請在每個連接器事件記錄檔中搜尋相關項目。 記錄檔位於 `C:\ProgramData\Microsoft\Microsoft AAD Application Proxy Connector\Trace`。 

由於記錄檔是文字檔，因此您可以使用 [findstr](https://docs.microsoft.com/windows-server/administration/windows-commands/findstr) 來搜尋與使用者相關的文字項目。  

若要尋找個人資料，請搜尋 UserID 的記錄檔。 

若要尋找使用 Kerberos 限制委派之應用程式所記錄的個人資料，請搜尋使用者名稱類型的這些元件：

- 內部部署使用者主體名稱
- 使用者主體名稱的使用者名稱部分
- 內部部署使用者主體名稱的使用者名稱部分
- 內部部署安全性帳戶管理員 (SAM) 帳戶名稱 


### <a name="delete-specific-data"></a>刪除特定資料

若要刪除特定資料：

1. 重新啟動 Microsoft Azure AD 應用程式 Proxy 連接器服務來產生新的記錄檔。 新的記錄檔可讓您刪除或修改舊的記錄檔。 
2. 請遵循前述的[檢視或匯出特定資料](#view-or-export-specific-data)程序來尋找必須刪除的資訊。 搜尋所有連接器記錄檔。
3. 刪除相關記錄檔，或選擇性地刪除包含個人資料的欄位。 如果您不再需要舊的記錄檔，您也可以全部加以刪除。

### <a name="turn-off-connector-logs"></a>關閉連接器記錄

確保連接器記錄不會包含個人資料的一個選項是關閉記錄產生。 若要停止產生連接器記錄，請從 `C:\Program Files\Microsoft AAD App Proxy Connector\ApplicationProxyConnectorService.exe.config` 移除下列醒目提示行。 

![組態](./media/application-proxy-remove-personal-data/01.png)


## <a name="next-steps"></a>後續步驟

如需應用程式 Proxy 的概觀，請參閱[如何為內部部署應用程式提供安全的遠端存取](application-proxy.md)。

