---
title: 聲場的相關常見問題集 - 認知服務
description: 本頁面提供 Project Acoustics 相關常見問題的解答，其中包括下載指示和製作流程。
services: cognitive-services
author: kegodin
manager: noelc
ms.service: cognitive-services
ms.component: acoustics
ms.topic: article
ms.date: 08/17/2018
ms.author: kegodin
ms.openlocfilehash: a71e867bd23cf64b2ac7fc8cd1c54c55d92ce924
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47431783"
---
# <a name="frequently-asked-questions"></a>常見問題集

## <a name="what-is-project-acoustics"></a>何謂 Project Acoustics？

Project Acoustics Unity 外掛程式是一個聲場系統，會計算執行階段前的聲波行為，類似於靜態光照。 雲端可執行繁重的波動物理學計算，因此執行階段 CPU 成本很低。  

## <a name="where-can-i-download-the-plugin"></a>我可以從哪裡下載外掛程式？

如果您想要評估聲場外掛程式，請在[這裡](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRwMoAEhDCLJNqtVIPwQN6rpUOFRZREJRR0NIQllDOTQ1U0JMNVc4OFNFSy4u)註冊以加入「設計人員預覽」。

## <a name="is-azure-used-at-runtime"></a>在執行階段是否會使用 Azure？

否，只有在場景設定的計算前階段期間，才會使用雲端整合。
 
## <a name="what-is-simulation-input"></a>什麼是模擬輸入？ 

模擬輸入係指您的 3D 場景、虛擬環境或遊戲關卡。 Project Acoustics 會執行可仔細建立聲音物理特性模型的 3D 體積波動模擬，包括平滑的遮蔽和散射。
 
## <a name="what-is-the-runtime-cost"></a>有哪些執行階段成本？

聲場的每一畫面每一來源大約會耗用 0.01% 的 CPU。 RAM 使用量則取決於場景大小，範圍從 10 到 100 MB。
 
## <a name="do-i-need-to-simplify-the-level-geometry-control-triangle-count-make-meshes-watertight"></a>我是否需要簡化關卡幾何？ 控制三角面計數？ 讓網格防水？

否。 此系統會直接內嵌詳細的關卡幾何。 系統會將其體素化來進行內部處理。
 
## <a name="whats-in-the-runtime-lookup-table"></a>什麼是執行階段查閱資料表？

ACE 檔案是一個眾多來源與聆聽者位置之配對間的聲場參數資料表。
 
## <a name="can-it-handle-moving-sources"></a>它是否可以處理移動中的來源？

是，**Microsoft Acoustics** Unity 空間定位器外掛程式會針對與目前來源和聆聽者位置相關的每個音訊處理刻度，諮詢查閱資料表。 空間定位器的 DSP 會順暢地更新每個刻度上的聲場處理參數。
 
## <a name="can-it-handle-dynamic-geometry-closing-doors-walls-blown-away"></a>它是否可以處理動態幾何？ 關閉中的門？ 被吹走的牆面？

否。 聲場參數是根據遊戲關卡的靜態狀態預先計算的。 建議您讓門幾何維持沒有聲場，然後使用已確立的技術，根據可破壞和可移動之遊戲物件的狀態套用額外的遮蔽。
 
## <a name="does-it-handle-materials"></a>它是否會處理材質？

是。 它會從您關卡中的實體材質名稱挑選材質來決定吸收性。
 
## <a name="what-do-the-probes-represent"></a>「探查」代表什麼？

探查是針對可能之玩家位置進行的取樣。 每個探查代表一個源自探查位置的個別場景波動模擬。 在執行階段，會從附近的探查位置插補聆聽者位置的聲場參數。
 
## <a name="why-spend-so-much-compute-in-the-cloud-what-does-it-buy-me"></a>為什麼要在雲端進行那麼多的計算？ 它能為我提供什麼？

Project Acoustics 會將每個架構層面納入考量，即使針對極複雜的虛擬環境，也能提供既準確又可靠的聲場參數。 它無須任何手動工作，即可提供平滑的遮蔽/障礙，且無須繪製音量，即可提供動態的殘響變化。 所有這些都可在執行階段減輕 CPU 的負擔。

## <a name="what-exactly-happens-during-baking"></a>進行「製作」時，實際會發生什麼事？

系統會考量可能的玩家位置，以產生一組間距一致的「探查」範例位置。 關卡的製作會由每個探查的獨立工作所組成：系統會考量一個以探查為中心的立體「模擬區域」，然後在該區域內以最高可達 25 公分的解析度進行詳細的波動模擬。

## <a name="next-steps"></a>後續步驟
* 探索[範例場景](sample-walkthrough.md)

