---
title: 將 Windows AWS VM 移至 Azure | Microsoft Docs
description: 使用 Azure PowerShell 將 Amazon Web Services (AWS) EC2 Windows 執行個體移至 Azure 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 06/01/2018
ms.author: cynthn
ms.openlocfilehash: cb5b68e7bd0a1b247327e7147fe38eae19395f50
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34726528"
---
# <a name="move-a-windows-vm-from-amazon-web-services-aws-to-azure-using-powershell"></a>使用 PowerShell 將 Windows VM 從 Amazon Web Services (AWS) 移至 Azure

如果您正在評估 Azure 虛擬機器來裝載您的工作負載，您可以匯出現有的 Amazon Web Services (AWS) EC2 Windows VM 執行個體，然後將虛擬硬碟 (VHD) 上傳至 Azure。 上傳 VHD 後，您可以從 VHD 在 Azure 中建立新的 VM。 

本文涵蓋將單一虛擬機器從 AWS 移至 Azure 的說明。 如果您想要將 VM 大規模從 AWS 移至 Azure，[使用 Azure Site Recovery 將 Amazon Web Services (AWS) 中的虛擬機器移轉至 Azure](../../site-recovery/site-recovery-migrate-aws-to-azure.md)。

## <a name="prepare-the-vm"></a>準備 VM 
 
您可以將一般化或特製化的 VHD 上傳至 Azure。 每個類型都需要您在從 AWS 匯出之前準備 VM。 

- **一般化 VHD** - 一般化 VHD 已使用 Sysprep 移除您所有的個人帳戶資訊。 如果您想要使用 VHD 作為建立新 VM 映像的來源，您應該︰ 
 
    * [準備 Windows VM](prepare-for-upload-vhd-image.md)。  
    * 使用 Sysprep 將虛擬機器一般化。  

 
- **特製化 VHD** - 特製化的 VHD 會從原始的 VM 維護使用者帳戶、應用程式和其他狀態資料。 如果您想要使用 VHD 現狀建立新的 VM，請確定完成下列步驟。  
    * [準備要上傳至 Azure 的 Windows VHD](prepare-for-upload-vhd-image.md). **不要**使用 Sysprep 一般化 VM。 
    * 移除任何 VM 上 (也就是 VMware 工具) 已安裝的來賓虛擬化工具和代理程式。 
    * 確認已透過 DHCP 設定 VM 提取其 IP 位址和 DNS 設定。 這可確保伺服器在啟動時取得 VNet 內的 IP 位址。  


## <a name="export-and-download-the-vhd"></a>匯出和下載 VHD 

將 EC2 執行個體匯出至 Amazon S3 貯體中的 VHD。 請依照 Amazon 文件文章[使用虛擬機器匯入/匯出將執行個體匯出為虛擬機器](http://docs.aws.amazon.com/vm-import/latest/userguide/vmexport.html)中的步驟，並執行 [create-instance-export-task](http://docs.aws.amazon.com/cli/latest/reference/ec2/create-instance-export-task.html) 命令，將 EC2 執行個體匯出到 VHD 檔案。 

已匯出的 VHD 檔案會儲存在您指定的 Amazon S3 貯體中。 匯出 VHD 的基本語法如下，只要以您的資訊取代 <brackets> 中的預留位置文字。

```
aws ec2 create-instance-export-task --instance-id <instanceID> --target-environment Microsoft \
  --export-to-s3-task DiskImageFormat=VHD,ContainerFormat=ova,S3Bucket=<bucket>,S3Prefix=<prefix>
```

匯出 VHD 後，請依照[如何從 S3 貯體下載物件？](http://docs.aws.amazon.com/AmazonS3/latest/user-guide/download-objects.html)中的指示，從 S3 貯體下載 VHD 檔案。 

> [!IMPORTANT]
> AWS 會酌收下載 VHD 的資料傳輸費用。 如需詳細資訊，請參閱 [Amazon S3 價格](https://aws.amazon.com/s3/pricing/)。


## <a name="next-steps"></a>後續步驟

現在您可以將 VHD 上傳至 Azure 並建立新的 VM。 

- 如果您在來源上執行 Sysprep，以在匯出前將其**一般化**，請參閱[上傳一般化的 VHD 並使用它在 Azure 中建立新的 VM](upload-generalized-managed.md)
- 如果您未在匯出之前執行 Sysprep，VHD 會被視為**特製化**，請參閱[將特製化的 VHD 上傳至 Azure 並建立新的 VM](create-vm-specialized.md)

 
