---
title: 快速入門 - 使用 PowerShell 在 Azure 中建立私用的 Docker 登錄
description: 快速了解如何在 Azure 中使用 PowerShell 建立私用的 Docker 容器登錄。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: quickstart
ms.date: 05/08/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: fe21ab3e29b7a4cbf204be82246d37a4b91128e4
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48856678"
---
# <a name="quickstart-create-an-azure-container-registry-using-powershell"></a>快速入門：使用 PowerShell 建立 Azure Container Registry

Azure Container Registry 是受控的私用 Docker 容器登錄服務，用於建立、儲存及提供 Docker 容器映像。 在此快速入門中，您會學到如何使用 PowerShell 建立 Azure 容器登錄。 建立登錄之後，您會將容器映像推送到此登錄，再從登錄將容器部署至 Azure 容器執行個體 (ACI)。

## <a name="prerequisites"></a>必要條件

本快速入門需要 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來判斷您安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

您也必須在本機上安裝 Docker。 Docker 提供 [macOS][docker-mac][Windows][docker-windows] 及 [Linux][docker-linux] 系統適用的封裝。

由於 Azure Cloud Shell 未包含所有必要的 Docker 元件 (`dockerd` 精靈)，因此您無法使用本快速入門中的 Cloud Shell。

## <a name="sign-in-to-azure"></a>登入 Azure

使用 [Connect-AzureRmAccount][Connect-AzureRmAccount] 命令登入 Azure 訂用帳戶，並遵循畫面上的指示進行。

```powershell
Connect-AzureRmAccount
```

## <a name="create-resource-group"></a>建立資源群組

在您已經過 Azure 的驗證後，請使用 [New-AzureRmResourceGroup][New-AzureRmResourceGroup] 建立資源群組。 資源群組是一種邏輯容器，您可在其中部署與管理 Azure 資源。

```powershell
New-AzureRmResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-container-registry"></a>建立容器登錄

接著，使用 [New-AzureRMContainerRegistry][New-AzureRMContainerRegistry] 命令在新的資源群組中建立容器登錄。

登錄名稱在 Azure 內必須是唯一的，且包含 5-50 個英數字元。 下列範例會建立名為「myContainerRegistry007」的登錄。 在下列命令中取代 *myContainerRegistry007*，然後執行命令建立登錄：

```powershell
$registry = New-AzureRMContainerRegistry -ResourceGroupName "myResourceGroup" -Name "myContainerRegistry007" -EnableAdminUser -Sku Basic
```

## <a name="log-in-to-registry"></a>登入登錄

推送和提取容器映像之前，您必須先登入登錄。 使用 [Get-AzureRmContainerRegistryCredential][Get-AzureRmContainerRegistryCredential] 命令先取得登錄的管理員登錄：

```powershell
$creds = Get-AzureRmContainerRegistryCredential -Registry $registry
```

接下來，執行 [docker login][docker-login] 登入：

```powershell
$creds.Password | docker login $registry.LoginServer -u $creds.Username --password-stdin
```

成功的登入會傳回 `Login Succeeded`：

```console
PS Azure:\> $creds.Password | docker login $registry.LoginServer -u $creds.Username --password-stdin
Login Succeeded
```

## <a name="push-image-to-registry"></a>將映像推送至登錄

現在您已登入至登錄，您可以將容器映像推送到登錄。 若要取得您可推送到登錄的映像，請從 Docker Hub 提取公用 [aci-helloworld][aci-helloworld-image]。 [aci-helloworld][aci-helloworld-github] 映像是小型 Node.js 應用程式，可提供靜態 HTML 頁面，其中顯示 Azure 容器執行個體標誌。

```powershell
docker pull microsoft/aci-helloworld
```

當提取映像時，輸出如下所示：

```console
PS Azure:\> docker pull microsoft/aci-helloworld
Using default tag: latest
latest: Pulling from microsoft/aci-helloworld
88286f41530e: Pull complete
84f3a4bf8410: Pull complete
d0d9b2214720: Pull complete
3be0618266da: Pull complete
9e232827e52f: Pull complete
b53c152f538f: Pull complete
Digest: sha256:a3b2eb140e6881ca2c4df4d9c97bedda7468a5c17240d7c5d30a32850a2bc573
Status: Downloaded newer image for microsoft/aci-helloworld:latest
```

在您將映像推送至 Azure 容器登錄前，您必須先使用登錄的完整網域名稱 (FQDN) 標記影像。 Azure 容器登錄的 FQDN 格式為 *\<registry-name\>.azurecr.io*。

將完整的影像標記填入變數中。 包含登入伺服器、存放庫名稱 ("aci-helloworld") 及映像版本 ("v1")：

```powershell
$image = $registry.LoginServer + "/aci-helloworld:v1"
```

接著，使用 [docker 標記][docker-tag]將映像標記：

```powershell
docker tag microsoft/aci-helloworld $image
```

最後，[docker 推送][docker-push]影像到您的登錄：

```powershell
docker push $image
```

當 Docker 用戶端推送您的映像時，輸出應如下所示：

```console
PS Azure:\> docker push $image
The push refers to repository [myContainerRegistry007.azurecr.io/aci-helloworld]
31ba1ebd9cf5: Pushed
cd07853fe8be: Pushed
73f25249687f: Pushed
d8fbd47558a8: Pushed
44ab46125c35: Pushed
5bef08742407: Pushed
v1: digest: sha256:565dba8ce20ca1a311c2d9485089d7ddc935dd50140510050345a1b0ea4ffa6e size: 1576
```

恭喜！ 您剛才已將第一個容器映像推送至登錄。

## <a name="deploy-image-to-aci"></a>將映像部署至 ACI

現在映像已在您的登錄中，請將容器直接部署到 Azure 容器執行個體，以了解它在 Azure 中執行的狀況。

首先，將登錄認證轉換為 *PSCredential*。 `New-AzureRmContainerGroup` 命令 (可用於建立容器執行個體) 需它採用以下格式。

```powershell
$secpasswd = ConvertTo-SecureString $creds.Password -AsPlainText -Force
$pscred = New-Object System.Management.Automation.PSCredential($creds.Username, $secpasswd)
```

此外，您容器的 DNS 名稱標籤在您建立容器的 Azure 區域中必須是唯一的。 執行下列命名將產生的名稱填入變數中：

```powershell
$dnsname = "aci-demo-" + (Get-Random -Maximum 9999)
```

最後，執行 [New-AzureRmContainerGroup][New-AzureRmContainerGroup] 從您登錄中的映像部署 1 CPU 核心與 1 GB 記憶體的容器：

```powershell
New-AzureRmContainerGroup -ResourceGroup myResourceGroup -Name "mycontainer" -Image $image -RegistryCredential $pscred -Cpu 1 -MemoryInGB 1 -DnsNameLabel $dnsname
```

您應該會從 Azure 得到初始回應，以及您容器的詳細資料，回應的狀態一開始為「Pending」：

```console
PS Azure:\> New-AzureRmContainerGroup -ResourceGroup myResourceGroup -Name "mycontainer" -Image $image -RegistryCredential $pscred -Cpu 1 -MemoryInGB 1 -DnsNameLabel $dnsname
ResourceGroupName        : myResourceGroup
Id                       : /subscriptions/<subscriptionID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/mycontainer
Name                     : mycontainer
Type                     : Microsoft.ContainerInstance/containerGroups
Location                 : eastus
Tags                     :
ProvisioningState        : Creating
Containers               : {mycontainer}
ImageRegistryCredentials : {myContainerRegistry007}
RestartPolicy            : Always
IpAddress                : 40.117.255.198
DnsNameLabel             : aci-demo-8751
Fqdn                     : aci-demo-8751.eastus.azurecontainer.io
Ports                    : {80}
OsType                   : Linux
Volumes                  :
State                    : Pending
Events                   : {}
```

若要監視其狀態並判斷其何時為執行中，請執行 [Get-AzureRmContainerGroup][Get-AzureRmContainerGroup] 命令數次。 容器啟動所需的時間應不到一分鐘。

```powershell
(Get-AzureRmContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer).ProvisioningState
```

在此，您會看見容器的 ProvisioningState 一開始為 *Creating*，然後啟動並執行時變成 *Succeeded* 狀態：

```console
PS Azure:\> (Get-AzureRmContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer).ProvisioningState
Creating
PS Azure:\> (Get-AzureRmContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer).ProvisioningState
Succeeded
```

## <a name="view-running-application"></a>檢視執行中應用程式

一旦成功部署至 ACI，且您的容器已啟動並在執行中，請在瀏覽器中瀏覽至其完整網域名稱 (FQDN)，以了解應用程式在 Azure 中執行的狀況。

使用 [Get-AzureRmContainerGroup][Get-AzureRmContainerGroup] 取得您容器的 FQDN：


```powershell
(Get-AzureRmContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer).Fqdn
```

命令的輸出是您容器執行個體的 FQDN：

```console
PS Azure:\> (Get-AzureRmContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer).Fqdn
aci-demo-8571.eastus.azurecontainer.io
```

在知道 FQDN 後，在瀏覽器中瀏覽至該 FQDN：

![瀏覽器中的 Hello World 應用程式][qs-psh-01-running-app]

恭喜！ 您已經有一個正在 Azure 中執行應用程式，並直接從您私用 Azure 容器登錄中的容器映像部署的容器。

## <a name="clean-up-resources"></a>清除資源

當您使用在本快速入門中建立的資源完成工作後，請使用 [Remove-AzureRmResourceGroup][Remove-AzureRmResourceGroup] 命令移除資源群組、容器登錄，以及容器執行個體：

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已使用 Azure CLI 建立 Azure Container Registry，並在 Azure 容器執行個體中啟動其執行個體。 請繼續進行 Azure 容器執行個體教學課程，以深入了解 ACI。

> [!div class="nextstepaction"]
> [Azure 容器執行個體教學課程](../container-instances/container-instances-tutorial-prepare-app.md)

<!-- LINKS - external -->
[aci-helloworld-github]: https://github.com/Azure-Samples/aci-helloworld
[aci-helloworld-image]: https://hub.docker.com/r/microsoft/aci-helloworld/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- Links - internal -->
[Connect-AzureRmAccount]: /powershell/module/azurerm.profile/connect-azurermaccount
[Get-AzureRmContainerGroup]: /powershell/module/azurerm.containerinstance/get-azurermcontainergroup
[Get-AzureRmContainerRegistryCredential]: /powershell/module/azurerm.containerregistry/get-azurermcontainerregistrycredential
[Get-Module]: /powershell/module/microsoft.powershell.core/get-module
[New-AzureRmContainerGroup]: /powershell/module/azurerm.containerinstance/new-azurermcontainergroup
[New-AzureRMContainerRegistry]: /powershell/module/azurerm.containerregistry/New-AzureRMContainerRegistry
[New-AzureRmResourceGroup]: /powershell/module/azurerm.resources/new-azurermresourcegroup
[Remove-AzureRmResourceGroup]: /powershell/module/azurerm.resources/remove-azurermresourcegroup

<!-- IMAGES> -->
[qs-psh-01-running-app]: ./media/container-registry-get-started-powershell/qs-psh-01-running-app.png
