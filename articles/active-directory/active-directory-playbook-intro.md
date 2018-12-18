---
title: Azure Active Directory PoC 腳本簡介 | Microsoft Docs
description: 探索並快速實作身分識別和存取管理案例
services: active-directory
keywords: azure active directory, 腳本, 概念證明, PoC
documentationcenter: ''
author: dstefanMSFT
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2017
ms.author: dstefan
ms.openlocfilehash: 80b7ba9507b85621e7b0623362774edc0c8cc729
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
ms.locfileid: "26731428"
---
# <a name="azure-active-directory-proof-of-concept-playbook-introduction"></a>Azure Active Directory 概念證明腳本：簡介

這篇文章提供在概念證明 (PoC) 中探索不同 Azure AD 功能的指導方針。 本文件的目標對象是身分識別架構設計人員、IT 專業人員和系統整合者

## <a name="how-to-use-this-playbook"></a>如何使用這個腳本

1. 使用[佈景主題](active-directory-playbook-ingredients.md#theme)區段，然後根據您的需求選取感興趣的區域。  
2. 藉由選取與您的商務目標一致的案例，來設定 PoC 的範圍。 愈短愈好。 我們建議這麼做是因為越簡潔越能將價值傳達給專案關係人，同時將實現作業的複雜度降至最低。  
3. 使用[實作](active-directory-playbook-implementation.md)區段以了解案例，以及它們對於您的環境有什麼意義。 在每個案例中，我們會說明如何設定 (所謂的[建置組塊](active-directory-playbook-building-blocks.md))，以及如何瀏覽案例。 
4. 每個建置區塊均會說明所需的必要條件，以及完成的大約時間。 這可以在計劃程序期間協助您。 
5. 根據上述的 1-3，定義要在其中執行的[環境](active-directory-playbook-ingredients.md#environment)。 我們鼓勵尋求生產環境，為使用者取得良好的經驗。 
6. 當需求有衝突的時候，請使用這個實用的取捨矩陣 
   * 以佈景主題為主的顯示值  
   * 準備、設定及執行案例的流暢度 
   * 執行目標案例的最少時間 
   * 在您的限制之內儘可能接近生產 

>[!NOTE]
> 在本文中，您會看到一些特定第三方應用程式和產品，以便作為範例。 Azure AD 支援我們的[應用程式庫](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps)中數千個應用程式，您可以根據需求和環境加以使用。 



[!INCLUDE [active-directory-playbook-toc](../../includes/active-directory-playbook-steps.md)]