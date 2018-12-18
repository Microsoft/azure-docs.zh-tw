---
title: 保護 Azure 中的 PaaS 資料庫 | Microsoft Docs
description: '了解用來保護 PaaS Web 與行動應用程式的 Azure SQL Database 和 SQL 資料倉儲安全性最佳做法。 '
services: security
documentationcenter: na
author: techlake
manager: MBaldwin
editor: ''
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/28/2018
ms.author: terrylan
ms.openlocfilehash: 72d5ec09becc1f1d9e23e284e18bcc037ccb3072
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452474"
---
# <a name="best-practices-for-securing-paas-databases-in-azure"></a>保護 Azure 中 PaaS 資料庫的最佳做法

在此文章中，我們將說明用來保護平台即服務 (PaaS) Web 與行動應用程式的 [Azure SQL Database](../sql-database/sql-database-technical-overview.md) 和 [SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)安全性最佳做法。 這些最佳做法衍生自我們的 Azure 經驗和客戶 (例如您自己) 的經驗。

Azure SQL Database 和 SQL 資料倉儲可為您的網際網路型應用程式提供關聯式資料庫服務。 讓我們來了解當使用 PaaS 部署中的 Azure SQL Database 和「SQL 資料倉儲」時，可協助保護您應用程式和資料的服務：

- Azure Active Directory 驗證 (而不是 SQL Server 驗證)
- Azure SQL 防火牆
- 透明資料加密 (TDE)

## <a name="use-a-centralized-identity-repository"></a>使用集中式身分識別存放庫
您可以將 Azure SQL Database 設定成使用下列兩種驗證類型其中之一：

- **SQL 驗證**會使用使用者名稱和密碼。 當您為資料庫建立邏輯伺服器時，採取使用者名稱和密碼指定了「伺服器管理員」登入。 使用這些認證時，您能夠以資料庫擁有者身分向該伺服器上的任何資料庫進行驗證。

- **Azure Active Directory 驗證**會使用由 Azure Active Directory 管理的身分識別，並且受控網域和整合式網域都支援此驗證。 若要使用「Azure Active Directory 驗證」，您必須建立另一個名為「Azure AD 管理員」的伺服器管理員，此管理員能夠管理 Azure AD 使用者和群組。 此管理員也可以執行一般伺服器管理員可執行的所有作業。

[Azure Active Directory 驗證](../active-directory/develop/authentication-scenarios.md)是使用 Azure Active Directory (AD) 中的身分識別來連線到 Azure SQL Database 和「SQL 資料倉儲」的機制。 Azure AD 提供一個 SQL Server 驗證替代方案，可讓您停止在各個資料庫伺服器擴散使用者身分識別。 Azure AD 驗證可讓您在一個中央位置集中管理資料庫使用者及其他 Microsoft 服務的身分識別。 中央識別碼管理提供單一位置以管理資料庫使用者並簡化權限管理。  

### <a name="benefits-of-using-azure-ad-instead-of-sql-authentication"></a>使用 Azure AD 而不使用 SQL 驗證的好處
- 允許在單一位置變換密碼。
- 使用外部 Azure AD 群組來管理資料庫權限。
- 透過啟用整合式 Windows 驗證和 Azure AD 支援的其他形式驗證來避免儲存密碼。
- 使用自主資料庫使用者，在資料庫層級驗證身分。
- 針對連線到 SQL Database 的應用程式支援權杖型驗證。
- 支援 Active Directory Federation Services (ADFS) 的網域同盟或本機 Azure AD 的原生使用者/密碼驗證，而不需進行網域同步處理。
- 支援來自 SQL Server Management Studio 之使用「Active Directory 通用驗證」(包括 [Multi-Factor Authentication (MFA)](../active-directory/authentication/multi-factor-authentication.md)) 的連線。 MFA 包含增強式驗證功能，其中提供一系列簡易的驗證選項，例如電話、簡訊、含有 PIN 的智慧卡或行動應用程式通知。 如需詳細資訊，請參閱 [SQL Database 和 SQL 資料倉儲的通用驗證](../sql-database/sql-database-ssms-mfa-authentication.md)。

若要深入了解 Azure AD 驗證，請參閱：

- [使用 Azure Active Directory 驗證向 SQL Database、受控執行個體或 SQL 資料倉儲進行驗證](../sql-database/sql-database-aad-authentication.md)
- [適用於 Azure SQL 資料倉儲的驗證](../sql-data-warehouse/sql-data-warehouse-authentication.md)
- [使用 Azure AD 驗證之 Azure SQL DB 的權杖型驗證支援 (英文)](../sql-database/sql-database-aad-authentication.md)

> [!NOTE]
> 若要確定 Azure Active Directory 適合您的環境，請參閱 [Azure AD 功能和限制](../sql-database/sql-database-aad-authentication.md#azure-ad-features-and-limitations)。
>
>

## <a name="restrict-access-based-on-ip-address"></a>根據 IP 位址限制存取
您可以建立指定可接受之 IP 位址範圍的防火牆規則。 這些規則可以同時以伺服器和資料庫層級作為目標。 建議您儘可能使用資料庫層級防火牆規則來增強安全性，並且讓您的資料庫更具有可攜性。 伺服器層級防火牆規則最適用於系統管理員，以及當您有許多資料庫具有相同的存取需求，但您又不想花時間個別設定每個資料庫的情況。

SQL Database 的預設來源 IP 位址限制會允許來自任何 Azure 位址 (包括其他訂用帳戶和租用戶) 的存取。 您可以將其限制為只允許您的 IP 位址存取該執行個體。 即使在有 SQL 防火牆和 IP 位址限制的情況下，仍然需要增強式驗證。 請參閱此文章稍早所提出的建議。

若要深入了解 Azure SQL 防火牆和 IP 限制，請參閱：

- [Azure SQL Database 和 SQL 資料倉儲存取控制](../sql-database/sql-database-control-access.md)
- [Azure SQL Database 和 SQL 資料倉儲防火牆規則](../sql-database/sql-database-firewall-configure.md)


## <a name="encrypt-data-at-rest"></a>加密待用資料
[透明資料加密 (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) 預設為啟用。 TDE 會透明加密 SQL Server、Azure SQL Database 和 Azure SQL 資料倉儲資料和記錄檔。 TDE 會保護對檔案或其備份的直接存取，免於遭受入侵。 這可讓您加密待用資料，而不需要變更現有應用程式。 TDE 應保持啟用，不過這無法阻止攻擊者使用一般存取路徑。 TDE 提供遵守各種產業中所確立的眾多法律、規定及指導方針的功能。

Azure SQL 會管理 TDE 的金鑰相關問題。 如同 TDE，在移動資料庫時，內部部署必須特別小心，如此才能保障復原能力。 在更複雜的情況下，您可以透過可延伸金鑰管理在 Azure Key Vault 中明確管理金鑰。 請參閱[使用 EKM 在 SQL Server 上啟用 TDE](/sql/relational-databases/security/encryption/enable-tde-on-sql-server-using-ekm)。 也可以透過 Azure Key Vault BYOK 功能，實行自備金鑰 (BYOK)。

Azure SQL 可透過 [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) 提供資料行加密。 如此僅有獲得授權的應用程式得以存取敏感的資料行。 使用此類加密可讓加密資料行的 SQL 查詢限制於相等值。

應用程式層級加密也應該用於選擇性資料。 透過使用保存在正確國家/地區的金鑰來加密資料，有時即可緩和資料主權問題的疑慮。 這甚至可以防止意外資料轉送造成問題，因為在沒有金鑰的情況下，就無法將資料解密，這裡是假設使用增強式演算法 (例如 AES 256)。

您可以採取其他預防措施來協助保護資料庫，例如設計安全系統、加密機密資產，以及建置圍繞資料庫伺服器的防火牆。

## <a name="next-steps"></a>後續步驟
此文章介紹用來保護 PaaS Web 與行動應用程式的一組 SQL Database 和「SQL 資料倉儲」安全性最佳做法。 若要深入了解如何保護您的 PaaS 部署，請參閱︰

- [保護 PaaS 部署](security-paas-deployments.md)
- [使用 Azure App Service 來保護 PaaS Web 與行動應用程式](security-paas-applications-using-app-services.md)
