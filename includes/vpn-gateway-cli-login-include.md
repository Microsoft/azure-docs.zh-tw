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
ms.openlocfilehash: e3961878663a26d721478effc01d968e21e34c18
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47020790"
---
使用 [az login](/cli/azure/#login) 命令登入 Azure 訂用帳戶並遵循畫面上的指示。 如需有關登入的詳細資訊，請參閱[開始使用 Azure CLI](/cli/azure/get-started-with-azure-cli)。

```azurecli
az login
```

如果您有多個 Azure 訂用帳戶，請列出帳戶的所有訂用帳戶。

```azurecli
az account list --all
```

指定您要使用的訂用帳戶。

```azurecli
az account set --subscription <replace_with_your_subscription_id>
```
