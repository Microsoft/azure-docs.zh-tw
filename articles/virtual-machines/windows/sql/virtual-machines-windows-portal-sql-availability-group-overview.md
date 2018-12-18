---
title: SQL Server 可用性群組 - Azure 虛擬機器 - 概觀 | Microsoft Docs
description: 本文介紹 Azure 虛擬機器上的「SQL Server 可用性群組」。
services: virtual-machines
documentationCenter: na
authors: MikeRayMSFT
manager: craigg
editor: monicar
tags: azure-service-management
ms.assetid: 601eebb1-fc2c-4f5b-9c05-0e6ffd0e5334
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/13/2017
ms.author: mikeray
ms.openlocfilehash: b71ba12f6c533d67b04366f05b9334e1993823fa
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43382023"
---
# <a name="introducing-sql-server-always-on-availability-groups-on-azure-virtual-machines"></a>Azure 虛擬機器上的 SQL Server Always On 可用性群組簡介 #

本文介紹 Azure 虛擬機器上的 SQL Server 可用性群組。 

「Azure 虛擬機器」上的 Always On 可用性群組與內部部署的 Always On 可用性群組類似。 如需詳細資訊，請參閱 [Always On 可用性群組 (SQL Server)](http://msdn.microsoft.com/library/hh510230.aspx)。 

下圖顯示「Azure 虛擬機器」中完整「SQL Server 可用性群組」的各個部分。

![可用性群組](./media/virtual-machines-windows-portal-sql-availability-group-tutorial/00-EndstateSampleNoELB.png)

「Azure 虛擬機器」中「可用性群組」的主要差異在於 Azure 虛擬機器需要[負載平衡器](../../../load-balancer/load-balancer-overview.md)。 負載平衡器會保有可用性群組接聽程式的 IP 位址。 如果您有多個可用性群組，則每個群組都需要一個接聽程式。 一個負載平衡器可以支援多個接聽程式。

當您已準備好在「Azure 虛擬機器」上建置 SQL Server 可用性群組時，請參考這些教學課程。

## <a name="automatically-create-an-availability-group-from-a-template"></a>從範本自動建立可用性群組

[在 Azure VM 中自動設定 Always On 可用性群組 - Resource Manager](virtual-machines-windows-portal-sql-alwayson-availability-groups.md)

## <a name="manually-create-an-availability-group-in-azure-portal"></a>在 Azure 入口網站中手動建立可用性群組

您也可以自行建立虛擬機器，而不使用範本。 首先，請完成必要條件，然後再建立可用性群組。 請參閱下列主題： 

- [設定 Azure 虛擬機器上 SQL Server Always On 可用性群組的必要條件](virtual-machines-windows-portal-sql-availability-group-prereq.md)

- [建立 Always On 可用性群組以改善可用性和災害復原](virtual-machines-windows-portal-sql-availability-group-tutorial.md)

## <a name="next-steps"></a>後續步驟

[在不同區域中的 Azure 虛擬機器上設定 SQL Server Always On 可用性群組](virtual-machines-windows-portal-sql-availability-group-dr.md)。
