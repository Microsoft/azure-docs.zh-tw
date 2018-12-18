---
title: 使用封包擷取搭配警示和 Azure Functions 進行主動式網路監視 | Microsoft Docs
description: 本文說明如何使用 Azure 網路監看員建立警示觸發的封包擷取
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 75e6e7c4-b3ba-4173-8815-b00d7d824e11
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 4c96ca70b9b6a82dcccec443ac0b1e06f96a2396
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31597406"
---
# <a name="use-packet-capture-for-proactive-network-monitoring-with-alerts-and-azure-functions"></a>使用封包擷取搭配警示和 Azure Functions 進行主動式網路監視

網路監看員封包擷取可建立擷取工作階段來追蹤虛擬機器的流入和流出流量。 擷取檔案可以有定義來只追蹤您想要監視之流量的篩選。 接著，此資料會儲存在儲存體 blob 或本機客體機器上。

這項功能可以從其他的自動化案例遠端啟動，例如 Azure Functions。 封包擷取可讓您根據定義的網路異常狀況來執行主動式擷取。 其他用途包括收集網路統計資料、取得有關網路入侵的資訊，以及偵錯用戶端與伺服器間的通訊等等。

在 Azure 中部署的資源會全年無休地執行。 您和您的人員無法全年無休地主動監視所有資源的狀態。 例如，如果凌晨 2 點發生問題，會發生什麼事？

藉由使用 Azure 生態系統內的網路監看員、警示及函式，您可以使用資料和工具主動回應以解決網路中的問題。

![案例][scenario]

## <a name="prerequisites"></a>先決條件

* 最新版的 [Azure PowerShell](/powershell/azure/install-azurerm-ps)。
* 網路監看員的現有執行個體。 如果您還沒有，請[建立網路監看員執行個體](network-watcher-create.md)。
* 在含有 [Windows 擴充功能](../virtual-machines/windows/extensions-nwa.md)或 [Linux 虛擬機器擴充功能](../virtual-machines/linux/extensions-nwa.md)的網路監看員所在區域中的現有虛擬機器。

## <a name="scenario"></a>案例

在此範例中，您的 VM 將傳送比平常還要多的 TCP 區段，而且您想要收到警示。 此處使用 TCP 區段做為範例，但您可以使用任何警示條件。

當您收到警示時，會想要收到封包層級資料，以了解通訊為何增加。 接著，您可以採取步驟，讓虛擬機器回復為一般通訊。

此案例假設您有現有的網路監看員執行個體，以及具有有效虛擬機器的資源群組。

下列清單是所進行工作流程的概觀：

1. 會在您的 VM 上觸發警示。
1. 警示會透過 webhook 呼叫您的 Azure 函式。
1. 您的 Azure 函式會處理警示，並啟動網路監看員封包擷取工作階段。
1. 封包擷取會在 VM 上執行並收集流量。
1. 封包擷取檔案會上傳至儲存體帳戶以供您檢閱和診斷。

為自動化此程序，我們在 VM 上建立並連接一個在事件發生時觸發的警示。 我們也會建立一個呼叫網路監看員的函式。

此案例會執行下列動作︰

* 建立啟動封包擷取的 Azure 函式。
* 在虛擬機器上建立警示規則，並設定警示規則以呼叫 Azure 函式。

## <a name="create-an-azure-function"></a>建立 Azure 函式

第一個步驟是建立 Azure 函式來處理警示，以及建立封包擷取。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [建立資源] > [計算] > [函數應用程式]。

    ![建立函數應用程式][1-1]

2. 在 [函數應用程式] 刀鋒視窗上，輸入下列值，然後選取 [確定] 以建立應用程式︰

    |**設定** | **值** | **詳細資料** |
    |---|---|---|
    |**應用程式名稱**|PacketCaptureExample|函數應用程式的名稱。|
    |**訂用帳戶**|[您的訂用帳戶]要建立函數應用程式的訂用帳戶。||
    |**資源群組**|PacketCaptureRG|要包含函數應用程式的資源群組。|
    |**主控方案**|取用方案| 函數應用程式所使用的方案類型。 選項有「取用」或 Azure App Service 方案。 |
    |**位置**|美國中部| 要在其中建立函數應用程式的區域。|
    |**儲存體帳戶**|{自動產生}| Azure Functions 針對一般用途的儲存所需的儲存體帳戶。|

3. 在 [PacketCaptureExample 函數應用程式] 刀鋒視窗上，選取 [函式] > [自訂函式] >[+]。

4. 選取 [HttpTrigger-Powershell]，然後輸入其餘資訊。 最後，若要建立函式，請選取 [建立]。

    |**設定** | **值** | **詳細資料** |
    |---|---|---|
    |**案例**|實驗性|案例類型|
    |**函式命名**|AlertPacketCapturePowerShell|函式的名稱|
    |**授權層級**|函式|函式的授權層級|

![函式範例][functions1]

> [!NOTE]
> PowerShell 範本是實驗性功能，因此並未擁有完整支援。

此範例需要進行自訂，下列步驟會有相關說明。

### <a name="add-modules"></a>新增模組

若要使用網路監看員 PowerShell Cmdlet，請將最新的 PowerShell 模組上傳至函數應用程式。

1. 在已安裝最新 Azure PowerShell 模組的本機電腦上，執行下列 PowerShell 命令︰

    ```powershell
    (Get-Module AzureRM.Network).Path
    ```

    此範例會提供您 Azure PowerShell 模組的本機路徑。 在接下來的步驟中會用到這些資料夾。 此案例中使用的模組有︰

    * AzureRM.Network

    * AzureRM.Profile

    * AzureRM.Resources

    ![PowerShell 資料夾][functions5]

1. 選取 [函數應用程式設定] > [前往 App Service 編輯器]。

    ![函數應用程式設定][functions2]

1. 以滑鼠右鍵按一下 **AlertPacketCapturePowershell** 資料夾，然後建立名為 **azuremodules** 的資料夾。 

4. 為您需要的每個模組建立子資料夾。

    ![資料夾和子資料夾][functions3]

    * AzureRM.Network

    * AzureRM.Profile

    * AzureRM.Resources

1. 以滑鼠右鍵按一下 **AzureRM.Network** 子資料夾，然後選取 [上傳檔案]。 

6. 前往 Azure 模組。 在本機 **AzureRM.Network** 資料夾中，選取資料夾中的所有檔案。 然後選取 [確定]。 

7. 針對 **AzureRM.Profile** 和 **AzureRM.Resources** 重複這些步驟。

    ![上傳檔案][functions6]

1. 完成之後，每個資料夾應該都會有來自本機電腦的 PowerShell 模組檔案。

    ![PowerShell 檔案][functions7]

### <a name="authentication"></a>驗證

若要使用 PowerShell Cmdlet，您必須進行驗證。 您可以在函數應用程式中設定驗證。 若要設定驗證，您必須設定環境變數，並將加密金鑰檔案上傳至函數應用程式。

> [!NOTE]
> 此案例只是提供了其中一種如何對 Azure Functions 實作驗證的範例。 有其他方法可以執行這項作業。

#### <a name="encrypted-credentials"></a>加密的認證

下列 PowerShell 指令碼會建立名為 **PassEncryptKey.key** 的金鑰檔案。 它也會提供所提供密碼的加密版本。 此密碼與為了讓 Azure Active Directory 應用程式能夠進行驗證而定義的密碼相同。

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

在函數應用程式的 App Service 編輯器中，於 **AlertPacketCapturePowerShell** 下方建立名為 **keys** 的資料夾。 然後上傳先前 PowerShell 範例中所建立的 **PassEncryptKey.key** 檔案。

![函式金鑰][functions8]

### <a name="retrieve-values-for-environment-variables"></a>擷取環境變數的值

最後需求是設定存取驗證值所需的環境變數。 下列清單顯示所建立的環境變數：

* AzureClientID

* AzureTenant

* AzureCredPassword


#### <a name="azureclientid"></a>AzureClientID

用戶端識別碼是 Azure Active Directory 中應用程式的應用程式識別碼。

1. 如果您還沒有要使用的應用程式，請執行下列範例來建立應用程式。

    ```powershell
    $app = New-AzureRmADApplication -DisplayName "ExampleAutomationAccount_MF" -HomePage "https://exampleapp.com" -IdentifierUris "https://exampleapp1.com/ExampleFunctionsAccount" -Password "<same password as defined earlier>"
    New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
    Start-Sleep 15
    New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $app.ApplicationId
    ```

   > [!NOTE]
   > 建立應用程式時所使用的密碼，應該與稍早在儲存金鑰檔案時所建立的密碼相同。

1. 在 Azure 入口網站中，選取 [訂用帳戶]。 選取要使用的訂用帳戶，然後選取 [存取控制 (IAM)]。

    ![函式 IAM][functions9]

1. 選擇要使用的帳戶，然後選取 [屬性]。 複製應用程式識別碼。

    ![函數應用程式識別碼][functions10]

#### <a name="azuretenant"></a>AzureTenant

執行下列 PowerShell 範例，以取得租用戶識別碼︰

```powershell
(Get-AzureRmSubscription -SubscriptionName "<subscriptionName>").TenantId
```

#### <a name="azurecredpassword"></a>AzureCredPassword

AzureCredPassword 環境變數的值是執行下列 PowerShell 範例所取得的值。 這個範例與前面的＜加密的認證＞一節所顯示的範例相同。 所需的值是 `$Encryptedpassword` 變數的輸出。  這是使用 PowerShell 指令碼所加密的服務主體密碼。

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

### <a name="store-the-environment-variables"></a>儲存環境變數

1. 返回函數應用程式。 然後選取 [函數應用程式設定] > [設定應用程式設定]。

    ![進行應用程式設定][functions11]

1. 將環境變數及其值新增至應用程式設定，然後選取 [儲存]。

    ![應用程式設定][functions12]

### <a name="add-powershell-to-the-function"></a>將 PowerShell 加入函式

現在是時候從 Azure 函式呼叫網路監看員。 根據需求，此函式的實作可能會不同。 不過，程式碼的一般流程如下︰

1. 處理輸入參數。
2. 查詢現有封包擷取以驗證限制，並且解決名稱衝突。
3. 使用適當的參數建立封包擷取。
4. 定期輪詢封包擷取，直到完成。
5. 通知使用者封包擷取工作階段已完成。

下列範例是可在函式中使用的 PowerShell 程式碼。 需要取代 **subscriptionId**、**resourceGroupName** 和 **storageAccountName** 的值。

```powershell
            #Import Azure PowerShell modules required to make calls to Network Watcher
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\AzureRM.Profile\AzureRM.Profile.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\AzureRM.Network\AzureRM.Network.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\AzureRM.Resources\AzureRM.Resources.psd1" -Global

            #Process alert request body
            $requestBody = Get-Content $req -Raw | ConvertFrom-Json

            #Storage account ID to save captures in
            $storageaccountid = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}"

            #Packet capture vars
            $packetcapturename = "PSAzureFunction"
            $packetCaptureLimit = 10
            $packetCaptureDuration = 10

            #Credentials
            $tenant = $env:AzureTenant
            $pw = $env:AzureCredPassword
            $clientid = $env:AzureClientId
            $keypath = "D:\home\site\wwwroot\AlertPacketCapturePowerShell\keys\PassEncryptKey.key"

            #Authentication
            $secpassword = $pw | ConvertTo-SecureString -Key (Get-Content $keypath)
            $credential = New-Object System.Management.Automation.PSCredential ($clientid, $secpassword)
            Connect-AzureRmAccount -ServicePrincipal -Tenant $tenant -Credential $credential #-WarningAction SilentlyContinue | out-null


            #Get the VM that fired the alert
            if($requestBody.context.resourceType -eq "Microsoft.Compute/virtualMachines")
            {
                Write-Output ("Subscription ID: {0}" -f $requestBody.context.subscriptionId)
                Write-Output ("Resource Group:  {0}" -f $requestBody.context.resourceGroupName)
                Write-Output ("Resource Name:  {0}" -f $requestBody.context.resourceName)
                Write-Output ("Resource Type:  {0}" -f $requestBody.context.resourceType)

                #Get the Network Watcher in the VM's region
                $nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq $requestBody.context.resourceRegion}
                $networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName

                #Get existing packetCaptures
                $packetCaptures = Get-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher

                #Remove existing packet capture created by the function (if it exists)
                $packetCaptures | %{if($_.Name -eq $packetCaptureName)
                { 
                    Remove-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName $packetCaptureName
                }}

                #Initiate packet capture on the VM that fired the alert
                if ((Get-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher).Count -lt $packetCaptureLimit){
                    echo "Initiating Packet Capture"
                    New-AzureRmNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -TargetVirtualMachineId $requestBody.context.resourceId -PacketCaptureName $packetCaptureName -StorageAccountId $storageaccountid -TimeLimitInSeconds $packetCaptureDuration
                    Out-File -Encoding Ascii -FilePath $res -inputObject "Packet Capture created on ${requestBody.context.resourceID}"
                }
            } 
 ``` 
#### <a name="retrieve-the-function-url"></a>擷取函式 URL 
1. 在您建立函式之後，請設定警示以呼叫與函式建立關聯的 URL。 若要取得此值，請從函數應用程式複製函式 URL。

    ![尋找函式 URL][functions13]

2. 複製函數應用程式的函式 URL。

    ![複製函式 URL][2]

如果您在 Webhook POST 要求的承載中需要自訂屬性，請參閱[針對 Azure 計量警示設定 Webhook](../monitoring-and-diagnostics/insights-webhooks-alerts.md)。

## <a name="configure-an-alert-on-a-vm"></a>在 VM 上設定警示

警示可以設定為當特定計量超出指派給它的臨界值時通知使用者。 在此範例中，警示是針對傳送的 TCP 區段，但還可針對許多其他計量觸發警示。 在此範例中，警示會設定為呼叫 webhook 來呼叫函式。

### <a name="create-the-alert-rule"></a>建立警示規則

前往現有的虛擬機器，然後新增警示規則。 如需設定警示相關的詳細文件，請參閱[在 Azure 服務的 Azure 監視器中建立警示 - Azure 入口網站](../monitoring-and-diagnostics/insights-alerts-portal.md)。 在 [警示規則] 刀鋒視窗中輸入下列值，然後選取 [確定]。

  |**設定** | **值** | **詳細資料** |
  |---|---|---|
  |**名稱**|TCP_Segments_Sent_Exceeded|警示規則的名稱。|
  |**說明**|傳送的 TCP 區段超出閾值|警示規則的描述。||
  |**計量**|傳送的 TCP 區段| 用以觸發警示的計量。 |
  |**Condition**|大於| 評估計量所用的條件。|
  |**閾值**|100| 觸發警示的計量值。 此值應該設為您環境的有效值。|
  |**期間**|過去五分鐘| 決定尋找計量閾值的期間。|
  |**Webhook**|[函數應用程式中的 Webhook URL]| 先前步驟中所建立函數應用程式中的 Webhook URL。|

> [!NOTE]
> TCP 區段計量預設不會啟用。 若要深入了解如何啟用其他計量，請造訪[啟用監視和診斷](../monitoring-and-diagnostics/insights-how-to-use-diagnostics.md)。

## <a name="review-the-results"></a>檢閱結果

觸發警示的準則之後，會建立封包擷取。 前往網路監看員，然後選取 [封包擷取]。 在這個頁面上，您可以選取封包擷取檔案連結以下載封包擷取。

![檢視封包擷取][functions14]

如果擷取檔案儲存在本機，則可登入虛擬機器進行擷取。

如需從 Azure 儲存體帳戶下載檔案的指示，請參閱[以 .NET 開始使用 Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。 另一個可用的工具是[儲存體總管](http://storageexplorer.com/)。

下載您的擷取後，您可以使用可讀取 **.cap** 檔案的任何工具來檢視它。 以下是其中兩個工具的連結︰

- [Microsoft Message Analyzer](https://technet.microsoft.com/library/jj649776.aspx)
- [WireShark](https://www.wireshark.org/)

## <a name="next-steps"></a>後續步驟

瀏覽[使用 Wireshark 的封包擷取分析](network-watcher-deep-packet-inspection.md)，了解如何檢視封包擷取。


[1]: ./media/network-watcher-alert-triggered-packet-capture/figure1.png
[1-1]: ./media/network-watcher-alert-triggered-packet-capture/figure1-1.png
[2]: ./media/network-watcher-alert-triggered-packet-capture/figure2.png
[3]: ./media/network-watcher-alert-triggered-packet-capture/figure3.png
[functions1]:./media/network-watcher-alert-triggered-packet-capture/functions1.png
[functions2]:./media/network-watcher-alert-triggered-packet-capture/functions2.png
[functions3]:./media/network-watcher-alert-triggered-packet-capture/functions3.png
[functions4]:./media/network-watcher-alert-triggered-packet-capture/functions4.png
[functions5]:./media/network-watcher-alert-triggered-packet-capture/functions5.png
[functions6]:./media/network-watcher-alert-triggered-packet-capture/functions6.png
[functions7]:./media/network-watcher-alert-triggered-packet-capture/functions7.png
[functions8]:./media/network-watcher-alert-triggered-packet-capture/functions8.png
[functions9]:./media/network-watcher-alert-triggered-packet-capture/functions9.png
[functions10]:./media/network-watcher-alert-triggered-packet-capture/functions10.png
[functions11]:./media/network-watcher-alert-triggered-packet-capture/functions11.png
[functions12]:./media/network-watcher-alert-triggered-packet-capture/functions12.png
[functions13]:./media/network-watcher-alert-triggered-packet-capture/functions13.png
[functions14]:./media/network-watcher-alert-triggered-packet-capture/functions14.png
[scenario]:./media/network-watcher-alert-triggered-packet-capture/scenario.png
