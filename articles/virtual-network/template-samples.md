---
title: 虛擬網路的 Azure Resource Manager 範本範例 | Microsoft Docs
description: 了解可供您用來部署 Azure 虛擬網路的不同 Azure Resource Manager 範本。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 04/13/2018
ms.author: jdial
ms.openlocfilehash: af0affc549afd8b63fe0d566fac198de054554c1
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31528982"
---
# <a name="azure-resource-manager-template-samples-for-virtual-network"></a>虛擬網路的 Azure Resource Manager 範本範例

下表包含 Azure Resource Manager 範本範例的連結。 您可以使用 Azure [入口網站](../azure-resource-manager/resource-group-template-deploy-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、Azure [CLI](../azure-resource-manager/resource-group-template-deploy-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 Azure [PowerShell](../azure-resource-manager/resource-group-template-deploy.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 來部署範本。 若要了解如何撰寫您自己的範本，請參閱[建立您的第一個範本](../azure-resource-manager/resource-manager-create-first-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)和[了解 Azure Resource Manager 範本的結構和語法](../azure-resource-manager/resource-group-authoring-templates.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。


| | |
|----|----|
|[建立具有兩個子網路的虛擬網路](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets)| 建立具有兩個子網路的虛擬網路。|
|[透過網路虛擬設備路由傳送流量](https://github.com/Azure/azure-quickstart-templates/tree/master/201-userdefined-routes-appliance)| 建立具有三個子網路的虛擬網路。 將虛擬機器部署到每個子網路。 建立包含路由的路由表，以將流量從某個子網路透過另一個子網路中的虛擬機器導向第三個子網路。 將路由表關聯至其中一個子網路。|
|[建立 Azure 儲存體的虛擬網路服務端點](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vnet-2subnets-service-endpoints-storage-integration)|建立具有兩個子網路、且每個子網路中各有一個虛擬介面的新虛擬網路。 讓服務端點可連線到其中一個子網路的 Azure 儲存體，並保護該子網路的新儲存體帳戶。|
|[將兩個虛擬網路連線](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vnet-to-vnet-peering)| 建立兩個虛擬網路，以及兩者之間的虛擬網路對等互連。|
|[建立具有多個 IP 位址的虛擬機器](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-multiple-ipconfig)| 建立具有多個 IP 位址的 Windows 或 Linux VM。|
