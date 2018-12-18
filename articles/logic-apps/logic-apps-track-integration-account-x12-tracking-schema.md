---
title: 適用於 B2B 訊息的 X12 追蹤結構描述 - Azure Logic Apps | Microsoft Docs
description: 建立 X12 追蹤結構描述，以針對採用 Enterprise Integration Pack 的 Azure Logic Apps 監視整合帳戶中的 B2B 訊息
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: a5413f80-eaad-4bcf-b371-2ad0ef629c3d
ms.date: 01/27/2017
ms.openlocfilehash: cfd195f2f812c8b2e09058d325d0dbb6f7a60d59
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125596"
---
# <a name="create-schemas-for-tracking-x12-messages-in-integration-accounts-for-azure-logic-apps"></a>建立結構描述，以便追蹤 Azure Logic Apps 整合帳戶中的 X12 訊息

您可以在整合帳戶中使用這些 X12 追蹤結構描述，協助您監視企業對企業 (B2B) 交易的成功、錯誤及訊息屬性︰

* X12 交易集追蹤結構描述
* X12 交易集通知追蹤結構描述
* X12 交換追蹤結構描述
* X12 交換通知追蹤結構描述
* X12 功能群組追蹤結構描述
* X12 功能群組通知追蹤結構描述

## <a name="x12-transaction-set-tracking-schema"></a>X12 交易集追蹤結構描述

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "transactionSetControlNumber": "",
      "CorrelationMessageId": "",
      "messageType": "",
      "isMessageFailed": "",
      "isTechnicalAcknowledgmentExpected": "",
      "isFunctionalAcknowledgmentExpected": "",
      "needAk2LoopForValidMessages":  "",
      "segmentsCount": ""
   }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | X12 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | X12 訊息接收者的夥伴名稱。 (選用) |
| senderQualifier | 字串 | 傳送合作夥伴辨識符號。 (必要) |
| senderIdentifier | 字串 | 傳送合作夥伴識別碼。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴辨識符號。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴識別碼。 (必要) |
| agreementName | 字串 | 據以解析訊息的 X12 合約名稱。 (選用) |
| direction | 例舉 | 訊息流程的方向，接收或傳送。 (必要) |
| interchangeControlNumber | 字串 | 交換控制編號。 (選用) |
| functionalGroupControlNumber | 字串 | 功能的控制編號。 (選用) |
| transactionSetControlNumber | 字串 | 交易集控制編號。 (選用) |
| CorrelationMessageId | 字串 | 相互關聯訊息識別碼。 {AgreementName}{*GroupControlNumber*}{TransactionSetControlNumber} 組合。 (選用) |
| messageType | 字串 | 交易集或文件類型。 (選用) |
| isMessageFailed | BOOLEAN | X12 訊息是否失敗。 (必要) |
| isTechnicalAcknowledgmentExpected | BOOLEAN | X12 合約中是否設定技術通知。 (必要) |
| isFunctionalAcknowledgmentExpected | BOOLEAN | X12 合約中是否設定功能通知。 (必要) |
| needAk2LoopForValidMessages | BOOLEAN | 有效的訊息是否需要 AK2 迴圈。 (必要) |
| segmentsCount | Integer | X12 交易集的區段數目。 (選用) |
||||

## <a name="x12-transaction-set-acknowledgement-tracking-schema"></a>X12 交易集通知追蹤結構描述

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "isaSegment": "",
      "gsSegment": "",
      "respondingfunctionalGroupControlNumber": "",
      "respondingFunctionalGroupId": "",
      "respondingtransactionSetControlNumber": "",
      "respondingTransactionSetId": "",
      "statusCode": "",
      "processingStatus": "",
      "CorrelationMessageId": "",
      "isMessageFailed": "",
      "ak2Segment": "",
      "ak3Segment": "",
      "ak5Segment": ""
   }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | X12 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | X12 訊息接收者的夥伴名稱。 (選用) |
| senderQualifier | 字串 | 傳送合作夥伴辨識符號。 (必要) |
| senderIdentifier | 字串 | 傳送合作夥伴識別碼。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴辨識符號。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴識別碼。 (必要) |
| agreementName | 字串 | 據以解析訊息的 X12 合約名稱。 (選用) |
| direction | 例舉 | 訊息流程的方向，接收或傳送。 (必要) |
| interchangeControlNumber | 字串 | 功能通知的交換控制編號。 僅針對傳送至夥伴的訊息而接收功能通知的傳送端，才填入值。 (選用) |
| functionalGroupControlNumber | 字串 | 功能群組的交換控制編號。 僅針對傳送至夥伴的訊息而接收功能通知的傳送端，才填入值。 (選用) |
| isaSegment | 字串 | 訊息的 ISA 區段。 僅針對傳送至夥伴的訊息而接收功能通知的傳送端，才填入值。 (選用) |
| isaSegment | 字串 | 訊息的 GS 區段。 僅針對傳送至夥伴的訊息而接收功能通知的傳送端，才填入值。 (選用) |
| respondingfunctionalGroupControlNumber | 字串 | 回應交換控制編號。 (選用) |
| respondingFunctionalGroupId | 字串 | 回應功能群組識別碼，在通知中對應至 AK101。 (選用) |
| respondingtransactionSetControlNumber | 字串 | 回應交易集控制編號。 (選用) |
| respondingTransactionSetId | 字串 | 回應交易集識別碼，在通知中對應至 AK201。 (選用) |
| StatusCode | BOOLEAN | 交易集通知狀態碼。 (必要) |
| segmentsCount | 例舉 | 通知狀態碼。 允許的值為 **Accepted**、**Rejected** 和 **AcceptedWithErrors**。 (必要) |
| processingStatus | 例舉 | 通知的處理狀態。 允許的值為 **Received**、**Generated** 和 **Sent**。 (必要) |
| CorrelationMessageId | 字串 | 相互關聯訊息識別碼。 {AgreementName}{*GroupControlNumber*}{TransactionSetControlNumber} 組合。 (選用) |
| isMessageFailed | BOOLEAN | X12 訊息是否失敗。 (必要) |
| ak2Segment | 字串 | 接收的功能群組內交易集的通知。 (選用) |
| ak3Segment | 字串 | 報告資料區段中的錯誤。 (選用) |
| ak5Segment | 字串 | 報告是否接受或拒絕 AK2 區段中識別的交易集，以及原因。 (選用) |
||||

## <a name="x12-interchange-tracking-schema"></a>X12 交換追蹤結構描述

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "isaSegment": "",
      "isTechnicalAcknowledgmentExpected": "",
      "isMessageFailed": "",
      "isa09": "",
      "isa10": "",
      "isa11": "",
      "isa12": "",
      "isa14": "",
      "isa15": "",
      "isa16": ""
   }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | X12 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | X12 訊息接收者的夥伴名稱。 (選用) |
| senderQualifier | 字串 | 傳送合作夥伴辨識符號。 (必要) |
| senderIdentifier | 字串 | 傳送合作夥伴識別碼。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴辨識符號。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴識別碼。 (必要) |
| agreementName | 字串 | 據以解析訊息的 X12 合約名稱。 (選用) |
| direction | 例舉 | 訊息流程的方向，接收或傳送。 (必要) |
| interchangeControlNumber | 字串 | 交換控制編號。 (選用) |
| isaSegment | 字串 | 訊息 ISA 區段。 (選用) |
| isTechnicalAcknowledgmentExpected | BOOLEAN | X12 合約中是否設定技術通知。 (必要) |
| isMessageFailed | BOOLEAN | X12 訊息是否失敗。 (必要) |
| isa09 | 字串 | X12 文件交換日期。 (選用) |
| isa10 | 字串 | X12 文件交換時間。 (選用) |
| isa11 | 字串 | X12 交換控制標準識別碼。 (選用) |
| isa12 | 字串 | X12 交換控制版本號碼。 (選用) |
| isa14 | 字串 | 已要求 X12 通知。 (選用) |
| isa15 | 字串 | 用於測試或生產的指標。 (選用) |
| isa16 | 字串 | 元素分隔符號。 (選用) |
||||

## <a name="x12-interchange-acknowledgement-tracking-schema"></a>X12 交換通知追蹤結構描述

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "isaSegment": "",
      "respondingInterchangeControlNumber": "",
      "isMessageFailed": "",
      "statusCode": "",
      "processingStatus": "",
      "ta102": "",
      "ta103": "",
      "ta105": ""
   }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | X12 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | X12 訊息接收者的夥伴名稱。 (選用) |
| senderQualifier | 字串 | 傳送合作夥伴辨識符號。 (必要) |
| senderIdentifier | 字串 | 傳送合作夥伴識別碼。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴辨識符號。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴識別碼。 (必要) |
| agreementName | 字串 | 據以解析訊息的 X12 合約名稱。 (選用) |
| direction | 例舉 | 訊息流程的方向，接收或傳送。 (必要) |
| interchangeControlNumber | 字串 | 從合作夥伴收到的技術通知交換控制編號。 (選用) |
| isaSegment | 字串 | 從合作夥伴收到的技術通知 ISA 區段。 (選用) |
| respondingInterchangeControlNumber |字串 | 從合作夥伴收到的技術通知交換控制編號。 (選用) |
| isMessageFailed | BOOLEAN | X12 訊息是否失敗。 (必要) |
| StatusCode | 例舉 | 交換通知狀態碼。 允許的值為 **Accepted**、**Rejected** 和 **AcceptedWithErrors**。 (必要) |
| processingStatus | 例舉 | 通知狀態。 允許的值為 **Received**、**Generated** 和 **Sent**。 (必要) |
| ta102 | 字串 | 交換日期。 (選用) |
| ta103 | 字串 | 交換時間。 (選用) |
| ta105 | 字串 | 交換說明碼。 (選用) |
||||

## <a name="x12-functional-group-tracking-schema"></a>X12 功能群組追蹤結構描述

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "gsSegment": "",
      "isTechnicalAcknowledgmentExpected": "",
      "isFunctionalAcknowledgmentExpected": "",
      "isMessageFailed": "",
      "gs01": "",
      "gs02": "",
      "gs03": "",
      "gs04": "",
      "gs05": "",
      "gs07": "",
      "gs08": ""
   }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | X12 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | X12 訊息接收者的夥伴名稱。 (選用) |
| senderQualifier | 字串 | 傳送合作夥伴辨識符號。 (必要) |
| senderIdentifier | 字串 | 傳送合作夥伴識別碼。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴辨識符號。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴識別碼。 (必要) |
| agreementName | 字串 | 據以解析訊息的 X12 合約名稱。 (選用) |
| direction | 例舉 | 訊息流程的方向，接收或傳送。 (必要) |
| interchangeControlNumber | 字串 | 交換控制編號。 (選用) |
| functionalGroupControlNumber | 字串 | 功能的控制編號。 (選用) |
| isaSegment | 字串 | 訊息 GS 區段。 (選用) |
| isTechnicalAcknowledgmentExpected | BOOLEAN | X12 合約中是否設定技術通知。 (必要) |
| isFunctionalAcknowledgmentExpected | BOOLEAN | X12 合約中是否設定功能通知。 (必要) |
| isMessageFailed | BOOLEAN | X12 訊息是否失敗。 (必要)|
| gs01 | 字串 | 功能識別碼。 (選用) |
| gs02 | 字串 | 應用程式寄件者的代碼。 (選用) |
| gs03 | 字串 | 應用程式接收者的代碼。 (選用) |
| gs04 | 字串 | 功能群組的日期。 (選用) |
| gs05 | 字串 | 功能群組的時間。 (選用) |
| gs07 | 字串 | 負責單位的代碼。 (選用) |
| gs08 | 字串 | 版本/版次/產業識別碼。 (選用) |
||||

## <a name="x12-functional-group-acknowledgement-tracking-schema"></a>X12 功能群組通知追蹤結構描述

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "senderQualifier": "",
      "senderIdentifier": "",
      "receiverQualifier": "",
      "receiverIdentifier": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "interchangeControlNumber": "",
      "functionalGroupControlNumber": "",
      "isaSegment": "",
      "gsSegment": "",
      "respondingfunctionalGroupControlNumber": "",
      "respondingFunctionalGroupId": "",
      "isMessageFailed": "",
      "statusCode": "",
      "processingStatus": "",
      "ak903": "",
      "ak904": "",
      "ak9Segment": ""
   }
}
```

| 屬性 | 類型 | 說明 |
| --- | --- | --- |
| senderPartnerName | 字串 | X12 訊息傳送者的夥伴名稱。 (選用) |
| receiverPartnerName | 字串 | X12 訊息接收者的夥伴名稱。 (選用) |
| senderQualifier | 字串 | 傳送合作夥伴辨識符號。 (必要) |
| senderIdentifier | 字串 | 傳送合作夥伴識別碼。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴辨識符號。 (必要) |
| receiverQualifier | 字串 | 接收合作夥伴識別碼。 (必要) |
| agreementName | 字串 | 據以解析訊息的 X12 合約名稱。 (選用) |
| direction | 例舉 | 訊息流程的方向，接收或傳送。 (必要) |
| interchangeControlNumber | 字串 | 從合作夥伴收到技術通知時，傳送端會填入的交換控制編號。 (選用) |
| functionalGroupControlNumber | 字串 | 從合作夥伴收到技術通知時，傳送端會填入的技術通知功能群組控制編號。 (選用) |
| isaSegment | 字串 | 同交換控制編號，但只在特定情況下才會填入。 (選用) |
| isaSegment | 字串 | 同功能群組控制編號，但只在特定情況下才會填入。 (選用) |
| respondingfunctionalGroupControlNumber | 字串 | 原始功能群組的控制編號。 (選用) |
| respondingFunctionalGroupId | 字串 | 在通知功能群組識別碼中對應至 AK101。 (選用) |
| isMessageFailed | BOOLEAN | X12 訊息是否失敗。 (必要) |
| StatusCode | 例舉 | 通知狀態碼。 允許的值為 **Accepted**、**Rejected** 和 **AcceptedWithErrors**。 (必要) |
| processingStatus | 例舉 | 通知的處理狀態。 允許的值為 **Received**、**Generated** 和 **Sent**。 (必要) |
| ak903 | 字串 | 已接收交易集數目。 (選用) |
| ak904 | 字串 | 所識別的功能群組中接受的交易集數目。 (選用) |
| ak9Segment | 字串 | 是否接受或拒絕 AK1 區段中識別的功能群組，以及原因。 (選用) |
|||| 

## <a name="b2b-protocol-tracking-schemas"></a>B2B 通訊協定追蹤結構描述

如需 B2B 通訊協定追蹤結構描述的相關資訊，請參閱︰

* [AS2 追蹤結構描述](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)   
* [B2B 自訂追蹤結構描述](logic-apps-track-integration-account-custom-tracking-schema.md)

## <a name="next-steps"></a>後續步驟

* [深入了解監視 B2B 訊息](logic-apps-monitor-b2b-message.md)。
* 了解[在 Log Analytics 中追蹤 B2B 訊息](../logic-apps/logic-apps-track-b2b-messages-omsportal.md)。
