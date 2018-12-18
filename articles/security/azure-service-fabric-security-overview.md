---
title: Azure Service Fabric 安全性概觀 | Microsoft Docs
description: 本文提供 Azure Service Fabric 安全性概觀。
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
ms.date: 08/04/2017
ms.author: tomsh
ms.openlocfilehash: 629b6fba9ced5fa2ccf22f473fe25c87d1cc4818
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37436805"
---
# <a name="azure-service-fabric-security-overview"></a>Azure Service Fabric 安全性概觀
[Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-overview) 是分散式系統平台，可讓您輕鬆封裝、部署及管理可調整和可信賴的微服務。 Service Fabric 可解決開發與管理雲端應用程式時遭遇的挑戰。 開發人員與管理員能夠避免複雜的基礎結構問題，全心實作可調整、可信賴的關鍵性高需求工作負載。

本文將概略說明 Service Fabric 部署的安全性考量。

## <a name="secure-your-cluster"></a>保護叢集
Azure Service Fabric 是跨電腦叢集的服務協調者。 叢集必須受到安全保護，以防止未經授權使用者加以連線，特別是在它們執行生產工作負載時。 雖然可以建立不安全的叢集，但這樣做會在叢集向公用網際網路公開管理端點時，允許匿名使用者連線至叢集。

對於單獨執行或在 Azure 上執行的叢集，應考量的兩種案例為節點對節點的安全性和用戶端對節點的安全性。 您可以使用不同的技術來實作這些案例。

### <a name="node-to-node-security"></a>節點對節點安全性
節點對節點的安全性會套用至叢集中的 VM 與機器之間的通訊。 使用節點對節點安全性，只有獲得授權加入叢集的電腦可以參與裝載應用程式和叢集中的服務。

在 Azure 上執行的叢集或在 Windows 上執行的獨立叢集可以使用[憑證安全性](https://msdn.microsoft.com/library/ff649801.aspx)或針對 Windows Server 機器使用 [Windows 安全性](https://msdn.microsoft.com/library/ff649396.aspx)。

#### <a name="node-to-node-certificate-security"></a>節點對節點憑證安全性

Service Fabric 會使用您建立叢集時指定的 X.509 伺服器憑證。 若要快速了解這些憑證是什麼，以及如何取得或建立這些憑證，請參閱[使用憑證](https://docs.microsoft.com/dotnet/framework/wcf/feature-details/working-with-certificates)。

您在透過 Azure 入口網站、Azure Resource Manager 範本或獨立 JSON 範本建立叢集時，會設定憑證安全性。 您可以指定用於憑證變換的主要憑證和選用次要憑證。 您指定的主要和次要憑證，應該不同於您為[用戶端對節點安全性](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security)指定的系統管理員用戶端憑證和唯讀用戶端憑證。

### <a name="client-to-node-security"></a>用戶端對節點安全性
您使用用戶端身分識別來設定用戶端對節點安全性。 若要建立用戶端與叢集之間的信任，您必須設定叢集才能知道用戶端可以信任的身分識別。

Service Fabric 針對連線至 Service Fabric 叢集的用戶端，可支援兩種存取控制類型：

-   **系統管理員**：可完整存取管理功能，包括讀取/寫入功能。
-   **使用者**：僅具有管理功能的讀取存取權 (例如查詢功能)，以及解析應用程式和服務的能力。

使用存取控制，叢集系統管理員可以限制特定類型叢集作業的存取權。 這可讓叢集更安全。

#### <a name="client-to-node-certificate-security"></a>用戶端對節點憑證安全性

您在透過 Azure 入口網站、Resource Manager 範本或獨立 JSON 範本建立叢集時，會設定用戶端對節點的憑證安全性。 您需要指定管理員用戶端憑證和/或使用者用戶端憑證。 請確定這些憑證不同於您為節點對節點安全性指定的主要和次要憑證。

用戶端如果是使用系統管理憑證來連接到叢集，就會擁有管理功能的完整存取權。 用戶端如果是使用唯讀使用者用戶端憑證來連接到叢集，則只會擁有管理功能的唯讀存取權。 換句話說，這些憑證會用於角色型存取控制 (RBAC)。

若要了解如何在叢集中設定憑證安全性，請參閱[使用 Azure Resource Manager 範本來設定叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm) 。

#### <a name="client-to-node-azure-active-directory-security"></a>用戶端對節點 Azure Active Directory 安全性

在 Azure 上執行的叢集也可以使用 Azure Active Directory (Azure AD) 來保護對管理端點的存取。 如需如何建立必要的 Azure Active Directory 構件、如何在建立叢集時填入這些構件，以及如何連接到這些叢集的詳細資訊，請參閱[使用 Azure Resource Manager 範本來設定叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)。

Azure AD 可讓組織 (稱為租用戶) 管理使用者對應用程式的存取。 有具有 Web 型登入 UI 的應用程式，以及具有原生用戶端體驗的應用程式。

Service Fabric 叢集提供其管理功能的數個進入點，包括 Web 架構的 Service Fabric Explorer 和 Visual Studio。 因此，您將建立兩個 Azure AD 應用程式來控制對叢集的存取：一個 Web 應用程式和一個原生應用程式。

對於 Azure 叢集，建議您針對節點對節點安全性使用 Azure AD 安全性來驗證用戶端和憑證。

對於具有 Windows Server 2012 R2 和 Active Directory 的獨立 Windows Server 叢集，建議您使用 Windows 安全性與群組受控服務帳戶 (gMSA)。 否則，使用 Windows 安全性與 Windows 帳戶。

## <a name="understand-monitoring-and-diagnostics-in-service-fabric"></a>了解 Service Fabric 中的監視與診斷
不論任何環境，針對應用程式與服務的開發、測試及部署進行[監視和診斷](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-overview)，都極為重要。 如果您要實作監視和診斷功能，以確保應用程式和服務可在本機開發環境或生產環境中如預期般運作，Service Fabric 解決方案就是理想之選。

從安全性的觀點來看，監視和診斷的主要目標是：

-   偵測及診斷安全性事件可能造成的硬體和基礎結構問題。
-   偵測可能是入侵指標 (Indicator of Compromise，IoC) 的軟體和應用程式問題。
-   了解資源耗用量，以協助避免不小心發生阻斷服務。

監視與診斷的工作流程包含三個步驟：

1.  **事件產生**：事件產生包含基礎結構 (叢集) 層級和應用程式/服務層級的事件 (記錄、追蹤、自訂事件)。 深入了解[基礎結構層級事件](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-generation-infra)和[應用程式層級事件](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-generation-app)，以了解提供的項目，以及如何新增其他的檢測。

2.  **事件彙總**：產生的事件必須經過收集與彙總後，才能夠顯示。 我們通常建議使用 [Azure 診斷](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-aggregation-wad) (類似代理程式型記錄收集) 或 [EventFlow](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-aggregation-eventflow) (同處理序記錄收集)。

3.  **分析**︰事件必須經過視覺化並可藉由某種格式來存取，才能進行分析及顯示。 有數個平台可用於監視和診斷資料的分析和視覺效果。 我們建議使用 [Azure Log Analytics](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-analysis-oms) 和 [Azure Application Insights](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-analysis-appinsights)，因為它們能夠與 Service Fabric 妥善整合。

您也可以使用 [Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview)，監視構成 Service Fabric 叢集的許多 Azure 資源。

監視程式是個別的服務，可以觀察服務之間的健全狀況和負載，並報告健全狀況模型階層中任何項目的健全狀況。 使用監視程式可避免在單一服務檢視中無法偵測到的錯誤。 

監視程式也非常適合裝載能夠執行修復動作的程式碼，而無需使用者互動。 例如，以特定時間間隔清除儲存體中的記錄檔。 您可以在 [Azure Service Fabric 監視程式範例](https://azure.microsoft.com/resources/samples/service-fabric-watchdog-service/)找到範例監視程式服務實作。

## <a name="secure-communication-by-using-certificates"></a>使用憑證保護通訊
憑證可協助您保護您的獨立 Windows 叢集不同節點之間的通訊安全。 藉由使用 X.509 憑證，您也可以驗證要連線到此叢集的用戶端。 這可確保只有授權的使用者可以存取叢集。 我們建議您在建立叢集時，在叢集上啟用憑證。

X509 數位憑證通常用來驗證用戶端與伺服器。 它們也可用於訊息的加密和數位簽署。

下表列出您在設定叢集時所需的憑證：

|憑證資訊設定 |說明|
|-------------------------------|-----------|
|ClusterCertificate|    需有此憑證，才能保護叢集上節點之間的通訊。 您可以使用兩個叢集憑證：主要憑證和用於升級的次要憑證。|
|ServerCertificate| 用戶端嘗試連線到此叢集時，會向用戶端此憑證顯示此憑證。 您可以使用兩個伺服器憑證：主要憑證和用於升級的次要憑證。|
|ClientCertificateThumbprints|  這是在經過驗證的用戶端上安裝的一組憑證。|
|ClientCertificateCommonNames|  這是 CertificateCommonName 第一個用戶端憑證的一般名稱。 CertificateIssuerThumbprint 是此憑證的簽發者指紋。|
|ReverseProxyCertificate|   這是選擇性憑證，經指定可保護您的[反向 Proxy](https://docs.microsoft.com/azure/service-fabric/service-fabric-reverseproxy)。|

如需保護憑證的詳細資訊，請參閱[使用 X.509 憑證保護 Windows 上的獨立叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-x509-security)。

## <a name="understand-role-based-access-control"></a>了解角色型存取控制
您可以在建立叢集時，分別為系統管理員和使用者用戶端角色提供個別的身分識別 (包括憑證)，來指定這兩個角色。 如需關於預設存取控制設定及如何變更預設設定的詳細資訊，請參閱[適用於 Service Fabric 用戶端的角色型存取控制](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security-roles)。

## <a name="secure-standalone-clusters-by-using-windows-security"></a>使用 Windows 安全性保護獨立叢集
為避免有人未經授權存取 Service Fabric 叢集，您必須保護叢集。 當叢集執行生產工作負載時，安全性尤其重要。 您可以在 ClusterConfig.JSON 檔案中使用 Windows 安全性，以設定節點對節點和用戶端對節點的安全性。

需要在 gMSA 下執行 Service Fabric 時，可透過設定 [ClustergMSAIdentity](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-windows-security) 來設定節點對節點安全性。 若要建置節點之間的信任關係，您必須使其可相互感知。

如果您想要使用 Active Directory 網域內的電腦群組，可以設定 ClusterIdentity 來設定節點對節點安全性。 如需詳細資訊，請參閱[在 Active Directory 中建立電腦群組](https://msdn.microsoft.com/library/aa545347)。

您使用 ClientIdentities 來設定用戶端對節點安全性。 您必須設定叢集，使其辨識它可以信任的用戶端身分識別。 您可以透過兩種方式來建立信任︰

-   指定可以連線的網域群組使用者。
-   指定可以連線的網域節點使用者。

## <a name="configure-application-security-in-service-fabric"></a>在 Service Fabric 中設定應用程式安全性
### <a name="manage-secrets-in-service-fabric-applications"></a>管理 Service Fabric 應用程式中的祕密
密碼可以是任何機密資訊，例如儲存體連接字串、密碼或其他不會以純文字處理的值。

您可以使用 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis) 來管理金鑰和密碼。 不過，在應用程式中使用密碼時不會依賴特定的雲端平台。 您可以將應用程式部署至任何位置裝載的叢集。 此流程有四個主要步驟︰

1.  取得資料加密憑證。
2.  在叢集中安裝憑證。
3.  在部署應用程式時以憑證來加密密碼的值，並將其插入服務的 Settings.xml 組態檔。
4.  藉由以相同的編密憑證進行解密，從 Settings.xml 讀取加密的值。

如需詳細資訊，請參閱[管理 Service Fabric 應用程式中的密碼](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-secret-management)。

### <a name="configure-security-policies-for-an-application"></a>設定應用程式的安全性原則
使用 Azure Service Fabric 安全性，您可以協助保護叢集中以不同使用者帳戶執行的應用程式。 以這些使用者帳戶部署時，Service Fabric 安全性也會協助保護應用程式所使用的資源，例如檔案、目錄和憑證。 如此一來，即使在共用主控環境中，執行中的應用程式就會更安全。

設定安全性原則的工作包括：

-   設定服務安裝程式進入點的原則
-   從安裝程式進入點啟動 PowerShell 命令
-   使用主控台重新導向進行本機偵錯
-   設定服務程式碼套件的原則
-   為 HTTP 和 HTTPS 端點指派安全性存取原則

## <a name="secure-communication-for-services"></a>保護服務的通訊安全
安全性是通訊最為重視的其中一個部分。 Reliable Services 應用程式架構會提供可用來改善安全性的一些預先建置通訊堆疊和工具。 如需詳細資訊，請參閱[保護服務的服務遠端作業通訊](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-secure-communication)。

## <a name="next-steps"></a>後續步驟
- 如需叢集安全性的概念資訊，請參閱[使用 Azure Resource Manager 建立 Service Fabric 叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)和[使用 Azure 入口網站建立 Service Fabric 叢集](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-portal)。
- 若要深入了解 Service Fabric 中的叢集安全性，請參閱 [Service Fabric 叢集安全性案例](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security)。
