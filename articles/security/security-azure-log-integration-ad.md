---
title: 使用 Azure Active Directory 稽核記錄進行 Azure 記錄整合 | Microsoft Docs
description: 了解如何安裝 Azure 記錄整合服務，整合 Azure 的稽核記錄檔
services: security
documentationcenter: na
author: Barclayn
manager: MBaldwin
editor: TomShinder
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ums.workload: na
ms.date: 06/07/2018
ms.author: barclayn
ms.custom: azlog
ms.openlocfilehash: 07299b960747528dde8dcefc70055442c2150486
ms.sourcegitcommit: 4e36ef0edff463c1edc51bce7832e75760248f82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35235986"
---
# <a name="integrate-azure-active-directory-audit-logs"></a>整合 Azure Active Directory 稽核記錄

Azure Active Directory (Azure AD) 稽核事件可協助您識別 Azure Active Directory 中發生的特殊權限動作。 您可以藉由檢視 [Azure Active Directory 稽核報告事件](/active-directory/active-directory-reporting-audit-events#list-of-audit-report-events.md)看到您可以追蹤的事件類型。


>[!IMPORTANT]
> Azure 記錄整合功能將於 2019/06/01 淘汰。 AzLog 下載將於 2018 年 6 月 27 日停用。 如需繼續進行的指導，請檢閱 [Use Azure monitor to integrate with SIEM tools](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/) (使用 Azure 監視器與 SIEM 工具整合) 一文 

## <a name="steps-to-integrate-azure-active-directory-audit-logs"></a>整合 Azure Active Directory 稽核記錄的步驟

> [!NOTE]
> 在嘗試執行本文中的步驟之前，您必須先檢閱[開始使用](security-azure-log-integration-get-started.md)一文並完成該文所述的步驟。

1. 開啟命令提示字元並執行此命令：

   ``cd c:\Program Files\Microsoft Azure Log Integration``

2. 請執行這個命令： 
 
   ``azlog createazureid``

   此命令會提示您登入 Azure。 此命令接著會建立託管 Azure 訂用帳戶 (其中登入的使用者是系統管理員、共同管理員或擁有者) 的 Azure AD 租用戶中的 Azure Active Directory 服務主體。 若登入的使用者只是 Azure AD 租用戶中的來賓使用者，則此命令將會失敗。 對 Azure 的驗證會透過 Azure AD 來進行。 建立 Azure 記錄整合的服務主體會建立 Azure AD 身分識別，以獲得 Azure 訂用帳戶的讀取權限。

3. 執行下列命令以提供您的租用戶識別碼。 您必須是租用戶系統管理員角色的成員才能執行命令。

   ``Azlog.exe authorizedirectoryreader tenantId``

   範例：

   ``AZLOG.exe authorizedirectoryreader ba2c0000-d24b-4f4e-92b1-48c4469999``

4. 檢查下列資料夾以確認其中是否建立了 Azure Active Directory 稽核記錄 JSON 檔案：

   * **C:\Users\azlog\AzureActiveDirectoryJson**
   * **C:\Users\azlog\AzureActiveDirectoryJsonLD**

下列影片會示範本文所述的步驟：

> [!VIDEO https://channel9.msdn.com/Blogs/Azure-Security-Videos/Azure-Log-Integration-Videos-Azure-AD-Integration/player]


> [!NOTE]
> 如需有關將 JSON 檔案中的資訊帶入到安全性資訊和事件管理 (SIEM) 系統的特定指示，請連絡您的 SIEM 廠商。

可透過 [Azure 記錄檔整合 MSDN 論壇](https://social.msdn.microsoft.com/Forums/office/home?forum=AzureLogIntegration)取得社群協助。 此論壇可透過分享問題、解答、秘訣和技巧，讓 Azure 記錄整合社群的成員彼此支援。 此外，Azure 記錄整合小組會監看這個論壇，並且會盡全力提供協助。

您也可以建立[支援要求](../azure-supportability/how-to-create-azure-support-request.md)。 選取 [記錄整合] 作為您要求支援的服務。

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure 記錄整合，請參閱：

* [Azure 記錄的 Microsoft Azure 記錄整合](https://www.microsoft.com/download/details.aspx?id=53324)：此下載中心頁面會提供關於 Azure 記錄整合的詳細資料、系統需求和安裝指示。
* [Azure 記錄整合簡介](security-azure-log-integration-overview.md)：本文會為您介紹 Azure 記錄整合、其主要功能以及運作方式。
* [Azure 記錄整合常見問題集](security-azure-log-integration-faq.md)：本文提供 Azure 記錄整合的相關問題解答。
* [Azure 診斷和 Azure 稽核記錄的新功能](https://azure.microsoft.com/blog/new-features-for-azure-diagnostics-and-azure-audit-logs/)：此部落格文章為您介紹 Azure 稽核記錄和其他功能，協助您深入了解您的 Azure 資源的作業。
