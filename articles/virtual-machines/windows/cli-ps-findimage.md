---
title: 在 Azure 中選取 Linux VM 映像 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 來判斷發行者、供應項目、SKU 和 Marketplace VM 映像的版本。
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 188b8974-fabd-4cd3-b7dc-559cbb86b98a
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/28/2018
ms.author: danlep
ms.openlocfilehash: 269d1392e00d02a79a360e3528fdde174563f2cf
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36295205"
---
# <a name="how-to-find-windows-vm-images-in-the-azure-marketplace-with-azure-powershell"></a>如何使用 Azure PowerShell 在 Azure Marketplace 中尋找 Windows VM 映像

本文描述如何使用 Azure PowerShell 在 Azure Marketplace 中尋找 Windows VM 映像。 當您使用 PowerShell、Resource Manager 範本或其他工具以程式設計方式建立 VM 時，使用此資訊來指定 Marketplace 映像。

也請透過 [Azure Marketplace](https://azuremarketplace.microsoft.com/) 店面、[Azure 入口網站](https://portal.azure.com)或 [Azure CLI](../linux/cli-ps-findimage.md) 瀏覽可用的映像和供應項目。 

確定您已安裝並設定最新的 [Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

[!INCLUDE [virtual-machines-common-image-terms](../../../includes/virtual-machines-common-image-terms.md)]

## <a name="table-of-commonly-used-windows-images"></a>常用 Windows 映像表
| 發行者 | 供應項目 | SKU |
|:--- |:--- |:--- |:--- |
| MicrosoftWindowsServer |WindowsServer |2016-Datacenter |
| MicrosoftWindowsServer |WindowsServer |2016-Datacenter-Server-Core |
| MicrosoftWindowsServer |WindowsServer |2016-Datacenter-with-Containers |
| MicrosoftWindowsServer |WindowsServer |2016-Nano-Server |
| MicrosoftWindowsServer |WindowsServer |2012-R2-Datacenter |
| MicrosoftWindowsServer |WindowsServer |2008-R2-SP1 |
| MicrosoftDynamicsNAV |DynamicsNAV |2017 |
| MicrosoftSharePoint |MicrosoftSharePointServer |2016 |
| MicrosoftSQLServer |SQL2014SP2-WS2012R2 |Enterprise |
| MicrosoftWindowsServerHPCPack |WindowsServerHPCPack |2012R2 |
| MicrosoftWindowsServerEssentials |WindowsServerEssentials |WindowsServerEssentials |

## <a name="navigate-the-images"></a>瀏覽映像

在位置中找到映像的另一個方法是依序執行 [Get-AzureRMVMImagePublisher](/powershell/module/azurerm.compute/get-azurermvmimagepublisher)、[Get-AzureRMVMImageOffer](/powershell/module/azurerm.compute/get-azurermvmimageoffer) 和 [Get-AzureRMVMImageSku](/powershell/module/azurerm.compute/get-azurermvmimagesku) Cmdlet。 您可以使用這些命令來判斷下列的值：

1. 列出映像發行者。
2. 針對指定的發行者，列出其供應項目。
3. 針對指定的供應項目，列出其 SKU。

然後，針對選取的 SKU 執行 [Get-AzureRMVMImage](/powershell/module/azurerm.compute/get-azurermvmimage) 以列出要部署的版本。

首先，使用以下命令列出發行者：

```powershell
$locName="<Azure location, such as West US>"
Get-AzureRMVMImagePublisher -Location $locName | Select PublisherName
```

填入您選擇的發行者名稱，然後執行以下命令：

```powershell
$pubName="<publisher>"
Get-AzureRMVMImageOffer -Location $locName -Publisher $pubName | Select Offer
```

填入您選擇的供應項目名稱，然後執行以下命令：

```powershell
$offerName="<offer>"
Get-AzureRMVMImageSku -Location $locName -Publisher $pubName -Offer $offerName | Select Skus
```

填入您選擇的 SKU 名稱，然後執行以下命令：

```powershell
$skuName="<SKU>"
Get-AzureRMVMImage -Location $locName -Publisher $pubName -Offer $offerName -Sku $skuName | Select Version
```

從 `Get-AzureRMVMImage` 命令的輸出中，您可以選取要部署新虛擬機器的版本映像。

下列命令會顯示完整的範例：

```powershell
$locName="West US"
Get-AzureRMVMImagePublisher -Location $locName | Select PublisherName

```

輸出：

```
PublisherName
-------------
a10networks
aiscaler-cache-control-ddos-and-url-rewriting-
alertlogic
AlertLogic.Extension
Barracuda.Azure.ConnectivityAgent
barracudanetworks
basho
boxless
bssw
Canonical
...
```

針對 MicrosoftWindowsServer 發佈者：

```powershell
$pubName="MicrosoftWindowsServer"
Get-AzureRMVMImageOffer -Location $locName -Publisher $pubName | Select Offer
```

輸出：

```
Offer
-----
Windows-HUB
WindowsServer
WindowsServerSemiAnnual
```

針對 WindowsServer 供應項目：

```powershell
$offerName="WindowsServer"
Get-AzureRMVMImageSku -Location $locName -Publisher $pubName -Offer $offerName | Select Skus
```

輸出：

```
Skus
----
2008-R2-SP1
2008-R2-SP1-smalldisk
2012-Datacenter
2012-Datacenter-smalldisk
2012-R2-Datacenter
2012-R2-Datacenter-smalldisk
2016-Datacenter
2016-Datacenter-Server-Core
2016-Datacenter-Server-Core-smalldisk
2016-Datacenter-smalldisk
2016-Datacenter-with-Containers
2016-Datacenter-with-RDSH
2016-Nano-Server
```

然後，針對 2016-Datacenter SKU：

```powershell
$skuName="2016-Datacenter"
Get-AzureRMVMImage -Location $locName -Publisher $pubName -Offer $offerName -Sku $skuName | Select Version
```

現在您可以將選取的發行者、供應項目、SKU 和版本結合為 URN (以 : 分隔的值)。 當您使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) Cmdlet 建立 VM 時，請傳遞此 URN 與 `--image` 參數。 請記住，您可以使用 "latest" 來取代 URN 中的版本號碼。 此版本一律為最新的映像版本。 您也可以使用 URN 搭配 [Set-AzureRMVMSourceImage](/powershell/module/azurerm.compute/set-azurermvmsourceimage) PowerShell Cmdlet。 

如果您使用 Resource Manager 範本來部署 VM，請在 `imageReference` 屬性中個別設定映像參數。 請參閱[範本參考](/azure/templates/microsoft.compute/virtualmachines)。

[!INCLUDE [virtual-machines-common-marketplace-plan](../../../includes/virtual-machines-common-marketplace-plan.md)]


### <a name="view-plan-properties"></a>檢視方案屬性

若要檢視映像的購買方案資訊，請執行 `Get-AzureRMVMImage` Cmdlet。 如果輸出中的 `PurchasePlan` 屬性不是 `null`，在以程式設計方式部署之前，必須接受映像包含的條款。  

例如，Windows Server 2016 Datacenter 映像沒有其他條款，因為 `PurchasePlan` 資訊為 `null`：

```powershell
$version = "2016.127.20170406"
Get-AzureRMVMImage -Location $locName -Publisher $pubName -Offer $offerName -Skus $skuName -Version $version
```

輸出：

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/
                   Versions/2016.127.20170406
Location         : westus
PublisherName    : MicrosoftWindowsServer
Offer            : WindowsServer
Skus             : 2016-Datacenter
Version          : 2016.127.20170406
FilterExpression :
Name             : 2016.127.20170406
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : null
DataDiskImages   : []

```

針對資料科學虛擬機器 - Windows 2016 映像執行類似的命令會顯示下列 `PurchasePlan` 屬性：`name`、`product` 和 `publisher`。 (某些映像也有 `promotion code` 屬性。)若要部署此映像，請參閱下列各節，以接受條款並啟用以程式設計方式部署。

```powershell
Get-AzureRMVMImage -Location "westus" -Publisher "microsoft-ads" -Offer "windows-data-science-vm" -Skus "windows2016" -Version "0.2.02"
```

輸出：

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/microsoft-ads/ArtifactTypes/VMIma
                   ge/Offers/windows-data-science-vm/Skus/windows2016/Versions/0.2.02
Location         : westus
PublisherName    : microsoft-ads
Offer            : windows-data-science-vm
Skus             : windows2016
Version          : 0.2.02
FilterExpression :
Name             : 0.2.02
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : {
                     "publisher": "microsoft-ads",
                     "name": "windows2016",
                     "product": "windows-data-science-vm"
                   }
DataDiskImages   : []

```

### <a name="accept-the-terms"></a>接受條款

若要檢視授權條款，請使用 [Get-AzureRmMarketplaceterms](/powershell/module/azurerm.marketplaceordering/get-azurermmarketplaceterms) Cmdlet，並傳入購買方案的參數。 輸出會提供 Marketplace 映像的條款連結，並顯示您先前是否已接受條款。 例如︰

```powershell
Get-AzureRmMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"
```

輸出：

```
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DV
                    M%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : 2UMWH6PHSAIM4U22HXPXW25AL2NHUJ7Y7GRV27EBL6SUIDURGMYG6IIDO3P47FFIBBDFHZHSQTR7PNK6VIIRYJRQ3WXSE6BTNUNENXA
Accepted          : False
Signdate          : 2/23/2018 7:43:00 PM
```

使用 [Set-AzureRmMarketplaceterms](/powershell/module/azurerm.marketplaceordering/set-azurermmarketplaceterms) Cmdlet 以接受或拒絕條款。 您只需針對映像的每個訂用帳戶接受一次條款。 例如︰

```powershell

$agreementTerms=Get-AzureRmMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"

Set-AzureRmMarketplaceTerms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016" -Terms $agreementTerms -Accept

```

輸出：

```
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DV
                    M%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : VNMTRJK3MNJ5SROEG2BYDA2YGECU33GXTD3UFPLPC4BAVKAUL3PDYL3KBKBLG4ZCDJZVNSA7KJWTGMDSYDD6KRLV3LV274DLBJSS4GQ
Accepted          : True
Signdate          : 2/23/2018 7:49:31 PM
```

### <a name="deploy-using-purchase-plan-parameters"></a>使用購買方案的參數進行部署
接受映像的條款之後，您可以在訂用帳戶中部署 VM。 如下列程式碼片段所示，使用 [Set-AzureRmVMPlan](/powershell/module/azurerm.compute/set-azurermvmplan) Cmdlet 來設定 VM 物件的 Marketplace 方案資訊。 如需建立 VM 網路設定及完成部署的完整指令碼，請參閱 [PowerShell 指令碼範例](powershell-samples.md)。

```powershell
...
$vmConfig = New-AzureRmVMConfig -VMName "myVM" -VMSize Standard_D1

# Set the Marketplace plan information
$vmConfig = Set-AzureRmVMPlan -VM $vmConfig -Publisher "imagePlanPublisher" -Product "imagePlanProduct" -Name "imagePlanName"

$cred=Get-Credential

$vmConfig = Set-AzureRmVMOperatingSystem -Windows -VM $vmConfig -ComputerName "myVM" -Credential $cred

# Set the Marketplace image
$vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig -PublisherName "imagePublisher" -Offer "imageOffer" -Skus "imageSku" -Version "imageVersion"
...
```
接著，您會將 VM 組態以及網路組態物件傳遞至 `New-AzureRmVM` cmdlet。

## <a name="next-steps"></a>後續步驟
若要使用基本映像資訊透過 `New-AzureRmVM` 快速建立虛擬機器，請參閱[使用 PowerShell 建立 Windows 虛擬機器](quick-create-powershell.md)。

請參閱 PowerShell 指令碼範例來[建立完整設定的虛擬機器](../scripts/virtual-machines-windows-powershell-sample-create-vm.md)。


