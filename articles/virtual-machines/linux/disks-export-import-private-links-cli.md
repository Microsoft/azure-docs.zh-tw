---
title: Azure CLI - 使用 Private Link 限制受控磁碟的匯入/匯出存取
description: 使用 Azure CLI 為受控磁碟啟用 Private Link。 只允許您在虛擬網路內安全地匯出和匯入磁碟。
author: roygara
ms.service: virtual-machines
ms.topic: overview
ms.date: 08/11/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 68b96401fc4fc6ea8916664978dbd4c094d9e5b4
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98684520"
---
# <a name="azure-cli---restrict-importexport-access-for-managed-disks-with-private-links"></a>Azure CLI - 使用 Private Link 限制受控磁碟的匯入/匯出存取

您可以使用[私人端點](../../private-link/private-endpoint-overview.md)來限制受控磁碟的匯出和匯入，並透過 Azure 虛擬網路上用戶端的 [Private Link](../../private-link/private-link-overview.md) 安全地存取資料。 私人端點會使用來自虛擬網路位址空間的 IP 位址來處理您的受控磁碟服務。 虛擬網路上的用戶端與受控磁碟之間的網路流量僅會流經虛擬網路和 Microsoft 骨幹網路上的私人連結，以排除公開網際網路的風險。

若要使用 Private Link 來匯出/匯入受控磁碟，請先建立一個磁碟存取資源，並藉由建立私人端點，將其連結至相同訂用帳戶中的虛擬網路。 然後，將磁碟或快照集與磁碟存取的執行個體產生關聯。 最後，將磁碟或快照集的 NetworkAccessPolicy 屬性設定為 `AllowPrivate`。 這會限制對您虛擬網路的存取。 

您可以將 NetworkAccessPolicy 屬性設定為 `DenyAll`，以防止任何人匯出磁碟或快照集的資料。 NetworkAccessPolicy 屬性的預設值是 `AllowAll`。

## <a name="limitations"></a>限制

[!INCLUDE [virtual-machines-disks-private-links-limitations](../../../includes/virtual-machines-disks-private-links-limitations.md)]


## <a name="log-in-into-your-subscription-and-set-your-variables"></a>登入訂用帳戶並設定變數

```azurecli-interactive
subscriptionId=yourSubscriptionId
resourceGroupName=yourResourceGroupName
region=northcentralus
diskAccessName=yourDiskAccessForPrivateLinks
vnetName=yourVNETForPrivateLinks
subnetName=yourSubnetForPrivateLinks
privateEndPointName=yourPrivateLinkForSecureMDExportImport
privateEndPointConnectionName=yourPrivateLinkConnection

#The name of an existing disk which is the source of the snapshot
sourceDiskName=yourSourceDiskForSnapshot

#The name of the new snapshot which will be secured via Private Links
snapshotNameSecuredWithPL=yourSnapshotNameSecuredWithPL

az login

az account set --subscription $subscriptionId

```

## <a name="create-a-disk-access-using-azure-cli"></a>使用 Azure CLI 建立磁碟存取
```azurecli
az disk-access create -n $diskAccessName -g $resourceGroupName -l $region

diskAccessId=$(az disk-access show -n $diskAccessName -g $resourceGroupName --query [id] -o tsv)
```

## <a name="create-a-virtual-network"></a>建立虛擬網路

私人端點不支援類似網路安全性群組 (NSG) 的網路原則。 若要在指定的子網路上部署私人端點，該子網路上需要有明確的停用設定。 

```azurecli
az network vnet create --resource-group $resourceGroupName \
    --name $vnetName \
    --subnet-name $subnetName
```
## <a name="disable-subnet-private-endpoint-policies"></a>停用子網路的私人端點原則

Azure 會將資源部署到虛擬網路內的子網路，因此您必須更新子網路，以停用私人端點網路原則。 

```azurecli
az network vnet subnet update --resource-group $resourceGroupName \
    --name $subnetName  \
    --vnet-name $vnetName \
    --disable-private-endpoint-network-policies true
```
## <a name="create-a-private-endpoint-for-the-disk-access-object"></a>建立磁碟存取物件的私人端點

```azurecli
az network private-endpoint create --resource-group $resourceGroupName \
    --name $privateEndPointName \
    --vnet-name $vnetName  \
    --subnet $subnetName \
    --private-connection-resource-id $diskAccessId \
    --group-ids disks \
    --connection-name $privateEndPointConnectionName
```

## <a name="configure-the-private-dns-zone"></a>設定私人 DNS 區域

建立儲存體 Blob 網域的私人 DNS 區域、建立與虛擬網路的關聯連結，以及建立 DNS 區域群組，以將私人端點與私人 DNS 區域建立關聯。 

```azurecli
az network private-dns zone create --resource-group $resourceGroupName \
    --name "privatelink.blob.core.windows.net"

az network private-dns link vnet create --resource-group $resourceGroupName \
    --zone-name "privatelink.blob.core.windows.net" \
    --name yourDNSLink \
    --virtual-network $vnetName \
    --registration-enabled false 

az network private-endpoint dns-zone-group create \
   --resource-group $resourceGroupName \
   --endpoint-name $privateEndPointName \
   --name yourZoneGroup \
   --private-dns-zone "privatelink.blob.core.windows.net" \
   --zone-name disks
```

## <a name="create-a-disk-protected-with-private-links"></a>建立受 Private Link 保護的磁碟
```azurecli-interactive
resourceGroupName=yourResourceGroupName
region=northcentralus
diskAccessName=yourDiskAccessName
diskName=yourDiskName
diskSkuName=Standard_LRS
diskSizeGB=128

diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)

az disk create -n $diskName \
-g $resourceGroupName \
-l $region \
--size-gb $diskSizeGB \
--sku $diskSkuName \
--network-access-policy AllowPrivate \
--disk-access $diskAccessId 
```

## <a name="create-a-snapshot-of-a-disk-protected-with-private-links"></a>建立以 Private Link 保護之磁碟的快照集
```azurecli-interactive
resourceGroupName=yourResourceGroupName
region=northcentralus
diskAccessName=yourDiskAccessName
sourceDiskName=yourSourceDiskForSnapshot
snapshotNameSecuredWithPL=yourSnapshotName

diskId=$(az disk show -n $sourceDiskName -g $resourceGroupName --query [id] -o tsv)

diskAccessId=$(az resource show -n $diskAccessName -g $resourceGroupName --namespace Microsoft.Compute --resource-type diskAccesses --query [id] -o tsv)

az snapshot create -n $snapshotNameSecuredWithPL \
-g $resourceGroupName \
-l $region \
--source $diskId \
--network-access-policy AllowPrivate \
--disk-access $diskAccessId 
```

## <a name="next-steps"></a>後續步驟

- [Private Link 常見問題集](../faq-for-disks.md#private-links-for-securely-exporting-and-importing-managed-disks)
- [使用 CLI 將受控快照集匯出/複製到不同區域的儲存體帳戶當做 VHD](/previous-versions/azure/virtual-machines/scripts/virtual-machines-cli-sample-copy-managed-disks-vhd)