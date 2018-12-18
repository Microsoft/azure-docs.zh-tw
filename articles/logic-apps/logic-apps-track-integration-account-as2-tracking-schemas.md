---
title: 適用於 B2B 訊息的 AS2 追蹤結構描述 - Azure Logic Apps | Microsoft Docs
description: 建立 AS2 追蹤結構描述，以針對採用 Enterprise Integration Pack 的 Azure Logic Apps 監視整合帳戶中的 B2B 訊息
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: f169c411-1bd7-4554-80c1-84351247bf94
ms.date: 01/27/2017
ms.openlocfilehash: 6c4144d26042729684e507b1afaa5e3006d8a34e
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125925"
---
# <a name="create-schemas-for-tracking-as2-messages-and-mdns-in-integration-accounts-for-azure-logic-apps"></a>建立結構描述，以便追蹤 Azure Logic Apps 整合帳戶中的 AS2 訊息和 MDN

您可以在整合帳戶中使用這些 AS2 追蹤結構描述，協助您監視企業對企業 (B2B) 交易的成功、錯誤及訊息屬性︰

* AS2 訊息追蹤結構描述
* AS2 MDN 追蹤結構描述

## <a name="as2-message-tracking-schema"></a>AS2 訊息追蹤結構描述

```json
{
   "agreementProperties": {  
      "senderPartnerName": "",  
      "receiverPartnerName": "",  
      "as2To": "",  
      "as2From": "",  
      "agreementName": ""  
   },  
   "messageProperties": {
      "direction": "",
      "messageId": "",
      "dispositionType": "",
      "fileName": "",
      "isMessageFailed": "",
      "isMessageSigned": "",
      "isMessageEncrypted": "",
      "isMessageCompressed": "",
      "correlationMessageId": "",
      "incomingHeaders": {
       },
      "outgoingHeaders": {
       },
      "isNrrEnabled": "",
      "isMdnExpected": "",
      "mdnType": ""
    }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | AS2 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | AS2 訊息接收者的夥伴名稱。 (選用) |
| as2To | 字串 | AS2 訊息接收者的名稱，從 AS2 訊息的標頭。 (必要) |
| as2From | 字串 | AS2 訊息傳送者的名稱，從 AS2 訊息的標頭。 (必要) |
| agreementName | 字串 | 據以解析訊息的 AS2 合約名稱。 (選用) |
| direction | 字串 | 訊息流程的方向，接收或傳送。 (必要) |
| messageId | 字串 | AS2 訊息 ID，從 AS2 訊息的標頭 (選用) |
| dispositionType |字串 | 訊息處理通知 (MDN) 配置類型值。 (選用) |
| fileName | 字串 | 檔案名稱，從 AS2 訊息的標頭。 (選用) |
| isMessageFailed |BOOLEAN | AS2 訊息是否失敗。 (必要) |
| isMessageSigned | BOOLEAN | AS2 訊息是否簽署。 (必要) |
| isMessageEncrypted | BOOLEAN | AS2 訊息是否加密。 (必要) |
| isMessageCompressed |BOOLEAN | AS2 訊息是否壓縮。 (必要) |
| correlationMessageId | 字串 | AS2 訊息識別碼，將訊息與 MDN 相互關聯。 (選用) |
| incomingHeaders |JToken 的字典 | 內送 AS2 訊息標頭詳細資料。 (選用) |
| outgoingHeaders |JToken 的字典 | 外寄 AS2 訊息標頭詳細資料。 (選用) |
| isNrrEnabled | BOOLEAN | 如果不知道值，則使用預設值。 (必要) |
| isMdnExpected | Boolean | 如果不知道值，則使用預設值。 (必要) |
| mdnType | 例舉 | 允許的值為 **NotConfigured**、**Sync** 和 **Async**。 (必要) |
||||

## <a name="as2-mdn-tracking-schema"></a>AS2 MDN 追蹤結構描述

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "as2To": "",
      "as2From": "",
      "agreementName": "g"
   },
   "messageProperties": {
      "direction": "",
      "messageId": "",
      "originalMessageId": "",
      "dispositionType": "",
      "isMessageFailed": "",
      "isMessageSigned": "",
      "isNrrEnabled": "",
      "statusCode": "",
      "micVerificationStatus": "",
      "correlationMessageId": "",
      "incomingHeaders": {
      },
      "outgoingHeaders": {
      }
   }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | AS2 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | AS2 訊息接收者的夥伴名稱。 (選用) |
| as2To | 字串 | 接收該 AS2 訊息的夥伴名稱。 (必要) |
| as2From | 字串 | 傳送該 AS2 訊息的夥伴名稱。 (必要) |
| agreementName | 字串 | 據以解析訊息的 AS2 合約名稱。 (選用) |
| direction |字串 | 訊息流程的方向，接收或傳送。 (必要) |
| messageId | 字串 | AS2 訊息識別碼。 (選用) |
| originalMessageId |字串 | AS2 原始訊息識別碼。 (選用) |
| dispositionType | 字串 | MDN 處置類型值。 (選用) |
| isMessageFailed |BOOLEAN | AS2 訊息是否失敗。 (必要) |
| isMessageSigned |BOOLEAN | AS2 訊息是否簽署。 (必要) |
| isNrrEnabled | BOOLEAN | 如果不知道值，則使用預設值。 (必要) |
| StatusCode | 例舉 | 允許的值為 **Accepted**、**Rejected** 和 **AcceptedWithErrors**。 (必要) |
| micVerificationStatus | 例舉 | 允許的值為 **NotApplicable**、**Succeeded** 和 **Failed**。 (必要) |
| correlationMessageId | 字串 | 相互關連識別碼。 原始 messaged 識別碼 (設定 MDN 之訊息的訊息識別碼)。 (選用) |
| incomingHeaders | JToken 的字典 | 指出內送訊息標頭詳細資料。 (選用) |
| outgoingHeaders |JToken 的字典 | 指出外寄訊息標頭詳細資料。 (選用) |
||||

## <a name="b2b-protocol-tracking-schemas"></a>B2B 通訊協定追蹤結構描述

如需 B2B 通訊協定追蹤結構描述的相關資訊，請參閱︰

* [X12 追蹤結構描述](logic-apps-track-integration-account-x12-tracking-schema.md)
* [B2B 自訂追蹤結構描述](logic-apps-track-integration-account-custom-tracking-schema.md)

## <a name="next-steps"></a>後續步驟

* 了解[監視 B2B 訊息](logic-apps-monitor-b2b-message.md)
* 了解[在 Log Analytics 中追蹤 B2B 訊息](../logic-apps/logic-apps-track-b2b-messages-omsportal.md)