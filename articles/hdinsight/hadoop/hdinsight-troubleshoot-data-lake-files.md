---
title: 無法存取 Azure HDInsight 中的 Data Lake 儲存體檔案
description: 無法存取 Azure HDInsight 中的 Data Lake 儲存體檔案
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/13/2019
ms.openlocfilehash: f4c5a23b604334952730fcc4cf1fcb3fcbed6237
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98944392"
---
# <a name="unable-to-access-data-lake-storage-files-in-azure-hdinsight"></a>無法存取 Azure HDInsight 中的 Data Lake 儲存體檔案

本文說明與 Azure HDInsight 叢集互動時，問題的疑難排解步驟和可能的解決方式。

## <a name="issue-acl-verification-failed"></a>問題： ACL 驗證失敗

您會收到類似下列的錯誤訊息：

```
LISTSTATUS failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.).
```

### <a name="cause"></a>原因

使用者可能已撤銷檔案/資料夾上 (SP) 的服務主體許可權。

### <a name="resolution"></a>解決方案

1. 確認 SP 具有「x」許可權，可沿著路徑進行往返。 如需詳細資訊，請參閱 [權限](https://hdinsight.github.io/ClusterCRUD/ADLS/adls-create-permission-setup.html)。 `dfs`檢查存取 Data Lake 儲存體帳戶中的檔案/資料夾的範例命令：

    ```
    hdfs dfs -ls /<path to check access>
    ```

1. 設定必要的許可權，以根據所執行的讀取/寫入操作來存取路徑。 請參閱這裡以取得各種檔案系統作業所需的許可權。

---

## <a name="issue-service-principal-certificate-expiry"></a>問題：服務主體憑證過期

您會收到類似下列的錯誤訊息：

```
Token Refresh failed - Received invalid http response: 500
```

### <a name="cause"></a>原因

為服務主體存取提供的憑證可能已過期。

1. 透過 SSH 連線到前端節點。 使用下列命令檢查儲存體帳戶的存取權 `dfs` ：

    ```
    hdfs dfs -ls /
    ```

1. 確認錯誤訊息類似于下列輸出：

    ```
    {"stderr": "-ls: Token Refresh failed - Received invalid http response: 500, text = Response{protocol=http/1.1, code=500, message=Internal Server Error, url=http://gw0-abccluster.24ajrd4341lebfgq5unsrzq0ue.fx.internal.cloudapp.net:909/api/oauthtoken}}...
    ```

1. 從取得其中一個 url `core-site.xml property`  -  `fs.azure.datalake.token.provider.service.urls` 。

1. 執行下列捲曲命令以取出 OAuth 權杖。

    ```
    curl gw0-abccluster.24ajrd4341lebfgq5unsrzq0ue.fx.internal.cloudapp.net:909/api/oauthtoken
    ```

1. 有效服務主體的輸出應該類似如下：

    ```
    {"AccessToken":"MIIGHQYJKoZIhvcNAQcDoIIGDjCCBgoCAQA…….","ExpiresOn":1500447750098}
    ```

1. 如果服務主體憑證已過期，則輸出看起來會像這樣：

    ```
    Exception in OAuthTokenController.GetOAuthToken: 'System.InvalidOperationException: Error while getting the OAuth token from AAD for AppPrincipalId 23abe517-2ffd-4124-aa2d-7c224672cae2, ResourceUri https://management.core.windows.net/, AADTenantId https://login.windows.net/80abc8bf-86f1-41af-91ab-2d7cd011db47, ClientCertificateThumbprint C49C25705D60569884EDC91986CEF8A01A495783 ---> Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException: AADSTS70002: Error validating credentials. AADSTS50012: Client assertion contains an invalid signature. **[Reason - The key used is expired.**, Thumbprint of key used by client: 'C49C25705D60569884EDC91986CEF8A01A495783', Found key 'Start=08/03/2016, End=08/03/2017, Thumbprint=C39C25705D60569884EDC91986CEF8A01A4956D1', Configured keys: [Key0:Start=08/03/2016, End=08/03/2017, Thumbprint=C39C25705D60569884EDC91986CEF8A01A4956D1;]]
    Trace ID: e4d34f1c-a584-47f5-884e-1235026d5000
    Correlation ID: a44d870e-6f23-405a-8b23-9b44aebfa4bb
    Timestamp: 2017-10-06 20:44:56Z ---> System.Net.WebException: The remote server returned an error: (401) Unauthorized.
    at System.Net.HttpWebRequest.GetResponse()
    at Microsoft.IdentityModel.Clients.ActiveDirectory.HttpWebRequestWrapper.<GetResponseSyncOrAsync>d__2.MoveNext()
    ```

1. 任何其他 Azure Active Directory 相關錯誤/憑證相關的錯誤，都可以藉由 ping 閘道 url 來取得 OAuth 權杖來辨識。

1. 如果您在嘗試從 HDI 叢集中存取 ADLS 時收到下列錯誤。 遵循上述步驟來檢查憑證是否已過期。

    ```
    Error: java.lang.IllegalArgumentException: Token Refresh failed - Received invalid http response: 500, text = Response{protocol=http/1.1, code=500, message=Internal Server Error, url=http://clustername.hmssomerandomstringc.cx.internal.cloudapp.net:909/api/oauthtoken}
    ```

### <a name="resolution"></a>解決方案

使用下列 PowerShell 腳本建立新的憑證或指派現有的憑證：

```powershell
$clusterName = 'CLUSTERNAME'
$resourceGroupName = 'RGNAME'
$subscriptionId = 'SUBSCRIPTIONID'
$appId = 'APPLICATIONID'
$generateSelfSignedCert = $false
$addNewCertKeyCredential = $true
$certFilePath = 'NEW_CERT_PFX_LOCAL_PATH'
$certPassword = Read-Host "Enter Certificate Password"

if($generateSelfSignedCert)
{
    Write-Host "Generating new SelfSigned certificate"
    
    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=hdinsightAdlsCert" -KeySpec KeyExchange
    $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword);
    $certString = [System.Convert]::ToBase64String($certBytes)
}
else
{

    Write-Host "Reading the cert file from path $certFilePath"

    $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath, $certPassword)
    $certString = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($certFilePath))
}


Login-AzureRmAccount

if($addNewCertKeyCredential)
{
    Write-Host "Creating new KeyCredential for the app"

    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    
    New-AzureRmADAppCredential -ApplicationId $appId -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore

    Write-Host "Waiting for 30 seconds for the permissions to get propagated"

    Start-Sleep -s 30
}

Select-AzureRmSubscription -SubscriptionId $subscriptionId

Write-Host "Updating the certificate on HDInsight cluster."

Invoke-AzureRmResourceAction `
    -ResourceGroupName $resourceGroupName `
    -ResourceType 'Microsoft.HDInsight/clusters' `
    -ResourceName $clusterName `
    -ApiVersion '2015-03-01-preview' `
    -Action 'updateclusteridentitycertificate' `
    -Parameters @{ ApplicationId = $appId.ToString(); Certificate = $certString; CertificatePassword = $certPassword.ToString() } `
    -Force

```

若要指派現有的憑證，請建立憑證、備妥 .pfx 檔案和密碼。 使用 AppId ready，將憑證與建立叢集的服務主體建立關聯。

將參數取代為實際值之後，請執行 PowerShell 命令。

## <a name="next-steps"></a>後續步驟

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]