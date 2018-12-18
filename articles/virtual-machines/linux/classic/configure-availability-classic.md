---
title: 傳統 Linux VM 的可用性設定組 | Microsoft Docs
description: 使用 Azure 入口網站和 Azure PowerShell，以傳統部署模型設定新的或現有 Linux 虛擬機器的可用性設定組。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: b8624315-beca-4ec7-8441-2e98b166b548
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/12/2016
ms.author: cynthn
ms.openlocfilehash: c5ca51f854e2358dc57286759d5b219e0d954a15
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38308084"
---
# <a name="how-to-configure-an-availability-set-for-linux-virtual-machines-in-the-classic-deployment-model"></a>如何以傳統部署模型設定 Linux 虛擬機器的可用性設定組
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 您也可以在 Resource Manager 部署中 [設定可用性設定組](../../azure-cli-arm-commands.md#azure-availset-commands-to-manage-your-availability-sets) 。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

[!INCLUDE [virtual-machines-common-classic-configure-availability](../../../../includes/virtual-machines-common-classic-configure-availability.md)]