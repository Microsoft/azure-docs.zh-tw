---
title: 包含檔案
description: 包含檔案
services: batch
author: dlepow
ms.service: batch
ms.topic: include
ms.date: 05/04/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 62bb91a2e51c39caf31719f72d68a6edab9205bc
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38944465"
---
> [!NOTE]
> 建立 Batch 帳戶時，您可以選擇兩種「集區配置」模式：[使用者訂用帳戶] 和 [Batch 服務]。 在大部分情況下，您應該使用預設 Batch 服務模式，以在幕後將集區配置在 Azure 管理的訂用帳戶中。 在其他使用者訂用帳戶模式中，在建立集區時，會直接在您的訂用帳戶中建立 Batch VM 和其他資源。 如果您想要使用 [Azure 保留的 VM 執行個體](https://azure.microsoft.com/pricing/reserved-vm-instances/)建立 Batch 集區，則需要使用者訂用帳戶模式。 若要在使用者訂用帳戶模式中建立 Batch 帳戶，您也必須向 Azure Batch 註冊訂用帳戶，並與 Azure Key Vault 中的帳戶建立關聯。