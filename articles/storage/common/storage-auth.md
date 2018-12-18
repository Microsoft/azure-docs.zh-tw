---
title: 授權存取 Azure 儲存體 | Microsoft Docs
description: 了解可授權存取 Azure 儲存體的不同方式，包括 Azure Active Directory、共用金鑰驗證或共用存取簽章。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 05/18/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: ab4f9d6cbdbc047d688b57e906ea60aec6fff2fa
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39529713"
---
# <a name="authorizing-access-to-azure-storage"></a>授權存取 Azure 儲存體

每次存取儲存體帳戶中的資料時，用戶端都會透過 HTTP/HTTPS 對 Azure 儲存體發出要求。 針對受保護資源所發出的每個要求都必須經過授權，服務才能確保用戶端具有所需的資料存取權限。 Azure 儲存體提供了下列選項，供您授權存取受保護的資源：

- **Azure Active Directory (Azure AD) 整合 (預覽)**，適用於 Blob 和佇列。 Azure AD 提供了角色型存取控制 (RBAC)，供您更細微地控制用戶端對儲存體帳戶資源的存取。 如需詳細資訊，請參閱[使用 Azure Active Directory 來驗證對 Azure 儲存體所發出的要求 (預覽)](storage-auth-aad.md)。
- **共用金鑰授權**，適用於 Blob、檔案、佇列和資料表。 使用共用金鑰的用戶端會在每個要求中，傳遞使用儲存體帳戶存取金鑰所簽署的標頭。 如需詳細資訊，請參閱[使用共用金鑰進行授權](https://docs.microsoft.com/rest/api/storageservices/authenticate-with-shared-key/)。
- **共用存取簽章**，適用於 Blob、檔案、佇列和資料表。 共用存取簽章 (SAS) 可提供儲存體帳戶資源的有限委派存取權。 對簽章的有效時間間隔或對其授與的權限新增條件約束，可讓您彈性管理存取權。 如需詳細資訊，請參閱[使用共用存取簽章 (SAS)](storage-dotnet-shared-access-signature-part-1.md)。
- **匿名公開讀取存取**，適用於容器和 Blob。 不需要授權。 如需詳細資訊，請參閱 [管理對容器與 Blob 的匿名讀取權限](../blobs/storage-manage-access-to-resources.md)。  

根據預設，Azure 儲存體中的所有資源都會受到保護，並只供帳戶擁有者使用。 雖然您可以使用上述任何授權策略來對用戶端授與儲存體帳戶資源的存取權，但 Microsoft 建議您盡可能使用 Azure AD，以獲得最大安全性並方便您使用。 



