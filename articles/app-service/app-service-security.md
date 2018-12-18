---
title: Azure App Service 和 Azure Functions 中的安全性 | Microsoft Docs
description: 深入了解 App Service 如何協助保護您的應用程式，以及您如何進一步鎖定您的應用程式免於遭受威脅。
keywords: azure app service, Web 應用程式, 行動裝置應用程式, api 應用程式, 函式應用程式, 安全性, 安全, 安全的, 合規性, 符合規範, 憑證, 憑證, https, ftps, tls, 信任, 加密, 加密, 已加密, ip 限制, 驗證, 授權, authn, autho, msi, 受控服務身分識別, 祕密, 祕密, 修補, 修補程式, 修補程式, 版本, 隔離, 網路隔離, ddos, mitm
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/24/2018
ms.author: cephalin
ms.openlocfilehash: 40fdd22bdbb3fc0676688430069d58c0422a7ca2
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43382111"
---
# <a name="security-in-azure-app-service-and-azure-functions"></a>Azure App Service 和 Azure Functions 中的安全性

本文說明 [Azure App Service](app-service-web-overview.md) 如何協助保護您的 Web 應用程式、行動裝置應用程式後端、API 應用程式和 [Azure Functions](/azure/azure-functions/)。 它也會示範如何使用內建的 App Service 功能，進一步保護您的應用程式。

App Service 的平台元件 (包括 Azure VM、儲存體、網路連線、Web 架構、管理和整合功能) 會主動受到保護與強化。 App Service 會持續經歷加強的合規性檢查，以確保：

- 您的應用程式資源[受到保護](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)免於遭受其他客戶的 Azure 資源威脅。
- [VM 執行個體和執行階段軟體會定期更新](app-service-patch-os-runtime.md)，以因應新發現的弱點。 
- 您的應用程式與其他 Azure 資源 (例如 [SQL Database](https://azure.microsoft.com/services/sql-database/)) 之間的祕密通訊 (例如連接字串) 仍在 Azure 內，不會跨越任何網路界限。 祕密會在儲存時一律加密。
- 透過 App Service 連線功能 (例如[混合式連線](app-service-hybrid-connections.md)) 的所有通訊都會加密。 
- 透過 Azure PowerShell、Azure CLI、Azure SDK、REST API 等遠端管理工具進行的連線全都會加密。
- 全天候威脅管理會保護基礎結構和平台，免於遭受惡意程式碼、分散式拒絕服務 (DDoS)、攔截式 (MITM) 和其他威脅。

如需在 Azure 中的基礎結構與平台安全性的詳細資訊，請參閱 [Azure 信任中心](https://azure.microsoft.com/overview/trusted-cloud/)。

下列各節說明如何進一步保護您的 App Service 應用程式免於遭受威脅。

## <a name="https-and-certificates"></a>HTTPS 和憑證

App Service 可讓您利用 [HTTPS](https://wikipedia.org/wiki/HTTPS) 保護您的應用程式。 建立您的應用程式後，已可使用 HTTPS 存取其預設網域名稱 (\<app_name>.azurewebsites.net)。 如果您[為應用程式設定自訂網域](app-service-web-tutorial-custom-domain.md)，您也應該[使用自訂憑證保護其安全](app-service-web-tutorial-custom-ssl.md)，以便用戶端瀏覽器可以對您的自訂網域進行安全的 HTTPS 連線。 有兩種方式可以完成：

- **App Service 憑證** - 直接在 Azure 中建立憑證。 此憑證會在 [Azure Key Vault](/azure/key-vault/) 中保護，而且可以匯入您的 App Service 應用程式中。 如需詳細資訊，請參閱[購買並設定您 Azure App Service 的 SSL 憑證](web-sites-purchase-ssl-web-site.md)。
- **第三方憑證** - 上傳您向受信任憑證授權單位購買的自訂 SSL 憑證，並將它繫結至您的 App Service 應用程式。 App Service 支援單一網域憑證和萬用字元憑證。 它也支援用於測試的自我簽署憑證。 如需詳細資訊，請參閱[將現有的自訂 SSL 憑證繫結至 Azure Web Apps](app-service-web-tutorial-custom-ssl.md)。

## <a name="insecure-protocols-http-tls-10-ftp"></a>不安全的通訊協定 (HTTP、TLS 1.0、FTP)

為了保護應用程式以抵禦所有未加密的 (HTTP) 連線，App Service 會提供單鍵設定來強制使用 HTTPS。 不安全的要求會在觸達您應用程式的程式碼之前離開。 如需詳細資訊，請參閱[強制使用 HTTPS](app-service-web-tutorial-custom-ssl.md#enforce-https)。

業界標準 (例如 [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)) 不再將 [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 視為安全的。 App Service 可讓您藉由[強制使用 TLS 1.1/1.2](app-service-web-tutorial-custom-ssl.md#enforce-tls-versions) 來停用過時的通訊協定。

App Service 同時支援使用 FTP 和 FTPS 來部署您的檔案。 不過，應該使用 FTPS，而不是 FTP (如果全都可行)。 如有其中一個或兩個通訊協定都不在使用中，您應該[停用它們](app-service-deploy-ftp.md#enforce-ftps)。

## <a name="static-ip-restrictions"></a>靜態 IP 限制

根據預設，App Service 應用程式會接受來自網際網路所有 IP 位址的要求，但是您可以限制存取一小部分的 IP 位址。 Windows 上的 App Service 可讓您定義允許存取應用程式的 IP 位址清單。 允許清單可以包含個別 IP 位址，或以子網路遮罩所定義的 IP 位址範圍。 如需詳細資訊，請參閱 [Azure App Service 靜態 IP 限制](app-service-ip-restrictions.md)。

對於 Windows 上的 App Service，您也可以藉由設定 _web.config_ 來動態限制 IP 位址。如需詳細資訊，請參閱 [動態 IP 安全性<dynamicIpSecurity>](https://docs.microsoft.com/iis/configuration/system.webServer/security/dynamicIpSecurity/)。

## <a name="client-authentication-and-authorization"></a>用戶端驗證與授權

Azure App Service 會提供周全的使用者或用戶端應用程式驗證和授權。 啟用以後，它可以透過少量或不需任何應用程式程式碼，登入使用者和用戶端應用程式。 您可以實作自己的驗證和授權解決方案，或允許 App Service 為您處理。 驗證和授權模組在將 Web 要求交給您的應用程式程式碼前加以處理，而且會在未經授權的要求觸達您的程式碼前予以拒絕。

App Service 驗證和授權支援多個驗證提供者，包括 Azure Active Directory、Microsoft 帳戶、Facebook、Google 和 Twitter。 如需詳細資訊，請參閱 [Azure App Service 中的驗證與授權](app-service-authentication-overview.md)。

## <a name="service-to-service-authentication"></a>服務對服務驗證

針對後端服務進行驗證時，App Service 會視您的需求而定，提供兩個不同的機制：

- **服務識別** - 使用應用程式本身的身分識別來登入遠端資源。 App Service 可讓您輕鬆地建立[受控服務識別](app-service-managed-service-identity.md)，將它用來驗證其他服務，例如 [Azure SQL Database](/azure/sql-database/) 或 [Azure Key Vault](/azure/key-vault/)。 如需這個方法的端對端教學課程，請參閱[使用受控服務識別保護來自 App Service 的 Azure SQL Database 連線](app-service-web-tutorial-connect-msi.md)。
- **代表 (OBO)** - 代表使用者進行遠端資源的委派存取。 以 Azure Active Directory 作為驗證提供者，您的 App Service 應用程式可以執行遠端服務的委派登入，例如 [Azure Active Directory 圖形 API](../active-directory/develop/active-directory-graph-api.md)或 App Service 中的遠端 API 應用程式。 如需這個方法的端對端教學課程，請參閱[在 Azure App Service 中端對端驗證和授權使用者](app-service-web-tutorial-auth-aad.md)。

## <a name="connectivity-to-remote-resources"></a>連線到遠端資源

您的應用程式可能需要存取三種類型的遠端資源： 

- [Azure 資源](#azure-resources)
- [Azure 虛擬網路內的資源](#resources-inside-an-azure-virtual-network)
- [內部部署資源](#on-premises-resources)

在上述每種情況下，App Service 可讓您進行安全連線，但您仍應會發現安全性最佳作法。 例如，即使後端資源允許未加密的連線，但一律使用加密的連線。 此外，確定您的後端 Azure 服務允許最小一組 IP 位址。 您可以在 [Azure App Service 中的輸入和輸出 IP 位址](app-service-ip-addresses.md)找到您應用程式的輸出 IP 位址。

### <a name="azure-resources"></a>Azure 資源

當您的應用程式連線到 Azure 資源 (例如 [SQL Database](https://azure.microsoft.com/services/sql-database/) 和 [Azure 儲存體](/azure/storage/)) 時，連線仍在 Azure 內，不會跨越任何網路界限。 不過，連線會通過 Azure 中的共用網路，因此務必確定您的連線已加密。 

如果您的應用程式裝載於 [App Service 環境](environment/intro.md)中，您應該[使用虛擬網路服務端點來連線到支援的 Azure 服務](../virtual-network/virtual-network-service-endpoints-overview.md)。

### <a name="resources-inside-an-azure-virtual-network"></a>Azure 虛擬網路內的資源

您的應用程式可以透過[虛擬網路整合](web-sites-integrate-with-vnet.md)來存取 [Azure 虛擬網路](/azure/virtual-network/)中的資源。 使用點對站 VPN 可建立與虛擬網路的整合。 應用程式可接著使用其私人 IP 位址，存取虛擬網路中的資源。 不過，點對站連線仍會在 Azure 中周遊共用的網路。 

若要完全隔離您的資源與 Azure 中共用網路的連線，請在 [App Service 環境](environment/intro.md)中建立您的應用程式。 App Service 環境一律會部署至專用的虛擬網路，因此您的應用程式與虛擬網路內的資源之間的連線會完全隔離。 如需 App Service 環境中的其他層面，請參閱[網路隔離](#network-isolation)。

### <a name="on-premises-resources"></a>內部部署資源

您可以用三種方式安全地存取內部部署資源，例如資料庫： 

- [混合式連線](app-service-hybrid-connections.md) - 透過 TCP 通道，建立遠端資源的點對點連線。 使用 TLS 1.2 搭配共用存取簽章 (SAS) 金鑰，可以建立 TCP 通道。
- 透過站對站 VPN 的[虛擬網路整合](web-sites-integrate-with-vnet.md) – 如 [Azure 虛擬網路內的資源](#resources-inside-an-azure-virtual-network)所述，但是虛擬網路可以透過[站對站 VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) 連線到您的內部部署網路。 在此網路拓撲中，您的應用程式可以連接到內部部署資源，例如虛擬網路中的其他資源。
- 採用站對站 VPN 的 [App Service 環境](environment/intro.md) – 如 [Azure 虛擬網路內的資源](#resources-inside-an-azure-virtual-network)所述，但是虛擬網路可以透過[站對站 VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) 連線到您的內部部署網路。 在此網路拓撲中，您的應用程式可以連接到內部部署資源，例如虛擬網路中的其他資源。

## <a name="application-secrets"></a>應用程式祕密

請勿將應用程式祕密 (例如資料庫認證、API 權杖和私密金鑰) 存放在您的程式碼或組態檔中。 普遍接受的方法是使用您所選語言的標準模式，將它們當作[環境變數](https://wikipedia.org/wiki/Environment_variable)存取。 在 App Service 中，環境變數是透過[應用程式設定](web-sites-configure.md#app-settings) (以及特別適用於 .NET 應用程式的[連接字串](web-sites-configure.md#connection-strings)) 來定義。 應用程式設定和連接字串都會經過加密並儲存在 Azure 中，而且只會在應用程式啟動時，要插入您應用程式的程序記憶體之前解密。 加密金鑰會定期輪替。

或者，您可以整合 App Service 應用程式與 [Azure Key Vault](/azure/key-vault/)，以便進行進階祕密管理。 App Service 應用程式可以[利用受控服務識別存取 Key Vault](../key-vault/tutorial-web-application-keyvault.md)，安全地存取您所需的祕密。

## <a name="network-isolation"></a>網路隔離

除了 [隔離式] 定價層，所有定價層都會在 App Service 中的共用網路基礎結構上執行您的應用程式。 例如，公用 IP 位址與前端負載平衡器會與其他租用戶共用。 [隔離式] 定價層可讓您在專屬 [App Service 環境](environment/intro.md)中執行您的應用程式，藉此完成網路隔離。 App Service 環境就會在您擁有的 [Azure 虛擬網路](/azure/virtual-network/)執行個體中執行。 它可讓您： 

- 使用[網路安全性群組](../virtual-network/virtual-networks-nsg.md)來限制網路存取。 
- 透過具有專屬前端的專屬公用端點來提供您的應用程式。
- 使用內部負載平衡器 (ILB) 來提供內部應用程式，只允許從 Azure 虛擬網路內部進行存取。 ILB 有來自私人子網路的 IP 位址，這可讓您的應用程式與網際網路完全隔離。
- [使用 Web 應用程式防火牆 (WAF) 後方的 ILB](environment/integrate-with-application-gateway.md)。 WAF 可為您的公開應用程式提供企業層級保護，例如 DDoS 保護、URI 篩選和 SQL 插入式攻擊的防護。

如需詳細資訊，請參閱 [Azure App Service 環境簡介](environment/intro.md)。 
