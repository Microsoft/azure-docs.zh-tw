---
title: Azure CLI 指令碼範例 - 建立受控應用程式定義 | Microsoft Docs
description: Azure CLI 指令碼範例 - 建立受控應用程式定義
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
ms.service: managed-applications
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/25/2017
ms.author: tomfitz
ms.openlocfilehash: 1cc407e07aee307b00116aad1c44bf8e2b97e33e
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39432468"
---
# <a name="create-a-managed-application-definition-with-azure-cli"></a>使用 Azure CLI 建立受控應用程式定義

此指令碼會將受控應用程式定義發佈到服務類別目錄。 


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli[main](../../../cli_scripts/managed-applications/create-definition/create-definition.sh "Create definition")]


## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立受控應用程式定義。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az managedapp definition create](https://docs.microsoft.com/cli/azure/managedapp/definition#az-managedapp-definition-create) | 建立受控應用程式定義。 提供內含所需檔案的套件。 |


## <a name="next-steps"></a>後續步驟

* 如需受控應用程式的簡介，請參閱 [Azure 受控應用程式概觀](../overview.md)。
* 如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。
