---
title: Data Lake Store 安全性概觀 | Microsoft Docs
description: 了解 Azure Data Lake Store 如何成為更安全的巨量資料存放區
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: ebd5b2ac-c5cc-46d4-9cfd-1a1ee70024c2
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: nitinme
ms.openlocfilehash: f4014d304d1e0a7d841e797c3e3bdfe31918c151
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34198152"
---
# <a name="security-in-azure-data-lake-store"></a>Azure Data Lake Store 安全性
許多企業會運用巨量資料分析來獲得商業見解，以利他們做出明智的決策。 但企業的環境可能既複雜又受到規範，並且還有數目日益增加的各類使用者。 因此，企業必須確定重要的商業資料有受到更妥善的保存，並授與正確層級的存取權給個別使用者。 Azure Data Lake Store 的設計便是要協助企業符合這些安全性需求。 請透過本文了解 Data Lake Store 的安全性功能，包括︰

* 驗證
* 授權
* 網路隔離
* 資料保護
* 稽核

## <a name="authentication-and-identity-management"></a>驗證和識別管理
驗證是在使用者與 Data Lake Store 互動或與連線到 Data Lake Store 的任何服務互動時，用來確認使用者身分識別的程序。 為了進行身分識別管理和驗證，Data Lake Store 會使用 [Azure Active Directory](../active-directory/active-directory-whatis.md)，這是一個綜合身分識別和存取管理雲端解決方案，可以簡化使用者和群組的管理。

每個 Azure 訂用帳戶都可與 Azure Active Directory 的執行個體相關聯。 只有 Azure Active Directory 服務中所定義的使用者和服務身分識別可以存取 Data Lake Store 帳戶，方法是使用 Azure 入口網站、命令列工具，或透過您的組織使用 Azure Data Lake Store SDK 所建立的用戶端應用程式。 使用 Azure Active Directory 做為集中式存取控制機制的主要優點如下︰

* 簡化的身分識別生命週期管理。 只要刪除或停用目錄中的帳戶，即可快速建立及快速撤銷使用者或服務的身分識別 (服務主體身分識別)。
* 多因素驗證。 [Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md) 可為使用者登入和交易提供額外的安全性。
* 可透過標準的開放通訊協定 (例如 OAuth 或 OpenID) 從任何用戶端進行驗證。
* 可與企業目錄服務和雲端識別提供者同盟。

## <a name="authorization-and-access-control"></a>授權和存取控制
在使用者通過 Azure Active Directory 驗證因而可以存取 Azure Data Lake Store 之後，授權便會控制 Data Lake Store 的存取權限。 Data Lake Store 會以下列方式分隔授權的帳戶以及資料相關的活動：

* [角色型存取控制](../role-based-access-control/overview.md) (RBAC)
* 適用於存取存放區資料的 POSIX ACL

### <a name="rbac-for-account-management"></a>用於帳戶管理的 RBAC
預設會為 Data Lake Store 定義四種基本角色。 這些角色允許透過 Azure 入口網站、PowerShell Cmdlet 和 REST API，在 Data Lake Store 帳戶上執行不同的作業。 [擁有者] 和 [參與者] 角色可在帳戶上執行各種管理功能。 您可以將「讀取者」角色指派給只檢視帳戶管理資料的使用者。

![RBAC 角色](./media/data-lake-store-security-overview/rbac-roles.png "RBAC 角色")

請注意，雖然指派角色是為了管理帳戶，但某些角色會影響資料的存取權。 您必須使用 ACL 來控制使用者可在檔案系統上執行之作業的存取權。 下表顯示預設角色的管理權限和資料存取權限摘要。

| 角色 | 管理權限 | 資料存取權限 | 說明 |
| --- | --- | --- | --- |
| 未指派角色 |None |由 ACL 控管 |使用者無法使用 Azure 入口網站或 Azure PowerShell Cmdlet 來瀏覽 Data Lake Store。 使用者只能使用命令列工具。 |
| 擁有者 |全部 |全部 |擁有者角色是超級使用者。 此角色可管理所有事項，且具有資料的完整存取權。 |
| 讀取者 |唯讀 |由 ACL 控管 |讀取者角色可以檢視有關帳戶管理的所有事項，例如哪個使用者被指派給哪個角色。 讀取者角色無法進行任何變更。 |
| 參與者 |除了新增和移除角色以外的一切 |由 ACL 控管 |參與者角色可以管理帳戶的某些層面，例如部署以及警示建立和管理。 參與者無法新增或移除角色。 |
| 使用者存取系統管理員 |新增和移除角色 |由 ACL 控管 |使用者存取管理員角色可管理使用者對帳戶的存取權。 |

如需相關指示，請參閱 [指派使用者或安全性群組給 Data Lake Store 帳戶](data-lake-store-secure-data.md#assign-users-or-security-groups-to-azure-data-lake-store-accounts)。

### <a name="using-acls-for-operations-on-file-systems"></a>在檔案系統上使用 ACL 執行作業
Data Lake Store 是 Hadoop 分散式檔案系統 (HDFS) 之類的階層式檔案系統，並可支援 [POSIX ACL](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html#ACLs_Access_Control_Lists)。 它會控制擁有者角色、擁有者群組以及其他使用者和群組對於資源的讀取 (r)、寫入 (w) 和執行 (x) 權限。 在 Data Lake Store 中，可在根資料夾、子資料夾和個別檔案上啟用 ACL。 如需 ACL 如何在 Data Lake Store 的內容中運作的詳細資訊，請參閱 [Data Lake Store 中的存取控制](data-lake-store-access-control.md)。

建議您使用 [安全性群組](../active-directory/active-directory-groups-create-azure-portal.md)為多個使用者定義 ACL。 將使用者新增到安全性群組，然後將檔案或資料夾的 ACL 指派給該安全性群組。 因為指派權限的上限為 28 個項目，所以當您想要提供指派權限時，此方法非常有用。 如需如何使用 Azure Active Directory 安全性群組更加妥善地保護 Data Lake Store 中所儲存之資料的詳細資訊，請參閱 [將使用者或安全性群組以 ACL 形式指派給 Azure Data Lake Store 檔案系統](data-lake-store-secure-data.md#filepermissions)。

![列出存取權限](./media/data-lake-store-security-overview/adl.acl.2.png "列出存取權限")

## <a name="network-isolation"></a>網路隔離
使用 Data Lake Store 以協助在網路層級控制資料存放區的存取。 您可以建立防火牆，並且為受信任的用戶端定義 IP 位址範圍。 使用 IP 位址範圍時，只有具有定義範圍內 IP 位址的用戶端可以連線到 Data Lake Store。

![防火牆設定和 IP 存取](./media/data-lake-store-security-overview/firewall-ip-access.png "防火牆設定和 IP 位址")

## <a name="data-protection"></a>資料保護
Azure Data Lake Store 在資料的生命週期中保護您的資料。 對於傳輸中的資料，Data Lake Store 會使用業界標準的傳輸層安全性 (TLS) 通訊協定來保護網路上的資料。

![Data Lake Store 中的加密](./media/data-lake-store-security-overview/adls-encryption.png "Data Lake Store 中的加密")

Data Lake Store 也會為帳戶中儲存的資料提供加密功能。 您可以選擇將資料加密，或選擇不使用加密。 如果您選擇進行加密，Data Lake Store 中儲存的資料會在儲存在持續性媒體之前進行加密。 在這種情況下，Data Lake Store 會在保存之前自動加密資料，並在擷取之前解密資料，因此它對存取資料的用戶端來說是完全透明的。 用戶端不需進行任何程式碼變更，即可加密/解密資料。

金鑰管理，如 Data Lake Store 會提供兩種模式用於管理您的主要加密金鑰 (MEK)，它是解密 Data Lake Store 中儲存任何資料所需。 您可以讓 Data Lake Store 管理 MEK，或使用 Azure 金鑰保存庫帳戶，選擇保留 MEK 的擁有權。 您會在建立 Data Lake Store 帳戶時指定金鑰管理的模式。 如需如何提供加密相關組態的詳細資訊，請參閱[使用 Azure 入口網站開始使用 Azure Data Lake Store](data-lake-store-get-started-portal.md)。

## <a name="activity-and-diagnostic-logs"></a>活動和診斷記錄
視您尋找的是帳戶管理相關活動或資料相關活動，您可以使用活動或診斷記錄。

* 帳戶管理相關活動是使用 Azure Resource Manager API，而且會透過活動記錄呈現在 Azure 入口網站中。
* 資料相關活動會使用 WebHDFS REST API 並透過診斷記錄檔呈現在 Azure 入口網站中。

### <a name="activity-log"></a>活動記錄檔
若要符合規定，組織可能需要適當的帳戶管理活動稽核線索，才能深入了解特定事件。 Data Lake Store 有內建的監視，而且它會記錄所有帳戶管理活動。

如需帳戶管理稽核線索，請檢視並選擇您想要記錄的資料行。 您也可以將活動記錄匯出至 Azure 儲存體。

![活動記錄](./media/data-lake-store-security-overview/activity-logs.png "活動記錄")

如需使用活動記錄的詳細資訊，請參閱[檢視活動記錄以稽核對資源的動作](../azure-resource-manager/resource-group-audit.md)。

### <a name="diagnostics-logs"></a>診斷記錄檔
您可以在 Azure 入口網站中啟用資料存取稽核和診斷記錄，並將記錄傳送到 Azure Blob 儲存體帳戶、事件中樞或 Log Analytics。

![診斷記錄](./media/data-lake-store-security-overview/diagnostic-logs.png "診斷記錄")

如需搭配 Azure Data Lake Store 使用診斷記錄的詳細資訊，請參閱[存取 Data Lake Store 的診斷記錄](data-lake-store-diagnostic-logs.md)。

## <a name="summary"></a>總結
企業客戶要求安全且容易使用的資料分析雲端平台。 Azure Data Lake Store 的設計目的是要透過藉由 Azure Active Directory 整合的身分識別管理和驗證、以 ACL 為基礎的授權、網路隔離、傳輸中和待用資料加密以及稽核，來協助滿足這些需求。

如果您想要在 Data Lake Store 中看到新功能，請在 [Data Lake Store UserVoice 論壇](https://feedback.azure.com/forums/327234-data-lake)將您的意見反應傳給我們。

## <a name="see-also"></a>另請參閱
* [Azure Data Lake Store 概觀](data-lake-store-overview.md)
* [開始使用 Data Lake Store](data-lake-store-get-started-portal.md)
* [保護 Data Lake Store 中的資料](data-lake-store-secure-data.md)

