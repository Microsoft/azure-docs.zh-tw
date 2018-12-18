---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 1c3103889bd11df45710cc0d6afaeeba022c9ace
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38763000"
---
您可以使用 [az network vpn-connection show](/cli/azure/network/vpn-connection#show) 命令來確認連線是否成功。 在範例中， '--name' 是指您想要測試之連線的名稱。 當系統在建立連線時，其連線狀態會顯示「連線中」。 連線建立好之後，狀態會變更為 [已連線]。

```azurecli
az network vpn-connection show --name VNet1toSite2 --resource-group TestRG1
```