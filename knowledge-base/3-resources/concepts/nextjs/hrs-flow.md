# hrs-flow

> Nguồn: https://www.hrs.com/ — Phân tích ngày 2026-05-27

---

## Tổng quan

HRS.com là nền tảng đặt phòng khách sạn doanh nghiệp. Search flow gồm 2 giai đoạn chính: **Autocomplete** (gợi ý địa điểm) và **Hotel Search** (tìm kiếm + hiển thị kết quả). Frontend dùng Next.js (thấy qua `/_next/` paths).

---

## API Endpoints

### 1. Autocomplete — `GET /locations`

**Trigger:** Mỗi keystroke trong search box, sau debounce ~300ms

**Query params:**
| Param | Ví dụ | Mô tả |
|-------|-------|-------|
| `query` | `Ber` | Keyword người dùng gõ |
| `language` | `en` | Ngôn ngữ hiển thị |
| `goog` | `[token]` | Google token (auth/tracking) |

**Response:** Array of objects (200 OK, ~7ms)

```json
[
  {
    "name": "Berlin",
    "additionalName": "",
    "locationId": 55133,
    "latitude": 52.516281,
    "longitude": 13.3776,
    "category": "City",
    "group": "LOCATION",
    "score": 253
  },
  {
    "name": "Hotel De France Centre Francais",
    "additionalName": "Berlin, Germany Berlin",
    "locationId": 55133,
    "hotelId": 571704,
    "latitude": 52.54596,
    "longitude": 13.41362,
    "category": "Hotel",
    "group": "HOTEL",
    "score": 255
  }
]
```

**Groups trả về:** `LOCATION` (City/Region), `HOTEL`, `TRAIN_STATION_AIRPORT`, `POINT_OF_INTEREST`

---

### 2. Navigate to Results Page

Sau khi user chọn location và submit, browser navigate sang:

```
GET /en/list
  ?location=%2555133         ← encoded locationId
  &orderBy=Recommendations
  &startDateDay=27
  &startDateMonth=5
  &startDateYear=2026
  &endDateDay=28
  &endDateMonth=5
  &endDateYear=2026
  &currency=EUR
  &rooms=1
  &numberOfAdults[0]=1
  &type[0]=SINGLEROOM
  &adults=1
  &children=0
  &singleRooms=1
  &doubleRooms=0
```

---

### 3. POI by Location — `GET /pec/v2/geo/poi/by-location`

**Trigger:** Ngay sau khi load results page (song song với hotel search)

**Query params:**
| Param | Ví dụ |
|-------|-------|
| `locationId` | `55133` |

**Response:** Array ~394 items (200 OK, ~1465ms)

```json
[
  {
    "id": 2049,
    "name": "Berlin Subway Moritzplatz",
    "geoCoordinate": { "latitude": 52.5041, "longitude": 13.4127 },
    "poiType": "PUBLIC_TRANSPORT",
    "classification": 0,
    "status": "ACTIVE"
  }
]
```

**poiType values:** `PUBLIC_TRANSPORT`, `EXIT_MOTORWAY`, `POI`

Dùng cho: bản đồ map view + filter "Distance to" trên UI.

---

### 4. Hotel Search — `POST /pec/v2/hotel-search/hotels` ← **API chính**

**Trigger:** Load results page (gọi **2 lần song song**)

**Request Body (JSON):**

```json
{
  "findParameter": {
    "locationId": 55133,
    "matchType": "LOCATION",
    "matchGroup": "CITY",
    "perimeter": 0,
    "maxPerimeter": 0
  },
  "requestParameter": {
    "dateRange": {
      "fromDate": "2026-05-27",
      "toDate": "2026-05-28"
    },
    "offerType": "NORMAL",
    "requestType": "ROOM",
    "roomRequests": [
      {
        "roomNumber": 1,
        "roomType": "SINGLEROOM",
        "occupancy": {
          "adults": 1,
          "children": 0
        }
      }
    ]
  },
  "options": {
    "statusFilterType": "BOOKABLE",
    "skipHotelsIfNoValidOffer": true,
    "useLegacyOa": false,
    "offerServiceOptions": {
      "considerOnlyCrsByTypes": ["HRS"],
      "crsTarget": "PRODUCTION",
      "buildAvgOffers": false,
      "calculateCheaperOfferFromSecondaryRooms": true,
      "allowEconomyRoomWithBath": true,
      "allowEconomyRoomSharedBath": true,
      "skipBreakfastFilter": false,
      "strictRoomTypeHandling": false,
      "provideNetAmount": true,
      "returnFailedHotelCauses": true,
      "useNewTaxesAndFees": true
    },
    "dataServiceOptions": {
      "pictureRequestParameter": {
        "width": 269,
        "height": 213
      }
    }
  },
  "recommendationParameter": {
    "distributionChannel": "PRIVATE_WEB",
    "costCenterId": 6011,
    "btc": {
      "skipFallback": true,
      "params": { "abParam": "a" }
    }
  }
}
```

**Response (200 OK, ~3.2s):**

```
{
  hotels[]: array (199 items per call, tổng 343),
  totalCount: 343,
  requestedHotels: ...,
  failedHotels: ...,
  usedPerimeter: ...
}
```

**Mỗi hotel object có 4 keys:**

```
hotelData     → address, averageRating, features, amenities,
                greenstay cert, hotelTypes, paymentTypes...
hotelOffer    → pricePerNight, totalPrice, tariffType,
                availability, roomCategories, meal...
hotelId       → number
geoCoordinate → { latitude, longitude }
```

**Chi tiết `hotelData`:**
- `hotelName`, `hotelId`, `hotelCategory.hrsStars`
- `address`: city, street, zipCode, country
- `averageRating`: ratings theo segment (ALLHRS, BUSINESS, PRIVATE...)
- `featureTypes`: array (PARKING, RESTAURANT, INTERNET_WLANINROOM, AIRCONDITION...)
- `hotelTypes`: CITY_HOTEL, BUSINESS_HOTEL, DESIGN_HOTEL, APARTMENT_HOTEL...
- `greenstayAmenity`: efficiencyClass, kilogramCarbonPOC
- `distanceCityCenter`, `distanceAirport`, `distanceRailway`
- `thumbnailPicture`: CDN URL từ `hrsstatic.com`

**Chi tiết `hotelOffer`:**
- `totalPrice.amount`, `totalPrice.currency`
- `tariffType`: `BASIC`
- `contingented`: `FALSE`
- `reservationOptions`: `GUARANT_WITH_CREDIT_CARD`
- `topRankingBooster`: number (dùng cho sort)
- `avgOffers.SINGLEROOM.pricePerNightAndRoom`
- `avgOffers.SINGLEROOM.meal.availability`

---

### 5. Hotel Pictures — `GET /pec/v2/hotel-data/sales-hotel-data/hotel-pictures`

**Trigger:** Sau khi nhận hotel list (gọi **2 lần song song**, batch nhiều hotelId)

**Query params:**
| Param | Giá trị |
|-------|---------|
| `hotelIds` | `875998,571704,...` (batch) |
| `salesHotelDataServiceOptions.fetchPictures` | `true` |
| `picturesParameters.allPicturesCachedWidth` | `269` |
| `picturesParameters.allPicturesCachedHeight` | `213` |
| `salesHotelDataServiceOptions.pictureRequestParameter.language` | `en` |
| `limit` | `10` |

**Response:** CDN URLs từ `foto.hrsstatic.com` với path format:
```
/fotos/0/2/{width}/{height}/80/000000/{origin_url}/{checksum}/{w},{h}/{num}/{seo_name}.jpg
```

---

## Analytics & Tracking (fire-and-forget)

| Service | Endpoint | Mục đích |
|---------|----------|---------|
| Bing Ads | `bat.bing.com/p/insights/c/k` | Conversion tracking |
| New Relic | `bam.nr-data.net/1/{key}` | Performance monitoring |
| New Relic | `bam.nr-data.net/ins/1/{key}` | SPA instrumentation |
| Internal | `POST /log` | Server-side logging (202) |

---

## Search Flow Summary

```
User gõ keyword
    ↓ debounce ~300ms
GET /locations?query=...
    ↓ ~7ms
Hiện dropdown (Location / Hotel / Airport / POI)
    ↓ user chọn + click Search
Navigate → /en/list?location=...&dates...&rooms...
    ↓ page load
    ├── GET /geo/poi/by-location?locationId=...   (~1.5s, cho map)
    └── POST /hotel-search/hotels × 2 parallel    (~3.2s, main data)
            ↓ response
        GET /hotel-data/.../hotel-pictures × 2    (~1.2s, lazy images)
            ↓
        Render 343 hotels, sorted by Recommendations
```

---

## Notes

- Frontend: **Next.js** (App Router, thấy qua `/_next/static/chunks/app/[language]/list/`)
- Deployment: Vercel (`dpl=dpl_BzTRGAgAM3D3TwsH8AE95JhHxh9e`, `_vercel/speed-insights`)
- Hotel search gọi 2 lần song song → có thể là pagination/chunking (199 + 144 = 343)
- `locationId=55133` là ID của Berlin trong hệ thống HRS
- `topRankingBooster` field trong offer ảnh hưởng đến thứ tự sort "Recommendations"
- Tất cả price đều có cả `amount` (gross) và `amountNet` (net before tax)
