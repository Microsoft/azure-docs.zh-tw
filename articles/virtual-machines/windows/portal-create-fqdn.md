---
title: 在 Azure 入口網站中建立 Windows VM 的 FQDN | Microsoft Docs
description: 了解如何在 Azure 入口網站中為基於 Resource Manager 的虛擬機器建立完整格式的網域名稱或 FQDN。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: a2ae5887-76df-485e-ae19-0efd96df8600
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/15/2018
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 885003863b8d5a5a81adc7f0310bbf2238edc68e
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42141491"
---
# <a name="create-a-fully-qualified-domain-name-in-the-azure-portal-for-a-windows-vm"></a>在 Azure 入口網站中建立 Windows VM 的完整網域名稱

在 [Azure 入口網站](https://portal.azure.com) 中建立虛擬機器 (VM) 時，會自動建立虛擬機器的公用 IP 資源。 您可以使用此 IP 位址從遠端存取 VM。 雖然入口網站不會建立 [fully qualified domain name (完整網域名稱)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)(或稱 FQDN)，但是您可以在 VM 建立之後建立一個。 這篇文章示範建立 DNS 名稱或 FQDN 的步驟。

## <a name="create-a-fqdn"></a>建立 FQDN
此文章假設您已經建立一個 VM。 如果需要，您可以[在入口網站中建立一個 VM](quick-create-portal.md) 或[使用 Azure PowerShell](quick-create-powershell.md) 來建立。 在您的 VM 已啟動並執行之後，請遵循下列步驟：

[!INCLUDE [virtual-machines-common-portal-create-fqdn](../../../includes/virtual-machines-common-portal-create-fqdn.md)]

您現在可以使用這個 DNS 名稱 (例如用於「遠端桌面通訊協定」(RDP)) 從遠端連接到 VM。

## <a name="next-steps"></a>後續步驟
既然您的 VM 已經有公用 IP 和 DNS 名稱，您便可以部署通用應用程式架構或服務，例如 IIS、SQL 或 SharePoint。

您也可以進一步閱讀[使用 Resource Manager](../../azure-resource-manager/resource-group-overview.md)，以取得建置 Azure 部署的相關秘訣。

