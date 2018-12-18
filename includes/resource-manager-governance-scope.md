---
title: 包含檔案
description: 包含檔案
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/21/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 4f5dbbd1da8a221fb5f8b9641e016829987db538
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38740548"
---
在建立任何項目之前，讓我們先檢閱範圍的概念。 Azure 提供四個管理層級：管理群組、訂用帳戶、資源群組和資源。 [管理群組](../articles/billing/billing-enterprise-mgmt-group-overview.md)是預覽版本。 下圖顯示這些層級的範例。

![影響範圍](./media/resource-manager-governance-scope/scope-levels.png)

您可以在任何範圍層級套用管理設定。 您選取的層級會決定套用設定的範圍。 較低層級會從較高層級繼承設定。 當您將設定套用到訂用帳戶時，訂用帳戶中的所有資源群組和資源都會套用該設定。 當您在資源群組上套用設定時，資源群組及其所有資源都會套用該設定。 不過，另一個資源群組沒有該設定。

通常我們會在較高層級套用重要設定，而在較低層級套用專案特定需求。 例如，您可能想要確定組織的所有資源都部署到特定區域。 若要完成這項需求，將原則套用至訂用帳戶，該訂用帳戶會指定允許的位置。 當貴組織中其他使用者新增新的資源群組和資源時，會自動強制執行允許的位置。 
