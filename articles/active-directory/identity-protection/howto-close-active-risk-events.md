---
title: 如何在 Azure Active Directory Identity Protection 中關閉作用中風險事件 | Microsoft Docs
description: 了解您可用於關閉作用中風險事件的選項。
services: active-directory
keywords: azure active directory identity protection, cloud app discovery, 管理應用程式, 安全性, 風險, 風險層級, 弱點, 安全性原則
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: e7434eeb-4e98-4b6b-a895-b5598a6cccf1
ms.service: active-directory
ms.component: identity-protection
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/24/2018
ms.author: markvi
ms.reviewer: raluthra
ms.openlocfilehash: 230a4df2c43e8d5e6ae1a5a7e8188cf207461022
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47057694"
---
# <a name="how-to-close-active-risk-events"></a>如何：關閉作用中風險事件

利用[風險事件](../reports-monitoring/concept-risk-events.md)，Azure Active Directory 會偵測可能遭入侵使用者帳戶的指標。 系統管理員會想要取得所有已關閉的風險事件，所以受影響的使用者不再有風險。

本文概述您可用於關閉作用中風險事件的其他選項。

## <a name="options-to-close-risk-events"></a>用於關閉風險事件的選項 

風險事件的狀態為 [作用中] 或 [已關閉]。 所有作用中風險事件會納入稱為使用者風險層級的值計算。 使用者風險層級是帳戶遭到入侵的可能性指標 (低、中、高)。 

若要關閉作用中風險事件，您有下列選項︰

- 使用者風險原則需要密碼重設

- 手動重設密碼
 
- 解除所有風險事件 

- 手動關閉個別風險事件



## <a name="require-password-reset-with-a-user-risk-policy"></a>使用者風險原則需要密碼重設

藉由設定[使用者風險條件式存取原則](howto-user-risk-policy.md)，如果自動偵測到指定的使用者風險層級，您可以要求變更密碼。 

![重設密碼](./media/howto-close-active-risk-events/13.png)

密碼重設會關閉相關使用者的所有作用中風險事件，並且讓身分識別回到安全狀態。 使用使用者風險原則是關閉作用中風險事件的慣用方法，因為這個方法會自動執行。 受影響的使用者與技術支援中心或系統管理員之間目前不需要互動。

不過，使用使用者風險原則不一定適用。 例如，這適用於：

- 尚未註冊多重要素驗證 (MFA) 的使用者。
- 已刪除具有作用中風險事件的使用者。
- 顯示合法使用者已執行報告風險事件的調查。


## <a name="manual-password-reset"></a>手動重設密碼

如果需要使用使用者風險原則重設密碼不可行，您可以針對已使用手動密碼重設關閉的使用者取得所有風險事件。

![重設密碼](./media/howto-close-active-risk-events/04.png)


相關的對話方塊提供兩個不同的方法，可以將密碼重設為︰

![重設密碼](./media/howto-close-active-risk-events/05.png)


**產生臨時密碼** - 藉由產生暫時密碼，您可以立即讓身分識別回到安全狀態。 此方法需要與受影響的使用者互動，因為他們需要知道暫時密碼。 例如，您可以將新的暫時密碼傳送到使用者的備用電子郵件地址，或傳送給使用者的經理。 因為此密碼是暫時的，所以會提示使用者在下次登入時變更密碼。


**要求使用者重設密碼** - 要求使用者重設密碼能夠自行復原，而不需要連絡技術支援中心或系統管理員。 如同在使用者風險原則的範例中，這個方法僅適用於註冊 MFA 的使用者。 位於尚未註冊 MFA 的使用者，無法使用此選項。


## <a name="dismiss-all-risk-events"></a>解除所有風險事件

如果密碼重設不是適合您的選項，您也可以解除所有風險事件。 

![重設密碼](./media/howto-close-active-risk-events/03.png)

當您按一下 [關閉所有事件] ，所有事件都已關閉，且受影響的使用者不再有風險。 不過，因為這個方法不會影響現有的密碼，所以不會讓相關的身分識別回到安全狀態。 這個方法的慣用使用案例是具有作用中風險事件的已刪除使用者。 


## <a name="close-individual-risk-events-manually"></a>手動關閉個別風險事件

您可以手動關閉個別風險事件。 手動關閉風險事件，即可降低使用者風險層級。 一般而言，會手動關閉風險事件以回應相關的調查。 例如，在與使用者交談揭露不再需要作用中風險事件時。 
 
手動關閉風險事件時，您可以選擇採取下列任何動作，以變更風險事件的狀態：

![動作](./media/howto-close-active-risk-events/06.png)

- **解決** - 如果在調查風險事件之後，您在 Identity Protection 外部採取適當的補救動作，而且您認為應該將風險事件視為已關閉，請將事件標示為 [已解決]。 解決的事件會將風險事件的狀態設定為 [已關閉]，而此風險事件便不再算是使用者風險。

- **標示為誤判** - 在某些情況下，您可能會調查某個風險事件並發現該事件被誤標為有風險。 您可以將風險事件標示為誤判，以減少發生這種情況。 這可協助機器學習演算法未來改善類似事件的分類。 誤判事件的狀態為 [已關閉]，而且不再算是使用者風險。

- **忽略** - 如果您尚未採取任何補救動作，但希望風險事件從作用中清單中移除，您可忽略風險事件，且事件狀態將會是 [已關閉]。 忽略的事件不算是使用者風險。 這個選項只能用於不尋常的情況下。

- **重新啟用** - 可以重新啟用已手動關閉的風險事件 (藉由選擇 [解決]、[誤判] 或 [略過])，並將事件狀態設回 [作用中]。 重新啟用的風險事件會納入使用者風險層級計算。 無法重新啟用透過補救 (例如重設安全的密碼) 關閉的風險事件。
  

## <a name="next-steps"></a>後續步驟

若要取得 Azure AD Identity Protection 的概觀，請參閱 [Azure AD Identity Protection 概觀](overview.md)。
