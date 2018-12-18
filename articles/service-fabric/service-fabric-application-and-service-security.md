---
title: 深入了解 Azure Service Fabric 應用程式安全性 | Microsoft Docs
description: 如何安全地在 Service Fabric 上執行微服務應用程式的概觀。 深入了解如何在不同的安全性帳戶底下執行服務和啟動指令碼、驗證和授權使用者、管理應用程式祕密、保護服務通訊、使用 API 閘道，以及保護應用程式待用資料。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: 4242a1eb-a237-459b-afbf-1e06cfa72732
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/16/2018
ms.author: ryanwi
ms.openlocfilehash: fa6d46186ad833b68e60c24f742d210b7845759a
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34207905"
---
# <a name="service-fabric-application-and-service-security"></a>Service Fabric 應用程式和服務安全性
微服務架構可以帶來[許多優點](service-fabric-overview-microservices.md)。 不過，管理微服務的安全性是一種挑戰，而且與管理傳統的整合型應用程式安全性有所不同。 

使用整合型時，應用程式通常會在網路中的一或多部伺服器上執行，而且更容易識別公開的連接埠和 API 與 IP 位址。 通常有一個周邊或界限，以及一個要保護的資料庫。 如果該系統因為安全性缺口或攻擊而遭到入侵，則系統內的一切都可能會被攻擊者利用。 使用微服務的系統比較複雜。  服務是非集中式且分散於許多主機，並可在主機之間移轉。  有了適當的安全性，您就可以限制攻擊者可取得的權限，以及在單一攻擊中藉由入侵一項服務可取得的資料量。  通訊不在內部，而是透過網路發生，且服務之間有許多公開的連接埠和互動。 了解這些服務互動為何以及它們的發生時機，對於您的應用程式安全性而言很重要。

本文不是微服務安全性的指南 (線上有許多此類資源可供取用)，但會說明如何在 Service Fabric 中完成不同層面的安全性。

## <a name="authentication-and-authorization"></a>驗證和授權
通常需要將服務所公開的資源和 API 限制為適用於特定信任的使用者或用戶端。 驗證是可靠地查明使用者身分識別的程序。  授權是讓 API 或服務可供某些已驗證使用者使用 (其他使用者無法使用) 的程序。

### <a name="authentication"></a>驗證
進行 API 層級信任決策的第一個步驟是驗證。 驗證是可靠地查明使用者身分識別的程序。  在微服務案例中，驗證通常會集中處理。 如果您使用 API 閘道，您可以[將驗證卸載](/azure/architecture/patterns/gateway-offloading)至閘道。 如果您使用此方法，請確定可以直接與個別服務連線 (不需要 API 閘道)，除非設立了額外的安全性來驗證訊息是否來自閘道。

如果可以直接存取服務，則驗證服務 (例如 Azure Active Directory 或專用驗證微服務) 會作為安全性權杖服務 (STS)，可用來驗證使用者。 信任決策會在服務與安全性權杖或 Cookie 之間共用。 

針對 ASP.NET Core，[驗證使用者](/dotnet/standard/microservices-architecture/secure-net-microservices-web-applications/)的主要機制是 ASP.NET Core 身分識別成員資格系統。 ASP.NET Core 身分識別會將使用者資訊 (包括登入資訊、角色及宣告) 儲存在由開發人員設定的資料存放區中。 ASP.NET Core 身分識別支援雙因素驗證。  另外也支援外部驗證提供者，以便使用者使用 Microsoft、Google、Facebook 或 Twitter 等提供者的現有驗證程序進行登入。 

### <a name="authorization"></a>Authorization
驗證之後，服務需要授權使用者存取權，或判斷使用者可以執行哪些作業。 這個程序允許服務讓 API 可供某些已驗證的使用者使用，而不是所有使用者都可使用。 授權與驗證彼此獨立且互不影響，後者是查明使用者是誰的程序。 驗證可為目前使用者建立一或多個身分識別。

[ASP.NET Core 授權](/dotnet/standard/microservices-architecture/secure-net-microservices-web-applications/authorization-net-microservices-web-applications)可以根據使用者的角色或根據自訂原則來進行，其中包含檢查宣告或其他啟發學習法。

## <a name="restrict-and-secure-access-using-an-api-gateway"></a>使用 API 閘道來限制和保護存取權
雲端應用程式通常需要前端閘道來為使用者、裝置或其他應用程式提供單一輸入點。 [API 閘道](/azure/architecture/microservices/gateway)位於用戶端與服務之間，這是通往您應用程式所提供之所有服務的進入點。 它會作為反向 Proxy，將要求從用戶端路由傳送到服務。 它也會執行各種跨領域工作，例如驗證和授權、SSL 終止和速率限制。 如果您未部署閘道，用戶端就必須將要求直接傳送給前端服務。

在 Service Fabric 中，閘道可以是任何無狀態服務 (例如 [ASP.NET Core 應用程式](service-fabric-reliable-services-communication-aspnetcore.md))，或是專為流量輸入設計的另一項服務 (例如，[Træfik](https://docs.traefik.io/)、[事件中樞](https://docs.microsoft.com/azure/event-hubs/)、[IoT 中樞](https://docs.microsoft.com/azure/iot-hub/)或 [Azure API 管理](https://docs.microsoft.com/azure/api-management))。

「API 管理」直接與 Service Fabric 整合，可讓您將具有一組豐富路由規則的 API 發佈至後端 Service Fabric 服務。  您可以保護後端服務的存取、使用節流來預防 DOS 攻擊，或驗證 API 金鑰、JWT 權杖、憑證和其他認證。 若要深入了解，請參閱 [Service Fabric 搭配 Azure API 管理概觀](service-fabric-api-management-overview.md)。

## <a name="manage-application-secrets"></a>管理應用程式密碼
密碼可以是任何機密資訊，例如儲存體連接字串、密碼或其他不會以純文字處理的值。 本文使用 Azure Key Vault 來管理金鑰和祕密。 不過，應用程式中的密碼「使用」  是由平台驗證，讓應用程式可部署至裝載在任何位置的叢集。

建議透過[服務組態套件][config-package]來管理服務組態設定。 組態套件會有各種版本，並可透過含有健全狀況驗證和自動復原的受控輪流升級來進行升級。 這是慣用的全域組態，因為可以減少全域服務中斷的機會。 加密的密碼也不例外。 Service Fabric 具有內建的功能，可使用憑證加密來加密或解密組態套件 Settings.xml 檔案中的值。

下圖說明 Service Fabric 應用程式中密碼管理的基本流程︰

![密碼管理概觀][overview]

此流程有四個主要步驟︰

1. 取得資料編密憑證。
2. 在叢集中安裝憑證。
3. 在部署應用程式時以憑證來加密密碼的值，並將其插入服務的 Settings.xml 組態檔。
4. 藉由以相同的編密憑證進行解密，從 Settings.xml 讀取加密的值。 

[Azure Key Vault][key-vault-get-started] 在此是當做憑證的安全儲存位置，以及讓憑證安裝在 Azure 中的 Service Fabric 叢集上的方法。 如果您沒有要部署至 Azure，您不需要使用金鑰保存庫管理 Service Fabric 應用程式中的密碼。

如需範例，請參閱[管理應用程式祕密](service-fabric-application-secret-management.md)。

## <a name="secure-the-hosting-environment"></a>保護裝載環境
藉由使用 Azure Service Fabric，您便可以保護在叢集中以不同使用者帳戶執行的應用程式。 在以該使用者帳戶部署時，Service Fabric 也會協助保護應用程式所使用的資源，例如檔案、目錄和憑證。 如此一來，即使在共用主控環境中，執行中的應用程式就不會彼此干擾。

應用程式資訊清單會宣告執行服務和保護資源所需的安全性主體 (使用者和群組)。  這些安全性主體會在原則中參考，例如執行身分、端點繫結、套件共用或安全性存取原則。  然後原則會套用至應用程式資訊清單之 **ServiceManifestImport** 區段中的服務資源。

宣告主體時，您也可以定義和建立使用者群組，以便將一或多個使用者新增至每個群組一起進行管理。 當不同的服務進入點有多個使用者，而且他們需要具備可在群組層級取得的某些常見權限時，這會很有用。

根據預設，Service Fabric 應用程式會在用以執行 Fabric.exe 程序的帳戶之下執行。 Service Fabric 也能夠以本機使用者帳戶或本機系統帳戶 (在應用程式的資訊清單內指定) 執行應用程式。 如需詳細資訊，請參閱[以本機使用者帳戶或本機系統帳戶身分執行服務](service-fabric-application-runas-security.md)。  您也可[以本機使用者或系統帳戶身分執行服務啟動指令碼](service-fabric-run-script-at-service-startup.md)。

當您在 Windows 獨立叢集上執行 Service Fabric 時，您可以在 [Active Directory 網域帳戶](service-fabric-run-service-as-ad-user-or-group.md)或[群組受控服務帳戶](service-fabric-run-service-as-gmsa.md)底下執行服務。

## <a name="secure-containers"></a>安全容器
Service Fabric 為容器內的服務提供了一種機制，供其存取 Windows 或 Linux 叢集 (5.7 版或更高版本) 節點上所安裝的憑證。 此 PFX 憑證可用於驗證應用程式或服務，也可以用來保護與其他服務的通訊。 如需詳細資訊，請參閱[將憑證匯入容器](service-fabric-securing-containers.md)。

此外，Service Fabric 也會為 Windows 容器支援 gMSA (群組受控服務帳戶)。 如需詳細資訊，請參閱[為 Windows 容器設定 gMSA](service-fabric-setup-gmsa-for-windows-containers.md)。

## <a name="secure-service-communication"></a>保護服務通訊
在 Service Fabric 中，服務會在 Service Fabric 叢集中的某處執行，通常是分散到多個 VM。 Service Fabric 提供數個選項來保護您的服務通訊。

您可以在 [ASP.NET Core 或 Java](service-fabric-service-manifest-resources.md#example-specifying-an-https-endpoint-for-your-service) Web 服務中啟用 HTTPS 端點。

您可以建立反向 Proxy 與服務之間的安全連線，以確保端對端的安全通道。 僅有將反向 Proxy 設為接聽 HTTPS 時，才支援安全的服務連線。 如需設定反向 Proxy 的詳細資訊，請參閱 [Azure Service Fabric 中的反向 Proxy](service-fabric-reverseproxy.md)。  [連線至安全的服務](service-fabric-reverseproxy-configure-secure-communication.md)說明如何在反向 Proxy 與服務之間建立安全的連線。

Reliable Services 應用程式架構會提供可用來改善安全性的一些預先建置通訊堆疊和工具。 深入了解在您使用服務遠端作業 (在 [C#](service-fabric-reliable-services-secure-communication.md) 或 [Java](service-fabric-reliable-services-secure-communication-java.md) 中) 或使用 [WCF](service-fabric-reliable-services-secure-communication-wcf.md) 時，如何改善安全性。

## <a name="encrypt-application-data-at-rest"></a>加密應用程式待用資料
在 Azure 中執行之 Service Fabric 叢集中的每個[節點類型](service-fabric-cluster-nodetypes.md)都是由[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)提供支援。 使用 Azure Resource Manager 範本，您可以將資料磁碟連結到構成 Service Fabric 叢集的擴展集。  如果您的服務會將資料儲存至連結的資料磁碟，您可以[將這些資料磁碟加密](../virtual-machine-scale-sets/virtual-machine-scale-sets-encrypt-disks-ps.md)以保護您的應用程式資料。

<!--TO DO: Enable BitLocker on Windows standalone clusters?
TO DO: Encrypt disks on Linux clusters?-->


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>後續步驟
* [在服務啟動時執行安裝指令碼](service-fabric-run-script-at-service-startup.md)
* [在服務資訊清單中指定資源](service-fabric-service-manifest-resources.md)
* [部署應用程式](service-fabric-deploy-remove-applications.md)
* [深入了解叢集安全性](service-fabric-cluster-security.md)

<!-- Links -->
[key-vault-get-started]:../key-vault/key-vault-get-started.md
[config-package]: service-fabric-application-and-service-manifests.md
[service-fabric-cluster-creation-via-arm]: service-fabric-cluster-creation-via-arm.md

<!-- Images -->
[overview]:./media/service-fabric-application-and-service-security/overview.png