---
title: Azure Stack Syslog 轉送
description: 了解如何使用 Syslog 轉送將 Azure Stack 與監視解決方案整合
services: azure-stack
author: PatAltimore
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 08/14/2018
ms.author: patricka
ms.reviewer: fiseraci
keywords: ''
ms.openlocfilehash: 8e59f2e7e2fceda7f30e12571cd9e2a552f76231
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41946567"
---
# <a name="azure-stack-datacenter-integration---syslog-forwarding"></a>Azure Stack 資料中心整合 - Syslog 轉送

本文說明如何使用 Syslog 將 Azure Stack 基礎結構與您資料中心內已經部署的外部安全性解決方案整合。 例如，「安全性資訊與事件管理」(SIEM) 系統。 Syslog 通道會公開來自 Azure Stack 基礎結構之所有元件的稽核、警示及安全性記錄。 請使用 Syslog 轉送來與安全性監視解決方案整合和/或擷取所有稽核、警示及安全性記錄，來儲存它們以供保留。 

從 1805 更新開始，Azure Stack 便具有一個整合式 Syslog 用戶端，此用戶端在設定之後，就會發出 Syslog 訊息，訊息所含的承載會採用「常見事件格式」(CEF)。 

> [!IMPORTANT]
> Syslog 轉送目前為預覽版。 在生產環境中不應該倚賴此功能。 

下圖顯示參與 Syslog 整合的主要元件。

![Syslog 轉送圖](media/azure-stack-integrate-security/syslog-forwarding.png)

## <a name="configuring-syslog-forwarding"></a>設定 Syslog 轉送

Azure Stack 中的 Syslog 用戶端支援下列設定：

1. **使用相互驗證 (用戶端與伺服器) 和 TLS 1.2 加密的 Syslog over TCP：** 在此設定中，Syslog 伺服器與 Syslog 用戶端都可透過憑證驗證彼此的身分識別。 傳送訊息時，會透過 TLS 1.2 加密通道進行傳送。

2. **使用伺服器驗證和 TLS 1.2 加密的 Syslog over TCP：** 在此設定中，Syslog 用戶端可透過憑證驗證 Syslog 伺服器的身分識別。 傳送訊息時，會透過 TLS 1.2 加密通道進行傳送。

3. **不使用任何加密的 Syslog over TCP：** 在此設定中，Syslog 用戶端與 Syslog 伺服器都不會驗證彼此的身分識別。 傳送訊息時，會以純文字形式透過 TCP 傳送。

4. **不使用任何加密的 Syslog over UDP：** 在此設定中，Syslog 用戶端與 Syslog 伺服器都不會驗證彼此的身分識別。 傳送訊息時，會以純文字形式透過 UDP 傳送。

> [!IMPORTANT]
> 針對生產環境，Microsoft 強烈建議您採用使用驗證和加密的 TCP (第 1 種設定或至少第 2 種設定)，以抵禦攔截式攻擊和訊息竊聽。

### <a name="cmdlets-to-configure-syslog-forwarding"></a>用以設定 Syslog 轉送的 Cmdlet
若要設定 Syslog 轉送，必須存取具特殊權限的端點 (PEP)。 在 PEP 已新增兩個 PowerShell Cmdlet 來設定 Syslog 轉送：


```powershell
### cmdlet to pass the syslog server information to the client and to configure the transport protocol, the encryption and the authentication between the client and the server

Set-SyslogServer [-ServerName <String>] [-NoEncryption] [-SkipCertificateCheck] [-SkipCNCheck] [-UseUDP] [-Remove]

### cmdlet to configure the certificate for the syslog client to authenticate with the server

Set-SyslogClient [-pfxBinary <Byte[]>] [-CertPassword <SecureString>] [-RemoveCertificate] 
```
#### <a name="cmdlets-parameters"></a>Cmdlet 參數

*Set-SyslogServer* Cmdlet 的參數：

| 參數 | 說明 | 類型 |
|---------|---------| ---------|
| *ServerName* | Syslog 伺服器的 FQDN 或 IP 位址 | 字串 |
|*NoEncryption*| 強制用戶端以純文字形式傳送 Syslog 訊息 | 旗標 | 
|*SkipCertificateCheck*| 在初始 TLS 信號交換期間，針對 Syslog 伺服器所提供的憑證略過驗證 | 旗標 |
|*SkipCNCheck*| 在初始 TLS 信號交換期間，針對 Syslog 伺服器所提供憑證的「一般名稱」值略過驗證 | 旗標 |
|*UseUDP*| 使用 Syslog 搭配以 UDP 作為傳輸通訊協定 |旗標 |
|*Remove*| 從用戶端中移除伺服器設定並停止 Syslog 轉送| 旗標 |

*Set-SyslogClient* Cmdlet 的參數：
| 參數 | 說明 | 類型 |
|---------|---------| ---------|
| *pfxBinary* | 包含憑證的 pfx 檔案，該憑證是用戶端要用來作為身分識別以向 Syslog 伺服器進行驗證的憑證  | Byte[] |
| *CertPassword* |  可匯入與 pfx 檔案關聯之私密金鑰的密碼 | SecureString |
|*RemoveCertificate* | 從用戶端移除憑證 | 旗標|

### <a name="configuring-syslog-forwarding-with-tcp-mutual-authentication-and-tls-12-encryption"></a>設定使用 TCP、相互驗證及 TLS 1.2 加密的 Syslog 轉送

在此設定中，Azure Stack 中的 Syslog 用戶端會透過使用 TLS 1.2 加密的 TCP 將訊息轉送給 Syslog 伺服器。 在初始信號交換期間，用戶端會確認伺服器提供有效、受信任的憑證；同樣地，用戶端也會向伺服器提供憑證來作為其身分識別證明。 這是最安全的設定，因為它提供用戶端與伺服器兩者的完整身分識別驗證，並且是透過加密的通道傳送訊息。 

> [!IMPORTANT]
> Microsoft 強烈建議您針對生產環境使用此設定。 

若要設定使用 TCP、相互驗證及 TLS 1.2 加密的 Syslog 轉送，請執行以下兩個 Cmdlet：
```powershell
# Configure the server
Set-SyslogServer -ServerName <FQDN or ip address of syslog server>

# Provide certificate to the client to authenticate against the server
Set-SyslogClient -pfxBinary <Byte[] of pfx file> -CertPassword <SecureString, password for accessing the pfx file>
```
用戶端憑證的根必須與部署 Azure Stack 時所提供的根相同。 此外，此憑證也必須包含私密金鑰。

```powershell
##Example on how to set your syslog client with the ceritificate for mutual authentication. 
##Run these cmdlets from your hardware lifecycle host or privileged access workstation.

$ErcsNodeName = "<yourPEP>"
$password = ConvertTo-SecureString -String "<your cloudAdmin account password" -AsPlainText -Force
 
$cloudAdmin = "<your cloudAdmin account name>"
$CloudAdminCred = New-Object System.Management.Automation.PSCredential ($cloudAdmin, $password)
 
$certPassword = $password
$certContent = Get-Content -Path C:\cert\<yourClientCertificate>.pfx -Encoding Byte
 
$params = @{ 
    ComputerName = $ErcsNodeName 
    Credential = $CloudAdminCred 
    ConfigurationName = "PrivilegedEndpoint" 
}

$session = New-PSSession @params
 
$params = @{ 
    Session = $session 
    ArgumentList = @($certContent, $certPassword) 
}
Write-Verbose "Invoking cmdlet to set syslog client certificate..." -Verbose 
Invoke-Command @params -ScriptBlock { 
    param($CertContent, $CertPassword) 
    Set-SyslogClient -PfxBinary $CertContent -CertPassword $CertPassword 
```

### <a name="configuring-syslog-forwarding-with-tcp-server-authentication-and-tls-12-encryption"></a>設定使用 TCP、伺服器驗證及 TLS 1.2 加密的 Syslog 轉送

在此設定中，Azure Stack 中的 Syslog 用戶端會透過使用 TLS 1.2 加密的 TCP 將訊息轉送給 Syslog 伺服器。 在初始信號交換期間，用戶端也會確認伺服器提供有效、受信任的憑證。 這可防止用戶端將訊息傳送到不受信任的目的地。
使用驗證和加密的 TCP 是預設設定，並且代表 Microsoft 針對生產環境建議的最低安全性層級。 

```powershell
Set-SyslogServer -ServerName <FQDN or ip address of syslog server>
```

如果您想要使用自我簽署和/或未受信任的憑證來測試 Syslog 伺服器與 Azure Stack 用戶端的整合，您可以使用這些旗標，在初始信號交換期間略過用戶端所執行的伺服器驗證。

```powershell
 #Skip validation of the Common Name value in the server certificate. Use this flag if you provide an IP address for your syslog server
 Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -SkipCNCheck
 
 #Skip entirely the server certificate validation
 Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -SkipCertificateCheck
```
> [!IMPORTANT]
> Microsoft 建議針對生產環境不要使用 -SkipCertificateCheck 旗標。 


### <a name="configuring-syslog-forwarding-with-tcp-and-no-encryption"></a>設定使用 TCP 且不加密的 Syslog 轉送

在此設定中，Azure Stack 中的 Syslog 用戶端會透過 TCP，在不加密的情況下將訊息轉送給 Syslog 伺服器。 用戶端既不會驗證伺服器的身分識別，也不會向伺服器提供自己的身分識別來進行驗證。 

```powershell
Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -NoEncryption
```
> [!IMPORTANT]
> Microsoft 建議針對生產環境不要使用此設定。 


### <a name="configuring-syslog-forwarding-with-udp-and-no-encryption"></a>設定使用 UDP 且不加密的 Syslog 轉送

在此設定中，Azure Stack 中的 Syslog 用戶端會透過 UDP，在不加密的情況下將訊息轉送給 Syslog 伺服器。 用戶端既不會驗證伺服器的身分識別，也不會向伺服器提供自己的身分識別來進行驗證。 

```powershell
Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -UseUDP
```
雖然不使用加密的 UDP 在設定上最為簡單，但它對於攔截式攻擊和訊息竊聽無法提供任何防護。 

> [!IMPORTANT]
> Microsoft 建議針對生產環境不要使用此設定。 


## <a name="removing-syslog-forwarding-configuration"></a>移除 Syslog 轉送設定

將 Syslog 伺服器設定全部移除並停止 Syslog 轉送：

**從伺服器端中移除 Syslog 伺服器設定**

```PowerShell  
Set-SyslogServer -Remove
```

**從用戶端中移除用戶端憑證**

```PowerShell  
Set-SyslogClient -RemoveCertificate
```

## <a name="verifying-the-syslog-setup"></a>驗證 Syslog 設定

如果您已成功將 Syslog 用戶端連線到 Syslog 伺服器，應該很快就會開始收到事件。 如果沒有看到任何事件，請執行下列 Cmdlet 來驗證 Syslog 用戶端的設定：

**驗證 Syslog 用戶端中的伺服器設定**

```PowerShell  
Get-SyslogServer
```

**驗證 Syslog 用戶端中的憑證設定**

```PowerShell  
Get-SyslogClient
```

## <a name="syslog-message-schema"></a>Syslog 訊息結構描述

Azure Stack 基礎結構的 Syslog 轉送會傳送採用「常見事件格式」(CEF) 的訊息。
每個 Syslog 訊息都會根據此結構描述來建立結構： 

```Syslog
<Time> <Host> <CEF payload>
```

CEF 承載會根據下面的結構，但每個欄位的對應會依訊息的類型 (Windows 事件、已建立的警示、已關閉的警示) 而有所不同。

```CEF
# Common Event Format schema
CEF: <Version>|<Device Vendor>|<Device Product>|<Device Version>|<Signature ID>|<Name>|<Severity>|<Extensions>
* Version: 0.0 
* Device Vendor: Microsoft
* Device Product: Microsoft Azure Stack
* Device Version: 1.0
```

### <a name="cef-mapping-for-windows-events"></a>Windows 事件的 CEF 對應
```
* Signature ID: ProviderName:EventID
* Name: TaskName
* Severity: Level (for details, see the severity table below)
* Extension: Custom Extension Name (for details, see the Custom Extension table below)
```

Windows 事件的嚴重性表格： 
| CEF 嚴重性值 | Windows 事件層級 | 數值 |
|--------------------|---------------------| ----------------|
|0|Undefined|值：0。 指出所有層級的記錄|
|10|重要|值：1。 指出某個重要警示的記錄|
|8|Error| 值：2。 指出某個錯誤的記錄|
|5|警告|值：3。 指出某個警告的記錄|
|2|資訊|值：4。 指出某個告知性訊息的記錄|
|0|詳細資訊|值：5。 指出所有層級的記錄|

Azure Stack 中 Windows 事件的自訂延伸模組表格：
| 自訂延伸模組名稱 | Windows 事件範例 | 
|-----------------------|---------|
|MasChannel | 系統|
|MasComputer | test.azurestack.contoso.com|
|MasCorrelationActivityID| C8F40D7C-3764-423B-A4FA-C994442238AF|
|MasCorrelationRelatedActivityID| C8F40D7C-3764-423B-A4FA-C994442238AF|
|MasEventData| svchost!!4132,G,0!!!!EseDiskFlushConsistency!!ESENT!!0x800000|
|MasEventDescription| 已成功處理使用者的「群組原則」設定。 自上次成功處理「群組原則」之後，未偵測到任何變更。|
|MasEventID|1501|
|MasEventRecordID|26637|
|MasExecutionProcessID | 29380|
|MasExecutionThreadID |25480|
|MasKeywords |0x8000000000000000|
|MasKeywordName |稽核成功|
|MasLevel |4|
|MasOpcode |1|
|MasOpcodeName |info|
|MasProviderEventSourceName ||
|MasProviderGuid |AEA1B4FA-97D1-45F2-A64C-4D69FFFD92C9|
|MasProviderName |Microsoft-Windows-GroupPolicy|
|MasSecurityUserId |\<Windows SID\> |
|MasTask |0|
|MasTaskCategory| 處理序建立|
|MasUserData|KB4093112!!5112!!Installed!!0x0!!WindowsUpdateAgent Xpath: /Event/UserData/*|
|MasVersion|0|

### <a name="cef-mapping-for-alerts-created"></a>已建立之警示的 CEF 對應
```
* Signature ID: Microsoft Azure Stack Alert Creation : FaultTypeId
* Name: FaultTypeId : AlertId
* Severity: Alert Severity (for details, see alerts severity table below)
* Extension: Custom Extension Name (for details, see the Custom Extension table below)
```
警示嚴重性表格：
| 嚴重性 | 層級 |
|----------|-------|
|0|Undefined|
|10|重要|
|5|警告|

Azure Stack 中已建立之警示的自訂延伸模組表格：
| 自訂延伸模組名稱 | 範例 | 
|-----------------------|---------|
|MasEventDescription|描述：已為 \<TestDomain\> 建立使用者帳戶 \<TestUser\>。 這是一個潛在的安全性風險。 -- 補救：請連絡支援人員。 需要取得「客戶協助」來解決此問題。 請勿在沒有他們協助的情況下嘗試解決此問題。 在您開啟支援要求之前，請先參考 https://aka.ms/azurestacklogfiles 的指引來啟動記錄檔收集程序 |

### <a name="cef-mapping-for-alerts-closed"></a>已關閉之警示的 CEF 對應
```
* Signature ID: Microsoft Azure Stack Alert Creation : FaultTypeId
* Name: FaultTypeId : AlertId
* Severity: Information
```

以下範例顯示含有 CEF 承載的 Syslog 訊息：
```
2018:05:17:-23:59:28 -07:00 TestHost CEF:0.0|Microsoft|Microsoft Azure Stack|1.0|3|TITLE: User Account Created -- DESCRIPTION: A user account \<TestUser\> was created for \<TestDomain\>. It's a potential security risk. -- REMEDIATION: Please contact Support. Customer Assistance is required to resolve this issue. Do not try to resolve this issue without their assistance. Before you open a support request, start the log file collection process using the guidance from https://aka.ms/azurestacklogfiles|10
```
## <a name="next-steps"></a>後續步驟

[服務原則](azure-stack-servicing-policy.md)