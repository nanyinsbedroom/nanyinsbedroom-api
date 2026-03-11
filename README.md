# Tower of Fantasy/幻塔 — API Reference

> **Disclaimer:** This documentation is based on observed HTTP network traffic in the Tower of Fantasy. It is unofficial and not endorsed by Perfect World Games or Hotta Studio. Endpoints, parameters, and response shapes may change without notice.

---

## Table of Contents

1. [Overview](#overview)
2. [Regions & Versions](#regions--versions)
3. [Base URLs](#base-urls)
4. [Common Authentication Parameters](#common-authentication-parameters)
   - [deviceInfo Object](#deviceinfo-object)
   - [OS Type Values](#os-type-values)
   - [User ID Formats](#user-id-formats)
5. [Shop API](#shop-api)
   - [GET /c/api/order/webProductList](#get-capiorderwebproductlist)
     - [Request Parameters](#request-parameters)
     - [Response Schema](#response-schema)
     - [Product Catalogue](#product-catalogue)
6. [Appearance API](#appearance-api)
   - [Regional Endpoint Comparison](#regional-endpoint-comparison)
   - [Common Appearance Request Parameters](#common-appearance-request-parameters)
   - [GET /face/pageNewest](#get-facepagenewest)
   - [GET /face/pageHottest](#get-facepagehottest)
   - [GET /face/listRec](#get-facelistrec)
   - [GET /face/pageMine](#get-facepagemine)
   - [GET /face/pageMyCollections](#get-facepagmycollections)
   - [Appearance Response Schema](#appearance-response-schema)
     - [Paginated Response Wrapper](#paginated-response-wrapper)
     - [FaceEntry Object](#faceentry-object)
     - [ImageItem Object](#imageitem-object)
   - [Character Customization Data (`data` field)](#character-customization-data-data-field)
     - [Top-Level Fields](#top-level-fields)
     - [Hair Fields](#hair-fields)
     - [Facial Feature Index Fields](#facial-feature-index-fields)
     - [Eye Color Fields](#eye-color-fields)
     - [Body Proportion Fields](#body-proportion-fields)
     - [Face Mark Fields](#face-mark-fields)
     - [Outfit Fields](#outfit-fields)
     - [Headwear Fields](#headwear-fields)
     - [PlayerMorphData Format](#playermorphdata-format)
     - [HeadBonesData Format](#headbonesdata-format)
     - [Extended Fields (v5.6.6+ / v5.7.8+)](#extended-fields-v566--v578)
7. [Data Types & Conventions](#data-types--conventions)
   - [RGBA Color Format](#rgba-color-format)
   - [Timestamps](#timestamps)
   - [CDN Hosts & Image URLs](#cdn-hosts--image-urls)
8. [Response Codes](#response-codes)
9. [Known App IDs](#known-app-ids)
10. [Changelog / Version Notes](#changelog--version-notes)

---

## Overview

Tower of Fantasy operates two separate backend infrastructures for its **Oversea (OS)** and **China (CN)** server regions. The APIs share the same endpoint paths and data schemas but differ in base URLs, app identifiers, client packages, user ID formats, and database scale.

| Service | Purpose |
|---|---|
| **Shop API** (`mapi`) | Fetch real-money product listings and pricing (Oversea only; CN equivalent not captured) |
| **Appearance API** (`face` / `htface`) | Browse, upload, share, and save character appearance presets |

Both services use HTTPS **GET** requests with **URL-encoded query parameters**. Authentication uses a combination of session tokens, HMAC/MD5 request signatures, and device fingerprints.

---

## Regions & Versions

| | Oversea (OS) | China (CN) |
|---|---|---|
| **Publisher** | Perfect World Games | Perfect World Games |
| **Client package** | `com.levelinfinite.hotta.win` | `com.hottagames.qrsl` |
| **Client version** | `5.6.6` | `5.7.8` |
| **`oneAppId`** | `1000129` | `1256` |
| **`mediaId`** | `11` | `5966` |
| **`platform`** | `4` | `9` |
| **Appearance API host** | `face.perfectworldgames.com` | `htface.laohu.com` |
| **Shop API host** | `mapi.perfectworldgames.com` | *(not captured)* |
| **Appearance image CDN** | `htface-kr-1305865668.file.myqcloud.com` | `htface-1251008858.file.myqcloud.com` |
| **Legacy face CDN** | `htface-tower.wmupd.com` | — |
| **Appearance DB size** | 908,790 total entries | 8,613,035 total entries |
| **Earliest entries** | Aug 2022 | Nov 2021 |
| **Top hottest score** | 51,127 | 353,836 |

> The CN server launched ~8 months before Oversea and has an ~9.5× larger face database. Both regions share identical endpoint paths and `data` field schemas.

---

## Base URLs

| Region | Service | Base URL |
|---|---|---|
| Oversea (OS) | Shop | `https://mapi.perfectworldgames.com` |
| Oversea (OS) | Appearance | `https://face.perfectworldgames.com` |
| China (CN) | Appearance | `https://htface.laohu.com` |
| China (CN) | Shop | *(not captured)* |

---

## Common Authentication Parameters

These parameters appear across most or all endpoints in both regions:

| Parameter | Type | Description |
|---|---|---|
| `uid` / `userId` | string | User account identifier (see [User ID Formats](#user-id-formats)) |
| `roleId` | string | In-game character/role ID |
| `token` | string | Session bearer token (UUID v4); required for Oversea shop endpoints only |
| `sign` | string | Request signature — MD5 for shop endpoints, Base64 HMAC for face endpoints |
| `timestamp` | integer | Unix timestamp in **seconds** |
| `oneAppId` | string | Platform app identifier (`1000129` for Oversea, `1256` for CN) |
| `isSandbox` | integer | `0` = live production, `1` = sandbox |
| `ndid` | string | Device fingerprint string |
| `deviceInfo` | string | URL-encoded JSON device metadata object |
| `os` | integer | OS type code |

---

### `deviceInfo` Object

Sent URL-encoded as a JSON string with every face API request.

**Oversea (OS) example:**
```json
{
  "deviceLanguage": "English",
  "mediaId": "11",
  "ndid": "YOUR_NDID",
  "oneAppId": "1000129",
  "oneAppVersion": "5.6.6",
  "osType": "4",
  "packageName": "com.levelinfinite.hotta.win",
  "phoneSystemVersion": "10.0.19045",
  "platform": "4",
  "sdkVersion": "1.0.1.1"
}
```

**China (CN) example:**
```json
{
  "deviceLanguage": "English",
  "mediaId": "5966",
  "ndid": "YOUR_NDID",
  "oneAppId": "1256",
  "oneAppVersion": "5.7.8",
  "osType": "4",
  "packageName": "com.hottagames.qrsl",
  "phoneSystemVersion": "10.0.19045",
  "platform": "9",
  "sdkVersion": "1.0.1.1"
}
```

| Field | Type | Notes |
|---|---|---|
| `deviceLanguage` | string | Display language |
| `mediaId` | string | Distribution channel ID. `"11"` = Oversea PC, `"5966"` = CN PC |
| `ndid` | string | Device fingerprint. CN `ndid` ends with a trailing underscore and has no second segment |
| `oneAppId` | string | App ID as string |
| `oneAppVersion` | string | Game client version |
| `osType` | string | OS code as string (matches top-level `os` param) |
| `packageName` | string | APK/EXE package name |
| `phoneSystemVersion` | string | OS build string |
| `platform` | string | Distribution platform code. `"4"` = Oversea PC, `"9"` = CN PC |
| `sdkVersion` | string | PW SDK version |

---

### OS Type Values

| Value | Platform |
|---|---|
| `2` | Android |
| `3` | iOS |
| `4` | Windows PC |
| `15` | Unknown (observed for `oneAppId` `1000138` entries in `listRec`) |

---

### User ID Formats

User IDs differ significantly between regions due to different account systems.

**Oversea (OS):** Simple numeric strings.
```
60833201
78964579
1001601533
```
On PC, `userId` and `roleId` are equal. Mobile entries have empty string `""` or `null` for `roleId`.

**China (CN):** Prefixed `<channelId>_<accountId>` format.

| Prefix | Account Type | Example |
|---|---|---|
| `9_` | QQ account | `9_143523314` |
| `8_` | WeChat account | `8_388405300`, `8_g984844123189584586` |
| `2_` | Phone/other | `2_2881786190000000000115924024` |
| `38_` | Third-party | `38_be5ba0a9f7ec2fcf` |
| `23_` | Additional channel | `23_864149437` |
| `6_` | Another channel | `6_2022010903243817` |
| `57_` | Another channel | `57_oVKVr6RsSKahiYjJOgPVG2t2WEhs` |

On CN, `roleId` is a long numeric character ID (e.g. `60563335266353`, `146522809307366`). On CN PC, `roleId` matches `userId` (e.g. `9_123702014`). `serverId` is always populated for mobile CN entries (e.g. `"14102"`, `"14502"`, `"34115"`).

---

## Shop API

The shop API has only been observed on Oversea infrastructure. CN equivalent not captured.

### GET `/c/api/order/webProductList`

**Full URL:** `https://mapi.perfectworldgames.com/c/api/order/webProductList`

Fetches real-time pricing for one or more in-game purchasable products. The client sends multiple batched requests to look up all products, grouping 4–6 product IDs per call.

---

#### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `appId` | string | ✅ | Always `1000129` for Oversea |
| `productList` | string | ✅ | URL-encoded JSON array: `[{"productId":"..."},...]` |
| `sign` | string | ✅ | MD5 signature of the canonical parameter string |
| `timestamp` | integer | ✅ | Unix seconds |
| `token` | string | ✅ | Session bearer token (UUID v4) obtained after login |
| `uid` | string | ✅ | Authenticated user account ID |

**Three observed requests (decoded):**

Request 1 — Subscriptions & packs:
```
productList=[
  {"productId":"com.proximabeta.tof.monthlycard30"},
  {"productId":"com.proximabeta.tof.bp68"},
  {"productId":"com.proximabeta.tof.bp128"},
  {"productId":"com.proximabeta.tof.giftpack601"},
  {"productId":"com.proximabeta.tof.giftpack3001"}
]
timestamp=1744649245, uid=60623201
```

Request 2 — Tanium tiers:
```
productList=[
  {"productId":"com.proximabeta.tof.diamond1"},
  {"productId":"com.proximabeta.tof.diamond2"},
  {"productId":"com.proximabeta.tof.diamond3"},
  {"productId":"com.proximabeta.tof.diamond4"},
  {"productId":"com.proximabeta.tof.diamond5"},
  {"productId":"com.proximabeta.tof.diamond6"}
]
timestamp=1744649245, uid=60623201
```

Request 3 — Gift packs:
```
productList=[
  {"productId":"com.proximabeta.tof.1pack2"},
  {"productId":"com.proximabeta.tof.20pack1"},
  {"productId":"com.proximabeta.tof.10pack1"},
  {"productId":"com.proximabeta.tof.15pack1"},
  {"productId":"com.proximabeta.tof.12pack2"},
  {"productId":"com.proximabeta.tof.5pack2"}
]
timestamp=1744649245, uid=60623201
```

---

#### Response Schema

```json
{
  "code": 0,
  "message": "OK",
  "messageType": "ALERT",
  "traceId": "c978b47904fa4d36b81211a9c7c4dcd0",
  "result": {
    "productList": [
      {
        "productId": "com.proximabeta.tof.diamond1",
        "amount": 099,
        "currency": "USD",
        "title": "60 Tanium"
      }
    ]
  }
}
```

| Field | Type | Description |
|---|---|---|
| `code` | integer | `0` = success |
| `message` | string | `"OK"` on success |
| `messageType` | string | `"ALERT"` observed in all responses |
| `traceId` | string | Server-side trace ID for debugging |
| `result.productList` | array | Resolved product pricing objects |
| `result.productList[].productId` | string | The queried product ID |
| `result.productList[].amount` | integer | Price in smallest local currency unit |
| `result.productList[].currency` | string | ISO 4217 currency code |
| `result.productList[].title` | string | Localised display name |

---

#### Product Catalogue

**Tanium (Premium Currency)**

| Product ID | Title | Approx. USD |
|---|---|---|
| `com.proximabeta.tof.diamond1` | 60 Tanium | ~$0.99 |
| `com.proximabeta.tof.diamond2` | 310 Tanium | ~$4.99 |
| `com.proximabeta.tof.diamond3` | 1020 Tanium | ~$14.99 |
| `com.proximabeta.tof.diamond4` | 2080 Tanium | ~$29.99 |
| `com.proximabeta.tof.diamond5` | 3480 Tanium | ~$49.99 |
| `com.proximabeta.tof.diamond6` | 6980 Tanium | ~$99.99 |

**Subscriptions & Battle Pass**

| Product ID | Title | Approx. USD |
|---|---|---|
| `com.proximabeta.tof.monthlycard30` | Monthly Pass | ~$4.99 |
| `com.proximabeta.tof.bp68` | Pass | ~$9.99 |
| `com.proximabeta.tof.bp128` | Collector Edition Pass | ~$19.99 |

**Starter & Content Packs**

| Product ID | Title | Approx. USD |
|---|---|---|
| `com.proximabeta.tof.giftpack601` | Rookie Supplies | ~$0.99 |
| `com.proximabeta.tof.giftpack3001` | Adventure Pack | ~$4.99 |

**Gift Packs**

| Product ID | Title | Approx. USD |
|---|---|---|
| `com.proximabeta.tof.1pack2` | 0.99 USD gift pack2 | ~$0.99 |
| `com.proximabeta.tof.12pack2` | 1.99 USD gift pack | ~$1.99 |
| `com.proximabeta.tof.5pack2` | 4.99 USD gift pack | ~$4.99 |
| `com.proximabeta.tof.10pack1` | 9.99 USD gift pack | ~$9.99 |
| `com.proximabeta.tof.15pack1` | 14.99 USD gift pack | ~$14.99 |
| `com.proximabeta.tof.20pack1` | 19.99 USD gift pack | ~$19.99 |

---

## Appearance API

The Appearance API powers the in-game character appearance sharing system. Both regions expose the same five endpoint paths under their respective base URLs.

---

### Regional Endpoint Comparison

| Endpoint | Oversea (OS) Full URL | China (CN) Full URL |
|---|---|---|
| Latest | `https://face.perfectworldgames.com/face/pageNewest` | `https://htface.laohu.com/face/pageNewest` |
| Hottest | `https://face.perfectworldgames.com/face/pageHottest` | `https://htface.laohu.com/face/pageHottest` |
| Recommended | `https://face.perfectworldgames.com/face/listRec` | `https://htface.laohu.com/face/listRec` |
| My Uploads | `https://face.perfectworldgames.com/face/pageMine` | `https://htface.laohu.com/face/pageMine` |
| My Collections | `https://face.perfectworldgames.com/face/pageMyCollections` | `https://htface.laohu.com/face/pageMyCollections` |

---

### Common Appearance Request Parameters

| Parameter | Type | Required | Oversea Example | CN Example |
|---|---|---|---|---|
| `oneAppId` | string | ✅ | `1000129` | `1256` |
| `os` | integer | ✅ | `4` | `4` |
| `userId` | string | ✅ | `60833201` | `9_143523314` |
| `roleId` | string | ✅ | `60833201` | `9_143523314` |
| `sign` | string | ✅ | Base64 HMAC | Base64 HMAC |
| `timestamp` | integer | ✅ | `1772660345` | `1772707239` |
| `isSandbox` | integer | ✅ | `0` | `0` |
| `ndid` | string | ✅ | `YOUR_NDID` | `YOUR_NDID` *(trailing underscore, no suffix)* |
| `deviceInfo` | string | ✅ | URL-encoded JSON | URL-encoded JSON |

Paginated endpoints additionally require `pageNum` (1-indexed integer) and `pageSize` (integer, `12` observed for browse, `6` for mine/collections).

---

### GET `/face/pageNewest`

Returns the most recently published appearances sorted by `createTime` descending.

**Observed `total`:** 908,790 (Oversea) · 8,613,035 (CN)

**Oversea example request:**
```
GET https://face.perfectworldgames.com/face/pageNewest
  ?deviceInfo=<url-encoded-json>
  &isSandbox=0
  &ndid=YOUR_NDID
  &oneAppId=1000129&os=4&pageNum=1&pageSize=12
  &roleId=60833201
  &sign=YOUR_SIGN
  &timestamp=1744649245&userId=60623201
```

**CN example request:**
```
GET https://htface.laohu.com/face/pageNewest
  ?deviceInfo=<url-encoded-json>
  &isSandbox=0
  &ndid=YOUR_NDID
  &oneAppId=1256&os=4&pageNum=1&pageSize=12
  &roleId=9_143523314
  &sign=YOUR_SIGN
  &timestamp=1744649245&userId=9_143523314
```

**Behavior notes:**
- `data` field is `null` for all 12 entries in both observed Oversea and CN responses. Character customization data is not returned here.
- Mobile Android entries (os=2) use timestamp-suffixed filenames in image URLs; PC entries (os=4) and iOS entries (os=3) use hash-only filenames.
- CN entries always populate `serverId` and `roleId` for mobile users; PC CN entries set `roleId` equal to `userId` and `serverId` to `null`.
- Some `createTime` values have sub-second fractional millisecond precision (e.g. `1772632776492`, `1772628078895`).

---

### GET `/face/pageHottest`

Returns appearances ranked by all-time popularity score (`likeCount + collectCount`) descending.

Same parameters as `/face/pageNewest`.

**Observed score ranges:**

| Region | Rank 1 Score | Rank 1 likeCount | Rank 1 collectCount | `total` |
|---|---|---|---|---|
| Oversea | 51,127 | 19,023 | 32,104 | 908,790 |
| CN | 353,836 | 106,971 | 246,865 | 8,613,035 |

**Notable observations:**
- The Oversea all-time #1 entry (`faceId` 100001259) was uploaded on 2022-08-09 (the first day of Oversea open beta), uses the legacy `htface-tower.wmupd.com` CDN, and has `data: null`.
- The CN all-time #1 entry (`faceId` 161196) was uploaded on 2021-12-03, the first week of CN beta.
- Hottest page returns `data: null` for all entries — only `listRec` consistently returns populated `data`.

---

### GET `/face/listRec`

Returns a curated/recommended list of appearances. Does **not** accept `pageNum` or `pageSize`. Returns a **flat JSON array** in `result` rather than a paginated wrapper object.

**Response structure:**
```json
{
  "traceid": "c27108f5f9201eff",
  "code": 0,
  "message": "",
  "result": [ /* array of FaceEntry objects */ ]
}
```

**Key behavioral differences from paginated endpoints:**
- All entries in both observed Oversea and CN responses have fully populated `data` fields with complete character customization JSON.
- Oversea `listRec` returns entries from **two different `oneAppId` values**: `1000129` (TOF Oversea, `faceId` in 100M+ range) and `1000138` (unknown alternate build, `faceId` in 4M range with `os: 15`). This indicates cross-build curation.
- CN `listRec` returns only `oneAppId: 1256` entries.
- Oversea recommended entries span dates from Aug 2022 (`faceId` 105089539) through Jan 2026 (`faceId` 142516427).
- CN recommended entries span dates from Dec 2021 (`faceId` 11196400) through Aug 2025 (`faceId` 13002248).

**`oneAppId` 1000138 entries (seen in Oversea `listRec`):**
- `faceId` values in the 4M–5M range
- `os: 15` (unknown platform)
- `userId` format: simple numeric strings similar to Oversea
- `ext: null` on all observed entries
- `data` field contains full character JSON with OS morph set (Oversea morph targets)
- `roleId` and `serverId`: empty strings `""`

---

### GET `/face/pageMine`

Returns appearances uploaded by the authenticated user. Returns an empty `data` array if none.

**Observed response (both regions, new account):**
```json
{
  "traceid": "83ed2494bb86b4cd",
  "code": 0,
  "message": "",
  "result": { "pageNum": 1, "pageSize": 6, "total": 0, "data": [] }
}
```

---

### GET `/face/pageMyCollections`

Returns appearances saved/bookmarked by the authenticated user. Returns an empty `data` array if none.

**Observed response (both regions, new account):**
```json
{
  "traceid": "6008f33e35b10af7",
  "code": 0,
  "message": "",
  "result": { "pageNum": 1, "pageSize": 6, "total": 0, "data": [] }
}
```

---

### Appearance Response Schema

#### Paginated Response Wrapper

Used by `pageNewest`, `pageHottest`, `pageMine`, `pageMyCollections`:

```json
{
  "traceid": "ca804399b74a1ed1",
  "code": 0,
  "message": "",
  "result": {
    "pageNum": 1,
    "pageSize": 12,
    "total": 908790,
    "data": [ /* FaceEntry[] */ ]
  }
}
```

| Field | Type | Description |
|---|---|---|
| `traceid` | string | Server-side request trace ID |
| `code` | integer | `0` = success |
| `message` | string | Empty string on success |
| `result.pageNum` | integer | Current page (1-indexed) |
| `result.pageSize` | integer | Requested page size |
| `result.total` | integer | Total records in the dataset |
| `result.data` | FaceEntry[] | Entries for this page |

---

#### FaceEntry Object

Full example (OS, `listRec`, modern entry with extended fields):
```json
{
  "faceId": 142516427,
  "oneAppId": 1000129,
  "userId": "60895804",
  "roleId": "",
  "serverId": "",
  "os": 3,
  "imgItems": [ /* 3 ImageItem objects */ ],
  "data": "{ ... }",
  "gender": 2,
  "ext": "suzuchan76,Stella,47343427153834",
  "createTime": 1769698760000,
  "tagItems": null,
  "likeCount": 1,
  "collectCount": 1,
  "score": 2,
  "isLiked": 0,
  "isCollection": 0
}
```

Full example (CN, `listRec`, early 2022 entry):
```json
{
  "faceId": 11679581,
  "oneAppId": 1256,
  "userId": "8_388405300",
  "roleId": "103955388516387",
  "serverId": "24204",
  "os": 2,
  "imgItems": [ /* 3 ImageItem objects */ ],
  "data": "{ ... }",
  "gender": 2,
  "ext": null,
  "createTime": 1642668344000,
  "tagItems": null,
  "likeCount": 0,
  "collectCount": 0,
  "score": 0,
  "isLiked": 0,
  "isCollection": 0
}
```

| Field | Type | Nullable | Description |
|---|---|---|---|
| `faceId` | integer | No | Unique appearance ID, globally incremented per region |
| `oneAppId` | integer | No | App ID of origin region. `1000129` = Oversea, `1000138` = alternate build, `1256` = CN |
| `userId` | string | No | Account ID of the creator |
| `roleId` | string | Yes | Character/role ID. PC Oversea: equals `userId`. Mobile Oversea: empty string `""` or `null`. CN mobile: long numeric ID. CN PC: equals `userId`. `""` for `oneAppId` 1000138 entries |
| `serverId` | string | Yes | Game server ID. Always populated for CN mobile; `null` or empty for Oversea PC; `""` for 1000138 entries |
| `os` | integer | No | Creator platform (see [OS Type Values](#os-type-values)) |
| `imgItems` | ImageItem[] | No | Exactly 3 preview images |
| `data` | string/null | Yes | JSON-encoded character customization. Always `null` in `pageNewest`/`pageHottest`. Always populated in `listRec` |
| `gender` | integer | No | `1` = male, `2` = female |
| `ext` | string/null | Yes | Comma-separated metadata. See format below |
| `createTime` | integer | No | Upload time in **milliseconds** since Unix epoch. May have sub-second precision (e.g. `1772632776492`) |
| `tagItems` | null | Yes | Always `null` in all observed responses |
| `likeCount` | integer | No | Number of likes |
| `collectCount` | integer | No | Number of times saved/bookmarked |
| `score` | integer | No | Always equals `likeCount + collectCount` |
| `isLiked` | integer | No | `1` if authenticated user has liked this, else `0` |
| `isCollection` | integer | No | `1` if authenticated user has saved this, else `0` |

**`ext` Field Format:**

When present, a comma-delimited string:
```
"<characterName>,<serverName>,<internalCharacterId>"
```

| Observed Value | Region | Notes |
|---|---|---|
| `"Akumatica,WS-Europe-01,38174111708527166"` | OS | Full 3-part format |
| `"Shiemy,Blumous"` | OS | 2-part only (no character ID) |
| `"Donplax,Tempest"` | OS | 2-part only |
| `""` | OS/CN | Empty string |
| `null` | OS/CN | Missing entirely |
| `"é›¨æ²æ©™,æ˜Ÿå²›HT-02,60563335266353"` | CN | Chinese character name + CN server |
| `"mohun49,å…‹ç½—æ©HT-04"` | CN | Mixed-language |

CN server names follow the pattern `<区域名>HT-<编号>`, e.g. `星岛HT-01`, `海嘉德HT-02`, `悯雨岛HT-15`, `宇宙折跃`, `白月破晴`, `幽岩`.

---

#### ImageItem Object

Every `FaceEntry` contains exactly **3 images** in `imgItems`:

| Index | Dimensions | Description |
|---|---|---|
| `0` | 480×480 | Square portrait/avatar thumbnail |
| `1` | 1280×720 or 1152×648 | Widescreen screenshot, angle 1 |
| `2` | 1280×720 or 1152×648 | Widescreen screenshot, angle 2 |

Screenshot resolution by platform:
- **iOS (os=3) or PC (os=4):** 1280×720
- **Android (os=2):** 1152×648

> **Exception:** Some CN Android entries (`os=2`) from 2025 were observed at 1280×720, suggesting some Android devices can output full resolution.

| Field | Type | Description |
|---|---|---|
| `url` | string | Absolute HTTPS CDN URL |
| `width` | integer | Pixel width |
| `height` | integer | Pixel height |

**Image URL patterns:**

PC/iOS with hash filename:
```
https://<cdn>/img/face/<YYYYMMDD>/<userId>/<md5hash>.png
```

Android with timestamp-suffixed filename (no extension):
```
https://<cdn>/img/face/<YYYYMMDD>/<userId>/<md5hash><unixMs>
```
or (older Android):
```
https://<cdn>/img/face/<YYYYMMDD>/<userId>/<unixMs>
```

Legacy Oversea PC (pre-late 2022, `htface-tower.wmupd.com`):
```
https://htface-tower.wmupd.com/fantasy/face/<YYYYMMDD>/<internalNumericId>/<md5hash>.png
```

Note: On the legacy CDN, the path segment after the date is a long numeric string (e.g. `13509374856046355118`) rather than a `userId`. This appears to be an internal server-generated ID.

---

### Character Customization Data (`data` field)

When present, `data` is a **JSON-encoded string** requiring a second `JSON.parse()` call. It contains the complete character appearance definition importable into the game's character creator.

**Encoding by era and platform:**

| Era | Platform | Encoding |
|---|---|---|
| Post-2023 (Oversea/CN) | PC | Standard JSON with `\n\t` indentation |
| Early 2022 (CN) | PC | Standard JSON with `\r\n\t` indentation |
| Aug–Dec 2022 (Oversea) | PC | `rn\t` literal instead of `\r\n\t` (the string `rn` appears verbatim) |
| 2022 (Oversea/CN) | Android | `n\t` literal instead of `\n\t` |

Parsers should normalize line endings before parsing early entries.

---

#### Top-Level Fields

| Field | Type | Description |
|---|---|---|
| `RoleIndex` | integer | Base character model. `1` = female (Nan Yin type), `2` = male |
| `RoleCharacterType` | integer | Mirrors `RoleIndex` in all observed data |

---

#### Hair Fields

| Field | Type | Nullable | Description |
|---|---|---|---|
| `HairIndex` | integer | No | Main top/back hair style index |
| `ForeHairIndex` | integer | No | Fringe/front hair piece index |
| `CenterHairIndex` | integer | No | Center parting hair piece index |
| `BackHairIndex` | integer | No | Rear hair piece index |
| `UsePartHair` | integer | No | `1` = multi-part hair assembly, `0` = single mesh |
| `HatIndex` | integer | No | Hat/headgear index. `0` = none. Non-zero values (e.g. `1`, `5`, `9`, `11`, `52`, `80`) observed |
| `HairColor` | RGBA | No | Primary hair base color |
| `HairColor2` | RGBA | No | Secondary/gradient hair color |
| `HairColor3` | RGBA | No | Shimmer/highlight tint. Default value `(R=0.958333,G=0.875690,B=0.654723,A=1.000000)` seen universally |
| `HairSpecColor` | RGBA | No | Hair specular/shine color |
| `HairBlendScale` | float | No | Blend weight between `HairColor` and `HairColor2`. Range `0.0`–`1.0` |
| `HairHighlightColorData` | string | Yes | *(v5.6.6+/v5.7.8+ only)* Hair highlight config. Empty string `""` in all observed entries |

---

#### Facial Feature Index Fields

| Field | Type | Description |
|---|---|---|
| `EyelashIndex` | integer | Eyelash style preset index |
| `SkinIndex` | integer | Skin tone/texture preset index. Range observed: `0`–`21` (Oversea), `2`–`19` (CN) |
| `NoseIndex` | integer | Nose shape preset. `0` = default in all observed entries |
| `BrowIndex` | integer | Eyebrow style index |
| `EyeIndex` | integer | Eye shape/liner style index |
| `EyeballIndex` | integer | Left eye iris/pupil texture index |
| `RightEyeballIndex` | integer | Right eye iris/pupil texture index |
| `UseEyeTwo` | integer | `1` = heterochromia (left/right eyes use separate color sets), `0` = both eyes use same colors |

---

#### Eye Color Fields

Each eye has four composited color layers. When `UseEyeTwo = 0`, all `RightEye*` fields are identical to their `Eye*` counterparts. When `UseEyeTwo = 1`, they are independently set.

| Field | Description |
|---|---|
| `EyeColor` | Left eye — primary iris color |
| `EyeColor2` | Left eye — secondary iris ring color |
| `EyeColor3` | Left eye — sclera/white tint |
| `EyeColor4` | Left eye — highlight/gloss. `(R=1,G=1,B=1,A=1)` = white shine; `(R=0,G=0,B=0,A=0)` = disabled |
| `RightEyeColor` | Right eye — primary iris color |
| `RightEyeColor2` | Right eye — secondary iris ring |
| `RightEyeColor3` | Right eye — sclera tint |
| `RightEyeColor4` | Right eye — highlight/gloss |
| `EyebrowColor` | Eyebrow fill color |

All values are RGBA float strings (see [RGBA Color Format](#rgba-color-format)).

---

#### Body Proportion Fields

All floats in range `[-1.0, 1.0]`.

| Field | Description |
|---|---|
| `BodyValue` | Body weight/width. `-1` = very slim, `1` = full |
| `NeckValue` | Neck length/width |
| `ChestValue` | Chest size |
| `ChestOpenValue` | Chest neckline opening |
| `HeadValue` | Head size. `-1` = small, `1` = large |
| `LegValue` | Leg proportion |

---

#### Face Mark Fields

| Field | Type | Description |
|---|---|---|
| `FaceIndex` | integer | Face mark/blush/tattoo preset. `0` = none. Range observed: `0`–`13` |
| `FaceColor` | RGBA | Face mark color tint |
| `FaceMarkOffset` | RGBA | Repurposed: `(R=x_offset, G=y_offset, B=reserved, A=opacity)` for positioning |
| `FaceMirror` | integer | `1` = mirror symmetrically, `0` = single side |

> In entries with `FaceprintData` (v5.6.6+), the legacy `FaceIndex`/`FaceColor`/`FaceMarkOffset` fields are absent from the JSON entirely.

---

#### Outfit Fields

| Field | Type | Description |
|---|---|---|
| `DressFashionId` | string | Outfit preset ID. Examples: `"fashion_dress_2"`, `"fashion_dress_10"`, `"fashion_dress_61"`. Range observed: `2`–`61` (Oversea), `2`–`43` (CN 2022) |
| `DressFashionColor` | string | Dash-separated RGBA dye channel values. Empty string `""` observed in some early CN entries |

**`DressFashionColor` format:**
```
"(R=...,G=...,B=...,A=...)-(R=...,G=...,B=...,A=...)-(R=...,G=...,B=...,A=...)"
```

- 1–4 dash-separated RGBA segments depending on how many dye channels the outfit supports. Outfits with 3 channels (e.g. `fashion_dress_36`, `fashion_dress_61`) have 3 segments; most outfits have 4.
- Values are HDR linear floats — values above `1.0` produce emissive/bloom effects (e.g. `R=2.000000` = maximum brightness).
- Empty string `""` (observed in early CN 2022 entries) means default colors are used.

---

#### Headwear Fields

| Field | Type | Description |
|---|---|---|
| `HeadwearFashionId` | string | Headwear/accessory preset ID. `"fashion_decoration_0"` = none. Non-zero examples: `"fashion_decoration_33"`, `"fashion_decoration_99_4"` |
| `HeadwearFashionOffset` | string | Position offset `"X=<f> Y=<f> Z=<f>"`. `"X=0.000 Y=0.000 Z=0.000"` = default |
| `HeadwearFashionScale` | string | Scale `"X=<f> Y=<f> Z=<f>"`. `"X=1.000 Y=1.000 Z=1.000"` = default. Non-unit scales observed: `"X=1.005 Y=1.005 Z=1.005"`, `"X=1.050 Y=1.050 Z=1.050"` |

---

#### PlayerMorphData Format

A **semicolon-separated** string of blend shape morph targets:

```
"face_chang--0.933327;taiyangxue_zuo-0.000000;sai_zuo--0.117907;..."
```

Format per entry: `<morphName>-<value>`
- Positive: `morphName-0.500000`
- Negative: `morphName--0.500000` (double-dash)
- Range: `[-1.0, 1.0]`
- Values are always formatted to 6 decimal places

**Important:** Both Oversea and CN entries in the Oversea `listRec` use the **same OS morph set**. The CN-specific morph set (`biaozhun_ZB`, `eye_s_nei`, etc.) only appears in entries from `oneAppId` 1256 retrieved via the CN API (`htface.laohu.com`).

**Oversea (OS) morph target reference** — observed order in data:

| Morph Name | Description |
|---|---|
| `face_chang` | Face length (shorter/longer) |
| `taiyangxue_zuo` | Temple — left |
| `taiyangxue_shang` | Temple — up |
| `taiyangxue_qian` | Temple — forward |
| `quangu_zuo` | Cheekbone — left |
| `quangu_shang` | Cheekbone — up |
| `quangu_qian` | Cheekbone — forward |
| `sai_zuo` | Cheek — left |
| `sai_shang` | Cheek — up |
| `sai_qian` | Cheek — forward |
| `sai_jian_old` | Cheek slim (legacy, always 0.0) |
| `sai_gu` | Cheek bone (always 0.0) |
| `xiachunji_zuo` | Sub-cheek — left |
| `xiachunji_shang` | Sub-cheek — up |
| `xiachunji_qian` | Sub-cheek — forward |
| `xiaba_shang` | Chin — up/height |
| `xiaba_zuo` | Chin — left/width |
| `xiasai_gu` | Lower jaw bone (always 0.0) |
| `xiaba_jian` | Chin sharpness (always 0.0) |
| `bizi_xia` | Nose tip |
| `zuiba_shang` | Upper lip height |
| `zuiba_kuan` | Mouth width |
| `zuiba_weixiao` | Smile curl |
| `eye_shang` | Eye vertical position up (always 0.0) |
| `eye_xia` | Eye vertical position down (always 0.0) |
| `eyejiao_shang` | Eye corner angle (always 0.0) |
| `eyexia_kai` | Lower eye opening (always 0.0) |
| `eyeneishang_kai` | Inner upper eye opening (always 0.0) |
| `A` | Phoneme A mouth shape (always 0.0) |
| `O` | Phoneme O mouth shape (always 0.0) |
| `cartoonhead0051_ZB` | Cartoon/stylized head weight. `1.0` in all observed Oversea entries |
| `shangyanpi_shang` | Upper eyelid height |
| `xiayanpi_xia` | Lower eyelid depth |
| `neiyanjiao_shang` | Inner eye corner — up |
| `neiyanjiao_xia` | Inner eye corner — down |
| `waiyanjiao_shang` | Outer eye corner — up |
| `waiyanjiao_xia` | Outer eye corner — down |
| `xiaba_qian` | Chin depth/forward |
| `pingguoji_zuo` | Cheek apple — left |
| `pingguoji_shang` | Cheek apple — up |
| `pingguoji_qian` | Cheek apple — forward |
| `xiaegu_zuo` | Lower cheek/mandible — left |
| `xiaegu_shang` | Lower cheek/mandible — up |
| `xiaegu_qian` | Lower cheek/mandible — forward |
| `zuiba_qian` | Mouth depth |
| `bizi_huxing` | Nostril shape |
| `bizi_qian` | Nose depth/protrusion |
| `cartoonhead0051_ertong` | Child/younger face blend |
| `cartoonhead0051_ertong1` | Child face variant |
| `animation_laughingsmall` | Smile animation (always 0.0) |
| `animation_surprise` | Surprise animation (always 0.0) |
| `animation_laughing` | Laugh animation (always 0.0) |
| `cartoonhead0051_yujie` | Elegant/mature face blend |
| `eyebrow_h` | Eyebrow height |
| `face_kuan` | Face width |

**China (CN) morph target reference** — as observed in `oneAppId` 1256 `listRec` entries (`faceId` 4199847 from `oneAppId` 1000138 also uses this set):

| Morph Name | Description |
|---|---|
| `face_chang` | Face length |
| `xiasai_gu` | Lower jaw bone |
| `face_yanbian_gu` | Face jaw/cheek bone |
| `face_quangugu` | Cheekbone overall |
| `xiaba_jian` | Chin sharpness |
| `bizi_shang` | Nose height |
| `zuiba_shang` | Upper lip |
| `xiaba_shang` | Chin height |
| `xiahegu_shang` | Lower jaw arc — up |
| `zuiba_zai` | Mouth redo |
| `zuiba_weixiao` | Smile curl |
| `biliang_gu` | Nose bridge bone |
| `xiahegu_yuan` | Lower jaw arc — round |
| `xiahegu_you` | Lower jaw arc — right |
| `face_chang_change` | Face length variant |
| `face_quangushang` | Cheekbone — up |
| `face_quanguqian` | Cheekbone — forward |
| `xiasai_shang` | Lower jaw — up |
| `xiasai_zuo` | Lower jaw — left |
| `xiahegu_qian` | Lower jaw arc — forward |
| `zuiba_qian` | Mouth depth |
| `xiaba_qian` | Chin depth |
| `bizi_zhai` | Nose width |
| `bizi_qian` | Nose depth |
| `face_yanbianshang` | Face side — up |
| `face_yanbianyou` | Face side — right |
| `face_yanbianqian` | Face side — forward |
| `xiaba_ping` | Chin flat |
| `face_dabo` | Face fat/plump |
| `zuijiao_hou` | Mouth corner pull-back |
| `tutu_1` | Lip pout |
| `001`–`010` | Reserved/unnamed blend shapes |
| `eye_s_nei` | Eye size — inner |
| `eye_s_zhong` | Eye size — middle |
| `eye_s_wai` | Eye size — outer |
| `eye_x_nei` | Eye shape — inner |
| `eye_x_zhong` | Eye shape — middle |
| `eye_x_wai` | Eye shape — outer |
| `chunji_shang` | Lips — up |
| `chunji_you` | Lips — side |
| `chunji_qian` | Lips — forward |
| `pingguoji_qian` | Cheek apple — forward |
| `pingguoji_shang` | Cheek apple — up |
| `pingguoji_you` | Cheek apple — right |
| `eyebrow_h` | Eyebrow height |
| `biaozhun_ZB` | CN standard head blend (equivalent of Oversea's `cartoonhead0051_ZB`). `1.0` in all observed CN entries |
| `tutu_ertong` | Child pout |
| `kouxing_a` | Mouth phoneme A |
| `kouxing_o` | Mouth phoneme O |
| `eyebrow_f` | Eyebrow furrow |

---

#### HeadBonesData Format

A **semicolon-separated** list of exactly **14** bone transform entries:

```
"X=0.937 Y=0.937 Z=0.937;X=1.000 Y=1.000 Z=1.000;X=-0.065 Y=-0.026 Z=-0.234;..."
```

Each entry: `"X=<float> Y=<float> Z=<float>"` (3 decimal places)

The 14 slots encode scale, translation, and rotation for specific head rig bones. Based on observed patterns:

| Slot (0-indexed) | Common values | Notes |
|---|---|---|
| 0 | `0.850`–`1.070` uniform XYZ | Head overall scale |
| 1 | `1.000 1.000 1.000` | Always identity |
| 2 | Small values like `0.022`, `-0.251` | Translation offset; `0.000 0.000 0.000` = no offset |
| 3 | `1.000 0.865`–`1.444 0.964`–`1.340` | Jaw/cheek bone scale (non-uniform) |
| 4 | Values like `-0.211`, `9.388`, `-11.208` | Vertical translation. CN entries often use `0.000 Y=1.000 Z=0.000` style |
| 5 | `0.880`–`1.077` non-uniform | Neck/jaw width scale |
| 6 | `0.920`–`1.300` uniform | Bone scale, varies per character. CN commonly `1.200` |
| 7–11 | `1.000 1.000 1.000` | Always identity in all observed entries |
| 12 | `-0.502`–`0.451`, or `0.000` | Frequently negative. CN often `0.000 0.000 <±Z>` |
| 13 | `1.000 1.000 1.000` | Always identity |

**CN-specific HeadBonesData patterns:**
- Slot 4 frequently uses `X=0.000 Y=1.000 Z=0.000` or `X=0.000 Y=0.000 Z=0.000` (identity-like) while Oversea uses large world-space offsets like `X=9.388 Y=0.000 Z=-0.538`
- Slot 12 in CN often uses `X=0.000 Y=0.000 Z=<float>` (e.g. `Z=-1.000`, `Z=1.000`, `Z=0.738`) rather than the uniform negative scale seen in Oversea

---

#### Extended Fields (v5.6.6+ / v5.7.8+)

Present in entries created by Oversea v5.6.6+ or CN v5.7.8+ clients. These replace/supplement older face mark and decoration fields.

| Field | Present In | Description |
|---|---|---|
| `HairHighlightColorData` | Both | Hair highlight config. Always `""` in all observed entries; format TBD |
| `FaceprintData` | Both | Face mark config (replaces `FaceIndex`/`FaceColor`/`FaceMarkOffset`) |
| `EyeshadowData` | Both | Eyeshadow layer config |
| `ImitationId` | Both | Imitation/preset link ID. `"None"` when unused |
| `ImitationSwitch` | Both | 16-slot bitmask. `"{0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0}"` = all off. Non-zero observed: `"{5,0,0,...}"` (CN) |
| `ImitationPendant` | Both | Imitation pendant config. Always `""` in observed entries |
| `MakeupOrnament` | Both | Colon-separated list of makeup ornament entries |

**`FaceprintData` format:**
```
"<mirrorFlag>&<patternIndex>&<primaryColor>&<offsetVector>&<opacity>"
```

Observed example:
```
"1&12&(R=0.994944,G=0.250802,B=0.263205,A=1.000000)&(R=0.155316,G=0.017158,B=0.000000,A=0.280000)&1.000000"
```

| Segment | Type | Description |
|---|---|---|
| 1 | integer | Mirror flag: `1` = mirrored, `0` = single side |
| 2 | integer | Pattern/style index |
| 3 | RGBA | Primary face mark color |
| 4 | RGBA repurposed | Position offset: `R=x, G=y, B=z, A=reserved` |
| 5 | float | Opacity multiplier |

**`EyeshadowData` format:**
```
"<typeIndex>&<param1>&<param2>&<param3>&<param4>&<param5>&<color>"
```

No eyeshadow (OS example):
```
"255&1.000000&1.000000&1.000000&1.000000&0.000000&(R=0.000000,G=0.000000,B=0.000000,A=0.000000)"
```

With eyeshadow (CN example):
```
"1&0.507112&0.000000&6.765708&0.000000&0.000000&(R=0.169910,G=0.125000,B=0.240000,A=0.240000)"
```

| Segment | Description |
|---|---|
| 1 | Type/preset index. `255` = no eyeshadow |
| 2–6 | Float parameters (opacity layers, blend factors) |
| 7 | RGBA color |

**`MakeupOrnament` format:**

A colon-separated list of ornament entries. Each entry:
```
"<ornamentId>@<slotIndex>@<posOffset>&<rotation>&<scale1>&<scale2>&<flags>@<colorParam1>@<colorParam2>"
```

OS example (2 ornaments):
```
"Hat_40@39@X=0.000 Y=0.000 Z=0.000&P=0.000000 Y=0.000000 R=0.000000&0.000000&0.000000&0@0@0:Hat_16@18@X=0.000 Y=0.000 Z=0.000&P=0.000000 Y=0.000000 R=0.000000&0.000000&0.000000&0@0@0"
```

CN example (5 ornaments with custom transforms):
```
"Hotta_Ornament_38@208@X=-3.343 Y=1.152 Z=-0.654&P=11.211548 Y=-1.604004 R=-4.773560&-0.300000&0.000000&0:Hat_84@66@...:fashion_decoration_99_4@153@X=3.802 Y=5.000 Z=5.000&P=20.945435 Y=118.976440 R=-29.998169&...:Hat_20@22@...:fashion_decoration_91_4@145@..."
```

| Segment | Description |
|---|---|
| `<ornamentId>` | Asset ID. Examples: `Hat_40`, `Hat_16`, `Hotta_Ornament_38`, `fashion_decoration_99_4`, `fashion_decoration_91_4`, `Hat_84`, `Hat_20` |
| `<slotIndex>` | Attachment bone slot number |
| `X=... Y=... Z=...` | Position offset (world units) |
| `P=... Y=... R=...` | Pitch/Yaw/Roll Euler rotation in degrees |
| `<scale1>&<scale2>` | Scale parameters |
| `<flags>` | Integer flags |
| `<colorParam1>&<colorParam2>` | Color parameter slots (`0` = default) |

---

## Data Types & Conventions

### RGBA Color Format

Colors are expressed as Unreal Engine linear RGBA float strings:

```
"(R=0.573944,G=0.573944,B=0.258005,A=1.000000)"
```

- **Linear color space** (not gamma-corrected sRGB)
- Normal range `0.0`–`1.0`
- **HDR values above `1.0`** are valid and produce emissive/bloom effects (e.g. `R=2.000000` for maximum brightness on `DressFashionColor`)
- `A=1.0` = fully opaque, `A=0.0` = fully transparent/disabled
- Fully disabled color: `(R=0.000000,G=0.000000,B=0.000000,A=0.000000)` — seen in `EyeColor4` when highlight is disabled

---

### Timestamps

| Context | Unit |
|---|---|
| Request `timestamp` parameter (all endpoints, both regions) | Unix **seconds** |
| `createTime` in face responses | Unix **milliseconds** |

Some `createTime` values carry high-precision timestamps with sub-millisecond info (e.g. `1772632776492`, `1772628078895`, `1772705575087`). These are safe to treat as standard millisecond timestamps.

---

### CDN Hosts & Image URLs

| CDN Hostname | Bucket/Owner | Region | Used For |
|---|---|---|---|
| `htface-kr-1305865668.file.myqcloud.com` | Tencent Cloud KR | Korea | Oversea face images (post mid-2022) |
| `htface-tower.wmupd.com` | Perfect World CDN | — | Oversea legacy face images (Aug–Dec 2022) |
| `htface-1251008858.file.myqcloud.com` | Tencent Cloud | China | All CN face images |

**Note on legacy URL path structure:** The legacy `htface-tower.wmupd.com` URLs use `/fantasy/face/` as the path prefix (not `/img/face/`), and the segment after the date is an internal server-generated numeric ID rather than a `userId`.

---

## Response Codes

| `code` | Meaning |
|---|---|
| `0` | Success |

Only `code: 0` was observed across all captured traffic. Error codes were not captured.

---

## Known App IDs

| `oneAppId` | Region / Build | Package |
|---|---|---|
| `1000129` | Tower of Fantasy — Oversea | `com.levelinfinite.hotta.win` |
| `1000138` | Unknown alternate build (appears in Oversea `listRec`) | Unknown |
| `1256` | Tower of Fantasy — China (幻塔) | `com.hottagames.qrsl` |

`oneAppId` 1000138 entries are intermixed into the Oversea `listRec` alongside `1000129` entries. They have `os: 15`, `faceId` values in the 4M–5M range (mid-2023 era), use the OS morph set, and carry no `ext` data.

---

## Changelog / Version Notes

| Client | Version | Notes |
|---|---|---|
| OS | 5.6.6 | Adds `HairHighlightColorData`, `FaceprintData`, `EyeshadowData`, `ImitationId`, `ImitationSwitch`, `ImitationPendant`, `MakeupOrnament`. Removes `FaceIndex`/`FaceColor`/`FaceMarkOffset` from new entries. `DressFashionColor` supports 3-channel outfits |
| CN | 5.7.8 | Same extended fields as oversea 5.6.6. `ImitationSwitch` first slot can be non-zero (e.g. `"{5,0,...}"`) |
| OS | pre-5.x (PC) | `data` uses `rn\t` literal line endings |
| OS | pre-5.x (Android) | `data` uses `n\t` literal line endings |
| CN | early 2022 (PC) | `data` uses standard `\r\n\t`. Some entries have empty `DressFashionColor` `""` |
| OS | Aug 2022 | Earliest face entries (`faceId` ~100,000,000). CDN: `htface-tower.wmupd.com` with `/fantasy/face/` path |
| OS | Late 2022 | CDN migrated to `htface-kr-1305865668.file.myqcloud.com` with `/img/face/` path |
| CN | Nov–Dec 2021 | Earliest face entries (`faceId` ~160,000). CDN: `htface-1251008858.file.myqcloud.com` |
---