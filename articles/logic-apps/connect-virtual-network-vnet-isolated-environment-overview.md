---
title: 從 Azure Logic Apps 存取 Azure 虛擬網路
description: 此概觀會展示隔離式 Logic Apps 從使用私人及專用資源整合服務環境 (ISE) 連線到 Azure 虛擬網路的方式
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 09/24/2018
ms.openlocfilehash: 9546b8ca33ef7da2d570b547446858e2a4099234
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393128"
---
# <a name="access-to-azure-virtual-network-resources-from-isolated-azure-logic-apps"></a>從隔離式 Azure Logic Apps 存取 Azure 虛擬網路資源

> [!NOTE]
> 這項功能目前處於「個人預覽版」階段。 若要要求存取，請在[這裡建立您的加入要求](https://aka.ms/iseprivatepreview)。

有時候，您的 Logic Apps 和整合帳戶會需要存取安全資源 (例如虛擬機器 (VM)) 及 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md) 中的其他系統或服務。 若要提供此存取，您可以[建立*整合服務環境* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)，作為建立您 Logic Apps 和整合帳戶的位置。 

![選取整合服務環境](./media/connect-virtual-network-vnet-isolated-environment-overview/select-logic-app-integration-service-environment.png)

建立 ISE 會將私人及隔離式 Logic Apps 執行個體部署至您的 Azure 虛擬網路。 私人執行個體使用專用資源 (例如儲存體)，並會與公用「全域」Logic Apps 服務分別執行。 這種區隔也有助於減少其它 Azure 租用戶對您應用程式效能可能造成的影響，或是 ["noisy neighbors" effect](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors) (相鄰干擾效應)。 

此概觀會描述建立 ISE 協助 Logic Apps 和整合帳戶直接存取您 Azure 虛擬網路中資源的方式，並會比較 ISE 和全域 Logic Apps 服務的差異。

<a name="difference"></a>

## <a name="isolated-versus-global"></a>隔離式與全域

當您在 Azure 中建立整合服務環境 (ISE) 時，您可以選取 Azure 虛擬網路作為您環境的「對等」。 Azure 會將 Logic Apps 服務的私人執行個體部署至您的虛擬網路，形成一個隔離式環境。您可以於該環境內建立及在專用資源上執行您的 Logic Apps。 當您建立邏輯應用程式時，您可以選取此環境作為您的應用程式位置，讓邏輯應用程式可直接存取您虛擬網路中的資源。  

ISE 中的 Logic Apps 提供與全域 Logic Apps 服務相同的使用者體驗及相似功能。 您不僅可以使用全域 Logic Apps 服務提供的相同內建功能及連接器，也能從提供 ISE 版本的連接器中選擇。 例如，以下為一些標準連接器，提供在 ISE 中執行的版本：
 
* Azure Blob 儲存體、檔案儲存體及表格儲存體
* Azure 佇列
* Azure 服務匯流排
* FTP
* SSH FTP (SFTP)
* SQL Server
* AS2、X12 及 EDIFACT

ISE 與非 ISE 連接器的差異是觸發與動作執行的位置：

* 若您使用內建觸發及動作 (例如您 ISE 中的 HTTP)，這些觸發和動作一律會在與您 Logic App 相同的 ISE 內執行。 

* 針對提供兩個版本的連接器：其中一個版本會在 ISE 中執行，另一個版本則會在全域 Logic Apps 服務中執行。  

  具備 **ISE** 標籤的連接器一律會在與您 Logic App 相同的 ISE 中執行。 不具備 **ISE** 標籤的連接器則會在全域 Logic Apps 服務中執行。 

  ![選取 ISE 連接器](./media/connect-virtual-network-vnet-isolated-environment-overview/select-ise-connectors.png)

* 您在 ISE 中設定的連接器也可以在全域 Logic Apps 服務中使用。 

> [!IMPORTANT]
> Logic Apps、內建動作，以及在您 ISE 中執行的連接器會使用不同定價方案，而非使用量式定價方案。 如需詳細資訊，請參閱 [Logic Apps 定價](../logic-apps/logic-apps-pricing.md)。

<a name="vnet-access"></a>

## <a name="permissions-for-virtual-network-access"></a>虛擬網路存取權限

當您建立整合服務環境 (ISE) 時，您可以選取 Azure 虛擬網路作為您環境的「對等」。 但是，您在建立您的 ISE 時「只」能建立此關聯，或進行「對等互連」。 此關聯可讓 ISE 存取虛擬網路中的資源，進而讓該 ISE內的 Logic Apps 直接連線到您虛擬網路中的資源。 針對在與 ISE 連結虛擬網路中的內部部署系統，Logic Apps 可透過使用以下任何項目，直接存取這些系統： 

* 該系統的 ISE 連接器，例如 SQL Server

* HTTP 動作 

* 自訂連接器

針對不在虛擬網路內或不具備 ISE 連接器的內部部署系統，您仍然可以在[設定及使用內部部署的資料閘道](../logic-apps/logic-apps-gateway-install.md)之後進行連線。

在選取 Azure 虛擬網路作為您環境的對等之前，您必須先在虛擬網路中為 Azure Logic Apps 服務設定角色型存取控制 (RBAC) 權限。 您需要將**網路參與者**和**傳統參與者**角色指派給 Azure Logic Apps 服務，才能完成此工作。 如需對等互連所需角色權限的詳細資訊，請參閱[建立、變更或刪除虛擬網路對等互連中的權限一節](../virtual-network/virtual-network-manage-peering.md#permissions)。

<a name="create-integration-account-environment"></a>

## <a name="integration-accounts-with-ise"></a>使用 ISE 的整合帳戶

您可以將整合帳戶與在整合服務環境 (ISE) 中執行的 Logic Apps 搭配使用，但這些整合帳戶也必須使用與連結 Logic Apps *相同的 ISE*。 ISE 中的 Logic Apps 只能參考位於相同 ISE 中的整合帳戶。 當您建立整合帳戶時，您可以選取您的 ISE，作為您整合帳戶的位置。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 <a href="https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps" target="_blank">Azure Logic Apps 論壇</a>。
* 若要提交或票選功能構想，請造訪 <a href="http://aka.ms/logicapps-wish" target="_blank">Logic Apps 使用者意見反應網站</a>。

## <a name="next-steps"></a>後續步驟

* 了解如何[從隔離式 Logic Apps 連線到 Azure 虛擬網路](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)
* 深入了解 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md)
* 了解 [Azure 服務的虛擬網路整合](../virtual-network/virtual-network-for-azure-services.md)
