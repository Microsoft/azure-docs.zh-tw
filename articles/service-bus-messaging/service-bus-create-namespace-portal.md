---
title: 如何在 Azure 入口網站中建立服務匯流排命名空間 | Microsoft Docs
description: 使用 Azure 入口網站建立服務匯流排命名空間。
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: fbb10e62-b133-4851-9d27-40bd844db3ba
ms.service: service-bus-messaging
ms.devlang: tbd
ms.topic: get-started-article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 06/29/2018
ms.author: spelluru
ms.openlocfilehash: 699129e9f75cce76d1682e3e2e2fb83d248ea92e
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696020"
---
# <a name="create-a-service-bus-namespace-using-the-azure-portal"></a>使用 Azure 入口網站建立服務匯流排命名空間

命名空間是所有傳訊元件的範圍容器。 多個佇列和主題可以位於單一命名空間，而且命名空間通常會做為應用程式容器。 有 2 種方式可建立服務匯流排命名空間：

1. Azure 入口網站 (本文)
2. [Resource Manager 範本][create-namespace-using-arm]

## <a name="create-a-namespace-in-the-azure-portal"></a>在 Azure 入口網站中建立命名空間

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

恭喜！ 您現已建立服務匯流排訊息命名空間。

## <a name="next-steps"></a>後續步驟

查看服務匯流排 [GitHub 範例][github-samples]，其中會示範一些更進階的服務匯流排傳訊功能。

[create-namespace-using-arm]: service-bus-resource-manager-overview.md
[github-samples]: https://github.com/Azure/azure-service-bus/tree/master/samples
