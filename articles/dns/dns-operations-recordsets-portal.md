---
title: 使用 Azure DNS 管理 DNS 記錄集和記錄 |Microsoft Docs
description: Azure DNS 可在裝載您的網域時，提供管理 DNS 記錄集和記錄的功能。
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 18ed44a1-7bfe-454f-964e-922ad978264a
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/16/2016
ms.author: victorh
ms.openlocfilehash: e24aa8d0fb72147b0ac30e7c495b598d6e9ea96f
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451387"
---
# <a name="manage-dns-records-and-record-sets-by-using-the-azure-portal"></a>使用 Azure 入口網站管理 DNS 記錄和記錄集

本文說明如何使用 Azure 入口網站管理 DNS 區域的記錄集和記錄。

請務必了解 DNS 記錄集和個別 DNS 記錄之間的差別。 記錄集是指一個區域中有相同名稱和相同類型的記錄集合。 如需詳細資訊，請參閱 [使用 Azure 入口網站建立 DNS 記錄集和記錄](dns-getstarted-create-recordset-portal.md)。

## <a name="create-a-new-record-set-and-record"></a>建立新的記錄集和記錄

若要在 Azure 入口網站中建立記錄集，請參閱 [使用 Azure 入口網站建立 DNS 記錄](dns-getstarted-create-recordset-portal.md)。

## <a name="view-a-record-set"></a>檢視記錄集

1. 在 Azure 入口網站中，移至 [DNS 區域]  刀鋒視窗。
2. 搜尋記錄集並加以選取。 這會開啟記錄集屬性。

    ![搜尋資料錄集](./media/dns-operations-recordsets-portal/searchset500.png)

## <a name="add-a-new-record-to-a-record-set"></a>將新記錄加入記錄集

任何記錄集最多只能加入 20 筆記錄。 記錄集不能包含兩筆相同的記錄。 您可以建立空的記錄集 (沒有記錄)，但該記錄集不會出現在 Azure DNS 名稱伺服器上。 類型為 CNAME 的記錄集最多只能包含一筆記錄。

1. 在 DNS 區域的 [記錄集屬性]  刀鋒視窗中，按一下您想要在其中新增記錄的記錄集。

    ![選取記錄集](./media/dns-operations-recordsets-portal/selectset500.png)

2. 在欄位中填入資料，藉以指定記錄集屬性。

    ![新增記錄](./media/dns-operations-recordsets-portal/addrecord500.png)

3. 按一下刀鋒視窗頂端的 [儲存]  來儲存您的設定。 然後關閉刀鋒視窗。
4. 您會在角落中看到正在儲存記錄。

    ![儲存記錄集](./media/dns-operations-recordsets-portal/saving150.png)

儲存記錄之後，[DNS 區域]  刀鋒視窗上的值將會反映新的記錄。

## <a name="update-a-record"></a>更新記錄

在更新現有記錄集中的記錄時，您可更新的欄位取決於您正在使用的記錄類型。

1. 在記錄集的 [記錄集屬性]  刀鋒視窗中搜尋記錄。
2. 修改記錄。 當您修改記錄時，您可以變更記錄的可用設定。 在下列範例中，已選取 [IP 位址]  欄位，而該 IP 位址正在進行修改。

    ![修改記錄](./media/dns-operations-recordsets-portal/modifyrecord500.png)

3. 按一下刀鋒視窗頂端的 [儲存]  來儲存您的設定。 您將會在右上角看到記錄已儲存的通知。

    ![已儲存記錄集](./media/dns-operations-recordsets-portal/saved150.png)

儲存記錄之後，[DNS 區域]  刀鋒視窗上記錄集的值將會反映更新的記錄。

## <a name="remove-a-record-from-a-record-set"></a>從記錄集移除記錄

您可以使用 Azure 入口網站來從記錄集移除記錄。 請注意，移除記錄集的最後一筆記錄不會刪除記錄集。

1. 在記錄集的 [記錄集屬性]  刀鋒視窗中搜尋記錄。
2. 按一下您想要移除的記錄。 然後選取 [移除] 。

    ![移除記錄](./media/dns-operations-recordsets-portal/removerecord500.png)

3. 按一下刀鋒視窗頂端的 [儲存]  來儲存您的設定。
4. 移除記錄之後，[DNS 區域]  刀鋒視窗上記錄的值將會反映移除。

## <a name="delete"></a>刪除記錄集

1. 在記錄集的 [記錄集屬性] 刀鋒視窗中，按一下 [刪除]。

    ![刪除記錄集](./media/dns-operations-recordsets-portal/deleterecordset500.PNG)

2. 隨即會出現訊息，詢問您是否要刪除記錄集。
3. 確認該名稱符合您想要刪除的記錄集，然後按一下 [是] 。
4. 在 [DNS 區域]  刀鋒視窗中，確認不再看到該記錄集。

## <a name="work-with-ns-and-soa-records"></a>使用 NS 和 SOA 記錄

自動建立之 NS 和 SOA 記錄的管理方式不同於其他記錄類型。

### <a name="modify-soa-records"></a>修改 SOA 記錄

您無法在區域頂點 (名稱 = "\@") 自動建立的 SOA 記錄集中新增或移除記錄。 不過，您可以修改 SOA 記錄 (「主機」除外) 和記錄集 TTL 內的任何參數。

### <a name="modify-ns-records-at-the-zone-apex"></a>在區域頂點修改 NS 記錄

系統會自動使用每個 DNS 區域在區域頂點建立 NS 記錄集。 此記錄集包含指派給區域的 Azure DNS 名稱伺服器的名稱。

您可以將其他名稱伺服器新增至此 NS 記錄集，以支援使用多個 DNS 提供者的共同裝載網域。 您也可以修改此記錄集的 TTL 和中繼資料。 不過，您無法移除或修改預先填入的 Azure DNS 名稱伺服器。

請注意，這只適用於區域頂點的 NS 記錄集。 區域中的其他 NS 記錄集 (如用於委派子區域) 可以修改，沒有任何限制。

### <a name="delete-soa-or-ns-record-sets"></a>刪除 SOA 或 NS 記錄集

您無法在建立區域時所自動建立的區域頂點 (名稱 = "\@") 刪除 SOA 和 NS 記錄集。 當您刪除該區域時，就會自動刪除它們。

## <a name="next-steps"></a>後續步驟

* 如需 Azure DNS 的詳細資訊，請參閱 [Azure DNS 概觀](dns-overview.md)。
* 如需自動化 DNS 的相關資訊，請參閱 [使用 .NET SDK 建立 DNS 區域和記錄集](dns-sdk.md)。
* 如需反向 DNS 記錄的詳細資訊，請參閱 [Azure 反向 DNS 和支援概觀](dns-reverse-dns-overview.md)。
