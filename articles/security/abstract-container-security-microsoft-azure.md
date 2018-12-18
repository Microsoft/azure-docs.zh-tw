---
title: 摘要 - Microsoft Azure 中的容器安全性
description: 《Microsoft Azure 中的容器安全性》白皮書的摘要。
author: TomShinder
ms.author: TomSh
ms.date: 09/05/2018
ms.topic: article
ms.service: security
ms.openlocfilehash: 2ce3aec3b7a588076584ebdf400f8cafcdeb02fe
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43842849"
---
# <a name="container-security-in-microsoft-azure"></a>Microsoft Azure 中的容器安全性
## <a name="abstract"></a>摘要

容器技術為雲端運算的領域帶來了結構性的改變。 藉由容器，我們得以在作業系統的單一執行個體上執行應用程式的多個執行個體，因而能夠更有效地使用資源。 容器讓組織保有一致性與彈性。 它們成就了持續部署，因為應用程式可以在桌上型電腦上開發、在虛擬機器中測試，然後在雲端中部署於生產環境。 容器提供了靈活性、精簡的作業和延展性，並因為資源最佳化而降低了成本。

容器是較新的技術，因此許多 IT 專業人員認為它在生產環境中缺乏可見性和使用性，而有安全方面的疑慮。 開發小組通常不會思考安全性最佳做法這種事情。 此白皮書可協助安全性作業小組和開發人員選擇正確的方法，來保護 Microsoft Azure 平台上的容器開發與部署。

本文件說明容器、容器部署和管理，以及原生平台服務。 此外也說明在 Azure 平台上使用容器時所引發的執行階段安全性問題。 在數據和範例方面，本文件主要以 Docker 作為容器模型，並以 Kubernetes 作為容器協調器。 大部分的安全性建議也都適用於 Microsoft 合作夥伴針對 Azure 平台推出的其他容器模型。

[下載白皮書](https://azure.microsoft.com/mediahandler/files/resourcefiles/container-security-in-microsoft-azure/Open%20Container%20Security%20in%20Microsoft%20Azure.pdf)