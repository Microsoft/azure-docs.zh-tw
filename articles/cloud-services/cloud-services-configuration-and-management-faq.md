---
title: Microsoft Azure 雲端服務之設定和管理問題的常見問題集 | Microsoft Docs
description: 本文列出 Microsoft Azure 雲端服務之設定和管理的相關常見問題集。
services: cloud-services
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: top-support-issue
ms.assetid: 84985660-2cfd-483a-8378-50eef6a0151d
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2018
ms.author: genli
ms.openlocfilehash: 30a23010f326189ffd5886407d70e357abb9c53e
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038002"
---
# <a name="configuration-and-management-issues-for-azure-cloud-services-frequently-asked-questions-faqs"></a>Azure 雲端服務之設定和管理問題：常見問題集 (FAQ)

本文包含 [Microsoft Azure 雲端服務](https://azure.microsoft.com/services/cloud-services)之設定和管理問題的相關常見問題集。 您也可以參閱 [雲端服務 VM 大小頁面](cloud-services-sizes-specs.md) 以取得大小資訊。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

**Certificates**

- [為什麼我雲端服務 SSL 憑證的信任鏈結是不完整的？](#why-is-the-certificate-chain-of-my-cloud-service-ssl-certificate-incomplete)
- [「Windows Azure Tools 擴充功能的加密憑證」用途為何？](#what-is-the-purpose-of-the-windows-azure-tools-encryption-certificate-for-extensions)
- [如何能夠產生憑證簽署要求 (CSR)，而不 "RDP" 到執行個體中？](#how-can-i-generate-a-certificate-signing-request-csr-without-rdp-ing-in-to-the-instance)
- [我的雲端服務管理憑證即將到期。要如何續訂？](#my-cloud-service-management-certificate-is-expiring-how-to-renew-it)
- [如何將主要 SSL 憑證 (.pfx) 和中繼憑證 (.p7b) 的安裝自動化？](#how-to-automate-the-installation-of-main-ssl-certificatepfx-and-intermediate-certificatep7b)
- [「適用於 MachineKey 的 Microsoft Azure 服務管理」憑證的用途為何？](#what-is-the-purpose-of-the-microsoft-azure-service-management-for-machinekey-certificate)

**監視和記錄**

- [即將在 Azure 入口網站推出的雲端服務功能有哪些可協助管理和監視應用程式？](#what-are-the-upcoming-cloud-service-capabilities-in-the-azure-portal-which-can-help-manage-and-monitor-applications)
- [為什麼 IIS 會停止寫入記錄目錄？](#why-does-iis-stop-writing-to-the-log-directory)
- [如何啟用雲端服務的 WAD 記錄？](#how-do-i-enable-wad-logging-for-cloud-services)

**網路組態**

- [如何設定 Azure 負載平衡器的閒置逾時？](#how-do-i-set-the-idle-timeout-for-azure-load-balancer)
- [如何將靜態 IP 位址關聯到我的雲端服務？](#how-do-i-associate-a-static-ip-address-to-my-cloud-service)
- [Azure 的基本 IPS/IDS 和 DDOS 提供的特性和功能是什麼？](#what-are-the-features-and-capabilities-that-azure-basic-ipsids-and-ddos-provides)
- [如何啟用雲端服務虛擬機器上的 HTTP/2？](#how-to-enable-http2-on-cloud-services-vm)

**權限**

- [Microsoft 內部工程師是否可在沒有權限的情況下，從遠端桌面到雲端服務執行個體？](#can-microsoft-internal-engineers-remote-desktop-to-cloud-service-instances-without-permission)
- [我無法使用 RDP 檔案從遠端桌面登入雲端服務虛擬機器。我收到下列錯誤：發生驗證錯誤 (代碼：0x80004005)](#i-cannot-remote-desktop-to-cloud-service-vm--by-using-the-rdp-file-i-get-following-error-an-authentication-error-has-occurred-code-0x80004005)

**調整大小**

- [我不能調整超過 X 個執行個體](#i-cannot-scale-beyond-x-instances)
- [如何根據記憶體計量設定自動縮放？](#how-can-i-configure-auto-scale-based-on-memory-metrics)

**泛型**

- [如何將 "nosniff" 新增至我的網站？](#how-do-i-add-nosniff-to-my-website)
- [如何自訂 Web 角色的 IIS？](#how-do-i-customize-iis-for-a-web-role)
- [我的雲端服務配額限制是多少？](#what-is-the-quota-limit-for-my-cloud-service)
- [為什麼我雲端服務虛擬機器上的磁碟機顯示幾乎沒有可用的磁碟空間？](#why-does-the-drive-on-my-cloud-service-vm-show-very-little-free-disk-space)
- [如何以自動化方式新增雲端服務的反惡意程式碼擴充功能？](#how-can-i-add-an-antimalware-extension-for-my-cloud-services-in-an-automated-way)
- [如何啟用雲端服務的伺服器名稱指示 (SNI)？](#how-to-enable-server-name-indication-sni-for-cloud-services)
- [如何將標籤新增至我的 Azure 雲端服務？](#how-can-i-add-tags-to-my-azure-cloud-service)
- [Azure 入口網站不會顯示雲端服務的 SDK 版本。如何取得版本？](#the-azure-portal-doesnt-display-the-sdk-version-of-my-cloud-service-how-can-i-get-that)
- [我想要關閉雲端服務幾個月。如何降低雲端服務的計費成本，而不遺失 IP 位址？](#i-want-to-shut-down-the-cloud-service-for-several-months-how-to-reduce-the-billing-cost-of-cloud-service-without-losing-the-ip-address)


## <a name="certificates"></a>憑證

### <a name="why-is-the-certificate-chain-of-my-cloud-service-ssl-certificate-incomplete"></a>為什麼我雲端服務 SSL 憑證的信任鏈結是不完整的？
    
我們建議客戶安裝完整的憑證鏈結而不是分葉憑證 (分葉憑證、中繼憑證、和根憑證)。 當您安裝分葉憑證時，會依賴 Windows 透過查核 CTL 來建置憑證鏈結。 如果當 Windows 嘗試驗證憑證時，在 Azure 或 Windows Update 中發生間歇性網路或 DNS 問題，就可能會將憑證視為無效。 藉由安裝完整的憑證鏈結，就可以避免這個問題。 [如何安裝鏈結的 SSL 憑證](https://blogs.msdn.microsoft.com/azuredevsupport/2010/02/24/how-to-install-a-chained-ssl-certificate/)中的部落格會示範如何執行這項操作。

### <a name="what-is-the-purpose-of-the-windows-azure-tools-encryption-certificate-for-extensions"></a>「Windows Azure Tools 擴充功能的加密憑證」用途為何？

每當有擴充功能新增至雲端服務時，就會自動建立這些憑證。 大多數情況下，這是 WAD 擴充功能或 RDP 擴充功能，但也可能是其他的，例如反惡意程式碼軟體或記錄收集器擴充功能。 這些憑證僅用於將擴充功能的私用組態進行加密和解密。 永遠不會檢查到期日，因此憑證是否過期並不重要。 

您可以忽略這些憑證。 如果您想要清除憑證，可以嘗試將它們全部刪除。 如果您嘗試刪除的憑證正在使用中，Azure 就會擲回錯誤。

### <a name="how-can-i-generate-a-certificate-signing-request-csr-without-rdp-ing-in-to-the-instance"></a>如何能夠產生憑證簽署要求 (CSR)，而不 "RDP" 到執行個體中？

請參閱下列指導方針文件：

[使用 Windows Azure 網站 (WAWS) 取得要使用的憑證](https://azure.microsoft.com/blog/obtaining-a-certificate-for-use-with-windows-azure-web-sites-waws/)

CSR 只是文字檔。 不必從最終會使用憑證的電腦建立它。 雖然是針對 App Service 寫入這份文件，但 CSR 建立為泛型，且也適用於雲端服務。

### <a name="my-cloud-service-management-certificate-is-expiring-how-to-renew-it"></a>我的雲端服務管理憑證即將到期。 要如何續訂？

您可以使用下列 PowerShell 命令來更新管理憑證：

    Add-AzureAccount
    Select-AzureSubscription -Current -SubscriptionName <your subscription name>
    Get-AzurePublishSettingsFile

**Get-AzurePublishSettingsFile** 會在 Azure 入口網站的 [訂用帳戶] > [管理憑證] 中建立新的管理憑證。 新憑證的名稱如下 "YourSubscriptionNam]-[CurrentDate]-credentials"。

### <a name="how-to-automate-the-installation-of-main-ssl-certificatepfx-and-intermediate-certificatep7b"></a>如何將主要 SSL 憑證 (.pfx) 和中繼憑證 (.p7b) 的安裝自動化？

您可以使用啟動指令碼 (batch/cmd/PowerShell) 將這項工作自動化，並在服務定義檔中註冊該啟動指令碼。 將啟動指令碼和憑證 (.p7b 檔案) 新增至與啟動指令碼相同目錄的專案資料夾中。

### <a name="what-is-the-purpose-of-the-microsoft-azure-service-management-for-machinekey-certificate"></a>「適用於 MachineKey 的 Microsoft Azure 服務管理」憑證的用途為何？

此憑證用來加密 Azure Web 角色上的電腦金鑰。 若要進一步了解，請參閱此諮詢[https://docs.microsoft.com/security-updates/securityadvisories/2018/4092731]。

如需詳細資訊，請參閱下列文章：
- [如何設定和執行雲端服務的啟動工作](https://docs.microsoft.com/azure/cloud-services/cloud-services-startup-tasks)
- [常見的雲端服務啟動工作](https://docs.microsoft.com/azure/cloud-services/cloud-services-startup-tasks-common)

## <a name="monitoring-and-logging"></a>監視和記錄

### <a name="what-are-the-upcoming-cloud-service-capabilities-in-the-azure-portal-which-can-help-manage-and-monitor-applications"></a>即將在 Azure 入口網站推出的雲端服務功能有哪些可協助管理和監視應用程式？

即將推出為遠端桌面通訊協定 (RDP) 產生新憑證的功能。 或者，您也可以執行下列指令碼：

```powershell
$cert = New-SelfSignedCertificate -DnsName yourdomain.cloudapp.net -CertStoreLocation "cert:\LocalMachine\My" -KeyLength 20 48 -KeySpec "KeyExchange"
$password = ConvertTo-SecureString -String "your-password" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath ".\my-cert-file.pfx" -Password $password
```
即將推出可針對您的 csdef 和 cscfg 上傳位置選擇 Blob 或本機的功能。 使用 [New-AzureDeployment](/powershell/module/servicemanagement/azure/new-azuredeployment?view=azuresmps-4.0.0)，您可以設定每個位置的值。

監視執行個體層級計量的功能。 [如何監視雲端服務](cloud-services-how-to-monitor.md)中還有更多其他監視功能。

### <a name="why-does-iis-stop-writing-to-the-log-directory"></a>為什麼 IIS 會停止寫入記錄目錄？
您已耗盡寫入記錄目錄的本機儲存體配額。 若要修正此問題，您可以執行下列三個項目的其中一項：
* 啟用 IIS 的診斷，並定期將診斷移至 blob 儲存體中。
* 從記錄目錄中手動移除記錄檔。
* 增加本機資源的配額限制。

如需詳細資訊，請參閱下列文件：
* [在 Azure 儲存體中儲存和檢視診斷資料](cloud-services-dotnet-diagnostics-storage.md)
* [IIS 記錄會停止在雲端服務中寫入](https://blogs.msdn.microsoft.com/cie/2013/12/21/iis-logs-stops-writing-in-cloud-service/)

### <a name="how-do-i-enable-wad-logging-for-cloud-services"></a>如何為雲端服務啟用 WAD 記錄？
您可以透過下列選項來啟用 Windows Azure 診斷 (WAD) 記錄：
1. [從 Visual Studio 啟用](https://docs.microsoft.com/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines#turn-on-diagnostics-in-cloud-service-projects-before-you-deploy-them)
2. [透過 .Net 程式碼啟用](https://docs.microsoft.com/azure/cloud-services/cloud-services-dotnet-diagnostics)
3. [透過 PowerShell 啟用](https://docs.microsoft.com/azure/cloud-services/cloud-services-diagnostics-powershell)

若要取得雲端服務的目前 WAD 設定，您可以使用 [Get-AzureServiceDiagnosticsExtensions](https://docs.microsoft.com/azure/cloud-services/cloud-services-diagnostics-powershell#get-current-diagnostics-extension-configuration) ps cmd 或您可以從入口網站的 [雲端服務] --> [延伸模組] 刀鋒視窗中檢視它。


## <a name="network-configuration"></a>網路組態

### <a name="how-do-i-set-the-idle-timeout-for-azure-load-balancer"></a>如何設定 Azure Load Balancer 的閒置逾時？
您可以在服務定義 (csdef) 檔中指定逾時，就像這樣：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="mgVS2015Worker" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRole1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
    </ConfigurationSettings>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="tcp" port="10100"   idleTimeoutInMinutes="30" />
    </Endpoints>
  </WorkerRole>
```
如需詳細資訊，請參閱[新增：Azure Load Balancer 的可設定閒置逾時](https://azure.microsoft.com/blog/new-configurable-idle-timeout-for-azure-load-balancer/)。

### <a name="how-do-i-associate-a-static-ip-address-to-my-cloud-service"></a>如何將靜態 IP 位址關聯到我的雲端服務？
若要設定靜態 IP 位址，您必須建立保留的 IP。 這個保留的 IP 可以關聯到新的雲端服務或現有的部署。 請參閱以下文件了解詳細資料：
* [如何建立保留的 IP 位址](../virtual-network/virtual-networks-reserved-public-ip.md#manage-reserved-vips)
* [保留現有雲端服務的 IP 位址](../virtual-network/virtual-networks-reserved-public-ip.md#reserve-the-ip-address-of-an-existing-cloud-service)
* [建立保留的 IP 至新雲端服務的關聯](../virtual-network/virtual-networks-reserved-public-ip.md#associate-a-reserved-ip-to-a-new-cloud-service)
* [建立保留的 IP 至執行中部署的關聯](../virtual-network/virtual-networks-reserved-public-ip.md#associate-a-reserved-ip-to-a-running-deployment)
* [使用服務組態檔建立保留的 IP 至雲端服務的關聯](../virtual-network/virtual-networks-reserved-public-ip.md#associate-a-reserved-ip-to-a-cloud-service-by-using-a-service-configuration-file)

### <a name="what-are-the-features-and-capabilities-that-azure-basic-ipsids-and-ddos-provides"></a>Azure 的基本 IPS/IDS 和 DDOS 提供的特性和功能是什麼？
Azure 在資料中心實體伺服器中擁有 IP/ID 可以防範潛威脅。 此外，客戶可以部署第三方安全性解決方案，例如 Web 應用程式防火牆、網路防火牆、反惡意程式碼軟體、入侵偵測、預防系統 (IDS/IPS) 等等。 如需詳細資訊，請參閱[保護您的資料和資產，以及符合全域安全性標準](https://www.microsoft.com/en-us/trustcenter/Security/AzureSecurity)。

Microsoft 會持續監視伺服器、網路和應用程式來偵測威脅。 Azure 的多元化威脅管理方法會使用入侵偵測、分散式阻斷服務 (DDoS) 攻擊防護、滲透測試、行為分析、異常偵測和機器學習，不斷地加強其防禦並降低風險。 適用於 Azure 保護的 Azure 雲端服務和虛擬機器的 Microsoft Antimalware。 您可以選擇額外部署第三方安全性解決方案，例如 Web 應用程式防火牆、網路防火牆、反惡意程式碼軟體、入侵偵測以及預防系統 (IDS/IPS) 等等。

### <a name="how-to-enable-http2-on-cloud-services-vm"></a>如何啟用雲端服務 VM 上的 HTTP/2？

Windows 10 和 Windows Server 2016 隨附用戶端和伺服器端上的 HTTP/2 支援。 如果您的用戶端 (瀏覽器) 透過利用 TLS 擴充功交涉 HTTP/2 的 TLS 來連線到 IIS 伺服器，您就不需要在伺服器端上進行任何變更。 這是因為依預設會透過 TLS 傳送指定使用 HTTP/2 的 h2-14 標頭。 另一方面，如果您的用戶端要傳送升級標頭來升級至 HTTP/2，您就需要在伺服器端進行下列變更，以確保可進行升級，並以 HTTP/2 連線作為結尾。 

1. 執行 regedit.exe。
2. 瀏覽至登錄機碼：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HTTP\Parameters。
3. 建立名為 **DuoEnabled** 的新 DWORD 值。
4. 將值設為 1。
5. 重新啟動您的伺服器。
6. 移至**預設網站**，並在 [繫結] 下方 使用剛才建立的自我簽署憑證來建立新的 TLS 繫結。 

如需詳細資訊，請參閱

- [IIS 上的 HTTP/2](https://blogs.iis.net/davidso/http2)
- [影片：Windows 10 中的 HTTP/2：瀏覽器、應用程式和 Web 伺服器](https://channel9.msdn.com/Events/Build/2015/3-88)
         

您可透過啟動工作將這些步驟自動化，每當建立新的 PaaS 執行個體時，它可以在系統登錄中執行上述變更。 如需詳細資訊，請參閱[如何設定和執行雲端服務的啟動工作](cloud-services-startup-tasks.md)。

 
這項作業完成之後，您可以使用下列方法之一來確認是否已啟用 HTTP/2：

- 啟用 IIS 記錄中的通訊協定版本，並查看 IIS 記錄。 它會在記錄中顯示 HTTP/2。 
- 若要確認通訊協定，請在 Internet Explorer/Edge 中啟用 F12 開發人員工具，並切換至 [網路] 索引標籤。 

如需詳細資訊，請參閱 [IIS 上的 HTTP/2](https://blogs.iis.net/davidso/http2)。

## <a name="permissions"></a>權限

### <a name="how-can-i-implement-role-based-access-for-cloud-services"></a>如何實作雲端服務的角色型存取？
雲端服務不支援角色型存取控制 (RBAC) 模型，因為它不是以 Azure Resource Manager 為基礎的服務。

請參閱[了解 Azure 中的不同角色](../role-based-access-control/rbac-and-directory-admin-roles.md)。

## <a name="remote-desktop"></a>遠端桌面

### <a name="can-microsoft-internal-engineers-remote-desktop-to-cloud-service-instances-without-permission"></a>Microsoft 內部工程師是否可在沒有權限的情況下，從遠端桌面到雲端服務執行個體？
Microsoft 會遵循嚴格的程序，不允許內部工程師在沒有擁有者或其受指派者寫入權限的情況下，從遠端桌面登入您的雲端服務 (電子郵件或其他撰寫的通訊) 中。

### <a name="i-cannot-remote-desktop-to-cloud-service-vm--by-using-the-rdp-file-i-get-following-error-an-authentication-error-has-occurred-code-0x80004005"></a>我無法使用 RDP 檔案從遠端桌面登入雲端服務虛擬機器。 我收到下列錯誤：發生驗證錯誤 (代碼：0x80004005)

如果您使用的 RDP 檔案來自已加入 Azure Active Directory 的機器，即可能會發生這個錯誤。 若要解決此問題，請依照下列步驟執行︰

1. 以滑鼠右鍵按一下您下載的 RDP 檔案，然後選取 [編輯]。
2. 新增 "&#92;" 作為使用者名稱的前置詞。 例如，使用 **.\username** 而不是 **username**。

## <a name="scaling"></a>調整大小

### <a name="i-cannot-scale-beyond-x-instances"></a>我不能調整超過 X 個執行個體
您的 Azure 訂用帳戶對於您可以使用的核心數目有限制。 如果您已使用所有可用的核心，調整將無法運作。 例如，如果您有 100 個核心的限制，這表示您的雲端服務可以有 100 個 A1 大小的虛擬機器執行個體，或 50 個 A2 大小的虛擬機器執行個體。

### <a name="how-can-i-configure-auto-scale-based-on-memory-metrics"></a>如何根據記憶體計量設定自動縮放？

目前不支援雲端服務根據記憶體計量設定自動縮放。 

若要解決這個問題，您可以使用 Application Insights。 自動縮放可支援 Application Insights 作為計量來源，而且可以根據來賓計量 (如「記憶體」) 來縮放角色執行個體計數。  您必須在雲端服務專案封裝檔案 (*.cspkg) 中設定 Application Insights，並在服務上啟用 Azure 診斷擴充功能來實作此功能。

有關如何透過 Application Insights 使用自訂計量以在雲端服務上設定自動縮放的詳細資訊，請參閱[開始在 Azure 中依自訂計量自動縮放](../monitoring-and-diagnostics/monitoring-autoscale-scale-by-custom-metric.md)

有關如何將 Azure 診斷與雲端服務的 Application Insights 整合的詳細資訊，請參閱[傳送雲端服務、虛擬機器或 Service Fabric 診斷資料至 Application Insights](../monitoring-and-diagnostics/azure-diagnostics-configure-application-insights.md)

有關啟用雲端服務 Application Insights 的詳細資訊，請參閱 [Azure 雲端服務的 Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-cloudservices)

有關如何啟用雲端服務 Azure 診斷記錄的詳細資訊，請參閱[為 Azure 雲端服務和虛擬機器設定診斷](../vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md#turn-on-diagnostics-in-cloud-service-projects-before-you-deploy-them)

## <a name="generic"></a>泛型

### <a name="how-do-i-add-nosniff-to-my-website"></a>如何將 "nosniff" 新增至我的網站？
若要防止用戶端探查 MIME 類型，請在您的 *web.config* 檔案中加入一項設定。

```xml
<configuration>
   <system.webServer>
      <httpProtocol>
         <customHeaders>
            <add name="X-Content-Type-Options" value="nosniff" />
         </customHeaders>
      </httpProtocol>
   </system.webServer>
</configuration>
```

您也可以在 IIS 中將此加入為設定。 請參考[常見的啟動工作](cloud-services-startup-tasks-common.md#configure-iis-startup-with-appcmdexe)一文來使用下列命令。

```cmd
%windir%\system32\inetsrv\appcmd set config /section:httpProtocol /+customHeaders.[name='X-Content-Type-Options',value='nosniff']
```

### <a name="how-do-i-customize-iis-for-a-web-role"></a>如何自訂 Web 角色的 IIS？
請從[常見的啟動工作](cloud-services-startup-tasks-common.md#configure-iis-startup-with-appcmdexe)一文使用 IIS 啟動指令碼。

### <a name="what-is-the-quota-limit-for-my-cloud-service"></a>我的雲端服務配額限制是多少？
請參閱[特定服務的限制](../azure-subscription-service-limits.md#subscription-limits)。

### <a name="why-does-the-drive-on-my-cloud-service-vm-show-very-little-free-disk-space"></a>為什麼我雲端服務虛擬機器上的磁碟機顯示幾乎沒有可用的磁碟空間？
這是預期的行為，並不會對您的應用程式造成任何問題。 在 Azure PaaS 虛擬機器中會開啟 %approot% 磁碟機的日誌記錄，基本上會消耗兩倍檔案通常所佔用的空間量。 不過，要留意幾件事，基本上這就會變得沒有問題。

%approot% 磁碟機大小會以 <.cspkg 的大小 + 最大的日誌大小 + 可用空間的邊界> 來計算，或 1.5 GB，兩者取其較大。 您 VM 的大小對這個計算方式並無任何影響。 (VM 大小只會影響暫存 C: 磁碟機的大小。) 

它不支援寫入 %approot% 磁碟機。 如果您要寫入 Azure VM 中，必須在暫存 LocalStorage 資源中進行 (或其他選項，例如 Blob 儲存體、Azure 檔案等)。 因此在 %approot% 資料夾上的可用空間數量沒有任何意義。 如果您不確定應用程式是否要寫入 %approot% 磁碟機中，一律可以讓您的服務執行幾天，然後比較「之前」和「之後」的大小。 

Azure 不會將任何內容寫入 %approot% 磁碟機。 一旦從 .cspkg 建立 VHD 並將其掛接至 Azure VM 後，唯一可能會寫入此磁碟機的就是您的應用程式。 

日誌設定是不可設定的，因此您無法將它關閉。

### <a name="how-can-i-add-an-antimalware-extension-for-my-cloud-services-in-an-automated-way"></a>如何以自動化方式新增雲端服務的反惡意程式碼擴充功能？

您可以在「啟動工作」中使用 PowerShell 指令碼來啟用反惡意程式碼擴充功能。 請遵循下列這些文章中的步驟加以實作： 
 
- [建立 PowerShell 啟動工作](cloud-services-startup-tasks-common.md#create-a-powershell-startup-task)
- [Set-AzureServiceAntimalwareExtension](https://docs.microsoft.com/powershell/module/servicemanagement/azure/Set-AzureServiceAntimalwareExtension?view=azuresmps-4.0.0 )

如需反惡意程式碼部署情節及如何從入口網站加以啟用的詳細資訊，請參閱[反惡意程式碼部署情節](../security/azure-security-antimalware.md#antimalware-deployment-scenarios)。

### <a name="how-to-enable-server-name-indication-sni-for-cloud-services"></a>如何啟用雲端服務的伺服器名稱指示 (SNI)？

您可以使用下列其中一個方法來啟用雲端服務中的 SNI：

**方法 1：使用 PowerShell**

您可以在雲端服務角色執行個體的啟動工作中使用 PowerShell Cmdlet **New-WebBinding** 來設定 SNI 繫結，如下所示：
    
    New-WebBinding -Name $WebsiteName -Protocol "https" -Port 443 -IPAddress $IPAddress -HostHeader $HostHeader -SslFlags $sslFlags 
    
如[這裡](https://technet.microsoft.com/library/ee790567.aspx)所述，$sslFlags 可能是如下所示其中一個值：

|值|意義|
------|------
|0|沒有 SNI|
|1|已啟用 SNI |
|2 |使用中央憑證存放區的非 SNI 繫結|
|3|使用中央憑證存放區的 SNI 繫結 |
 
**方法 2：使用程式碼**

也可以透過角色啟動中的程式碼來設定 SNI 繫結，如這個[部落格文章](https://blogs.msdn.microsoft.com/jianwu/2014/12/17/expose-ssl-service-to-multi-domains-from-the-same-cloud-service/)所述：

    
    //<code snip> 
                    var serverManager = new ServerManager(); 
                    var site = serverManager.Sites[0]; 
                    var binding = site.Bindings.Add(“:443:www.test1.com”, newCert.GetCertHash(), “My”); 
                    binding.SetAttributeValue(“sslFlags”, 1); //enables the SNI 
                    serverManager.CommitChanges(); 
    //</code snip> 
    
使用上述的任何方法，必須先使用啟動工作或透過程式碼在角色執行個體上安裝特定主機名稱的個別憑證 (*.pfx)，SNI 繫結才會有效。

### <a name="how-can-i-add-tags-to-my-azure-cloud-service"></a>如何將標籤新增至我的 Azure 雲端服務？ 

雲端服務是傳統資源。 只有透過 Azure Resource Manager 建立的資源才能支援標籤。 您無法將標籤套用到傳統資源 (例如雲端服務)。 

### <a name="the-azure-portal-doesnt-display-the-sdk-version-of-my-cloud-service-how-can-i-get-that"></a>Azure 入口網站不會顯示雲端服務的 SDK 版本。 如何取得版本？

我們正設法將這項功能放到 Azure 入口網站上。 同時，您可以使用下列 PowerShell 命令取得 SDK 版本：

    Get-AzureService -ServiceName "<Cloud Service name>" | Get-AzureDeployment | Where-Object -Property SdkVersion -NE -Value "" | select ServiceName,SdkVersion,OSVersion,Slot

### <a name="i-want-to-shut-down-the-cloud-service-for-several-months-how-to-reduce-the-billing-cost-of-cloud-service-without-losing-the-ip-address"></a>我想要關閉雲端服務幾個月。 如何降低雲端服務的計費成本，而不遺失 IP 位址？

已部署的雲端服務會取得所使用的計算服務和儲存體之計費。 因此即使您關閉 Azure 虛擬機器，仍會對儲存體計費。 

以下方法可以降低您的計費，而不會遺失服務的 IP 位址：

1. 在刪除部署之前[保留 IP 位址](../virtual-network/virtual-networks-reserved-public-ip.md)。  只會針對此 IP 位址計費。 如需 IP 位址計費的詳細資訊，請參閱 [IP 位址定價](https://azure.microsoft.com/pricing/details/ip-addresses/)。
2. 刪除部署。 請勿刪除 xxx.cloudapp.net，以便您未來繼續使用。
3. 如果想要使用您在訂用帳戶中保留的同一保留 IP 來重新部署雲端服務，請參閱[雲端服務和虛擬機器的保留 IP 位址](https://azure.microsoft.com/blog/reserved-ip-addresses/)。

