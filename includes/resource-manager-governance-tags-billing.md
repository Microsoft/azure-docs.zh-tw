---
title: 包含檔案
description: 包含檔案
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/19/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 7843410043b726526380b2a916d96f414a2decda
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38740542"
---
將標記套用至資源之後，您可以檢視具有這些標記之資源的成本。 成本分析顯示最新的使用方式需要一些時間，因此您可能還看不到成本。 當有成本可顯示時，您可以檢視訂用帳戶中所有資源群組的資源成本。 使用者必須擁有[訂用帳戶層級的計費資訊存取權](../articles/billing/billing-manage-access.md)才能查看成本。

若要在入口網站中依標記檢視成本，請選取您的訂用帳戶，然後選取 [成本分析]。

![成本分析](./media/resource-manager-governance-tags-billing/select-cost-analysis.png)

接著，以標記值篩選，選取 [套用]。

![依標記檢視成本](./media/resource-manager-governance-tags-billing/view-costs-by-tag.png)

您也可以使用 [Azure 計費 API](../articles/billing/billing-usage-rate-card-overview.md) 透過程式設計的方式檢視成本。
