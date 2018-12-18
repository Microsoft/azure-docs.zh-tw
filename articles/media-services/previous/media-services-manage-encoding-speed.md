---
title: 使用 Azure 媒體服務管理編碼的速度和並行 | Microsoft Docs
description: 本文提供如何使用 Azure 媒體服務管理編碼作業/工作之速度和並行的簡短概觀。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 676313f8-a158-4e3a-a99b-2c29a341ecc9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2017
ms.author: juliako
ms.openlocfilehash: d7e3d6d0c176d0a903c3027ab4feddb332557566
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33782867"
---
#  <a name="manage-speed-and-concurrency-of-your-encoding"></a>管理編碼的速度和並行

本文提供如何管理編碼作業/工作之速度和並行的簡短概觀。

## <a name="overview"></a>概觀

在媒體服務中，**保留單元類型**會決定媒體處理工作的處理速度。 您可以選擇下列的保留單元類型：**S1**、**S2** 或 **S3**。 例如，在執行相同編碼作業的前提下，使用 **S2** 保留單元類型的速度會比 **S1** 類型快。 [調整編碼單位](media-services-scale-media-processing-overview.md)主題所顯示的資料表可協助您在不同的編碼速度之間做出決定。

除了指定保留單元類型之外，您還可以指定使用**保留單元**來佈建帳戶。 佈建的保留單元數目可決定指定帳戶中可同時處理的媒體工作數目。 例如，如果帳戶有五個保留單元，則只要有工作需要處理，就會同時執行五個媒體工作。 剩餘的工作會在佇列中等待，當執行中的工作完成時，就循序地挑選來處理。 如果帳戶未佈建任何保留單元，則會循序地挑選工作。 在此情況下，一件工作完成與下一件工作開始之間的等待時間，視系統中的資源可用性而定。

如需說明如何調整編碼單位的詳細資訊及範例，請參閱[這個](media-services-scale-media-processing-overview.md)主題。

## <a name="next-step"></a>後續步驟

[調整編碼單位](media-services-scale-media-processing-overview.md)

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

