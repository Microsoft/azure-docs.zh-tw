---
title: 範例 Azure 基礎結構逐步解說 | Microsoft Docs
description: 了解適合用來在 Azure 中部署範例基礎結構的關鍵設計和實作指導方針。
documentationcenter: ''
services: virtual-machines-linux
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 281fc2c0-b533-45fa-81a3-728c0049c73d
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 12/15/2017
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d4b8cd07e50697139f68084f47c847ef8728c429
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37932142"
---
# <a name="example-azure-infrastructure-walkthrough-for-linux-vms"></a>適用於 Linux VM 的範例 Azure 基礎結構逐步解說
本文將逐步解說建置範例應用程式基礎結構的方法。 我們會詳述設計簡單線上商店基礎結構的方式，此線上商店能將所有命名慣例、可用性設定組、虛擬網路及負載平衡器的指導方針和決定集合在一起，並實際部署您的虛擬機器 (VM)。

## <a name="example-workload"></a>範例工作負載
Adventure Works Cycles 想要在 Azure 中建置一個線上商店，該商店將包含：

* 兩個在 Web 層級中執行用戶端前端的 nginx 伺服器
* 兩個在應用程式層級中處理資料和訂單的 nginx 伺服器
* 兩個做為在資料庫層級中儲存產品資料及訂單的共用叢集一部分的 MongoDB 伺服器
* 兩個在驗證層級中針對客戶帳戶及供應商的 Active Directory 網域控制站
* 所有伺服器皆位於兩個子網路中：
  * 一個適用於網頁伺服器的前端子網路 
  * 一個適用於應用程式伺服器、MongoDB 叢集及網域控制站的後端子網路

![不同應用程式基礎結構層級的圖表](./media/infrastructure-example/example-tiers.png)

連入的安全網路流量必須在客戶瀏覽線上商店時，於網頁伺服器之間達成負載平衡。 以來自網頁伺服器的 HTTP 要求形式處理訂單流量，必須在應用程式伺服器之間達成負載平衡。 此外，基礎結構必須設計為高可用性。

所產生的設計必須包含下列各項：

* Azure 訂用帳戶和帳戶
* 單一資源群組
* Azure 受控磁碟
* 具有兩個子網路的虛擬網路
* 適用於具備類似角色之 VM 的可用性設定組
* 虛擬機器

以上各項會遵循下列命名慣例：

* Adventure Works Cycles 使用 **[IT workload]-[location]-[Azure resource]** 做為首碼
  * 針對此範例，"**azos**" (Azure 線上商店) 是 IT 工作負載名稱，而 "**use**" (美國東部 2) 是位置
* 虛擬網路會使用 AZOS-USE-VN **[number]**
* 可用性設定組會使用 azos-use-as-**[role]**
* 虛擬機器名稱會使用 azos-use-vm-**[vmname]**

## <a name="azure-subscriptions-and-accounts"></a>Azure 訂用帳戶與帳戶
Adventure Works Cycles 正在使用名稱為 Adventure Works Enterprise Subscription 的企業訂用帳戶，來提供這個 IT 工作負載的計費。

## <a name="storage"></a>儲存體
Adventure Works Cycles 決定他們應該使用 Azure 受控磁碟。 建立 VM 時，會使用這兩個可用儲存體的儲存層：

* **標準儲存體**，適用於 Web 伺服器、應用程式伺服器，以及網域控制站及其資料磁碟。
* **進階儲存體**，適用於 MongoDB 共用叢集伺服器及其資料磁碟。

## <a name="virtual-network-and-subnets"></a>虛擬網路和子網路
由於虛擬網路不需要持續連線到 Adventure Work Cycles 內部部署網路，因此他們決定使用僅限雲端的虛擬網路。

他們透過 Azure 入口網站，使用下列設定來建立僅限雲端的虛擬網路：

* 名稱：AZOS-USE-VN01
* 位置：East US 2
* 虛擬網路位址空間：10.0.0.0/8
* 第一個子網路：
  * 名稱：FrontEnd
  * 位址空間：10.0.1.0/24
* 第二個子網路：
  * 名稱：BackEnd
  * 位址空間：10.0.2.0/24

## <a name="availability-sets"></a>可用性設定組
為了維持這四個線上商店層級的高可用性，Adventure Works Cycles 決定使用四個可用性設定組：

* **azos-use-as-web** ，適用於網頁伺服器
* **azosuse-as-app** ，適用於應用程式伺服器
* **azos-use-as-db** ，適用於 MongoDB 共用叢集中的伺服器
* **azos-use-as-dc** ，適用於網域控制站

## <a name="virtual-machines"></a>虛擬機器
Adventure Works Cycles 決定為其 Azure VM 使用下列名稱：

* **azos-use-vm-web01** ，適用於第一部網頁伺服器
* **azos-use-vm-web02** ，適用於第二部網頁伺服器
* **azos-use-vm-app01** ，適用於第一部應用程式伺服器
* **azos-use-vm-app02** ，適用於第二部應用程式伺服器
* **azos-use-vm-db01** ，適用於叢集中的第一部 MongoDB 伺服器
* **azos-use-vm-db02** ，適用於叢集中的第二部 MongoDB 伺服器
* **azos-use-vm-dc01** ，適用於第一個網域控制站
* **azos-use-vm-dc02** ，適用於第二個網域控制站

以下是產生的組態。

![在 Azure 中部署的最終應用程式基礎結構](./media/infrastructure-example/example-config.png)

這個設定包含下列各項：

* 含有兩個子網路 (前端和後端) 且僅限雲端的虛擬網路
* 使用標準和進階磁碟的 Azure 受控磁碟
* 四個可用性設定組，每個層級的線上商店都有一組
* 適用於四個層級的虛擬機器
* 適用於以 HTTPS 為基礎之 Web 流量 (從網際網路到 Web 伺服器) 的外部負載平衡組
* 適用於未加密之 Web 流量 (從 Web 伺服器到應用程式伺服器) 的內部負載平衡組
* 單一資源群組
