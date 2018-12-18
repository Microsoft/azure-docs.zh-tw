---
title: 包含檔案
description: 包含檔案
services: virtual-machines-windows, virtual-machines-linux
author: dlepow
ms.service: multiple
ms.topic: include
ms.date: 02/28/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: a3c5290eb0179fe5842c495c2e08f22580d02bda
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38944936"
---
## <a name="deploy-an-image-with-marketplace-terms"></a>利用 Marketplace 條款部署映像

Azure Marketplace 中的特定 VM 映像有其他授權和購買條款，您必須先接受這些條款，才能以程式設計方式部署這些映像。  

若要從這類映像部署 VM，您必須接受映像的條款，並啟用以程式設計方式部署。 您只需要在您的訂用帳戶中執行此動作一次。 然後，您每次從映像以程式設計方式部署 VM 時，也必須指定「購買方案」參數。

下列各節將示範如何：

* 找出 Marketplace 映像是否有其他授權條款 
* 以程式設計方式接受條款
* 當您以程式設計方式部署 VM 時，提供購買方案參數

