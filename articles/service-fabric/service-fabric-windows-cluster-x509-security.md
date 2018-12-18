---
title: 使用憑證保護 Windows 上的 Azure Service Fabric 叢集 | Microsoft Docs
description: 保護獨立或內部部署叢集 Azure Service Fabric 內的通訊，以及用戶端和叢集之間的通訊。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: fe0ed74c-9af5-44e9-8d62-faf1849af68c
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/15/2017
ms.author: dekapur
ms.openlocfilehash: 8383d6afa4b6029aca3978c53eec1865d1d8c8ea
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44721835"
---
# <a name="secure-a-standalone-cluster-on-windows-by-using-x509-certificates"></a>使用 X.509 憑證保護 Windows 上的獨立叢集
本文說明如何保護獨立 Windows 叢集的不同節點之間的通訊。 此外，也會說明如何藉由使用 X.509 憑證，驗證連線到此叢集的用戶端。 驗證可確保只有已獲授權的使用者可以存取叢集和已部署的應用程式，以及執行管理工作。 憑證安全性應在叢集建立之時先在叢集上啟用。  

如需諸如節點對節點安全性、用戶端對節點安全性、角色型存取控制等有關叢集安全性的詳細資訊，請參閱[叢集安全性案例](service-fabric-cluster-security.md)。

## <a name="which-certificates-do-you-need"></a>您需要哪些憑證？
一開始，請[下載 Windows Server 套件的 Service Fabric](service-fabric-cluster-creation-for-windows-server.md#download-the-service-fabric-for-windows-server-package)，下載位置是叢集中的其中一個節點。 在下載的套件中，您會找到 ClusterConfig.X509.MultiMachine.json 檔案。 開啟檔案，檢閱 properties 區段下的 security 區段：

```JSON
"security": {
    "metadata": "The Credential type X509 indicates this cluster is secured by using X509 certificates. The thumbprint format is d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
    "ClusterCredentialType": "X509",
    "ServerCredentialType": "X509",
    "CertificateInformation": {
        "ClusterCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },        
        "ClusterCertificateCommonNames": {
            "CommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]"
            }
            ],
            "X509StoreName": "My"
        },
        "ClusterCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames" : "Root"
            }
        ],
        "ServerCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },
        "ServerCertificateCommonNames": {
            "CommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]"
            }
            ],
            "X509StoreName": "My"
        },
        "ServerCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames" : "Root"
            }
        ],
        "ClientCertificateThumbprints": [
            {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": false
            },
            {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": true
            }
        ],
        "ClientCertificateCommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]",
                "IsAdmin": true
            }
        ],
        "ClientCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames": "Root"
            }
        ]
        "ReverseProxyCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },
        "ReverseProxyCertificateCommonNames": {
            "CommonNames": [
                {
                "CertificateCommonName": "[CertificateCommonName]"
                }
            ],
            "X509StoreName": "My"
        }
    }
},
```

此區段描述保護獨立 Windows 叢集所需的憑證。 如果您指定叢集憑證，請將 ClusterCredentialType 的值設定為 X509。 如果您指定外部連線的伺服器憑證，請將 ServerCredentialType 設定為 X509。 雖然並非必要，但我們建議您具備這兩個憑證以適當保護叢集。 如果您將這些值設定為 X509，則您也必須指定對應的憑證，否則 Service Fabric 會擲回例外狀況。 在某些情況下，您可能只想要指定 ClientCertificateThumbprints 或 ReverseProxyCertificate。 在這些情況下，您不需要將 ClusterCredentialType 或 ServerCredentialType 設定為 X509。


> [!NOTE]
> [指紋](https://en.wikipedia.org/wiki/Public_key_fingerprint) 是憑證的主要身分識別。 若要找出您所建立之憑證的指紋，請參閱[擷取憑證的指紋](https://msdn.microsoft.com/library/ms734695.aspx)。
> 
> 

下表列出您在設定叢集時所需的憑證：

| **CertificateInformation 設定** | **說明** |
| --- | --- |
| ClusterCertificate |測試環境建議使用。 需有此憑證，才能保護叢集上節點之間的通訊。 您可以使用兩個不同的憑證 (主要和次要) 進行更新。 在 Thumbprint 區段中設定主要憑證的指紋，以及在 ThumbprintSecondary 變數中設定次要憑證的指紋。 |
| ClusterCertificateCommonNames |生產環境建議使用。 需有此憑證，才能保護叢集上節點之間的通訊。 您可以使用一或兩個叢集憑證通用名稱。 CertificateIssuerThumbprint 會對應至此憑證的簽發者指紋。 如果您使用多個具有同一個通用名稱的憑證，則可指定多個簽發者指紋。|
| ClusterCertificateIssuerStores |生產環境建議使用。 此憑證會對應到叢集憑證的簽發者。 您可以在此區段下提供簽發者一般名稱和對應存放區名稱，而不用在 ClusterCertificateCommonNames 底下指定簽發者指紋。  這樣讓變換叢集簽發者憑證變得容易。 如果使用一個以上的叢集憑證，可以指定多個簽發者。 空白的 IssuerCommonName 會讓 X509StoreNames 底下指定之對應存放區中的所有憑證都列入允許清單。|
| ServerCertificate |測試環境建議使用。 用戶端嘗試連線到此叢集時，會向用戶端此憑證顯示此憑證。 為了方便起見，您可以選擇對 ClusterCertificate 和 ServerCertificate 使用相同的憑證。 您可以使用兩個不同的伺服器憑證 (主要和次要) 進行更新。 在 Thumbprint 區段中設定主要憑證的指紋，以及在 ThumbprintSecondary 變數中設定次要憑證的指紋。 |
| ServerCertificateCommonNames |生產環境建議使用。 用戶端嘗試連線到此叢集時，會向用戶端此憑證顯示此憑證。 CertificateIssuerThumbprint 會對應至此憑證的簽發者指紋。 如果您使用多個具有同一個通用名稱的憑證，則可指定多個簽發者指紋。 為了方便起見，您可以選擇對 ClusterCertificateCommonNames 和 ServerCertificateCommonNames 使用相同的憑證。 您可以使用一或兩個伺服器憑證通用名稱。 |
| ServerCertificateIssuerStores |生產環境建議使用。 此憑證會對應到伺服器憑證的簽發者。 您可以在此區段下提供簽發者一般名稱和對應存放區名稱，而不用在 ServerCertificateCommonNames 底下指定簽發者指紋。  這樣讓變換伺服器簽發者憑證變得容易。 如果使用一個以上的伺服器憑證，可以指定多個簽發者。 空白的 IssuerCommonName 會讓 X509StoreNames 底下指定之對應存放區中的所有憑證都列入允許清單。|
| ClientCertificateThumbprints |請在經過驗證的用戶端上安裝這組憑證。 在您要允許存取叢集的電腦上，您可以安裝數個不同的用戶端憑證。 在 CertificateThumbprint 變數中設定每個憑證的指紋。 如果您將 IsAdmin 設為 true，則已安裝此憑證的用戶端可以對叢集執行系統管理員管理活動。 如果 IsAdmin 是 false，有此憑證的用戶端只能執行其使用者存取權限允許的動作，通常是唯讀。 如需角色的詳細資訊，請參閱[角色型存取控制 (RBAC)](service-fabric-cluster-security.md#role-based-access-control-rbac)。 |
| ClientCertificateCommonNames |針對 CertificateCommonName 設定第一個用戶端憑證的一般名稱。 CertificateIssuerThumbprint 是此憑證的簽發者指紋。 若要深入了解通用名稱和簽發者，請參閱[使用憑證](https://msdn.microsoft.com/library/ms731899.aspx)。 |
| ClientCertificateIssuerStores |生產環境建議使用。 此憑證會對應至用戶端憑證 (系統管理員和非系統管理員角色) 的簽發者。 您可以在此區段下提供簽發者一般名稱和對應存放區名稱，而不用在 ClientCertificateCommonNames 底下指定簽發者指紋。  這樣讓變換用戶端簽發者憑證變得容易。 如果使用一個以上的用戶端憑證，可以指定多個簽發者。 空白的 IssuerCommonName 會讓 X509StoreNames 底下指定之對應存放區中的所有憑證都列入允許清單。|
| ReverseProxyCertificate |測試環境建議使用。 如果您想要保護[反向 Proxy](service-fabric-reverseproxy.md)，則可以指定此選擇性憑證。 如果您使用此憑證，請務必在 nodeTypes 中設定 reverseProxyEndpointPort。 |
| ReverseProxyCertificateCommonNames |生產環境建議使用。 如果您想要保護[反向 Proxy](service-fabric-reverseproxy.md)，則可以指定此選擇性憑證。 如果您使用此憑證，請務必在 nodeTypes 中設定 reverseProxyEndpointPort。 |

以下是範例叢集組態，其中已提供叢集、伺服器和用戶端憑證。 若是叢集/伺服器/反向 Proxy 憑證，則指紋和通用名稱不可一起設定為相同的憑證類型。

 ```JSON
 {
    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "10-2017",
    "nodes": [{
        "nodeName": "vm0",
        "metadata": "Replace the localhost below with valid IP address or FQDN",
        "iPAddress": "10.7.0.5",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD0"
    }, {
        "nodeName": "vm1",
        "metadata": "Replace the localhost with valid IP address or FQDN",
        "iPAddress": "10.7.0.4",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r1",
        "upgradeDomain": "UD1"
    }, {
        "nodeName": "vm2",
        "iPAddress": "10.7.0.6",
        "metadata": "Replace the localhost with valid IP address or FQDN",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r2",
        "upgradeDomain": "UD2"
    }],
    "properties": {
        "diagnosticsStore": {
        "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
        "dataDeletionAgeInDays": "7",
        "storeType": "FileShare",
        "IsEncrypted": "false",
        "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
        }
        "security": {
            "metadata": "The Credential type X509 indicates this cluster is secured by using X509 certificates. The thumbprint format is d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
            "ClusterCredentialType": "X509",
            "ServerCredentialType": "X509",
            "CertificateInformation": {
                "ClusterCertificateCommonNames": {
                  "CommonNames": [
                    {
                      "CertificateCommonName": "myClusterCertCommonName"
                    }
                  ],
                  "X509StoreName": "My"
                },
                "ClusterCertificateIssuerStores": [
                    {
                        "IssuerCommonName": "ClusterIssuer1",
                        "X509StoreNames" : "Root"
                    },
                    {
                        "IssuerCommonName": "ClusterIssuer2",
                        "X509StoreNames" : "Root"
                    }
                ],
                "ServerCertificateCommonNames": {
                  "CommonNames": [
                    {
                      "CertificateCommonName": "myServerCertCommonName",
                      "CertificateIssuerThumbprint": "7c fc 91 97 13 16 8d ff a8 ee 71 2b a2 f4 62 62 00 03 49 0d"
                    }
                  ],
                  "X509StoreName": "My"
                },
                "ClientCertificateThumbprints": [{
                    "CertificateThumbprint": "c4 c18 8e aa a8 58 77 98 65 f8 61 4a 0d da 4c 13 c5 a1 37 6e",
                    "IsAdmin": false
                }, {
                    "CertificateThumbprint": "71 de 04 46 7c 9e d0 54 4d 02 10 98 bc d4 4c 71 e1 83 41 4e",
                    "IsAdmin": true
                }]
            }
        },
        "reliabilityLevel": "Bronze",
        "nodeTypes": [{
            "name": "NodeType0",
            "clientConnectionEndpointPort": "19000",
            "clusterConnectionEndpointPort": "19001",
            "leaseDriverEndpointPort": "19002",
            "serviceConnectionEndpointPort": "19003",
            "httpGatewayEndpointPort": "19080",
            "applicationPorts": {
                "startPort": "20001",
                "endPort": "20031"
            },
            "ephemeralPorts": {
                "startPort": "20032",
                "endPort": "20062"
            },
            "isPrimary": true
        }
         ],
        "fabricSettings": [{
            "name": "Setup",
            "parameters": [{
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
            }, {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
            }]
        }]
    }
}
 ```

## <a name="certificate-rollover"></a>憑證變換
當您使用憑證通用名稱而非指紋時，憑證變換並不需要叢集設定升級。 針對簽發者指紋升級，請確定新的指紋清單與舊清單有交集。 您必須先使用新的簽發者指紋進行設定升級，然後在存放區中安裝新的憑證 (叢集/服務憑證和簽發者憑證)。 在安裝新的簽發者憑證之後，請將舊的簽發者憑證保留在憑證存放區中，時間至少要兩個小時。
如果您使用簽發者存放區，則不需要為簽發者憑證變換執行設定升級。 在對應的憑證存放區中安裝具有較晚到期日的簽發者憑證，並且在數小時之後移除舊的簽發者憑證。

## <a name="acquire-the-x509-certificates"></a>取得 X.509 憑證
若要保護叢集內的通訊，您必須先取得叢集節點的 X.509 憑證。 此外，若要將此叢集的連線限制於經過授權的電腦/使用者，您必須取得並安裝這些用戶端電腦的憑證。

對於執行生產環境工作負載的叢集，請使用[憑證授權單位 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 簽署的 X.509 憑證來保護叢集。 如需如何取得這些憑證的詳細資訊，請參閱[如何取得憑證](http://msdn.microsoft.com/library/aa702761.aspx)。

若是用於測試的叢集，您可以選擇使用自我簽署憑證。

## <a name="optional-create-a-self-signed-certificate"></a>選擇性：建立自我簽署憑證
若要建立可以正確保護的自我簽署憑證，其中一個做法是使用 C:\Program Files\Microsoft SDKs\Service Fabric\ClusterSetup\Secure 目錄下 Service Fabric SDK 資料夾中的 CertSetup.ps1 指令碼。 編輯此檔案來變更憑證的預設名稱  (尋找值 CN=ServiceFabricDevClusterCert)。執行此指令碼為 `.\CertSetup.ps1 -Install`。

現在將憑證匯出至 .pfx 檔並設定保護密碼。 首先取得憑證的指紋。 
1. 從 [啟動] 功能表中，執行 [管理電腦憑證]。 

2. 移至 [本機電腦\個人] 資料夾，找到您建立的憑證。 

3. 按兩下憑證以開啟它，選取 [詳細資料] 索引標籤，然後向下捲動至 [指紋] 欄位。 

4. 移除空格，並將指紋值複製到下列 PowerShell 命令。 

5. 將 `String` 的值變更為適當的安全密碼加以保護，然後在 PowerShell 執行下列命令︰

   ```powershell   
   $pswd = ConvertTo-SecureString -String "1234" -Force –AsPlainText
   Get-ChildItem -Path cert:\localMachine\my\<Thumbprint> | Export-PfxCertificate -FilePath C:\mypfx.pfx -Password $pswd
   ```

6. 若要檢視電腦上安裝之憑證的詳細資訊，請執行下列 PowerShell 命令︰

   ```powershell
   $cert = Get-Item Cert:\LocalMachine\My\<Thumbprint>
   Write-Host $cert.ToString($true)
   ```

或者，如果您有 Azure 訂用帳戶，請依照[使用 Azure Resource Manager 建立 Service Fabric 叢集](service-fabric-cluster-creation-via-arm.md)中的步驟。

## <a name="install-the-certificates"></a>安裝憑證
有了憑證之後，您就可以將它們安裝在叢集節點上。 節點上須已安裝最新的 Windows PowerShell 3.x。 針對叢集和伺服器憑證及任何次要憑證，在每個節點上重複這些步驟。

1. 將 .pfx 檔案複製到節點。

2. 以系統管理員身分開啟 PowerShell 視窗並輸入下列命令。 以您用來建立此憑證的密碼取代 $pswd。 以複製到這個節點之 .pfx 的完整路徑取代 $PfxFilePath。
   
    ```powershell
    $pswd = "1234"
    $PfxFilePath ="C:\mypfx.pfx"
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\LocalMachine\My -FilePath $PfxFilePath -Password (ConvertTo-SecureString -String $pswd -AsPlainText -Force)
    ```
3. 現在您必須在此憑證上設定存取控制，讓在「網路服務」帳戶下執行的 Service Fabric 程序可以藉由執行下列指令碼來使用它。 提供憑證的指紋和服務帳戶的**網路服務**。 您可以檢查憑證上的 ACL 是否正確，方法是在 [啟動]  >  [管理電腦憑證] 開啟憑證，並查看 [所有工作]  >  [管理私密金鑰]。
   
    ```powershell
    param
    (
    [Parameter(Position=1, Mandatory=$true)]
    [ValidateNotNullOrEmpty()]
    [string]$pfxThumbPrint,
   
    [Parameter(Position=2, Mandatory=$true)]
    [ValidateNotNullOrEmpty()]
    [string]$serviceAccount
    )
   
    $cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object -FilterScript { $PSItem.ThumbPrint -eq $pfxThumbPrint; }
   
    # Specify the user, the permissions, and the permission type
    $permission = "$($serviceAccount)","FullControl","Allow"
    $accessRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $permission
   
    # Location of the machine-related keys
    $keyPath = Join-Path -Path $env:ProgramData -ChildPath "\Microsoft\Crypto\RSA\MachineKeys"
    $keyName = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName
    $keyFullPath = Join-Path -Path $keyPath -ChildPath $keyName
   
    # Get the current ACL of the private key
    $acl = (Get-Item $keyFullPath).GetAccessControl('Access')
   
    # Add the new ACE to the ACL of the private key
    $acl.SetAccessRule($accessRule)
   
    # Write back the new ACL
    Set-Acl -Path $keyFullPath -AclObject $acl -ErrorAction Stop
   
    # Observe the access rights currently assigned to this certificate
    get-acl $keyFullPath| fl
    ```
4. 為每個伺服器憑證重複前面的步驟。 您也可以使用上述步在您要允許存取叢集的電腦上安裝用戶端憑證。

## <a name="create-the-secure-cluster"></a>建立安全的叢集
設定 ClusterConfig.X509.MultiMachine.json 檔案的 security 區段後，您可以繼續進行[建立叢集](service-fabric-cluster-creation-for-windows-server.md#create-the-cluster)一節，以設定節點和建立獨立叢集。 請記得在建立叢集時使用 ClusterConfig.X509.MultiMachine.json 檔案。 例如，您的命令可能如下所示：

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json
```

順利執行安全的獨立 Windows 叢集，並已設定經過驗證的用戶端以進行連接之後，請依照[使用 PowerShell 來連線到叢集](service-fabric-connect-to-secure-cluster.md#connect-to-a-cluster-using-powershell)一節中的步驟來進行。 例如︰

```powershell
$ConnectArgs = @{  ConnectionEndpoint = '10.7.0.5:19000';  X509Credential = $True;  StoreLocation = 'LocalMachine';  StoreName = "MY";  ServerCertThumbprint = "057b9544a6f2733e0c8d3a60013a58948213f551";  FindType = 'FindByThumbprint';  FindValue = "057b9544a6f2733e0c8d3a60013a58948213f551"   }
Connect-ServiceFabricCluster $ConnectArgs
```

然後，您可以執行其他的 PowerShell 命令以使用此叢集。 例如，您可以執行 [Get-ServiceFabricNode](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) 以顯示此安全叢集上的節點清單。


若要移除叢集，連線至您下載 Service Fabric 套件的叢集節點，開啟命令列並移至套件資料夾。 現在執行下列命令：

```powershell
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json
```

> [!NOTE]
> 不正確的憑證設定可能會讓叢集無法在部署期間出現。 若要自我診斷安全性問題，請查看事件檢視器群組 [應用程式及服務記錄] > [Microsoft Service Fabric]。
> 
> 

