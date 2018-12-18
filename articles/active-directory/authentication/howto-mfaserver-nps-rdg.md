---
title: 使用 RADIUS 的 RDG 和 Azure MFA Server | Microsoft Docs
description: 此 Azure Multi-Factor Authentication 頁面協助您部署使用 RADIUS 的遠端桌面 (RD) 閘道器和 Azure Multi-Factor Authentication Server。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 99ed74a9664fbeeb65d90e51d0afcd97dd0b0ca0
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44346486"
---
# <a name="remote-desktop-gateway-and-azure-multi-factor-authentication-server-using-radius"></a>使用 RADIUS 的遠端桌面閘道和 Azure Multi-Factor Authentication Server

遠端桌面 (RD) 閘道通常會使用[本機網路原則服務 (NPS)](https://docs.microsoft.com/windows-server/networking/core-network-guide/core-network-guide#BKMK_optionalfeatures) 來驗證使用者。 本文說明如何將 RADIUS 要求從遠端桌面閘道器 (透過本機 NPS) 傳送至 Multi-Factor Authentication Server。 Azure MFA 和 RD 閘道的組合表示使用者可以從任何地方存取其工作環境，同時執行強式驗證。 

由於 Server 2012 R2 不支援終端機服務的 Windows 驗證，請使用 RD 閘道和 RADIUS 來與 MFA Server 整合。 

在不同的伺服器上安裝 Azure Multi-Factor Authentication Server，由它代理將 RADIUS 要求傳回到遠端桌面閘道器伺服器上的 NPS。 NPS 驗證使用者名稱和密碼之後，它會將回應傳回給 Multi-Factor Authentication Server。 然後，MFA Server 會執行第二因素驗證，然後將結果傳回給閘道。

## <a name="prerequisites"></a>必要條件

- 已加入網域的 Azure MFA Server。 如果尚未進行安裝，請遵循[開始使用 Azure Multi-Factor Authentication Server](howto-mfaserver-deploy.md)中的步驟。
- 已設定的現有 NPS 伺服器。
- 使用網路原則服務進行驗證的遠端桌面閘道。

## <a name="configure-the-remote-desktop-gateway"></a>設定遠端桌面閘道
設定 RD 閘道，以將 RADIUS 驗證傳送到 Azure Multi-Factor Authentication Server。 

1. 在 [RD 閘道管理員] 中，以滑鼠右鍵按一下伺服器名稱，然後選取 [屬性]。
2. 移至 [RD CAP 存放區] 索引標籤並選取 [執行 NPS 的中央伺服器]。 
3. 輸入每部伺服器的名稱或 IP 位址，將一或多部 Multi-Factor Authentication Server 新增為 RADIUS 伺服器。 
4. 為每一部伺服器建立共用密碼。

## <a name="configure-nps"></a>設定 NPS
RD 閘道器使用 NPS 將 RADIUS 要求傳送到 Azure Multi-Factor Authentication。 若要設定 NPS，首先變更逾時設定，以免 RD 閘道在雙步驟驗證完成之前逾時。 然後，您可更新 NPS，以從 MFA Server 接收 RADIUS 驗證。 使用下列程序來設定 NPS：

### <a name="modify-the-timeout-policy"></a>修改逾時原則

1. 在 NPS 中，開啟左欄中的 [RADIUS 用戶端及伺服器] 功能表，然後選取 [遠端 RADIUS 伺服器群組]。 
2. 選取 [TS 閘道伺服器群組]。 
3. 移至 [負載平衡] 索引標籤。 
4. 將 [要求被丟棄前，無回應的秒數] 和 [伺服器被識別為無法使用時，要求之間的間隔秒數] 變更為 30 至 60 秒之間。 (如果您在驗證期間發現伺服器仍然逾時，您可以回到這裡並增加秒數。)
5. 移至 [驗證/帳戶] 索引標籤，檢查指定的 RADIUS 連接埠符合 Multi-Factor Authentication Server 所接聽的連接埠。

### <a name="prepare-nps-to-receive-authentications-from-the-mfa-server"></a>準備 NPS 以從 MFA Server 接收驗證

1. 以滑鼠右鍵按一下左欄中的 [RADIUS 用戶端及伺服器] 之下的 [RADIUS 用戶端]，然後選取 [新增]。
2. 將 Azure Multi-Factor Authentication Server 新增為 RADIUS 用戶端。 選擇 [好記的名稱] 並指定共用密碼。
3. 開啟左欄中的 [原則] 功能表，然後選取 [連線要求原則]。 您應會看見在設定 RD 閘道時所建立的原則，其名稱為 [TS 閘道授權原則]。 此原則會將 RADIUS 要求轉送到 Multi-Factor Authentication Server。
4. 以滑鼠右鍵按一下 [TS 閘道授權原則]，然後選取 [重複原則]。 
5. 開啟新的原則，並移至 [條件] 索引標籤。
6. 新增條件以比對 [好記的用戶端名稱] 與步驟 2 中為 Azure Multi-Factor Authentication Server RADIUS 用戶端設定的 [好記的名稱]。 
7. 移至 [設定] 索引標籤，然後選取 [驗證]。
8. 將 [驗證提供者] 變更為 [驗證這個伺服器上的要求]。 此原則可確保當 NPS 收到來自 Azure MFA Server 的 RADIUS 要求時，就在本機進行驗證，而不會將 RADIUS 要求傳回給 Azure Multi-Factor Authentication Server，導致迴圈狀況。 
9. 若要避免迴圈狀況，請確定新原則的順序高於 [連線要求原則] 窗格中的原始原則。

## <a name="configure-azure-multi-factor-authentication"></a>設定 Azure Multi-Factor Authentication

Azure Multi-Factor Authentication Server 設定為 RD 閘道器和 NPS 之間的 RADIUS Proxy。  它應該安裝在 RD 閘道器伺服器之外另一部加入網域的伺服器上。 使用下列程序來設定 Azure Multi-Factor Authentication Server。

1. 開啟 Azure Multi-Factor Authentication Server，然後選取 [RADIUS 驗證] 圖示。 
2. 選取 [啟用 RADIUS 驗證] 核取方塊。
3. 在 [用戶端] 索引標籤上，確定連接埠符合 NPS 中設定的連接埠，然後選取 [新增]。
4. 新增 RD 閘道器伺服器 IP 位址、應用程式名稱 (選擇性) 和共用密碼。 Azure Multi-Factor Authentication Server 與 RD 閘道上的共用密碼必須相同。
3. 移至 [目標] 索引標籤，然後選取 [RADIUS 伺服器] 選項按鈕。
4. 選取 [新增] 並輸入 NPS 伺服器的 IP 位址、共用密碼和連接埠。 除非使用中央 NPS，否則 RADIUS 用戶端和 RADIUS 目標相同。 共用密碼必須符合 NPS 伺服器的 RADIUS 用戶端區段中設定的共用密碼。

![Radius 驗證](./media/howto-mfaserver-nps-rdg/radius.png)

## <a name="next-steps"></a>後續步驟

- 整合 Azure MFA 與 [IIS Web Apps](howto-mfaserver-iis.md)

- 在 [Azure Multi-Factor Authentication 常見問題集](multi-factor-authentication-faq.md)中取得答案
