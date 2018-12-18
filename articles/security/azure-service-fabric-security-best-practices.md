---
title: Azure Service Fabric 安全性最佳做法 | Microsoft Docs
description: 本文提供一組 Azure Service Fabric 安全性的最佳做法。
services: security
documentationcenter: na
author: unifycloud
manager: mbaldwin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/01/2017
ms.author: tomsh
ms.openlocfilehash: b908589903d243b2d284e2a23b6111785229c16f
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33894124"
---
# <a name="azure-service-fabric-security-best-practices"></a>Azure Service Fabric 安全性最佳做法
在 Azure 上部署應用程式很快速、輕鬆且符合成本效益。 將您的雲端應用程式部署至生產環境之前，請檢閱重要和建議最佳做法，以在應用程式中實作安全叢集。

Azure Service Fabric 是分散式系統平台，可讓您輕鬆封裝、部署及管理可調整和可信賴的微服務。 Service Fabric 也可解決開發與管理雲端應用程式時遭遇的重大挑戰。 開發人員與管理員能夠避免複雜的基礎結構問題，專注於實作關鍵且嚴格要求之可調整、可信賴且可管理的工作負載。 

針對每個最佳做法，我們會說明︰

-   最佳作法是什麼。
-   您應該實作最佳做法的原因。
-   如果您不實作最佳做法，可能會發生的狀況。
-   您學習實作最佳做法的方式。

建議下列 Azure Service Fabric 安全性最佳做法：

-   使用 Azure Resource Manager 範本和 Service Fabric PowerShell 模組來建立安全的叢集。
-   使用 X.509 憑證。
-   設定安全性原則。
-   實作 Reliable Actors 安全性設定。
-   設定 Azure Service Fabric 的 SSL。
-   使用網路隔離和安全性搭配 Azure Service Fabric。
-   設定 Azure Key Vault 以保持安全性。
-   將使用者指派給角色。


## <a name="best-practices-for-securing-your-clusters"></a>保護叢集的最佳做法

一律使用安全的叢集：
-   使用憑證來實作叢集安全性。
-   使用 Azure Active Directory (Azure AD) 來提供用戶端存取 (系統管理員和唯讀)。

使用自動化部署：
-   使用指令碼來產生、部署及變換祕密。
-   將祕密儲存在 Azure Key Vault 中，並將 Azure AD 用於其他所有的用戶端存取。
-   需經過驗證，人員才能存取祕密。

此外，請考慮下列設定選項：
-   使用 Azure 網路安全性群組 (NSG) 來建立周邊網路 (也稱為非軍事區域、DMZ 和遮蔽式子網路)。
-   使用跳躍伺服器搭配遠端桌面連線來存取叢集虛擬機器 (VM) 或管理您的叢集。

叢集必須受到安全保護，以防止未經授權使用者連線，特別是當叢集正在生產環境中執行時。 雖然可以建立不安全的叢集，但如果叢集向公用網際網路公開管理端點，匿名使用者就可以連線到您的叢集。

有三個[情節](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security)會使用各種技術來實作叢集安全性：

-   節點對節點安全性：這個情節會保護 VM 與叢集中電腦之間的通訊。 這種形式的安全性可確保只有獲得授權加入叢集的電腦可以裝載應用程式和叢集中的服務。
在此情節中，Azure 上執行的叢集或在 Windows 上執行的獨立叢集可以使用[憑證安全性](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-x509-security)或針對 Windows Server 機器使用 [Windows 安全性](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-windows-security)。
-   用戶端對節點安全性：這個情節會保護 Service Fabric 用戶端與叢集中個別節點之間的通訊。
-   角色型存取控制 (RBAC)：這個情節會針對存取叢集的每個系統管理員和使用者用戶端角色使用不同的身分識別 (憑證、Azure AD 等等)。 當您建立叢集時，可以指定角色身分識別。

>[!NOTE]
>**Azure 叢集的安全性建議：** 針對節點對節點安全性使用 Azure AD 安全性來驗證用戶端和憑證。

若要設定獨立 Windows 叢集，請參閱[設定獨立 Windows 叢集的設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-manifest)。

使用 Azure Resource Manager 範本和 Service Fabric PowerShell 模組來建立安全的叢集。
如需使用 Azure Resource Manager 範本來建立安全 Service Fabric 叢集的逐步指示，請參閱[建立 Service Fabric 叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)。

使用 Azure Resource Manager 範本：
-   使用範本來設定受控 VM 虛擬硬碟 (VHD) 的存放裝置，以自訂您的叢集。
-   使用範本輕鬆進行組態管理及稽核，對資源群組進行變更。

將叢集設定視為程式碼：
-   請徹底檢查部署組態。
-   避免使用隱含命令直接修改您的資源。

[Service Fabric 應用程式生命週期](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-lifecycle) 的許多層面都可以自動化。 [Service Fabric PowerShell 模組](https://docs.microsoft.com/azure/service-fabric/service-fabric-deploy-remove-applications#upload-the-application-package) 會將部署、升級、移除和測試 Azure Service Fabric 應用程式的常見工作自動化。 也可使用應用程式管理的受控 API 和 HTTP API。

## <a name="use-x509-certificates"></a>使用 X.509 憑證
請務必使用 X.509 憑證或 Windows 安全性來保護叢集。 僅在叢集建立期間才能設定安全性。 建立叢集之後，就無法開啟安全性。

如需指定[叢集憑證](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-x509-security)，請將 **ClusterCredentialType** 屬性的值設定為 X509。 如需指定外部連線的伺服器憑證，請將 **ServerCredentialType** 屬性設定為 X509。

此外，請遵循下列準則：
-   使用正確設定的 Windows 伺服器憑證服務，以建立生產叢集的憑證。 您也可以從核准的憑證授權單位 (CA) 取得憑證。
-   如果是使用 MakeCert.exe 或類似工具來建立憑證，請勿針對生產叢集使用暫存或測試憑證。
-   針對測試叢集，而非針對生產叢集使用自我簽署的憑證。

如果叢集不安全，任何人都可以匿名方式連線到叢集並執行管理作業。 基於這個理由，請務必使用 X.509 憑證或 Windows 安全性來保護生產叢集。

若要深入了解有關使用 X.509 憑證的資訊，請參閱[新增或移除 Service Fabric 叢集的憑證](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security-update-certs-azure)。

## <a name="configure-security-policies"></a>設定安全性原則
Service Fabric 也可保護應用程式所使用的資源。 部署應用程式時，諸如檔案、目錄和憑證等資源會儲存在使用者帳戶下。 即使在共用主控環境中，這個功能也能讓執行中的應用程式就不會彼此干擾。

-   使用 Active Directory 網域群組或使用者：以根據 Active Directory 使用者或群組帳戶的認證來執行服務。 請務必在您網域內，而非 Azure Active Directory 使用 Active Directory 內部部署。 使用網域使用者或群組，存取網域中已經被授與權限的其他資源。 例如，檔案共用等資源。

-   指派 HTTP 和 HTTPS 端點的安全性存取原則：當服務資訊清單使用 HTTP 宣告端點資源時，指定 **SecurityAccessPolicy** 屬性，將 **RunAs** 原則套用至服務。 配置給 HTTP 端點的通訊埠是針對執行服務的 RunAs 使用者帳戶正確存取控制的清單。 未設定原則時，http.sys 無法存取服務，而且您從用戶端呼叫時會失敗。

若要了解如何在 Service Fabric 叢集中啟用安全性原則，請參閱[設定應用程式的安全性原則](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-runas-security)。

## <a name="implement-the-reliable-actors-security-configuration"></a>實作 Reliable Actors 安全性設定
Service Fabric Reliable Actors 是動作項目設計模式的實作。 如同任何軟體設計模式，是以軟體問題是否符合模式作為基礎來決定使用特定模式的決策。

一般情況下，針對下列軟體問題或安全性情節，可使用動作項目設計模式來協助模型解決方案：
-   您的問題領域涉及大量 (數千個或更多) 小型、獨立且隔離的狀態和邏輯單位。
-   您使用的是單一執行緒物件，這類物件不需要與外部元件進行顯著的互動，包括跨一組動作項目查詢狀態。
-   您的動作項目執行個體不會藉由發出 I/O 作業，使用無法預期的延遲來封鎖呼叫端。

在 Service Fabric 中，動作項目是在 Reliable Actors 應用程式架構中進行實作。 此架構是以動作項目模式作為基礎，建置於 [Service Fabric Reliable Services](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-introduction) 之上。 您撰寫的每個 Reliable Actor 服務都是已分割的具狀態可靠服務。

每個動作項目都會定義為動作項目類型的執行個體，與 .NET 物件是 .NET 類型的執行個體的方式完全相同。 例如，**動作項目類型**會實作計算機的功能，而該類型會有許多動作項目散發到叢集的各種節點上。 每個分散式動作項目都唯一具備動作項目識別碼。

[複寫器安全性設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-actors-kvsactorstateprovider-configuration)用來保護在複寫期間使用的通訊通道。 此設定會使服務無法看到彼此的複寫流量，並可確保高可用性資料的安全。 依預設，空白的安全性組態區段會妨礙複寫安全性。
複寫器組態會設定負責讓動作項目狀態提供者狀態高度可靠的複寫器。

## <a name="configure-ssl-for-azure-service-fabric"></a>設定 Azure Service Fabric 的 SSL
伺服器驗證程序會[驗證](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)管理用戶端的叢集管理端點。 接著，管理用戶端會辨識其是與實際的叢集在交談。 此憑證也會為 HTTPS 管理 API 及透過 HTTPS 使用的 Service Fabric Explorer 提供 [SSL](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)。
您必須為您的叢集取得自訂網域名稱。 當您向憑證授權單位要求憑證時，憑證的主體名稱必須與用於您叢集的自訂網域名稱相符。

若要設定應用程式的 SSL，您必須先取得已由 CA 簽署的 SSL 憑證。 CA 是受信任的第三方，會就 SSL 安全性目的發出憑證。 如果您還沒有 SSL 憑證，就必須向銷售 SSL 憑證的公司取得。

憑證必須符合 Azure 中對於 SSL 憑證的下列要求：
-   憑證必須包含私密金鑰。

-   憑證必須是為了進行金鑰交換而建立，且可匯出成個人資訊交換檔 (.pfx)。

-   憑證的主體名稱必須符合用來存取您雲端服務的網域名稱。

    - 取得要在存取雲端服務時使用的自訂網域名稱。
    - 向 CA 要求包含主體名稱的憑證，可符合您服務的自訂網域名稱。 例如，如果您的自訂網域名稱為 __contoso__**.com**，您 CA 的憑證就要有 **.contoso.com** 或 __www__**.contoso.com** 主體名稱。

    >[!NOTE]
    >您無法從 CA 取得 __cloudapp__**.net** 網域的 SSL 憑證。
    
-   憑證至少必須以 2,048 位元加密。

HTTP 通訊協定並不安全，且會受到竊聽攻擊。 透過 HTTP 傳輸的資料是以純文字形式從網頁瀏覽器傳送到 web 伺服器，或是在其他端點之間傳送。 攻擊者可以攔截並檢視透過 HTTP 傳送的敏感性資料，例如信用卡詳細資料和帳戶登入。 當資料是透過 HTTPS 經由瀏覽器傳送或張貼時，SSL 可確保敏感性資訊經過加密及保護以避免遭到攔截。

若要深入了解使用 SSL 憑證的資訊，請參閱[設定 Azure 應用程式的 SSL](https://docs.microsoft.com/azure/cloud-services/cloud-services-configure-ssl-certificate)。

## <a name="use-network-isolation-and-security-with-azure-service-fabric"></a>使用網路隔離和安全性搭配 Azure Service Fabric
使用 [Azure Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates)作為範例來設定 3 節點類型安全叢集。 使用範本和網路安全性群組來控制傳入和傳出網路流量。

此範本針對每個虛擬機器擴展集各有一個 NSG，且會用來控制進出集合的流量。 預設會設定規則，允許範本中指定的系統服務和應用程式連接埠所需的所有流量。 請檢閱這些規則並進行任何變更以符合您的需求，包括為您的應用程式新增規則。

如需詳細資訊，請參閱 [Azure Service Fabric 的常見網路情節](https://docs.microsoft.com/azure/service-fabric/service-fabric-patterns-networking)。

## <a name="set-up-azure-key-vault-for-security"></a>設定 Azure Key Vault 以提供安全性
Service Fabric 會使用憑證來提供驗證與加密，以保護叢集和其應用程式。

Service Fabric 會使用 X.509 憑證來保護叢集，並提供應用程式的安全性功能。 您可以使用 Azure Key Vault 來[管理 Azure 中 Service Fabric 叢集的憑證](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security-update-certs-azure)。 建立叢集的 Azure 資源提供者會從 Key Vault 提取憑證。 接著，當您在 Azure 上部署叢集時，提供者會在 VM 上安裝憑證。

[Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault)、Service Fabric 叢集以及使用憑證的資源提供者之間有憑證關聯性。 建立叢集時，憑證關聯性的相關資訊會儲存在 Key Vault 中。

設定 Key Vault 有兩個基本步驟：
1. 特別針對 Key Vault 建立資源群組。

    建議您將 Key Vault 放在其自己的資源群組中。 如果移除其他資源群組，例如儲存體、計算或包含您叢集的群組，此動作有助於避免您遺失金鑰和祕密。 含有您金鑰保存庫的資源群組必須與正在使用它的叢集位於相同區域。

2. 在新的資源群組中建立 Key Vault。

    部署時，必須啟用 Key Vault。 接著，計算資源提供者可以從保存庫中取得憑證，並加以安裝在 VM 執行個體上。

若要深入了解如何設定 Key Vault 的相關資訊，請參閱[開始使用 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-get-started)。

## <a name="assign-users-to-roles"></a>將使用者指派給角色
建立應用程式來代表您的叢集之後，請將使用者指派給 Service Fabric 所支援的角色︰唯讀和系統管理員。您可以使用 Azure 入口網站來指派這些角色。

>[!NOTE]
> 如需在 Service Fabric 中使用角色的詳細資訊，請參閱[角色型存取控制 (適用於 Service Fabric 用戶端)](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security-roles)。

Azure Service Fabric 針對連線到 [Service Fabric 叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)的用戶端，支援兩種存取控制類型：系統管理員和使用者。 叢集系統管理員可以使用存取控制來限制存取不同使用者群組的特定叢集作業。 存取控制可讓叢集更安全。

## <a name="next-steps"></a>後續步驟
- 設定 Service Fabric [開發環境](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started)。
- 了解 [Service Fabric 支援選項](https://docs.microsoft.com/azure/service-fabric/service-fabric-support)。
