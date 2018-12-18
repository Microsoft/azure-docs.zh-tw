---
title: 存取防火牆後面的金鑰保存庫 | Microsoft Docs
description: 了解如何從防火牆後面的應用程式存取 Azure 金鑰保存庫
services: key-vault
documentationcenter: ''
author: amitbapat
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 50d21774-2ee1-4212-8995-570c9de603c5
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 01/07/2017
ms.author: ambapat
ms.openlocfilehash: 6bb757fe072f0fc3ecbda5ae125aa2bf928479fb
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41919106"
---
# <a name="access-azure-key-vault-behind-a-firewall"></a>在防火牆後存取 Azure 金鑰保存庫
### <a name="q-my-key-vault-client-application-needs-to-be-behind-a-firewall-what-ports-hosts-or-ip-addresses-should-i-open-to-enable-access-to-a-key-vault"></a>問︰我的金鑰保存庫用戶端應用程式必須位於防火牆後。 我應該開啟哪些連接埠、主機或 IP 位址以啟用金鑰保存庫的存取權？
若要存取金鑰保存庫，您的金鑰保存庫用戶端應用程式必須存取多個端點才能使用各種功能：

* 透過 Azure Active Directory (Azure AD) 進行的驗證。
* Azure 金鑰保存庫的管理。 這包括透過 Azure Resource Manager 建立、讀取、更新、刪除和設定存取原則。
* 經由金鑰保存庫專用端點 (例如 [https://yourvaultname.vault.azure.net](https://yourvaultname.vault.azure.net))，存取和管理金鑰保存庫本身儲存的物件 (金鑰和密碼)。  

視您的組態和環境而定，會有一些變化。   

## <a name="ports"></a>連接埠
三項功能 (驗證、管理和資料平面存取) 的所有金鑰保存庫流量都會透過 HTTPS︰連接埠 443 傳送。 不過，偶爾會有 CRL 的 HTTP (連接埠 80) 流量。 支援 OCSP 的用戶端不應該觸達 CRL，但可能偶爾會觸達 [http://cdp1.public-trust.com/CRL/Omniroot2025.crl](http://cdp1.public-trust.com/CRL/Omniroot2025.crl)。  

## <a name="authentication"></a>驗證
金鑰保存庫用戶端應用程式必須存取 Azure Active Directory 端點以便驗證。 使用的端點取決於 Azure AD 租用戶組態、主體類型 (使用者主體或服務主體) 和帳戶類型 (例如 Microsoft 帳戶，或是公司帳戶或學校帳戶)。  

| 主體類型 | 端點:連接埠 |
| --- | --- |
| 使用 Microsoft 帳戶的使用者<br> (例如，user@hotmail.com) |**全域：**<br> login.microsoftonline.com:443<br><br> **Azure 中國︰**<br> login.chinacloudapi.cn:443<br><br>**Azure 美國政府︰**<br> login.microsoftonline.us:443<br><br>**Azure 德國︰**<br> login.microsoftonline.de:443<br><br> 和 <br>login.live.com:443 |
| 搭配使用公司帳戶或學校帳戶與 Azure AD 的使用者或服務主體 (例如，user@contoso.com) |**全域：**<br> login.microsoftonline.com:443<br><br> **Azure 中國︰**<br> login.chinacloudapi.cn:443<br><br>**Azure 美國政府︰**<br> login.microsoftonline.us:443<br><br>**Azure 德國︰**<br> login.microsoftonline.de:443 |
| 使用公司帳戶或學校帳戶加上 Active Directory 同盟服務 (AD FS) 或其他同盟端點的使用者或服務主體 (例如，user@contoso.com) |公司帳戶或學校帳戶的所有端點，加上 AD FS 或其他同盟端點 |

還有其他可能的複雜案例。 如需其他資訊，請參閱 [Azure Active Directory 驗證流程](../active-directory/develop/authentication-scenarios.md)、[整合應用程式與 Azure Active Directory](../active-directory/develop/active-directory-how-to-integrate.md) 及 [Active Directory 驗證通訊協定](https://msdn.microsoft.com/library/azure/dn151124.aspx)。  

## <a name="key-vault-management"></a>金鑰保存庫管理
對於金鑰保存庫管理 (CRUD 和設定存取原則)，金鑰保存庫用戶端應用程式需要存取 Azure Resource Manager 端點。  

| 作業類型 | 端點:連接埠 |
| --- | --- |
| 透過 Azure Resource Manager 的<br> 金鑰保存庫控制項面作業 |**全域：**<br> management.azure.com:443<br><br> **Azure 中國︰**<br> management.chinacloudapi.cn:443<br><br> **Azure 美國政府︰**<br> management.usgovcloudapi.net:443<br><br> **Azure 德國︰**<br> management.microsoftazure.de:443 |
| Azure Active Directory 圖形 API |**全域：**<br> graph.windows.net:443<br><br> **Azure 中國︰**<br> graph.chinacloudapi.cn:443<br><br> **Azure 美國政府︰**<br> graph.windows.net:443<br><br> **Azure 德國︰**<br> graph.cloudapi.de:443 |

## <a name="key-vault-operations"></a>金鑰保存庫作業
對於所有金鑰保存庫物件 (金鑰和密碼) 管理和密碼編譯作業，金鑰保存庫用戶端需要存取金鑰保存庫端點。 視金鑰保存庫的位置而定，端點 DNS 尾碼會有所不同。 金鑰保存庫端點的格式為 vault-name.region-specific-dns-suffix，如下表所述。  

| 作業類型 | 端點:連接埠 |
| --- | --- |
| 包括金鑰的密碼編譯作業在內的作業；建立、讀取、更新和刪除金鑰與密碼；對金鑰保存庫物件 (金鑰或密碼) 設定或取得標籤和其他屬性 |**全域：**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Azure 中國︰**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure 美國政府︰**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure 德國︰**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 |

## <a name="ip-address-ranges"></a>IP 位址範圍
金鑰保存庫服務會使用其他 Azure 資源，例如 PaaS 基礎結構。 因此，不可能提供金鑰保存庫服務端點在任何特定時間會有的特定 IP 位址範圍。 如果您的防火牆只支援 IP 位址範圍，請參閱 [Microsoft Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)文件。 驗證和身分識別 (Azure Active Directory) 是全域服務，可能會在沒有通知的情況下，容錯移轉至其他地區或移動流量。 在此案例中，[驗證和身分識別 IP 位址](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity_ip)中所列出的 IP 範圍全都應該新增至防火牆。

## <a name="next-steps"></a>後續步驟
如果您有關於金鑰保存庫的問題，請造訪 [Azure 金鑰保存庫論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=AzureKeyVault)。

