---
title: 針對 Azure NetApp Files 調整容量集區或磁碟區的大小 | Microsoft Docs
description: 說明如何變更容量集區或磁碟區的大小。
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to-article
ms.date: 04/03/2018
ms.author: b-juche
ms.openlocfilehash: 72da85fb7296d02bc6d5f7fcd3279953c312c920
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39012006"
---
# <a name="resize-a-capacity-pool-or-a-volume"></a>調整容量集區或磁碟區的大小
您可以視需要變更容量集區或磁碟區的大小。 

## <a name="resize-the-capacity-pool"></a>調整容量集區的大小 

您可以用 4 TiB 作為遞增或遞減單位，來變更容量集區大小。 調整容量集區的大小後，將會變更已購買的 Azure NetApp Files 容量。

1. 在 [管理 NetApp 帳戶] 刀鋒視窗中，按一下要調整大小的容量集區。 
2. 以滑鼠右鍵按一下容量集區名稱，或按一下容量集區資料列結尾處的 ["..."] 圖示，以顯示內容功能表。 
3. 使用內容功能表選項來調整大小，或刪除容量集區。

## <a name="resize-a-volume"></a>調整磁碟區的大小

您可以視需要變更磁碟區的大小。 磁碟區的容量耗用量是根據其集區的佈建容量進行計算。

1. 在 [管理 NetApp 帳戶] 刀鋒視窗中，按一下 [磁碟區]。 
2. 以滑鼠右鍵按一下要調整大小的磁碟區名稱，或按一下磁碟區資料列結尾處的 ["..."] 圖示，以顯示內容功能表。
3. 使用內容功能表選項來調整大小，或刪除磁碟區。

