---
title: 包含檔案
description: 包含檔案
services: active-directory
author: daveba
ms.service: active-directory
ms.component: msi
ms.topic: include
ms.date: 05/31/2018
ms.author: daveba
ms.custom: include file
ms.openlocfilehash: 55814f5515649e0fc6d646d3580f281fd9ec37a5
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37860939"
---
| 類別 | 限制 |
| --- | --- |
| 使用者指派的受控身分識別 | <ul><li>在建立使用者指派的受控身分識別時，僅支援使用英數字元 (0-9、a-z、A-Z) 和連字號 (-)。 此外，指派至 VM/VMSS 的名稱應該限制為 24 個字元長度，才能正常運作。</li><li>如果您使用受控身分識別虛擬機器擴充功能，所支援的限制為 32 個使用者指派的受控身分識別。  若未使用受控身分識別虛擬機器擴充功能，則所支援的限制為 512 個使用者指派的身分識別。</li>|

