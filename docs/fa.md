# مستند API پیام آراد

این مستند برای اتصال به سرویس API پلتفرم پیام آراد تهیه شده است. محتوا بر اساس مستندات قبلی Arad SMS Hub API و Monitoring API و همچنین آخرین نسخه سرویس API به‌روز شده است.

## آدرس پایه و نسخه‌ها

به جای `{baseAddress}` آدرسی را قرار دهید که از آراد دریافت کرده‌اید.

```text
https://{baseAddress}
```

Swagger سرویس از مسیر زیر در دسترس است:

```text
https://{baseAddress}/swagger/index.html
```

API ارسال پیام نسخه‌بندی شده است:

```text
/api/v1.0/message
/api/v2.0/message
/api/v3.0/message
/api/v4.0/message
```

نسخه `v1.0` نسخه سازگار پیش‌فرض است و نسخه `v4.0` آخرین نسخه مسیرهای اصلی ارسال پیام است.

## احراز هویت

بیشتر متدها از سه روش احراز هویت پشتیبانی می‌کنند:

- توکن Bearer: مقدار `Authorization: Bearer {access_token}`
- احراز هویت Basic: مقدار `Authorization: Basic {base64(username:password)}`
- کلید API: مقدار `X-API-Key: {api_key}`

برای استفاده عملیاتی، IP درخواست‌دهنده نیز باید برای کاربر یا API Key مجاز شده باشد.

### احراز هویت Basic

نام کاربری و رمز عبور را با `:` کنار هم قرار داده و Base64 کنید.

```text
AradUser:AradPassword -> QXJhZFVzZXI6QXJhZFBhc3N3b3Jk
Authorization: Basic QXJhZFVzZXI6QXJhZFBhc3N3b3Jk
```

### دریافت Bearer Token

```http
POST /connect/token
Content-Type: application/x-www-form-urlencoded
```

پارامترهای فرم:

| پارامتر | نوع | اجباری | توضیح |
| --- | --- | --- | --- |
| `username` / `UserName` | string | بله | نام کاربری API. |
| `password` / `Password` | string | بله | رمز عبور API. |
| `scope` / `Scope` | string | خیر | اسکوپ درخواستی. مقدار پیش‌فرض `ApiAccess` است. |
| `applicationId` / `ApplicationId` | string | خیر | شناسه اختیاری برنامه کلاینت. |

نمونه پاسخ:

```json
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 300,
  "expires_at": "2026-06-29T08:30:00Z",
  "scope": "ApiAccess"
}
```

### استفاده از API Key

API Key از طریق داشبورد ایجاد می‌شود و در هدر هر درخواست قرار می‌گیرد:

```http
X-API-Key: {api_key}
```

در زمان اعتبارسنجی API Key، وضعیت کاربر، تاریخ انقضا، وضعیت والد، تنظیمات کاربر و IPهای مجاز هم بررسی می‌شوند.

## اسکوپ‌ها و سطح دسترسی

| اسکوپ | کاربرد |
| --- | --- |
| `ApiAccess` | ارسال تکی پیام، ارسال از طریق GET، ارسال با پارامتر خاص. |
| `BulkApiAccess` | ارسال Bulk، ارسال P2P Bulk، ساخت کمپین، Webengage. |
| `ApiPatternAccess` | ارسال پیام از طریق الگو. |
| `WhiteListAccess` | ارسال به لیست سفید/دفترچه تلفن. |
| `LiveDataAccess` | LiveData و گزارش‌ها. |
| `MessageRequestAccess` | دفترچه تلفن و زمان‌بندی. |
| `SupportTicketAccess` | تیکت‌ها. |
| `SmartSmsAccess` | سرویس Smart SMS. |
| `OtpAccess` | سرویس OTP. |
| `KycAccess` | سرویس KYC. |
| `PushNotificationAccess` | Push Notification، در صورت فعال بودن. |
| `SmsWalletAccess` | کیف پول پیامک، در صورت فعال بودن. |

سیاست دسترسی DLR/MO، اطلاعات کاربر و بررسی IP یکی از اسکوپ‌های `ApiAccess`، `BulkApiAccess`، `ApiPatternAccess`، `WhiteListAccess` یا `OtpAccess` را می‌پذیرد.

## قالب استاندارد پاسخ

اکثر پاسخ‌ها در قالب زیر برگردانده می‌شوند:

```json
{
  "message": "Successfully done.",
  "succeeded": true,
  "data": [],
  "resultCode": 100
}
```

در خطاها مقدار `succeeded` برابر `false` است و `resultCode` مقدار متناظر از `ApiResponse` را دارد.

## مدل‌های درخواست پیام

### `AradA2PMessage`

| فیلد | نوع | اجباری | توضیح |
| --- | --- | --- | --- |
| `sourceAddress` | string | بله | شماره فرستنده. |
| `destinationAddress` | string | بله | شماره گیرنده. |
| `messageText` | string | بله | متن پیام. |
| `dataCoding` | enum | خیر | نوع کدینگ. مقادیر رایج: `Default`/`0`، `ASCII`/`1`، `UCS2`/`8`. |
| `validityPeriod` | DateTime | خیر | زمان انقضای پیام. اگر ارسال نشود، مقدار پیش‌فرض سیستم استفاده می‌شود. |
| `targetUDH` | string[] | خیر | اطلاعات UDH مثل `port:{port}` و `referenceNumberType:8bit|16bit`. |
| `serviceID` | string | خیر | شناسه سرویس ارزش افزوده. |
| `priority` | enum | خیر | از `Lowest` برابر 0 تا `Highest` برابر 7. |
| `messageClass` | enum | خیر | مثل `TEXT`، `BINARY`، `FLASH` و غیره. |
| `udh` | string | خیر | تگ یا شناسه سمت کاربر برای گزارش‌گیری. |

### `AradBulkMessage`

| فیلد | نوع | اجباری | توضیح |
| --- | --- | --- | --- |
| `sourceAddress` | string | بله | شماره فرستنده. |
| `destinationAddress` | string[] | بله | لیست شماره گیرندگان. |
| `messageText` | string | بله | متن پیام. |
| `validityPeriod` | DateTime | خیر | زمان انقضای پیام. |
| `serviceID` | string | خیر | شناسه سرویس ارزش افزوده. |
| `servicePrice` | number | خیر | قیمت سرویس ارزش افزوده. |
| `targetUDH` | string[] | خیر | اطلاعات UDH اختیاری. |
| `udh` | string | خیر | تگ یا شناسه سمت کاربر. |

## متدهای پیام

در مسیرها، `{version}` یکی از `v1.0`، `v2.0`، `v3.0` یا `v4.0` است.

| متد | مسیر | اسکوپ | توضیح |
| --- | --- | --- | --- |
| `POST` | `/api/{version}/message/send` | `ApiAccess` | ارسال لیست پیام‌های A2P. |
| `GET` | `/api/{version}/message/send` | `ApiAccess` | ارسال از طریق Query String. گیرنده‌ها با کاما جدا می‌شوند. |
| `POST` | `/api/{version}/message/SendBySpecialParameter` | `ApiAccess` | ارسال بعد از تبدیل/یافتن گیرنده توسط سرویس استعلام یا پارامتر خاص. |
| `POST` | `/api/{version}/message/bulk` | `BulkApiAccess` | ارسال یک متن به چند گیرنده. |
| `POST` | `/api/{version}/message/P2PBulk` | `BulkApiAccess` | ارسال پیام‌های نظیر به نظیر؛ هر گیرنده می‌تواند متن متفاوت داشته باشد. |
| `GET` | `/api/{version}/message/CreateCampaign` | `BulkApiAccess` | ساخت کمپین با `campaignName`. |
| `GET` | `/api/{version}/message/SendToWhiteList` | `WhiteListAccess` | ارسال به دفترچه تلفن/لیست سفید. |
| `POST` | `/api/{version}/message/GetDLR` | DLR/MO policy | دریافت وضعیت تحویل بر اساس شناسه پیام. |
| `POST` | `/api/{version}/message/GetDLRByPartId` | DLR/MO policy | دریافت وضعیت تحویل بر اساس شناسه پارت. |
| `GET` | `/api/{version}/message/GetMO` | DLR/MO policy | دریافت حداکثر 1000 پیام ورودی خوانده‌نشده. |
| `GET` | `/api/{version}/message/GetMOByDate` | DLR/MO policy | دریافت پیام‌های ورودی در بازه زمانی. |

### ارسال پیام A2P

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
    "messageText": "متن تست",
    "targetUDH": ["port:5000", "referenceNumberType:16bit"],
    "udh": "customer-order-123"
  }
]
```

نمونه پاسخ موفق در `v4.0`:

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

### تفاوت خروجی نسخه‌های ارسال

| نسخه | قالب `data` |
| --- | --- |
| `v1.0` | آرایه‌ای از شناسه پیام‌ها. اگر `returnPartId=true` باشد، آرایه‌ای از `{ messageId, partIds }`. |
| `v2.0` | آرایه key/value که key شناسه پیام و value اطلاعات MCC/MNC یا اپراتور است. |
| `v3.0` | آرایه key/value که key شناسه پیام و value تعداد پارت پیام است. |
| `v4.0` | آرایه‌ای از `{ id, part, upstreamGateway }`. |

اگر در زمان ارسال `returnLongId=true` استفاده شود، شناسه‌ها با فرمت long برمی‌گردند. در زمان دریافت DLR نیز همان flag باید استفاده شود.

### ارسال Bulk

```http
POST /api/v4.0/message/bulk?returnLongId=false
Content-Type: application/json
```

```json
{
  "sourceAddress": "989000xxxx",
  "destinationAddress": ["98912xxxxxxx", "98913xxxxxxx"],
  "messageText": "متن پیام گروهی",
  "targetUDH": ["port:5000", "referenceNumberType:16bit"]
}
```

### ارسال P2P Bulk

```http
POST /api/v4.0/message/P2PBulk?returnLongId=false
Content-Type: application/json
```

```json
[
  {
    "sourceAddress": "989000xxxx",
    "destinationAddress": "98912xxxxxxx",
    "messageText": "پیام گیرنده اول"
  },
  {
    "sourceAddress": "989000xxxx",
    "destinationAddress": "98913xxxxxxx",
    "messageText": "پیام گیرنده دوم"
  }
]
```

### ارسال با GET

```http
GET /api/v4.0/message/send?sourceAddress=989000xxxx&destinationAddress=98912xxxxxxx,98913xxxxxxx&messageText=Hello&returnLongId=false
```

### ارسال به لیست سفید

```http
GET /api/v4.0/message/SendToWhiteList?sourceAddress=989000xxxx&phoneBookId={phoneBookId}&templateId={templateId}&messageText=&returnLongId=false&selectedMember=98912xxxxxxx
```

باید فقط یکی از `templateId` یا `messageText` ارسال شود. اگر دفترچه تلفن دارای template باشد، ارسال `templateId` الزامی است.

## دریافت دلیوری و پیام ورودی

### دریافت DLR

```http
POST /api/v4.0/message/GetDLR?returnLongId=false&returnUDH=false&returnSentDate=false
Content-Type: application/json
```

```json
[
  "668a5dac7f212cbf40fd50f2"
]
```

حداکثر تعداد شناسه در هر درخواست 1000 عدد است.

فیلدهای آیتم‌های پاسخ:

| فیلد | نوع | توضیح |
| --- | --- | --- |
| `id` | string | شناسه پیام. |
| `partStatus` | array | وضعیت پارت‌ها. در `GetDLR` مقادیر tuple شامل شماره پارت، وضعیت و تاریخ تحویل است. |
| `deliveryStatus` | enum | وضعیت کلی تحویل. |
| `deliveryDate` | DateTime? | زمان تحویل، در صورت وجود. |
| `createDate` | DateTime? | زمان ایجاد پیام. |
| `sentDate` | DateTime? | زمان ارسال، در صورت `returnSentDate=true`. |
| `udh` | string | در صورت `returnUDH=true`. |

### دریافت DLR بر اساس Part Id

```http
POST /api/v4.0/message/GetDLRByPartId?returnLongId=false&returnUDH=false&returnSentDate=false
Content-Type: application/json
```

```json
[
  "5575795555842650087"
]
```

### دریافت پیام‌های ورودی

```http
GET /api/v4.0/message/GetMO?returnId=false
```

حداکثر 1000 پیام ورودی خوانده‌نشده برای کاربر و IP معتبر برگردانده می‌شود.

در حالت `returnId=false` هر آیتم شامل فیلدهای زیر است:

```json
{
  "sourceAddress": "98912xxxxxxx",
  "destinationAddress": "989000xxxx",
  "messageText": "متن پاسخ",
  "receiveDateTime": "2026-06-29T08:10:00Z"
}
```

در حالت `returnId=true` فیلدهای `id` و `isRead` نیز برگردانده می‌شوند.

### دریافت پیام‌های ورودی در بازه زمانی

```http
GET /api/v4.0/message/GetMOByDate?startDateTime=2026-06-29T00:00:00Z&endDateTime=2026-06-29T23:59:59Z&returnId=true
```

## ارسال از طریق الگو

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

برای ارسال چند الگو:

```http
POST /api/patternMessage/sendMultiple?returnLongId=false
```

تعداد `parameters` باید با placeholderهای الگوی تاییدشده برابر باشد. `patternId` و دسترسی به شماره فرستنده با کاربر احراز هویت‌شده بررسی می‌شود.

## LiveData و مانیتورینگ

همه متدهای LiveData به اسکوپ `LiveDataAccess` نیاز دارند.

| متد | مسیر | توضیح |
| --- | --- | --- |
| `GET` | `/api/v1.0/LiveData/LiveData` | اطلاعات نمودار لحظه‌ای برای upstreamهای فعال. |
| `GET` | `/api/v1.0/LiveData/Last` | آخرین داده موجود، با جست‌وجو تا 10 دقیقه قبل. |
| `GET` | `/api/v1.0/LiveData/Day?date=yyyy/MM/dd` | داده روزانه گروه‌بندی‌شده بر اساس ساعت. |
| `GET` | `/api/v1.0/LiveData/UpStreamInfo` | وضعیت اتصال و ping سرویس‌دهنده‌ها. |
| `GET` | `/api/v1.0/LiveData/UsersInfo?userName={userName}` | اطلاعات کاربر، اعتبار و تعرفه زیرکاربران. |
| `GET` | `/api/v1.0/LiveData/GetUserConnectivityStatus` | وضعیت اتصال کاربران. |
| `GET` | `/api/v1.0/LiveData/MonitoringAZ?fullDateTime=false` | نمای کلی مانیتورینگ. |
| `GET` | `/api/v1.0/LiveData/MonitoringUpstream?fullDateTime=false` | نمای مانیتورینگ upstream. |
| `GET` | `/api/v1.0/LiveData/MonitoringQueue?fullDateTime=false` | نمای مانیتورینگ صف‌ها. |
| `GET` | `/api/v1.0/LiveData/MonitoringInputTps?fullDateTime=false` | نمای TPS ورودی. |
| `GET` | `/api/v1.0/LiveData/MonitoringTenHighTrafficUsers?fullDateTime=false` | ده کاربر پرترافیک. |
| `GET` | `/api/v1.0/LiveData/MonitoringPercentageUser?fullDateTime=false` | سهم درصدی ترافیک کاربران. |

خروجی `LiveData` و `Day` در فیلد `data` به شکل زیر است:

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

مقادیر رایج `type`: `Blocked`، `Received`، `Rejected`، `Undeliverable`، `Sent`، `NotSent`، `Errors` و `Delivered`.

## کاربر و ابزارها

| متد | مسیر | اسکوپ | توضیح |
| --- | --- | --- | --- |
| `GET` | `/api/user/UserInfo` | Get-data policy | اطلاعات حساب، شماره‌های فرستنده، اعتبار، نوع پرداخت و MPS. |
| `GET` | `/api/user/GetMapTraffics?username={username}` | Get-data policy | لیست Map Trafficها. |
| `POST` | `/api/user/AddMapTraffics` | Get-data policy | افزودن Map Traffic. |
| `PUT` | `/api/user/ModifyMapTraffics?id={id}` | Get-data policy | فعال/غیرفعال کردن Map Traffic. |
| `GET` | `/api/Tools/CheckIp` | Get-data policy | بررسی IP درخواست‌دهنده نسبت به IPهای مجاز. |
| `GET` | `/api/Tools/Ping` | عمومی | مقدار `PONG` را برمی‌گرداند. |

## Webengage

```http
POST /api/webengage/SendSMS
Authorization: Bearer {BulkApiAccess token}
Content-Type: application/json
```

این endpoint وضعیت‌های ارسال پلتفرم را به پاسخ‌های سازگار با Webengage مثل `sms_accepted` یا `sms_failed` تبدیل می‌کند.

## سایر گروه‌های endpoint

گروه‌های زیر در API فعلی وجود دارند و جزئیات کامل آن‌ها در Swagger قابل مشاهده است:

| گروه | مسیر پایه | اسکوپ |
| --- | --- | --- |
| OTP | `/api/v1.0/Otp` | `OtpAccess` |
| KYC | `/api/v1.0/Kyc` | `KycAccess` |
| Smart SMS | `/api/v1.0/SmartSms` | `SmartSmsAccess` |
| Phonebook | `/api/v1.0/PhoneBook` | `MessageRequestAccess` |
| Schedule | `/api/v1.0/Schedule` | `MessageRequestAccess` |
| Report | `/api/v1.0/Report` | `LiveDataAccess` |
| Ticket | `/api/v1.0/Ticket` | `SupportTicketAccess` |

## کدهای وضعیت تحویل

| کد | نام |
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

## کدهای بازگشتی API

| کد | نام |
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

## کدهای خطای ارسال

| کد | نام |
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

## نکات تکمیلی

- تاریخ‌های خروجی MO/DLR عموماً UTC هستند، مگر اینکه endpoint خاصی آن‌ها را به زمان محلی تبدیل کند.
- `GetMO` و `GetMOByDate` حداکثر 1000 رکورد برمی‌گردانند.
- `GetDLR` و `GetDLRByPartId` حداکثر 1000 شناسه در هر درخواست می‌پذیرند.
- تفاوت نسخه‌های API ارسال بیشتر در metadata خروجی است و فیلدهای اصلی ورودی ارسال تغییر نکرده‌اند.
