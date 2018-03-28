# 跨链协议 ILP

## 前言

本文内容为 Interledger Protocol (ILP)跨链协议。本协议描述了[RFC 791](https://tools.ietf.org/html/rfc791)中定义的 IP 协议。跨链协议是过去十余年来去中心化支付协议中的研究高峰。这项工作始于2004年 Ryan Fugger ，在2008年比特币开发中也有众多贡献者由此开始讨论。

## 介绍

### 动机

今天，各个支付网络是相互孤立的，隔离的。在单个国家内进行支付还是一件相对容易的事，或者付款收款方都在同一个账本或网络中拥有账号的话，也还算容易。然而，从一个账本向另一个账本支付却通常是非常困难的。虽然系统间存在各种各样的联系，但他们却是人工的、迟缓的、或是昂贵的。

跨账本协议提供了一种不同数字资产账本间的支付路由，即使这些账本的收发双方是相互隔离的，并因此存在失败风险的。安全的多次转发后的支付和自动路由机制，使得在全球多个不同资产网络中进行价值转移成为可能。

### 范围

### 接口

### 操作方法

## 概述

### 与其他协议的关系

### 操作模型

本协议采用“转账锁定机制”，来确保发送者的资金会被发送到接收者，否则会返回发送者账户。下面的例子来说明这个模型：

```
      (1,21)                                        (11)
       应用                   连接器                  应用
         \                     |                     /
         (2,20)              (6,16)             (10,12)
          跨链模块            跨链模块            跨链模块
              \            /       \            /
             (3,19)    (5,17)     (7,15)    (9,13)
                本地账本接口-1        本地账本接口-2
                      |                  |
                    (4,18)            (8,14)
                   本地账本 1         本地账本 2
```

1. 发送方应用程序先使用本地的高级别协议确定地址、金额、密码学信息，并确定一个目标账本的超时条件。然后用这些参数拼装成一个“支付消息”发送给跨链模块。

2. 跨链模块准备好跨链协议报文，选定用于向本地账本发送交易的账号，并发往本地账本接口。

3. 本地账本接口创建一个本地账本的转账交易，包含加密信息参数，然后在本地账本中验证这次转账请求。

4. 锁定资金，先不转账，通知connector

5. 连接器收到通知，传给跨链模块

6. 连接器的跨链模块解析出跨链请求报文，判断出应该往哪进行支付。跨链模块调用目标账本的本地接口，发送第二个交易报文，包含sender交易中的相同内容

7. 本地接口收到交易，然后认证授权本次交易发往本地账本。

8. 账本锁定连接器的资金——并不会直接转移给收款人——并且通知目标主机

9. 目标主机本地账本接口收到通知，传给跨链模块

10. 跨链模块解析跨链支付信息包，并确定支付到哪个应用。然后它将数据传递给应用程序。

11. 收款方应用收到提示，识别资金，并锁定履行情况为暂存状态。检查sender声明的入款执行情况。如果检查通过，收款方应用就产生一个履行结果并传递给跨链模块。

12. 收款人的跨链账本模块把操作履行情况传给本地账本接口。

13. 本地账本接口向账本提交支付结果情况

14. 收款方账本验证履行结果中对应的被锁定交易情况。如果履行情况检查通过，并且转账没有过期，则账本执行转账，并通知收款方连接器。

15. 连接器本地账本接口收到履行结果通知，并传给跨链模块

16. 连接器的跨链模块接收到履行结果，传递给本地账本，作为付款账本的响应。

17. 这个账本接口提交履行给付款账本。

18. 原账本验证支付结果，检查付款锁定条件。如果履行结果有效，转账没有过期，则账本执行转账，并通知给付款方。

19. 付款方本地账本接口收到支付结果通知，并传给跨链模块。

20. 发送者跨链模块接收到支付结果提醒，并传递给应用程序。

21. 付款方应用收到支付结果消息。

### 功能描述

跨链协议的目的，是要帮助主机提供一个支付过程中的路由能力，使得交易可以在一组账本间彼此联通。主要方法就是在等到收款方确认之后，再通过一个跨链模块把交易发送到另一个跨链模块。在跨链系统中，跨链模块是作为应用主机和连接器中的一部分存在的。为支付信息进行跨链交易路由时，每个账本内的地址是由跨链寻址功能来进行解释的。这样，跨链协议的一个中心组件就是跨链寻址功能。

在大金额交易场景中，连接器和中间账本选择的路由的算法可能是不够可信的。有潜在账本提供的锁定机制“可能”会保护发送者和接受者信息。在这种情况下，跨链交易报文需要含有加密信息和有效时限。

#### 跨链寻址

### 连接器 Connectors

### 错误 Errors

## 规范

### 跨账本支付报文格式

#### Delivered Payment Packet


| Field | Type | Short Description |
|:--|:--|:--|
| type | UInt8 | Always `1`, indicates that this ILP packet is an ILP Payment Packet (type 1) |
| length | Length Determinant | Indicates how many bytes the rest of the packet has |
| amount | UInt64 | Amount the destination account should receive, denominated in the asset of the destination ledger |
| account | Address | Address corresponding to the destination account |
| data | OCTET STRING | Transport layer data attached to the payment |
| extensions | Length Determinant | Always `0`

Here's an example:

| Type | Length, 8+(1+14)+(1+3)+1=28 | Amount (123,000,000) ... |
|:--|:--|:--|
| 1    |  28    | 0 0 0 0 7 84 |


| .. Amount (123,000,000) | Length | Address ... ('g.us.') |
|:--|:--|:--|
| 212 192 | 14     | 103 46 117 115 46 |

| ... Address ('nexus.bo') ... |
|:--|
| 110 101 120 117 115 46 98 111 |

| ... Address ('b') | length | data    | extensions |
|:--|:--|:--|:--|
| 98 | 3      | 4 16 65 | 0          |

### Forwarded Payment Packet (experimental)

| Field | Type | Short Description |
|:--|:--|:--|
| type | UInt8 | Always `10`, indicates that this ILP packet is an ILP Forwarded Payment Packet (type 10) |
| length | Length Determinant | Indicates how many bytes the rest of the packet has |
| account | Address | Address corresponding to the destination account |
| data | OCTET STRING | Transport layer data attached to the payment |
| extensions | Length Determinant | Always `0`

Example:

| Type | Length, (1+14)+(1+3)+1=20 | Length | Address ... ('g.us.nexus.bo')
|:--|:--|:--|:--|
| 10   |  20                       | 14     | 103 46 117 115 46 110 101 120 117 115 46 98 111 |

| ... Address ('b') | length | data    | extensions |
|:--|:--|:--|:--|
| 98 | 3      | 4 16 65 | 0          |


Let's look more closely at the three important fields: `amount`, `address`, and `data`.

#### amount

    UInt64 ::= INTEGER (0..18446744073709551615)

Amount in discrete units of the receiving ledger's asset type. Note that the amount is counted in terms of the smallest indivisible unit on the receiving ledger.

#### account

    -- Readable names for special characters that may appear in ILP addresses
    hyphen IA5String ::= "-"
    period IA5String ::= "."
    underscore IA5String ::= "_"
    tilde IA5String ::= "~"

    -- A standard interledger address
    Address ::= IA5String
      (FROM
        ( hyphen
        | period
        | "0".."9"
        | "A".."Z"
        | underscore
        | "a".."z"
        | tilde )
      )
      (SIZE (1..1023))

[Interledger Address](../0015-ilp-addresses/) of the receiving account.

#### data

    OCTET STRING (SIZE(0..32767))

Arbitrary data that is attached to the payment. The contents are defined by the transport layer protocol.

### ILP Rejection Format

Here is a summary of the fields in the ILP error format:

| Field | Type | Short Description |
|:--|:--|:--|
| code | IA5String | [ILP Error Code](#ilp-error-codes) |
| triggeredBy | Address | ILP address of the entity that originally emitted the error |
| message | UTF8String | Error data provided for debugging purposes |
| data | OCTET STRING | Error data provided for debugging purposes |

#### code

    IA5String (SIZE(3))

Error code. For example, `F00`. See [ILP Error Codes](#ilp-error-codes) for the list of error codes and their meanings.

#### triggeredBy

    Address

[ILP Address](#account) of the entity that originally emitted the error.

#### message

    UTF8String (SIZE(0..8191))

Human-readable error message. When emitting a rejection, implementations (receivers or connectors) MAY include additional debug information. When including additional debug information, implementations SHOULD use the format `Error message. key1=value1 key2=value2`, e.g.:

```
Invalid destination. destination=example.us.bob
```

This field MUST be encoded as UTF-8.


#### data

    OCTET STRING (SIZE(0..32767))

Machine-readable data. The format is defined for each error code. Implementations MUST follow the correct format for the code given in the `code` field. When parsing rejection packets, implementations MUST ignore extra bytes in the `data` field.

### ILP Error Format

> **Deprecated:** Use [ILP Rejection Format](#ilp-rejection-format) instead.

Here is a summary of the fields in the ILP error format:

| Field | Type | Short Description |
|:--|:--|:--|
| code | IA5String | [ILP Error Code](#ilp-error-codes) |
| name | IA5String | [ILP Error Code Name](#ilp-error-codes) |
| triggeredBy | Address | ILP address of the entity that originally emitted the error |
| forwardedBy | SEQUENCE OF Address | ILP addresses of connectors that relayed the error message |
| triggeredAt | GeneralizedTime | Time when the error was initially emitted |
| data | OCTET STRING | Error data provided for debugging purposes |

#### code

    IA5String (SIZE(3))

Error code. For example, `F00`. See [ILP Error Codes](#ilp-error-codes) for the list of error codes and their meanings.

#### name

    IA5String

Error name. For example, `Bad Request`. See [ILP Error Codes](#ilp-error-codes) for the list of error codes and their meanings.

Implementations of ILP SHOULD NOT depend on the `name` instead of the `code`. The name is primarily provided as a convenience to facilitate debugging by humans. If the `name` does not match the `code`, the `code` is the definitive identifier of the error.

#### triggeredBy

[ILP Address](#account) of the entity that originally emitted the error. This MAY be an address prefix if the entity that originally omitted the error is a ledger.

#### forwardedBy

[ILP Addresses](#account) of the connectors that relayed the error message.

#### triggeredAt

    GeneralizedTime

Date and time when the error was initially emitted. This MUST be expressed in the UTC + 0 (Z) timezone.

#### data

    OCTET STRING (SIZE(0..8192))

Error data provided primarily for debugging purposes. Systems that emit errors SHOULD include additional explanation or context about the issue.

Protocols built on top of ILP that define behavior for certain errors SHOULD specify the encoding format of error `data`.

Unless otherwise specified, `data` SHOULD be encoded as UTF-8.

### ILP Error Codes

Inspired by [HTTP Status Codes](https://tools.ietf.org/html/rfc2616#section-10), ILP errors are categorized based on the intended behavior of the caller when they get the given error.

#### F__ - Final Error

Final errors indicate that the payment is invalid and should not be retried unless the details are changed.

| Code | Name | Description | Data Fields |
|---|---|---|---|
| **F00** | **Bad Request** | Generic sender error. | (empty) |
| **F01** | **Invalid Packet** | The ILP packet was syntactically invalid. | (empty) |
| **F02** | **Unreachable** | There was no way to forward the payment, because the destination ILP address was wrong or the connector does not have a route to the destination. | (empty) |
| **F03** | **Invalid Amount** | The amount is invalid, for example it contains more digits of precision than are available on the destination ledger or the amount is greater than the total amount of the given asset in existence. | (empty) |
| **F04** | **Insufficient Destination Amount** | The receiver deemed the amount insufficient, for example you tried to pay a $100 invoice with $10. | (empty) |
| **F05** | **Wrong Condition** | The receiver generated a different condition and cannot fulfill the payment. | (empty) |
| **F06** | **Unexpected Payment** | The receiver was not expecting a payment like this (the memo and destination address don't make sense in that combination, for example if the receiver does not understand the transport protocol used) | (empty) |
| **F07** | **Cannot Receive** | The receiver is unable to accept this payment due to a constraint. For example, the payment would put the receiver above its maximum account balance. | (empty) |
| **F99** | **Application Error** | Reserved for application layer protocols. Applications MAY use names other than `Application Error`. | (empty) |

#### T__ - Temporary Error

Temporary errors indicate a failure on the part of the receiver or an intermediary system that is unexpected or likely to be resolved soon. Senders SHOULD retry the same payment again, possibly after a short delay.

| Code | Name | Description | Data Fields |
|---|---|---|---|
| **T00** | **Internal Error** | A generic unexpected exception. This usually indicates a bug or unhandled error case. | (empty) |
| **T01** | **Ledger Unreachable** | The connector has a route or partial route to the destination but was unable to reach the next ledger. Try again later. | (empty) |
| **T02** | **Ledger Busy** | The ledger is rejecting requests due to overloading. Try again later. | (empty) |
| **T03** | **Connector Busy** | The connector is rejecting requests due to overloading. Try again later. | (empty) |
| **T04** | **Insufficient Liquidity** | The connector would like to fulfill your request, but it doesn't currently have enough money. Try again later. | (empty) |
| **T05** | **Rate Limited** | The sender is sending too many payments and is being rate-limited by a ledger or connector. If a connector gets this error because they are being rate-limited, they SHOULD retry the payment through a different route or respond to the sender with a `T03: Connector Busy` error. | (empty) |
| **T99** | **Application Error** | Reserved for application layer protocols. Applications MAY use names other than `Application Error`. | (empty) |

#### R__ - Relative Error

Relative errors indicate that the payment did not have enough of a margin in terms of money or time. However, it is impossible to tell whether the sender did not provide enough error margin or the path suddenly became too slow or illiquid. The sender MAY retry the payment with a larger safety margin.

| Code | Name | Description | Data Fields |
|---|---|---|---|
| **R00** | **Transfer Timed Out** | The transfer timed out, meaning the next party in the chain did not respond. This could be because you set your timeout too low or because something look longer than it should. The sender MAY try again with a higher expiry, but they SHOULD NOT do this indefinitely or a malicious connector could cause them to tie up their money for an unreasonably long time. | (empty) |
| **R01** | **Insufficient Source Amount** | Either the sender did not send enough money or the exchange rate changed before the payment was prepared. The sender MAY try again with a higher amount, but they SHOULD NOT do this indefinitely or a malicious connector could steal money from them. | (empty) |
| **R02** | **Insufficient Timeout** | The connector could not forward the payment, because the timeout was too low to subtract its safety margin. The sender MAY try again with a higher expiry, but they SHOULD NOT do this indefinitely or a malicious connector could cause them to tie up their money for an unreasonably long time. | (empty) |
| **R99** | **Application Error** | Reserved for application layer protocols. Applications MAY use names other than `Application Error`. | (empty) |

### ILP Fulfillment Data Format

This type of ILP packet is attached to a fulfillment and carries additional transport layer information that the sender may use to make further payments. Here is a summary of the fields in the ILP fulfillment data format:

| Field | Type | Short Description |
|:--|:--|:--|
| data | OCTET STRING | Transport layer fulfillment data |

#### Example

Consider a payload of `4 16 65` (payload length is `3`).
The length of the envelope content would then be `1+3+1 = 5`.
The ILP packet type is always `9` for fulfillment data, and extensions is always `0`, so on the wire
(and when passed between software modules like for instance ledger plugins),
the resulting ILP packet would look like this:

| type | content length | payload length | data | extensions |
|:--|:--|:--|:--|:--|
| 9 | 5 | 3 | 4 16 65 | 0 |

#### data

    OCTET STRING (SIZE(0..32767))

Arbitrary data that is attached to the fulfillment. The contents are defined by the transport layer protocol.

## Appendix A: ASN.1 Module

See [ASN.1 Definitions](../asn1/InterledgerProtocol.asn).

## Appendix B: IANA Considerations

### Header Type Registry

The following initial entries should be added to the Interledger Header Type registry to be created and maintained at (the suggested URI) http://www.iana.org/assignments/interledger-header-types:

| Header Type ID | Protocol | Message Type |
|:--|:--|:--|
| 1 | ILP | [IlpPayment](#ilp-payment-packet-format) |
| 2 | [ILQP][] | QuoteLiquidityRequest |
| 3 | [ILQP][] | QuoteLiquidityResponse |
| 4 | [ILQP][] | QuoteBySourceAmountRequest |
| 5 | [ILQP][] | QuoteBySourceAmountResponse |
| 6 | [ILQP][] | QuoteByDestinationAmountRequest |
| 7 | [ILQP][] | QuoteByDestinationAmountResponse |
| 8 | ILP | [IlpError](#ilp-error-format) |
| 9 | ILP | [IlpFulfillmentData](#ilp-fulfillment-data-format) |
| 10 | ILP | [IlpForwardedPaymentData](#ilp-forwarded-payment-packet-experimental) |
| 11 | ILP | [IlpRejectionData](#ilp-rejection-format) |
