---
title: 深度學習虛擬機器簡介 - Azure | Microsoft Docs
description: 適用於深度學習虛擬機器的重要分析案例與元件。
keywords: 深度學習, AI, 資料科學工具, 資料科學虛擬機器, 資料科學工具, linux 資料科學
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: d4f91270-dbd2-4290-ab2b-b7bfad0b2703
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.author: gokuma
ms.openlocfilehash: 35370d3d143fdc154728aa8b31b9874227e46168
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31406886"
---
# <a name="introduction-to-the-deep-learning-virtual-machine"></a>深度學習虛擬機器簡介

## <a name="why-deep-learning-virtual-machine"></a>為什麼需要深度學習虛擬機器？ 

深度學習演算法 / 深度類神經網路已逐漸成為許多機器學習問題中所採用的熱門方法之一。 它們在處理機器認知工作 (例如影像、文字、音訊/視訊了解) 時特別好用，這類工作通常會在一些具備進階深度類神經網路架構的特定網域中處理人類認知層次，以及存取大型資料集來定型模型。 深度學習需要大量計算能力，以使用這些大型資料集來定型模型。 利用雲端和圖形處理單元 (GPU) 的可用性，就能建置複雜的深度類神經架構，並在雲端上，於功能強大之計算基礎結構上的大型資料集中為它們定型。  [資料科學虛擬機器](overview.md)已針對資料準備、機器學習和深度學習提供一組豐富的工具和範例。 但使用者所面臨的挑戰之一就是探索這些工具和適用於特定案例的範例，例如，輕鬆地進行深度學習，同時也能更輕鬆地佈建以 GPU 為基礎的 VM 執行個體。 這個深度學習虛擬機器 (DLVM) 會解決這些挑戰。 

## <a name="what-is-deep-learning-virtual-machine"></a>什麼是深度學習虛擬機器？ 
深度學習虛擬機器是[資料科學虛擬機器](overview.md) (DSVM) 的特別設定變體，能夠更直接地使用以 GPU 為基礎的 VM 執行個體來定型深度學習模型。 它會在 Windows 2016 和 Ubuntu 資料科學虛擬機器上受到支援。  它共用與 DSVM 相同的核心 VM 映像 (因此全都會有豐富的工具組)，但卻設定來讓深度學習更容易。 我們也會提供端對端範例來進行影像和文字了解，其可廣泛套用至許多現實生活 AI 案例。 深度學習虛擬機器也會藉由在虛擬機器上呈現工具和範例的目錄，嘗試讓您能夠更輕鬆地探索 DSVM 上豐富的工具和範例組合。 在工具方面，深度學習虛擬機器提供數個熱門的深度學習架構，取得並預先處理影像、文字資料的工具。 如需工具的完整清單，您可以參考[資料科學虛擬機器概觀頁面](overview.md#whats-included-in-the-data-science-vm)。 

## <a name="next-steps"></a>後續步驟

使用下列步驟，開始使用深度學習虛擬機器：

* [佈建深度學習虛擬機器](provision-deep-learning-dsvm.md)
* [使用深度學習虛擬機器](use-deep-learning-dsvm.md)
* [工具參考](dsvm-deep-learning-ai-frameworks.md)
* [範例](dsvm-samples-and-walkthroughs.md)
