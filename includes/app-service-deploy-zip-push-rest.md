## <a name="rest"></a>使用 REST API 部署 ZIP 檔案 

您可以使用[部署服務 REST API](https://github.com/projectkudu/kudu/wiki/REST-API)，在 Azure 中將 .zip 檔案部署至您的應用程式。 若要部署，請將 POST 要求傳送至 https://<app_name>.scm.azurewebsites.net/api/zipdeploy。 POST 要求必須在訊息本文中包含 .zip 檔案。 系統會使用 HTTP 基本驗證，在要求中提供應用程式的部署認證。 如需詳細資訊，請參閱 [.zip 推送部署參考](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)。 

針對 HTTP 基本驗證，您需要 App Service 部署的認證。 若要了解如何設定部署認證，請參閱[設定及重設使用者層級的認證](../articles/app-service/app-service-deployment-credentials.md#userscope)。

### <a name="with-curl"></a>使用 cURL

下列範例會使用 cURL 工具來部署 .zip 檔案。 取代預留位置 `<username>`、`<password>`、`<zip_file_path>` 和 `<app_name>`。 當 cURL 顯示提示時，請輸入密碼。

```bash
curl -X POST -u <deployment_user> --data-binary @"<zip_file_path>" https://<app_name>.scm.azurewebsites.net/api/zipdeploy
```

此要求會觸發從上傳的 .zip 檔案推送部署。 您可以使用 `https://<app_name>.scm.azurewebsites.net/api/deployments` 端點來檢閱目前和過去的部署，如下列 cURL 範例所示。 再次使用您的應用程式名稱取代 `<app_name>`，以及使用部署認證的使用者名稱取代 `<deployment_user>`。

```bash
curl -u <deployment_user> https://<app_name>.scm.azurewebsites.net/api/deployments
```

### <a name="with-powershell"></a>透過 PowerShell

下列範例使用 [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) 來傳送包含 .zip 檔案的要求。 取代預留位置 `<deployment_user>`、`<deployment_password>`、`<zip_file_path>` 和 `<app_name>`。

```PowerShell
#PowerShell
$username = "<deployment_user>"
$password = "<deployment_password>"
$filePath = "<zip_file_path>"
$apiUrl = "https://<app_name>.scm.azurewebsites.net/api/zipdeploy"
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username, $password)))
$userAgent = "powershell/1.0"
Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -UserAgent $userAgent -Method POST -InFile $filePath -ContentType "multipart/form-data"
```

此要求會觸發從上傳的 .zip 檔案推送部署。 若要檢閱目前和過去的部署，請執行下列命令。 同樣地，取代 `<app_name>` 預留位置。

```bash
$apiUrl = "https://<app_name>.scm.azurewebsites.net/api/deployments"
Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -UserAgent $userAgent -Method GET
```
