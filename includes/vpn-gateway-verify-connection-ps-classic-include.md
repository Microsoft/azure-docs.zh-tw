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
ms.openlocfilehash: 591da67e6411d0e859076f0a3c3c38afc1ebe1f5
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38765916"
---
您可以使用 'Get-AzureVNetConnection' Cmdlet，驗證連線是否成功。

1. 請使用下列 Cmdlet 範例，並將值設定為與您狀況相符的值。 如果虛擬網路的名稱包含空格，則必須以引號括住。

  ```powershell
  Get-AzureVNetConnection "Group ClassicRG ClassicVNet"
  ```
2. 完成 Cmdlet 之後，請檢視值。 在下列範例中，[連線狀態] 會顯示為 [已連接]，而您可以看見輸入和輸出位元組。

        ConnectivityState         : Connected
        EgressBytesTransferred    : 181664
        IngressBytesTransferred   : 182080
        LastConnectionEstablished : 1/7/2016 12:40:54 AM
        LastEventID               : 24401
        LastEventMessage          : The connectivity state for the local network site 'RMVNetLocal' changed from Connecting to
                                    Connected.
        LastEventTimeStamp        : 1/7/2016 12:40:54 AM
        LocalNetworkSiteName      : RMVNetLocal