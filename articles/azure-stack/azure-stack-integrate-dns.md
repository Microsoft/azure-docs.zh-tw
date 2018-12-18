---
title: Azure Stack 資料中心整合 - DNS
description: 了解如何將 Azure Stack DNS 與您的資料中心 DNS 整合
services: azure-stack
author: jeffgilb
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 02/28/2018
ms.author: jeffgilb
ms.reviewer: wfayed
keywords: ''
ms.openlocfilehash: b4935dc95ccf525c0a40b10dcc8c59ec8aba710e
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "41946456"
---
# <a name="azure-stack-datacenter-integration---dns"></a>Azure Stack 資料中心整合 - DNS
若要能夠從 Azure Stack 外部存取 Azure Stack 端點 (`portal`、`adminportal`、`management`、`adminmanagement` 等)，您必須將 Azure Stack DNS 服務與裝載您想要在 Azure Stack 中使用之 DNS 區域的 DNS 伺服器整合。

## <a name="azure-stack-dns-namespace"></a>Azure Stack DNS 命名空間
部署 Azure Stack 時，您必須提供一些與 DNS 相關的重要資訊。


|欄位  |說明  |範例|
|---------|---------|---------|
|區域|您 Azure Stack 部署的地理位置。|`east`|
|外部網域名稱|您想要用於 Azure Stack 部署的區域名稱。|`cloud.fabrikam.com`|
|內部網域名稱|用於 Azure Stack 中基礎結構服務的內部區域名稱。  它是「目錄服務」整合的且是私人的 (無法從 Azure Stack 部署外部連線)。|`azurestack.local`|
|DNS 轉寄站|用來轉送裝載於 Azure Stack 外部 (公司內部網路或公用網際網路) 之 DNS 查詢、DNS 區域及記錄的 DNS 伺服器。|`10.57.175.34`<br>`8.8.8.8`|
|命名前置詞 (選擇性)|您想要讓 Azure Stack 基礎結構角色執行個體電腦名稱擁有的命名前置詞。  如果未提供，則預設值為 `azs`。|`azs`|

您 Azure Stack 部署和端點的完整網域名稱 (FQDN) 是「區域」參數和「外部網域名稱」參數的組合。 使用上表中的範例值時，此 Azure Stack 部署的 FQDN 會是以下名稱：

`east.cloud.fabrikam.com`

因此，此部署的一些端點範例會類似下列 URL：

`https://portal.east.cloud.fabrikam.com`

`https://adminportal.east.cloud.fabrikam.com`

若要將此範例 DNS 命名空間用於 Azure Stack 部署，必須符合下列條件：

- `fabrikam.com` 區域已向網域註冊機構、內部公司 DNS 伺服器或這兩者註冊 (視您的名稱解析需求而定)。
- 子網域 `cloud.fabrikam.com` 存在於 `fabrikam.com`區域底下。
- 從 Azure Stack 部署可以連線到裝載 `fabrikam.com` 和 `cloud.fabrikam.com` 區域的 DNS 伺服器。

若要能夠從 Azure Stack 外部解析 Azure Stack 端點和執行個體的 DNS 名稱，您必須將裝載 Azure Stack 外部 DNS 區域的 DNS 伺服器與裝載所要使用之上層區域的 DNS 伺服器整合。


## <a name="resolution-and-delegation"></a>解析和委派

有兩種類型的 DNS 伺服器：

- 權威 DNS 伺服器會裝載 DNS 區域。 它只會回答這些區域中的 DNS 記錄查詢。
- 遞迴 DNS 伺服器不會裝載 DNS 區域。 它會呼叫授權 DNS 伺服器來收集所需的資料，以回答所有 DNS 查詢。

Azure Stack 同時包含權威和遞迴 DNS 伺服器。 遞迴伺服器可用來解析所有項目的名稱，但該 Azure Stack 部署的內部私人區域和外部公用 DNS 區域除外。 

![Azure Stack DNS 架構](media/azure-stack-integrate-dns/Integrate-DNS-01.png)

## <a name="resolving-external-dns-names-from-azure-stack"></a>從 Azure Stack 解析外部 DNS 名稱

若要解析 Azure Stack 外部端點的 DNS 名稱 (例如 www.bing.com)，您必須提供 Azure Stack 可用來轉送 Azure Stack 對其而言不具權威性之 DNS 要求的 DNS 伺服器。 為了進行部署，「部署工作表」中必須要有作為 Azure Stack 轉送要求目的地的 DNS 伺服器 (在 [DNS 轉寄站] 欄位中)。 請至少在此欄位中提供兩個伺服器以供容錯使用。 如果沒有這些值，Azure Stack 部署將會失敗。

### <a name="configure-conditional-dns-forwarding"></a>設定條件式 DNS 轉送

> [!IMPORTANT]
> 這僅適用 AD FS 部署。

若要使用現有的 DNS 基礎結構啟用名稱解析，請設定條件式轉送。

若要新增條件式轉寄站，您必須使用特殊權限端點。

針對此程序，請使用您資料中心網路內能夠與 Azure Stack 中具特殊權限端點通訊的電腦。

1. 開啟一個已提高權限的 Windows PowerShell 工作階段 (以系統管理員身分執行)，然後連線到具特殊權限端點的 IP 位址。 使用適用於 CloudAdmin 驗證的認證。

   ```
   $cred=Get-Credential 
   Enter-PSSession -ComputerName <IP Address of ERCS> -ConfigurationName PrivilegedEndpoint -Credential $cred
   ```

2. 連線到具特殊權限的端點之後，執行下列 PowerShell 命令。 使用您想要使用之 DNS 伺服器的網域名稱和 IP 位址取代所提供的範例值。

   ```
   Register-CustomDnsServer -CustomDomainName "contoso.com" -CustomDnsIPAddresses "192.168.1.1","192.168.1.2"
   ```

## <a name="resolving-azure-stack-dns-names-from-outside-azure-stack"></a>從 Azure Stack 外部解析 Azure Stack DNS 名稱
權威伺服器是保存外部 DNS 區域資訊及任何使用者建立之區域的伺服器。 請與這些伺服器整合來啟用區域委派或條件性轉送功能，以從 Azure Stack 外部解析 Azure Stack DNS 名稱。

## <a name="get-dns-server-external-endpoint-information"></a>取得 DNS 伺服器外部端點資訊

若要將您的 Azure Stack 部署與 DNS 基礎結構整合，您需要下列資訊：

- DNS 伺服器 FQDN
- DNS 伺服器 IP 位址

Azure Stack DNS 伺服器的 FQDN 具有下列格式：

`<NAMINGPREFIX>-ns01.<REGION>.<EXTERNALDOMAINNAME>`

`<NAMINGPREFIX>-ns02.<REGION>.<EXTERNALDOMAINNAME>`

使用範例值時，DNS 伺服器的 FQDN 為：

`azs-ns01.east.cloud.fabrikam.com`

`azs-ns02.east.cloud.fabrikam.com`


在所有 Azure Stack 部署結束時，名為 `AzureStackStampDeploymentInfo.json` 的檔案中也會建立此資訊。 此檔案位於部署虛擬機器的 `C:\CloudDeployment\logs` 資料夾中。 如果您不確定用於 Azure Stack 部署的值是哪些值，便可以從該處取得那些值。

如果部署虛擬機器已不再可供使用或無法存取，則您可以連線到具特殊權限的端點並執行 `Get-AzureStackInfo` PowerShell Cmdlet 來取得那些值。 如需詳細資訊，請參閱 [具特殊權限的端點](azure-stack-privileged-endpoint.md)。

## <a name="setting-up-conditional-forwarding-to-azure-stack"></a>設定轉送到 Azure Stack 的條件性轉送

將 Azure Stack 與 DNS 基礎結構整合的最簡單且最安全的方式就是：從裝載上層區域的伺服器執行條件性區域轉送。 如果您可以直接控制裝載 Azure Stack 外部 DNS 命名空間之上層區域的 DNS 伺服器，便建議使用此方法。

如果您不熟悉如何使用 DNS 來進行條件性轉送，請參閱下列 TechNet 文章：[指派網域名稱的條件式轉寄站](https://technet.microsoft.com/library/cc794735) \(英文\)，或您 DNS 解決方案專屬的文件。

如果您將外部「Azure Stack DNS 區域」指定成看起來像公司網域名稱的子網域，就無法使用條件性轉送。 必須設定 DNS 委派。

範例：

- 公司 DNS 網域名稱：`contoso.com`
- Azure Stack 外部 DNS 網域名稱：`azurestack.contoso.com`

## <a name="delegating-the-external-dns-zone-to-azure-stack"></a>將外部 DNS 區域委派給 Azure Stack

若要能夠從 Azure Stack 部署外部解析 DNS 名稱，您必須設定 DNS 委派。

每個註冊機構都有自己的 DNS 管理工具，可變更網域的名稱伺服器記錄。 請在註冊機構的 DNS 管理頁面中，編輯 NS 記錄，然後以 Azure Stack 中的 NS 記錄取代區域的 NS 記錄。

大多數 DNS 註冊機構會要求您至少要提供兩個 DNS 伺服器，才能完成委派。

## <a name="next-steps"></a>後續步驟

[防火牆整合](azure-stack-firewall.md)
