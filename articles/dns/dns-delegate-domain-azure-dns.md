---
title: 教學課程 - 在 Azure DNS 中裝載您的網域和子網域
description: 本教學課程說明如何設定 Azure DNS 以裝載 DNS 區域。
services: dns
author: vhorne
manager: jeconnoc
ms.service: dns
ms.topic: tutorial
ms.date: 6/13/2018
ms.author: victorh
ms.openlocfilehash: ea0dc257d691326bc073b4cbff37e847a6990f02
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452289"
---
# <a name="tutorial-host-your-domain-in-azure-dns"></a>教學課程：在 Azure DNS 中裝載您的網域

您可以使用 Azure DNS 來裝載 DNS 網域，以及管理您的 DNS 記錄。 只要將您的網域裝載於 Azure，就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具和計費方式來管理 DNS 記錄。 

假設您項網域名稱註冊機構購買了網域 contoso.net，然後在 Azure DNS 中建立名為 contoso.net 的區域。 由於您是網域的擁有者，註冊機構會提供選項，讓您設定網域的名稱伺服器 (NS) 記錄。 註冊機構會將 NS 記錄儲存在 .net 父系網域中。 然後，當世界各地的網際網路使用者嘗試解析 contoso.net 中的 DNS 記錄時，系統會將他們導向至您在 Azure DNS 區域中的網域。


在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 DNS 區域
> * 擷取名稱伺服器清單
> * 委派網域
> * 確認委派正常運作


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-a-dns-zone"></a>建立 DNS 區域

1. 登入 Azure 入口網站。
1. 在左上方，選取 [建立資源] > [網路] > [DNS 區域]，以開啟 [建立 DNS 區域] 頁面。

   ![DNS 區域](./media/dns-delegate-domain-azure-dns/openzone650.png)

1. 在 [建立 DNS 區域] 頁面中輸入下列的值，然後選取 [建立]：

   | **設定** | **值** | **詳細資料** |
   |---|---|---|
   |**名稱**|[您的網域名稱] |您所購買的網域名稱。 本教學課程以 contoso.net 作為範例。|
   |**訂用帳戶**|[您的訂用帳戶]|選取要在其中建立區域的訂用帳戶。|
   |**資源群組**|**建立新的︰** contosoRG|建立資源群組。 資源群組名稱在您選取的訂用帳戶中必須是唯一的。 |
   |**位置**|美國東部||

> [!NOTE]
> 資源群組的位置不會對 DNS 區域造成任何影響。 DNS 區域一定是「全域」位置，且不會顯示出來。

## <a name="retrieve-name-servers"></a>擷取名稱伺服器

在委派 DNS 區域給 Azure DNS 之前，您必須知道區域的名稱伺服器。 每次建立區域時，Azure DNS 都會配置某個集區中的名稱伺服器。

1. 建立 DNS 區域之後，在 Azure 入口網站的 [我的最愛] 窗格中選取 [所有資源]。 在 [所有資源] 頁面上，選取您的 DNS 區域。 如果您選取的訂用帳戶已有幾個資源，您可以在 [依名稱篩選] 方塊中輸入您的網域名稱，以輕鬆存取應用程式閘道。 

1. 從 DNS 區域頁面中擷取名稱伺服器。 在此範例中，區域 contoso.net 已被指派名稱伺服器 *ns1-01.azure-dns.com*、*ns2-01.azure-dns.net*、*ns3-01.azure-dns.org* 和 *ns4-01.azure-dns.info*：

   ![名稱伺服器清單](./media/dns-delegate-domain-azure-dns/viewzonens500.png)

Azure DNS 會自動在您的區域中為指派的名稱伺服器建立權威 NS 記錄。


## <a name="delegate-the-domain"></a>委派網域

現已建立 DNS 區域且您擁有名稱伺服器，您必須使用 Azure DNS 名稱伺服器來更新父系網域。 每個註冊機構都有自己的 DNS 管理工具，可變更網域的名稱伺服器記錄。 在註冊機構的 DNS 管理頁面中，請編輯 NS 記錄，並將 NS 記錄取代為 Azure DNS 名稱伺服器。

委派網域給 Azure DNS 時，您必須使用 Azure DNS 所提供的名稱伺服器。 不論您的網域名稱為何，都建議將四個名稱伺服器全部用上。 網域委派不需要名稱伺服器，即可使用相同的最上層網域作為您的網域。

> [!NOTE]
> 當您複製每個名稱伺服器位址時，請務必複製位址結尾的句點。 結尾句點表示完整網域名稱結束。 有些註冊機構可能會在 NS 名稱不含結尾句點時附加句點。 但若要符合 DNS RFC，請務必包含結尾句點，因為您不能假設每個註冊機構都會為您附加句點。

Azure DNS 目前不支援使用您區域中名稱伺服器的委派 (有時稱為*虛名名稱伺服器*)。

## <a name="verify-that-the-delegation-is-working"></a>確認委派正常運作

完成委派之後，您可以使用 *nslookup* 之類的工具來查詢您區域的「起始點授權」SOA 記錄，以確認委派正常運作。 SOA 記錄是在區域建立時自動建立的。 在完成委派之後，您可能需要等候 10 分鐘左右，才能夠順利地驗證其運作性。 變更需要一些時間才能傳播至整個 DNS 系統。

您不需要指定 Azure DNS 名稱伺服器。 如果已正確設定委派，正常的 DNS 解析程序會自動尋找名稱伺服器。

在命令提示字元中，輸入如下的 nslookup 命令：

```
nslookup -type=SOA contoso.net
```

以下是上述命令的範例回應︰

```
Server: ns1-04.azure-dns.com
Address: 208.76.47.4

contoso.net
primary name server = ns1-04.azure-dns.com
responsible mail addr = msnhst.microsoft.com
serial = 1
refresh = 900 (15 mins)
retry = 300 (5 mins)
expire = 604800 (7 days)
default TTL = 300 (5 mins)
```

## <a name="clean-up-resources"></a>清除資源

如果您想要進行下一個教學課程，您可以保留 **contosoRG** 資源群組。 否則請刪除 **contosoRG** 資源群組，以刪除在本教學課程中建立的資源。 若要這麼做，請按一下 **contosoRG** 資源群組，然後按一下 [刪除資源群組]。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已為網域建立 DNS 區域，並將其委派給 Azure DNS。 若要了解 Azure DNS 和 Web 應用程式，請繼續進行 Web 應用程式的教學課程。

> [!div class="nextstepaction"]
> [在自訂網域中建立 Web 應用程式的 DNS 記錄](./dns-web-sites-custom-domain.md)
