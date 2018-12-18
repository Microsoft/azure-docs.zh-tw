---
title: 使用 Azure Active Directory 在 Azure Data Lake Storage Gen1 中進行驗證 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 向 Azure Data Lake Storage Gen1 進行驗證
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: b74696be433b00a5afb3416295b6317975aa6873
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967654"
---
# <a name="authentication-with-azure-data-lake-storage-gen1-using-azure-active-directory"></a>使用 Azure Active Directory 向 Azure Data Lake Storage Gen1 進行驗證

Azure Data Lake Storage Gen1 使用 Azure Active Directory 進行驗證。 撰寫搭配 Data Lake Storage Gen1 的應用程式之前，必須決定要如何向 Azure Active Directory (Azure AD) 驗證應用程式。

## <a name="authentication-options"></a>驗證選項

* **使用者驗證** - 運用使用者的 Azure 認證向 Data Lake Storage Gen1 進行驗證。 您建立來搭配 Data Lake Storage Gen1 的應用程式會提示這些使用者認證。 因此，此驗證機制是*互動式*的，應用程式會在已登入使用者的執行內容中執行。 如需詳細資訊和指示，請參閱 [Data Lake Storage Gen1 的使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)。

* **服務對服務驗證** - 如果您希望應用程式向 Data Lake Storage Gen1 驗證自己，請使用此選項。 在這種情況下，您要建立 Azure Active Directory (AD) 應用程式，並使用來自 Azure AD 應用程式的金鑰向 Data Lake Storage Gen1 驗證。 因此，此驗證機制是*非互動式*的。 如需詳細資訊和指示，請參閱 [Data Lake Storage Gen1 的服務對服務驗證](data-lake-store-service-to-service-authenticate-using-active-directory.md)。

下表說明如何為 Data Lake Storage Gen1 支援使用者驗證機制和服務對服務驗證機制。 以下是讀取表格的方式。

* ✔ * 符號表示支援該驗證選項，以及示範如何使用該驗證選項的文章連結。 
* ✔ 符號表示支援該驗證選項。 
* 空白儲存格表示不支援該驗證選項。


|使用與此驗證選項搭配...                   |.NET         |Java     |PowerShell |Azure CLI | Python   |REST     |
|:---------------------------------------------|:------------|:--------|:----------|:-------------|:---------|:--------|
|使用者 (沒有 MFA**)                        |   ✔ |    ✔    |    ✔      |       ✔      |    **[✔*](data-lake-store-end-user-authenticate-python.md#end-user-authentication-without-multi-factor-authentication)** (已取代)     |    **[✔*](data-lake-store-end-user-authenticate-rest-api.md)**    |
|使用者 (有 MFA)                           |    **[✔*](data-lake-store-end-user-authenticate-net-sdk.md)**        |    **[✔*](data-lake-store-end-user-authenticate-java-sdk.md)**     |    ✔      |       **[✔*](data-lake-store-get-started-cli-2.0.md)**      |    **[✔*](data-lake-store-end-user-authenticate-python.md#end-user-authentication-with-multi-factor-authentication)**     |    ✔    |
|服務對服務 (使用用戶端金鑰)         |    **[✔*](data-lake-store-service-to-service-authenticate-net-sdk.md#service-to-service-authentication-with-client-secret)** |    **[✔*](data-lake-store-service-to-service-authenticate-java.md)**    |    ✔      |       ✔      |    **[✔*](data-lake-store-service-to-service-authenticate-python.md#service-to-service-authentication-with-client-secret-for-account-management)**     |    **[✔*](data-lake-store-service-to-service-authenticate-rest-api.md)**    |
|服務對服務 (使用用戶端憑證) |    **[✔*](data-lake-store-service-to-service-authenticate-net-sdk.md#service-to-service-authentication-with-certificate)**        |    ✔    |    ✔      |       ✔      |    ✔     |    ✔    |

<i>* 按一下<b>✔\* </b>符號。它是連結。</i><br>
<i>** MFA 是指多重要素驗證</i>

如需如何使用 Azure Active Directory 進行驗證的詳細資訊，請參閱 [Azure Active Directory 的驗證案例](../active-directory/develop/authentication-scenarios.md)。

## <a name="next-steps"></a>後續步驟

* [使用者驗證](data-lake-store-end-user-authenticate-using-active-directory.md)
* [服務對服務驗證](data-lake-store-service-to-service-authenticate-using-active-directory.md)


