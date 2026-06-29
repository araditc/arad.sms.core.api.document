# Arad SMS Core API

This document describes the Arad SMS Core API for integrators. It is based on the previous Arad SMS Hub API and Monitoring API documents and the current API implementation in `V4.0_Codex`.

## Base URL and Versions

Replace `{baseAddress}` with the URL provided by Arad.

```text
https://{baseAddress}
```

Swagger is available at:

```text
https://{baseAddress}/swagger/index.html
```

The message API supports versioned routes:

```text
/api/v1.0/message
/api/v2.0/message
/api/v3.0/message
/api/v4.0/message
```

Version `v1.0` is the default-compatible version. Version `v4.0` is the latest message API version.

## Authentication

Most endpoints accept three authentication methods:

- Bearer token: `Authorization: Bearer {access_token}`
- Basic authentication: `Authorization: Basic {base64(username:password)}`
- API key: `X-API-Key: {api_key}`

For production access, the caller IP must also be allowed for the user/API key.

### Basic Authentication

Build a `username:password` string and Base64 encode it.

```text
AradUser:AradPassword -> QXJhZFVzZXI6QXJhZFBhc3N3b3Jk
Authorization: Basic QXJhZFVzZXI6QXJhZFBhc3N3b3Jk
```

### Bearer Token

```http
POST /connect/token
Content-Type: application/x-www-form-urlencoded
```

Form fields:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `username` / `UserName` | string | Yes | API username. |
| `password` / `Password` | string | Yes | API password. |
| `scope` / `Scope` | string | No | Requested scope. Defaults to `ApiAccess`. |
| `applicationId` / `ApplicationId` | string | No | Optional client application id. |

Example response:

```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 300,
  "expires_at": "2026-06-29T08:30:00Z",
  "scope": "ApiAccess"
}
```

### API Key

Create an API key from the dashboard and send it in every request:

```http
X-API-Key: {api_key}
```

The API key is validated together with the user status, expiration, parent account status, user settings, and allowed IPs.

## Scopes and Policies

| Scope | Used by |
| --- | --- |
| `ApiAccess` | Single message send, send by GET, special-parameter send. |
| `BulkApiAccess` | Bulk, P2P bulk, campaign creation, Webengage. |
| `ApiPatternAccess` | Pattern-based message send. |
| `WhiteListAccess` | Send to whitelist/phonebook. |
| `LiveDataAccess` | LiveData and report endpoints. |
| `MessageRequestAccess` | Phonebook and schedule endpoints. |
| `SupportTicketAccess` | Ticket endpoints. |
| `SmartSmsAccess` | Smart SMS endpoints. |
| `OtpAccess` | OTP endpoints. |
| `KycAccess` | KYC endpoints. |
| `PushNotificationAccess` | Push notification endpoints, if enabled. |
| `SmsWalletAccess` | SMS wallet endpoints, if enabled. |

Delivery-report, MO, user-data, and IP-check policies accept one of these scopes: `ApiAccess`, `BulkApiAccess`, `ApiPatternAccess`, `WhiteListAccess`, or `OtpAccess`.

## Standard Response Envelope

Most API responses use this envelope:

```json
{
  "message": "Successfully done.",
  "succeeded": true,
  "data": [],
  "resultCode": 100
}
```

For errors, `succeeded` is `false` and `resultCode` contains an `ApiResponse` value.

## Message Request Models

### `AradA2PMessage`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `sourceAddress` | string | Yes | Sender number. |
| `destinationAddress` | string | Yes | Recipient number. |
| `messageText` | string | Yes | Message body. |
| `dataCoding` | enum | No | Encoding. Common values: `Default`/`0`, `ASCII`/`1`, `UCS2`/`8`. |
| `validityPeriod` | DateTime | No | Message expiration. If not set, system default is used. |
| `targetUDH` | string[] | No | Optional UDH metadata such as `port:{port}` and `referenceNumberType:8bit|16bit`. |
| `serviceID` | string | No | VAS service id. |
| `priority` | enum | No | `Lowest` 0 through `Highest` 7. |
| `messageClass` | enum | No | `TEXT`, `BINARY`, `FLASH`, etc. |
| `udh` | string | No | User-defined tag/correlation value. |

### `AradBulkMessage`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `sourceAddress` | string | Yes | Sender number. |
| `destinationAddress` | string[] | Yes | Recipient numbers. |
| `messageText` | string | Yes | Message body. |
| `validityPeriod` | DateTime | No | Message expiration. |
| `serviceID` | string | No | VAS service id. |
| `servicePrice` | number | No | VAS service price. |
| `targetUDH` | string[] | No | Optional UDH metadata. |
| `udh` | string | No | User-defined tag/correlation value. |

## Message Endpoints

Use `{version}` as `v1.0`, `v2.0`, `v3.0`, or `v4.0`.

| Method | Path | Scope | Description |
| --- | --- | --- | --- |
| `POST` | `/api/{version}/message/send` | `ApiAccess` | Send a list of A2P messages. |
| `GET` | `/api/{version}/message/send` | `ApiAccess` | Send by query string. Multiple destinations are comma-separated. |
| `POST` | `/api/{version}/message/SendBySpecialParameter` | `ApiAccess` | Send after resolving a destination by an inquiry/special parameter. |
| `POST` | `/api/{version}/message/bulk` | `BulkApiAccess` | Send one text to multiple destinations. |
| `POST` | `/api/{version}/message/P2PBulk` | `BulkApiAccess` | Send peer-to-peer bulk messages where each destination can have its own text. |
| `GET` | `/api/{version}/message/CreateCampaign` | `BulkApiAccess` | Create a campaign by `campaignName`. |
| `GET` | `/api/{version}/message/SendToWhiteList` | `WhiteListAccess` | Send to a phonebook/whitelist. |
| `POST` | `/api/{version}/message/GetDLR` | DLR/MO policy | Pull delivery reports by message id. |
| `POST` | `/api/{version}/message/GetDLRByPartId` | DLR/MO policy | Pull delivery reports by part id. |
| `GET` | `/api/{version}/message/GetMO` | DLR/MO policy | Pull up to 1000 unread incoming messages. |
| `GET` | `/api/{version}/message/GetMOByDate` | DLR/MO policy | Pull incoming messages by date range. |

### Send A2P Messages

```http
POST /api/v4.0/message/send?returnLongId=false&returnPartId=false
Authorization: Bearer {access_token}
Content-Type: application/json
```

```json
[
  {
    "sourceAddress": "989000xxxx",
    "destinationAddress": "98912xxxxxxx",
    "messageText": "Test message",
    "targetUDH": ["port:5000", "referenceNumberType:16bit"],
    "udh": "customer-order-123"
  }
]
```

Successful `v4.0` response:

```json
{
  "message": "Successfully done.",
  "succeeded": true,
  "data": [
    {
      "id": "668a5dac7f212cbf40fd50f2",
      "part": "1",
      "upstreamGateway": "GatewayName"
    }
  ],
  "resultCode": 100
}
```

### Versioned Send Responses

| Version | `data` shape |
| --- | --- |
| `v1.0` | Array of message id strings. If `returnPartId=true`, array of `{ messageId, partIds }`. |
| `v2.0` | Array of key/value pairs where key is message id and value is MCC/MNC/operator metadata. |
| `v3.0` | Array of key/value pairs where key is message id and value is message part count. |
| `v4.0` | Array of `{ id, part, upstreamGateway }`. |

If `returnLongId=true`, returned ids follow the long-id format used by the platform. Use the same flag when querying DLR if long ids were requested during send.

### Bulk Send

```http
POST /api/v4.0/message/bulk?returnLongId=false
Content-Type: application/json
```

```json
{
  "sourceAddress": "989000xxxx",
  "destinationAddress": ["98912xxxxxxx", "98913xxxxxxx"],
  "messageText": "Bulk test message",
  "targetUDH": ["port:5000", "referenceNumberType:16bit"]
}
```

### P2P Bulk Send

```http
POST /api/v4.0/message/P2PBulk?returnLongId=false
Content-Type: application/json
```

```json
[
  {
    "sourceAddress": "989000xxxx",
    "destinationAddress": "98912xxxxxxx",
    "messageText": "Message for recipient 1"
  },
  {
    "sourceAddress": "989000xxxx",
    "destinationAddress": "98913xxxxxxx",
    "messageText": "Message for recipient 2"
  }
]
```

### Send by GET

```http
GET /api/v4.0/message/send?sourceAddress=989000xxxx&destinationAddress=98912xxxxxxx,98913xxxxxxx&messageText=Hello&returnLongId=false
```

### Send to Whitelist

```http
GET /api/v4.0/message/SendToWhiteList?sourceAddress=989000xxxx&phoneBookId={phoneBookId}&templateId={templateId}&messageText=&returnLongId=false&selectedMember=98912xxxxxxx
```

Use either `templateId` or `messageText`, not both. If the phonebook has templates, `templateId` is required.

## Delivery Reports and MO

### Get Delivery Reports

```http
POST /api/v4.0/message/GetDLR?returnLongId=false&returnUDH=false&returnSentDate=false
Content-Type: application/json
```

```json
[
  "668a5dac7f212cbf40fd50f2"
]
```

The request is limited to 1000 ids.

Response data items:

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Message id. |
| `partStatus` | array | Part statuses. For `GetDLR`, tuple values represent part number, delivery status, and delivery date. |
| `deliveryStatus` | enum | Overall delivery status. |
| `deliveryDate` | DateTime? | Delivery time, when available. |
| `createDate` | DateTime? | Message creation time. |
| `sentDate` | DateTime? | Sent time when `returnSentDate=true`. |
| `udh` | string | Returned when `returnUDH=true`. |

### Get Delivery Reports by Part Id

```http
POST /api/v4.0/message/GetDLRByPartId?returnLongId=false&returnUDH=false&returnSentDate=false
Content-Type: application/json
```

```json
[
  "5575795555842650087"
]
```

### Get Incoming Messages

```http
GET /api/v4.0/message/GetMO?returnId=false
```

Returns up to 1000 unread messages for the authenticated user/IP.

When `returnId=false`, each item contains:

```json
{
  "sourceAddress": "98912xxxxxxx",
  "destinationAddress": "989000xxxx",
  "messageText": "Reply text",
  "receiveDateTime": "2026-06-29T08:10:00Z"
}
```

When `returnId=true`, `id` and `isRead` are also returned.

### Get Incoming Messages by Date

```http
GET /api/v4.0/message/GetMOByDate?startDateTime=2026-06-29T00:00:00Z&endDateTime=2026-06-29T23:59:59Z&returnId=true
```

## Pattern-Based Messaging

```http
POST /api/patternMessage/send?returnLongId=false
Authorization: Bearer {ApiPatternAccess token}
Content-Type: application/json
```

```json
{
  "patternId": "welcome-template",
  "parameters": ["Ali", "12345"],
  "destinations": ["98912xxxxxxx"]
}
```

For multiple patterns:

```http
POST /api/patternMessage/sendMultiple?returnLongId=false
```

The number of `parameters` must match the placeholders in the approved pattern. `patternId` and sender access are validated against the authenticated user.

## LiveData and Monitoring

All LiveData endpoints require `LiveDataAccess`.

| Method | Path | Description |
| --- | --- | --- |
| `GET` | `/api/v1.0/LiveData/LiveData` | Recent live chart data for active upstream gateways. |
| `GET` | `/api/v1.0/LiveData/Last` | Latest available live data point, searching back up to 10 minutes. |
| `GET` | `/api/v1.0/LiveData/Day?date=yyyy/MM/dd` | Daily live data grouped by hour. |
| `GET` | `/api/v1.0/LiveData/UpStreamInfo` | Upstream connection and ping information. |
| `GET` | `/api/v1.0/LiveData/UsersInfo?userName={userName}` | User and credit/tariff information for sub-users. |
| `GET` | `/api/v1.0/LiveData/GetUserConnectivityStatus` | User connectivity status. |
| `GET` | `/api/v1.0/LiveData/MonitoringAZ?fullDateTime=false` | Aggregate monitoring view. |
| `GET` | `/api/v1.0/LiveData/MonitoringUpstream?fullDateTime=false` | Upstream monitoring view. |
| `GET` | `/api/v1.0/LiveData/MonitoringQueue?fullDateTime=false` | Queue monitoring view. |
| `GET` | `/api/v1.0/LiveData/MonitoringInputTps?fullDateTime=false` | Input TPS monitoring view. |
| `GET` | `/api/v1.0/LiveData/MonitoringTenHighTrafficUsers?fullDateTime=false` | Top traffic users monitoring view. |
| `GET` | `/api/v1.0/LiveData/MonitoringPercentageUser?fullDateTime=false` | User traffic-share monitoring view. |

`LiveData` and `Day` return this shape in `data`:

```json
{
  "date": "2026-06-29",
  "result": [
    {
      "upstreamGatewayName": "GatewayName",
      "assets": [
        {
          "type": "Delivered",
          "data": [
            { "time": "10:30:00", "count": 123 }
          ]
        }
      ]
    }
  ]
}
```

Asset types include `Blocked`, `Received`, `Rejected`, `Undeliverable`, `Sent`, `NotSent`, `Errors`, and `Delivered`.

## User and Tools

| Method | Path | Scope | Description |
| --- | --- | --- | --- |
| `GET` | `/api/user/UserInfo` | Get-data policy | Account information, sender ids, credit, payment type, and MPS. |
| `GET` | `/api/user/GetMapTraffics?username={username}` | Get-data policy | List mapped traffic records. |
| `POST` | `/api/user/AddMapTraffics` | Get-data policy | Add mapped traffic records. |
| `PUT` | `/api/user/ModifyMapTraffics?id={id}` | Get-data policy | Activate/deactivate a mapped traffic record. |
| `GET` | `/api/Tools/CheckIp` | Get-data policy | Check the caller IP against allowed IPs. |
| `GET` | `/api/Tools/Ping` | Public | Returns `PONG`. |

## Webengage

```http
POST /api/webengage/SendSMS
Authorization: Bearer {BulkApiAccess token}
Content-Type: application/json
```

The Webengage endpoint maps platform send statuses to Webengage-compatible responses such as `sms_accepted` or `sms_failed`.

## Other Endpoint Groups

These controllers are available in the current API and are documented through Swagger:

| Group | Base path | Scope |
| --- | --- | --- |
| OTP | `/api/v1.0/Otp` | `OtpAccess` |
| KYC | `/api/v1.0/Kyc` | `KycAccess` |
| Smart SMS | `/api/v1.0/SmartSms` | `SmartSmsAccess` |
| Phonebook | `/api/v1.0/PhoneBook` | `MessageRequestAccess` |
| Schedule | `/api/v1.0/Schedule` | `MessageRequestAccess` |
| Report | `/api/v1.0/Report` | `LiveDataAccess` |
| Ticket | `/api/v1.0/Ticket` | `SupportTicketAccess` |

## Delivery Status Values

| Value | Name |
| --- | --- |
| `1` | `Delivered` |
| `2` | `UnDelivered` |
| `3` | `Accepted` |
| `5` | `Rejected` |
| `7` | `ErrorInSending` |
| `8` | `WaitingForSend` |
| `9` | `Sent` |
| `10` | `NotSent` |
| `11` | `Expired` |
| `14` | `BlackList` |
| `15` | `SmsIsFilter` |
| `16` | `Deleted` |
| `29` | `Stored` |
| `32` | `Unknown` |
| `33` | `Enroute` |
| `34` | `Undeliverable` |
| `36` | `UnreachableNetwork` |

## API Result Codes

| Code | Name |
| --- | --- |
| `100` | `Succeeded` |
| `101` | `DatabaseError` |
| `102` | `RepositoryError` |
| `103` | `ModelError` |
| `107` | `UserNotFound` |
| `109` | `NotFound` |
| `116` | `HeaderError` |
| `118` | `RateLimitExceedResponse` |
| `119` | `UserTariffNull` |
| `120` | `LimitOnDailySubmissions` |
| `121` | `LimitOnMonthlySubmissions` |
| `122` | `SendToWhiteListOnly` |
| `130` | `Authorize` |
| `135` | `NotEnoughCredit` |
| `136` | `InvalidMessageId` |
| `138` | `InvalidScope` |
| `139` | `ApiKeyExpired` |
| `140` | `PendingApprove` |

## Send Error Codes

| Code | Name |
| --- | --- |
| `0` | `SendError` |
| `-1` | `NotEnoughCredit` |
| `-3` | `DeActiveAccount` |
| `-4` | `ExpiredAccount` |
| `-8` | `NumberAtBlackList` |
| `-11` | `InvalidSenderNumber` |
| `-12` | `InvalidReceiverNumber` |
| `-18` | `InvalidIpAddress` |
| `-19` | `InvalidPattern` |
| `-22` | `InvalidPort` |
| `-23` | `MessageTooLong` |
| `-25` | `InvalidReferenceNumberType` |
| `-26` | `InvalidTargetUDH` |
| `-28` | `DataCodingNotAllowed` |
| `-29` | `NotFoundRoute` |
| `-31` | `SettingNotFound` |
| `-32` | `ContentFilter` |
| `-33` | `InvalidCharacter` |
| `-41` | `LimitedInSchedule` |
| `-42` | `TooManyRequest` |
| `-43` | `CacheServerError` |
| `-44` | `MessageTextIsEmpty` |

## Notes

- Dates returned by MO/DLR endpoints are UTC unless a specific endpoint converts them to local time.
- `GetMO` and `GetMOByDate` are limited to 1000 records.
- `GetDLR` and `GetDLRByPartId` accept up to 1000 ids per request.
- API version differences affect response metadata, not the required send input fields.
