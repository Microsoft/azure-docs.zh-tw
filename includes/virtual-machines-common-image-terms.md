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
ms.openlocfilehash: 47ef014f31c628fed9d7b2b005e67e8138d3e7e9
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38941154"
---
## <a name="terminology"></a>術語

Azure 中的 Marketplace 映像具有下列屬性：

* **發行者** - 建立映像的組織。 範例：Canonical、MicrosoftWindowsServer
* **供應項目** - 發行者所建立的一組相關映像之名稱。 範例：Ubuntu Server、WindowsServer
* **SKU** - 供應項目執行個體，例如發佈的主要版本。 範例：16.04-LTS、2016-Datacenter
* **版本** - 映像 SKU 的版本號碼。 

當您以程式設計方式部署 VM 時，若要識別 Marketplace 映像，請個別提供這些值作為參數，否則某些工具會接受映像 URN。 URN 會結合這些值，並以冒號 (:) 字元分隔：發行者:供應項目:SKU:版本。 在 URN 中，您可以使用 "latest" 來取代版本號碼，這會選取最新的映像版本。 

如果映像發行者提供其他授權和購買條款，您必須接受這些條款，並啟用以程式設計方式部署。 當您以程式設計方式部署 VM 時，也必須提供「購買方案」參數。 請參閱[利用 Marketplace 條款部署映像](#deploy-an-image-with-marketplace-terms)。