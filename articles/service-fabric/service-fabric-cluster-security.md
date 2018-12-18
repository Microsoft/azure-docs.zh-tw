---
title: 保護 Azure Service Fabric 叢集 | Microsoft Docs
description: 深入了解 Azure Service Fabric 叢集的安全性情節，以及可用來加以實作的各種技術。
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: timlt
editor: ''
ms.assetid: 26b58724-6a43-4f20-b965-2da3f086cf8a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/14/2018
ms.author: aljo
ms.openlocfilehash: 52730ae24f4917ab593914c390df798f7f58dbde
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145551"
---
# <a name="service-fabric-cluster-security-scenarios"></a>Service Fabric 叢集安全性案例
Azure Service Fabric 叢集是您擁有的資源。 保護叢集是您的責任，從而協助避免未經授權的使用者與它們連線。 在叢集上執行生產工作負載時，安全的叢集尤其重要。 雖然可以建立不安全的叢集，但如果叢集向公用網際網路公開管理端點，匿名使用者就可以連線叢集。 不支援將不安全的叢集用作生產工作負載。 

本文是 Azure 叢集和獨立式叢集的安全性情節的概觀，以及可用來實作它們的各種技術：

* 節點對節點安全性
* 用戶端對節點安全性
* 角色型存取控制 (RBAC)

## <a name="node-to-node-security"></a>節點對節點安全性
節點對節點安全性會協助保護叢集中 VM 與電腦之間的通訊。 此安全性情節可確保只有獲得授權加入叢集的電腦可以參與裝載應用程式和叢集中的服務。

![節點對節點通訊的圖表][Node-to-Node]

在 Azure 上執行的叢集和在 Windows 上執行的獨立叢集都可以使用[憑證安全性](https://msdn.microsoft.com/library/ff649801.aspx)或針對 Windows Server 電腦使用 [Windows 安全性](https://msdn.microsoft.com/library/ff649396.aspx)。

### <a name="node-to-node-certificate-security"></a>節點對節點憑證安全性
Service Fabric 會使用您建立叢集時，在節點類型組態中指定的 X.509 伺服器憑證。 在本文結尾處，您可以看到這些憑證是什麼，以及如何取得或建立這些憑證的快速概觀。

建立叢集時，可在 Azure 入口網站中，使用 Azure Resource Manager 範本或使用獨立 JSON 範本設定憑證安全性。 Service Fabric SDK 的預設行為是部署和安裝到期日最晚的憑證；傳統行為允許定義主要和次要憑證，以允許手動起始憑證變換，但不建議用於新的功能。 即將要使用的主要憑證有最晚的到期日，應該不同於您為[用戶端對節點安全性](#client-to-node-security)設定的系統管理用戶端憑證和唯讀用戶端憑證。

若要了解如何在適用於 Azure 的叢集中設定憑證安全性，請參閱[使用 Azure Resource Manager 範本來設定叢集](service-fabric-cluster-creation-via-arm.md)。

若要了解如何在適用於獨立 Windows Server 叢集的叢集中設定憑證安全性，請參閱[使用 X.509 憑證來保護 Windows 上的獨立叢集](service-fabric-windows-cluster-x509-security.md)。

### <a name="node-to-node-windows-security"></a>節點對節點 Windows 安全性
若要了解如何設定適用於獨立 Windows Server 叢集的憑證安全性，請參閱[使用 Windows 安全性保護 Windows 上的獨立叢集](service-fabric-windows-cluster-windows-security.md)。

## <a name="client-to-node-security"></a>用戶端對節點安全性
用戶端對節點安全性會驗證用戶端，並協助保護用戶端與叢集中個別節點之間的通訊。 這個類型的安全性可協助確保只有獲得授權的使用者能存取叢集與叢集上部署的應用程式。 用戶端是透過其 Windows 安全性認證或其憑證安全性認證唯一識別。

![用戶端對節點通訊的圖表][Client-to-Node]

在 Azure 上執行的叢集或在 Windows 上執行的獨立叢集可以使用[憑證安全性](https://msdn.microsoft.com/library/ff649801.aspx)或 [Windows 安全性](https://msdn.microsoft.com/library/ff649396.aspx)。

### <a name="client-to-node-certificate-security"></a>用戶端對節點憑證安全性
建立叢集時，可在 Azure 入口網站中，使用 Resource Manager 範本或使用獨立 JSON 範本來設定用戶端對節點憑證安全性。 若要建立憑證，請指定管理員用戶端憑證或使用者用戶端憑證。 您指定的系統管理用戶端憑證和使用者用戶端憑證，應不同於您針對[節點對節點安全性](#node-to-node-security)指定的主要和次要憑證，以作為最佳做法。 節點對節點安全性的叢集憑證預設會新增到允許的用戶端系統管理員憑證清單中。

用戶端如果是使用系統管理憑證來連接到叢集，就會擁有管理功能的完整存取權。 用戶端如果是使用唯讀使用者用戶端憑證來連接到叢集，則只會擁有管理功能的唯讀存取權。 這些憑證是用於我們在本文稍後所述的 RBAC。

若要了解如何在適用於 Azure 的叢集中設定憑證安全性，請參閱[使用 Azure Resource Manager 範本來設定叢集](service-fabric-cluster-creation-via-arm.md)。

若要了解如何在適用於獨立 Windows Server 叢集的叢集中設定憑證安全性，請參閱[使用 X.509 憑證來保護 Windows 上的獨立叢集](service-fabric-windows-cluster-x509-security.md)。

### <a name="client-to-node-azure-active-directory-security-on-azure"></a>Azure 上的用戶端對節點 Azure Active Directory 安全性
Azure AD 可讓組織 (稱為租用戶) 管理使用者對應用程式的存取。 應用程式分成具有 Web 型登入 UI 的應用程式，以及具有原生用戶端體驗的應用程式。 如果您尚未建立租用戶，請從閱讀[如何取得 Azure Active Directory 租用戶][active-directory-howto-tenant]開始進行。

Service Fabric 叢集提供其管理功能的各種進入點 (包括 Web 型 [Service Fabric Explorer][service-fabric-visualizing-your-cluster] 和 [Visual Studio][service-fabric-manage-application-in-visual-studio])。 因此，您將建立兩個 Azure AD 應用程式來控制對叢集的存取：一個 Web 應用程式和一個原生應用程式。

針對在 Azure 上執行的叢集，您也可以使用 Azure Active Directory (Azure AD) 來保護對管理端點的存取。 若要了解如何建立所需的 Azure AD 構件，以及如何在建立叢集時填入這些購件，請參閱[設定 Azure AD 來驗證用戶端](service-fabric-cluster-creation-setup-aad.md)。

## <a name="security-recommendations"></a>安全性建議
對於 Azure 叢集，建議您針對節點對節點安全性使用 Azure AD 安全性來驗證用戶端和憑證。

對於獨立 Windows Server 叢集，如果您有 Windows Server 2012 R2 和 Windows Active Directory，建議您使用 Windows 安全性與群組受控服務帳戶。 否則，使用 Windows 安全性與 Windows 帳戶。

## <a name="role-based-access-control-rbac"></a>角色型存取控制 (RBAC)
您可以使用存取控制來限制存取不同使用者群組的特定叢集作業。 這樣有助於讓叢集更安全。 針對連線到叢集的用戶端，支援兩種存取控制類型：系統管理員角色和使用者角色。

指派為系統管理員角色的使用者可以完整存取管理功能，包括讀取和寫入功能。 獲指派使用者角色的使用者預設只具有管理功能的讀取存取權 (例如查詢功能)。 它們也可以解析應用程式和服務。

當您建立叢集時，請設定系統管理員和使用者用戶端角色。 為每種角色類型提供個別的身分識別 (例如，使用憑證或 Azure AD)，從而 指派角色。 如需有關預設存取控制設定及如何變更預設設定的詳細資訊，請參閱[適用於 Service Fabric 用戶端的角色型存取控制](service-fabric-cluster-security-roles.md)。

## <a name="x509-certificates-and-service-fabric"></a>X.509 憑證和 Service Fabric
X509 數位憑證通常用來驗證用戶端與伺服器。 它們也用來加密及數位簽署訊息。 Service Fabric 會使用 X.509 憑證來保護叢集，並提供應用程式的安全性功能。 如需關於 X.509 數位憑證的詳細資訊，請參閱[使用憑證](http://msdn.microsoft.com/library/ms731899.aspx)。 您可以使用 [Key Vault](../key-vault/key-vault-get-started.md) 來管理 Azure 中 Service Fabric 叢集的憑證。

一些需要考量的重要事項︰

* 若要建立執行生產環境工作負載之叢集的憑證，請使用已正確設定的 Windows Server 憑證服務，或是已核准[憑證授權單位 (CA)](https://en.wikipedia.org/wiki/Certificate_authority)中的服務。
* 請勿使用您透過諸如 MakeCert.exe 等工具，在生產環境中建立的任何暫存或測試憑證。
* 您可以使用自我簽署的憑證，但只能在測試叢集中。 請勿在生產環境中使用自我簽署的憑證。

### <a name="cluster-and-server-certificate-required"></a>叢集和伺服器憑證 (必要)
需有這些憑證 (一個主要憑證和選擇性次要憑證) 以便保護叢集，並避免未經授權的存取。 這些憑證會提供叢集和伺服器驗證。

叢集驗證會驗證叢集同盟的節點對節點通訊。 只有可使用此憑證提供其身分識別的節點可以加入叢集。 伺服器驗證會向管理用戶端驗證叢集管理端點，管理用戶端就能知道它正在交談的對象是真正的叢集，而非「中間人 (man in the middle)」。 此憑證也會為 HTTPS 管理 API 及透過 HTTPS 使用的 Service Fabric Explorer 提供 SSL。 當用戶端或節點驗證節點時，其中一項初始檢查是 [主體] 欄位中的一般名稱值。 此一般名稱或其中一個憑證主體別名 (SAN) 必須存在於允許的一般名稱清單中。

憑證必須符合下列要求：

* 憑證必須包含私密金鑰。 通常這些憑證的副檔名為 .pfx 或 .pem  
* 憑證必須是為了進行金鑰交換而建立，且可匯出成個人資訊交換 (.pfx) 檔案。
* **憑證的主體名稱必須與您用來存取 Service Fabric 叢集的網域相符**。 必須如此相符，才能為叢集的 HTTPS 管理端點和 Service Fabric Explorer 提供 SSL。 您無法從憑證授權單位 (CA) 取得 *.cloudapp.azure.com 網域的 SSL 憑證。 您必須為您的叢集取得自訂網域名稱。 當您向 CA 要求憑證時，憑證的主體名稱必須與用於您叢集的自訂網域名稱相符。

需考量的其他事項：

* [主體] 欄位可以有多個值。 每個值前面都會加上起首字母來表示實值類型。 通常，起首字母是 **CN** (針對一般名稱)；例如 **CN = www.contoso.com**。 
* [主體] 欄位可以是空白。 
* 如果選擇性 [主體別名] 欄位已填入資料，此欄位就必須具有憑證的一般名稱，以及每個 SAN 的一個項目。 這些會以 **DNS 名稱**值輸入。 若要深入了解如何產生具有 SAN 的憑證，請參閱[如何將主體別名新增至安全 LDAP 憑證](http://support.microsoft.com/kb/931351)。
* 憑證的 [預定目的] 欄位值應包含適當的值，例如**伺服器驗證**或**用戶端驗證**。

### <a name="application-certificates-optional"></a>應用程式憑證 (選用)
您可以針對應用程式安全性目的，在叢集上安裝任何數目的其他憑證。 在建立您的叢集之前，請考量需要在節點上安裝憑證的應用程式安全性案例，例如：

* 將應用程式組態值加密和解密。
* 在複寫期間將資料跨節點加密。

不論是 Linux 叢集還是 Windows 叢集，建立安全叢集的概念都是相同的。

### <a name="client-authentication-certificates-optional"></a>用戶端驗證憑證 (選擇性)
您可以針對管理員或使用者用戶端作業指定任意數目的其他憑證。 用戶端可以在需要相互驗證時使用這個憑證。 用戶端憑證通常不是由第三方 CA 所發行。 然而，目前使用者位置的個人存放區通常包含由根授權單位所放置的用戶端憑證。 憑證的**預定目的**值應是**用戶端驗證**。  

根據預設，叢集憑證具有管理員用戶端權限。 這些額外的用戶端憑證不會安裝到叢集中，但會指定為允許存在叢集組態中。  不過，用戶端憑證必須安裝在用戶端機器上，以連線到叢集並執行任何作業。

> [!NOTE]
> Service Fabric 叢集上的所有管理作業都需要伺服器憑證。 用戶端憑證無法用於管理作業。

## <a name="next-steps"></a>後續步驟
* [使用 Resource Manager 範本在 Azure 中建立叢集](service-fabric-cluster-creation-via-arm.md) 
* [使用 Azure 入口網站建立叢集](service-fabric-cluster-creation-via-portal.md)

<!--Image references-->
[Node-to-Node]: ./media/service-fabric-cluster-security/node-to-node.png
[Client-to-Node]: ./media/service-fabric-cluster-security/client-to-node.png

[active-directory-howto-tenant]:../active-directory/develop/quickstart-create-new-tenant.md
[service-fabric-visualizing-your-cluster]: service-fabric-visualizing-your-cluster.md
[service-fabric-manage-application-in-visual-studio]: service-fabric-manage-application-in-visual-studio.md
