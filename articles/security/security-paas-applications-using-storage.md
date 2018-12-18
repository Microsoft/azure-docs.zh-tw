---
title: 使用 Azure 儲存體來保護 PaaS 應用程式 | Microsoft Docs
description: 了解用來保護 PaaS Web 與行動應用程式的 Azure 儲存體安全性最佳做法。
services: security
documentationcenter: na
author: TomShinder
manager: MBaldwin
editor: ''
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/28/2018
ms.author: TomShinder
ms.openlocfilehash: ac01aaca8c147b1f474b59ac57424f5cdc5f8a8d
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451862"
---
# <a name="best-practices-for-securing-paas-web-and-mobile-applications-using-azure-storage"></a>使用 Azure 儲存體保護 PaaS Web 與行動應用程式的最佳做法
在此文章中，我們將討論用來保護平台即服務 (PaaS) Web 與行動應用程式的 Azure 儲存體安全性最佳做法。 這些最佳做法衍生自我們的 Azure 經驗和客戶 (例如您自己) 的經驗。

Azure 可讓您透過無法在內部部署中輕易實現的方式來部署和使用儲存體。 利用 Azure 儲存體，您不必大費周章就可以實現高水準的延展性和可用性。 Azure 儲存體不僅能作為 Windows 和 Linux Azure 虛擬機器的基礎，還可以支援大型的分散式應用程式。

Azure 儲存體提供以下四個服務：Blob 儲存體、表格儲存體、佇列儲存體和檔案儲存體。 若要深入了解，請參閱 [Microsoft Azure 儲存體簡介](../storage/storage-introduction.md)。

[Azure 儲存體安全性指南](../storage/common/storage-security-guide.md)是可讓您獲得 Azure 儲存體和安全性詳細資訊的絕佳來源。 這篇最佳做法文章會概括說明該安全性指南中提供的一些概念和安全性指南的連結，以及其他可供獲得詳細資訊的來源。

此文章會說明下列最佳做法：

- 共用存取簽章 (SAS)
- 角色型存取控制 (RBAC)
- 高價值資料的用戶端加密
- 儲存體服務加密


## <a name="use-a-shared-access-signature-instead-of-a-storage-account-key"></a>使用共用存取簽章而非儲存體帳戶金鑰
存取控制很重要。 為協助您控制對 Azure 儲存體的存取，Azure 會在您建立儲存體帳戶時產生兩個 512 位元的儲存體帳戶金鑰 (SAK)。 金鑰備援層級可讓您避免在金鑰例行輪替期間發生服務中斷。 

儲存體存取金鑰是高優先順序的祕密，因此只該開放給負責控制儲存體存取的人員存取。 如果讓不適當的人員獲得這些金鑰的存取權，他們就會完全控制儲存體，而能夠在儲存體中取代、刪除或新增檔案。 可能會危害組織或客戶的惡意程式碼和其他類型的內容也包括在內。

您還是要有辦法來為使用者提供儲存體物件的存取權。 若要提供更細微的存取權，您可以利用共用存取簽章 (SAS)。 SAS 可讓您在預先定義的時間間隔內，使用特定權限來共用儲存體中的特定物件。 共用存取簽章可讓您定義：

- SAS 的有效期間，包括開始時間和到期時間。
- SAS 所授與的權限。 例如，Blob 的 SAS 可能會對使用者授與該 Blob 的讀取和寫入權限，但不授與刪除權限。
- Azure 儲存體接受的 SAS 所來自的選擇性 IP 位址或 IP 位址範圍。 例如，您可以指定屬於組織的 IP 位址範圍。 這可讓 SAS 的安全性更上一層樓。
- Azure 儲存體在接受 SAS 時所透過的通訊協定。 您可以使用這個選擇性參數來限制使用 HTTPS 之用戶端的存取權。

SAS 可讓您以您想要的方式來共用內容，而不必交出儲存體帳戶金鑰。 一律在應用程式中使用 SAS 可讓您安全地共用儲存體資源，而不會外洩儲存體帳戶金鑰。

若要深入了解共用存取簽章，請參閱[使用共用存取簽章](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。 

## <a name="use-role-based-access-control"></a>使用角色型存取控制
另一種管理存取權的方式是使用[角色型存取控制](../role-based-access-control/overview.md) (RBAC)。 若使用 RBAC，您就可以根據須知事項和最低權限安全性原則來專注在為員工提供確實符合其所需的權限。 權限太多可能會讓帳戶暴露在攻擊者的威脅下。 權限太少則會讓員工無法有效率地完成工作。 RBAC 可以為 Azure 提供更細緻的存取管理來協助解決這個問題。 對於想要強制執行資料存取安全性原則的組織，這是必須做的事。

您可以利用 Azure 中內建的 RBAC 角色指派權限給使用者。 例如，將儲存體帳戶參與者用於需要管理儲存體帳戶的雲端操作者，並使用傳統儲存體帳戶參與者角色管理傳統儲存體帳戶。 對於需要管理 VM 而非其所連線的虛擬網路或儲存體帳戶的雲端操作者，您可以將他們新增至虛擬機器參與者角色。

未利用諸如 RBAC 等功能來強制執行資料存取控制的組織，可能會對其使用者提供超過所需的權限。 這可能會讓某些使用者一開始就能存取他們不應能夠存取的資料，而導致資料外洩。

若要深入了解 RBAC，請參閱：

- [使用 RBAC 和 Azure 入口網站來管理存取權](../role-based-access-control/role-assignments-portal.md)
- [適用於 Azure 資源的內建角色](../role-based-access-control/built-in-roles.md)
- [Azure 儲存體安全性指南](../storage/common/storage-security-guide.md) 

## <a name="use-client-side-encryption-for-high-value-data"></a>對高價值的資料使用用戶端加密
用戶端加密可讓您先以程式設計方式將傳輸中的資料加密再上傳到 Azure 儲存體，然後在擷取資料時以程式設計方式將資料解密。 此功能除了可以將傳輸中的資料加密，也可以將待用資料加密。 用戶端加密是最安全的資料加密方式，但會要求您透過程式設計方式變更應用程式，並將金鑰管理程序放在正確的位置上。

用戶端加密也可讓您成為唯一掌控加密金鑰的人。 您可以產生和管理自己的加密金鑰。 它使用信封技術，透過此技術，Azure 儲存體用戶端程式庫會先產生內容加密金鑰 (CEK)，隨後再使用金鑰加密金鑰 (KEK) 來包裝 (加密) 它。 KEK 由金鑰識別碼所識別，可以是非對稱金鑰組或對稱金鑰，且可以在本機管理或儲存在 [Azure Key Vault](../key-vault/key-vault-whatis.md) 中。

用戶端加密會內建在 Java 和 .NET 儲存體用戶端程式庫中。 如需有關將用戶端應用程式內資料加密並產生及管理自有加密金鑰的資訊，請參閱 [Microsoft Azure 儲存體的用戶端加密和 Azure Key Vault](../storage/storage-client-side-encryption.md)。

## <a name="enable-storage-service-encryption-for-data-at-rest"></a>啟用待用資料的儲存體服務加密
當檔案儲存體的[儲存體服務加密](../storage/storage-service-encryption.md)啟用時，系統會自動使用 AES-256 加密來將資料加密。 Microsoft 會處理所有的加密、解密和金鑰管理工作。 此功能適用於 LRS 及 GRS 備援類型。

## <a name="next-steps"></a>後續步驟

此文章介紹用來保護 PaaS Web 與行動應用程式的一組 Azure 儲存體安全性最佳做法。 若要深入了解如何保護您的 PaaS 部署，請參閱︰

- [保護 PaaS 部署](security-paas-deployments.md)
- [使用 Azure App Service 來保護 PaaS Web 與行動應用程式](security-paas-applications-using-app-services.md)
- [保護 Azure 中的 PaaS 資料庫](security-paas-applications-using-sql.md)
