---
title: SAP HANA on Azure (大型執行個體) 的資料分層和擴充節點 | Microsoft Docs
description: SAP HANA on Azure (大型執行個體) 的資料分層和擴充節點。
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/04/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 87c2cb3e373b76685fca09eb0cfeefdc9216df77
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030572"
---
# <a name="use-sap-hana-data-tiering-and-extension-nodes"></a>使用 SAP HANA 資料分層與擴充節點

對於不同 SAP NetWeaver 版本的 SAP BW 和 SAP BW/4HANA，SAP 可支援資料分層模型。 如需關於資料分層模型的詳細資訊，請參閱 SAP 文件：[具有 SAP HANA 擴充節點之 HANA 上的 SAP BW/4HANA 和 SAP BW](https://www.sap.com/documents/2017/05/ac051285-bc7c-0010-82c7-eda71af511fa.html#)。
透過 HANA 大型執行個體，您可以使用 SAP HANA 擴充節點的 option-1 組態，其相關說明請見常見問題集和 SAP 部落格文件。 option-2 設定可以搭配下列 HANA 大型執行個體 SKU 來進行設定：S72m、S192、S192m、S384 和 S384m。 

當您檢視文件時，您可能不會立即看出其優勢。 但只要看看 SAP 大小調整指導方針，您就可以使用 option-1 和 option-2 SAP HANA 擴充節點看出其優勢。 範例如下：

- SAP HANA 大小調整指導方針通常需要兩倍的資料磁碟區數量來作為記憶體。 當您使用熱資料來執行 SAP HANA 執行個體時，最多只會有 50% 的記憶體用於資料。 理論上，剩餘的記憶體會保留下來供 SAP HANA 執行其工作。
- 這表示，在具有 2 TB 記憶體且執行 SAP BW 資料庫的 HANA 大型執行個體 S192 單位中，只有 1 TB 會作為資料磁碟區。
- 如果您使用其他採用 option-1 的 SAP HANA 擴充節點 (同樣是 S192 HANA 大型執行個體 SKU)，您將額外獲得 2 TB 容量的資料磁碟區。 在 option-2 組態中，您會額外獲得 4 TB 的暖資料磁碟區。 相較於熱節點，「暖」擴充節點的記憶體容量全都可以用來儲存資料 (若為 option-1)。 在 option-2 的 SAP HANA 擴充節點組態中，則會有兩倍的記憶體可用於資料磁碟區。
- 您最終會有 3 TB 的容量可供資料使用，且熱資料與暖資料的比率為 1:2 (若為 option-1)。 使用 option-2 的擴充節點組態時，則會有 5 TB 的資料容量和 1:4 的比率。

相較於記憶體的資料磁碟區容量越高，您想要的暖資料越有可能會儲存在磁碟儲存體上。

**後續步驟**
- 請參閱 [SAP HANA on Azure (大型執行個體) 架構](hana-architecture.md)