---
title: Azure Data Lake Store 與 Azure 儲存體 Blob 比較 | Microsoft Docs
description: Azure Data Lake Store 與 Azure 儲存體 Blob 比較
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: b199525b-84de-4f79-9eb6-69a613b8b217
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: nitinme
ms.openlocfilehash: 0b374e92a1e1d9828bc8c095e29e1dfdfd13275b
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39492906"
---
# <a name="comparing-azure-data-lake-store-and-azure-blob-storage"></a>比較 Azure Data Lake Store 和 Azure Blob 儲存體
本文章中的表格摘要說明 Azure Data Lake Store 與 Azure Blob 儲存體之間的差異，以及巨量資料處理的一些重要層面。 Azure Blob 儲存體是一般用途的可調整物件存放區，針對各種不同的儲存體案例所設計。 Azure Data Lake Store 是超大規模儲存機制，已針對巨量資料分析的工作負載最佳化。

|  | Azure Data Lake Store | Azure Blob 儲存體 |
| --- | --- | --- |
| 目的 |適用於巨量資料分析工作負載的最佳化儲存機制 |適用於各種不同儲存體案例的一般用途物件存放區，包括巨量資料分析 |
| 使用案例 |批次、互動式、資料流分析、機器學習的資料，例如記錄檔、IoT 資料、點擊串流、大型資料集 |任何類型的文字或二進位資料，例如應用程式後端、備份資料、串流和一般用途資料的媒體儲存體。 此外，分析工作負載的完整支援；批次、互動式、串流分析、機器學習的資料，例如記錄檔、IoT 資料、點擊串流、大型資料集 |
| 重要概念 |Data Lake Store 帳戶包含資料夾，其中又包含儲存為檔案的資料 |儲存體帳戶包含容器，其中又包含 Blob 形式的資料 |
| Structure |階層式檔案系統 |具有扁平命名空間的物件存放區 |
| API |透過 HTTPS 的 REST API |透過 HTTP/HTTPS 的 REST API |
| 伺服器端 API |[WebHDFS 相容的 REST API](https://msdn.microsoft.com/library/azure/mt693424.aspx) |[Azure Blob 儲存體 REST API](https://msdn.microsoft.com/library/azure/dd135733.aspx) |
| Hadoop 檔案系統用戶端 |是 |是 |
| 資料作業 - 驗證 |採用 [Azure Active Directory 身分識別](../active-directory/develop/authentication-scenarios.md) |採用共用密碼 - [帳戶存取金鑰](../storage/common/storage-create-storage-account.md#manage-your-storage-account)和[共用存取簽章金鑰](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。 |
| 資料作業 - 驗證通訊協定 |OAuth 2.0。 呼叫必須包含由 Azure Active Directory 發行的有效 JWT (JSON Web 權杖) |雜湊式訊息驗證碼 (HMAC)。 呼叫必須包含透過 HTTP 要求之一部分的 Base64 編碼 SHA-256 雜湊。 |
| 資料作業 - 授權 |POSIX 存取控制清單 (ACL)  ACL 採用 Azure Active Directory 身分識別，可設為檔案或資料夾層級。 |針對帳戶層級授權 – 使用[帳戶存取金鑰](../storage/common/storage-create-storage-account.md#manage-your-storage-account)<br>針對帳戶、容器或 Blob 授權 - 使用[共用存取簽章金鑰](../storage/common/storage-dotnet-shared-access-signature-part-1.md) |
| 資料作業 - 稽核 |可用。 需詳細資訊請參閱 [這裡](data-lake-store-diagnostic-logs.md) 。 |可用 |
| 待用資料加密 |<ul><li>透明、伺服器端</li> <ul><li>使用服務管理的金鑰</li><li>在 Azure KeyVault 中使用客戶管理的金鑰</li></ul></ul> |<ul><li>透明、伺服器端</li> <ul><li>使用服務管理的金鑰</li><li>在 Azure KeyVault 中使用客戶管理的金鑰 (預覽)</li></ul><li>用戶端加密</li></ul> |
| 管理作業 (例如帳戶建立) |[角色型存取控制](../role-based-access-control/overview.md) (RBAC) |[角色型存取控制](../role-based-access-control/overview.md) (RBAC) |
| 開發人員 SDK |.NET、Java、Python、Node.js |.Net、JAVA、Python、Node.js、C++、Ruby、PHP、Go、Android、iOS |
| 分析的工作負載效能 |平行分析工作負載的效能最佳化。 高輸送量和 IOPS。 |平行分析工作負載的效能最佳化。 |
| 大小限制 |帳戶大小、檔案大小或檔案數目沒有限制 |特定限制記載在[這裡](../storage/common/storage-scalability-targets.md)。 可聯絡 [Azure 支援](https://azure.microsoft.com/support/faq/)以取得較大帳戶的限制 |
| 異地備援 |本機備援 (在一個 Azure 區域中多個資料複本） |本機備援 (LRS)、區域備援 (ZRS)、全域備援 (GRS)、讀取存取全域備援 (RA-GRS)。 詳細資訊請參閱 [這裡](../storage/common/storage-redundancy.md) |
| 服務狀態 |正式推出 |正式推出 |
| 區域可用性 |請參閱 [這裡](https://azure.microsoft.com/regions/#services) |所有 Azure 區域均可使用 |
| 價格 |請參閱 [價格](https://azure.microsoft.com/pricing/details/data-lake-store/) |請參閱 [價格](https://azure.microsoft.com/pricing/details/storage/) |


