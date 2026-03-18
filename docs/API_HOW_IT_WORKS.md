# 🔄 ระบบ Calendar Events API ทำงานอย่างไร

## 📋 สรุปการทำงาน

```
┌─────────────────┐
│  เว็บไซต์อื่นๆ  │
│ (External Site) │
└────────┬────────┘
         │
         │ 1. ส่ง Request
         │    GET /api/calendar/events
         ▼
┌─────────────────┐
│   API Server    │  ← Express.js หรือ Supabase REST API
│  (Filter/Process)│
└────────┬────────┘
         │
         │ 2. Query Database
         ▼
┌─────────────────┐
│   Supabase      │
│   PostgreSQL    │  ← ฐานข้อมูลตารางกิจกรรม
└────────┬────────┘
         │
         │ 3. Return Data
         ▼
┌─────────────────┐
│  JSON Response  │  ← ข้อมูล JSON ที่ส่งกลับไป
└─────────────────┘
```

---

## 📦 ข้อมูลที่ API ส่งออก (Response Format)

### ตัวอย่างข้อมูลที่ส่งออก (JSON Format)

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "Better Business",
      "category": "MEET CEO",
      "date": "2025-12-15",
      "time": "13:00:00",
      "lecturer": "Mr. Mick Napawat",
      "location": "Hat Yai Branch",
      "event_name": "Better Business",
      "details": "MLM entrepreneurship ideas in the AI era",
      "image_url": "https://example.com/images/event1.jpg",
      "is_active": true,
      "created_at": "2025-01-14T10:00:00.000Z",
      "updated_at": "2025-01-14T10:00:00.000Z"
    },
    {
      "id": 2,
      "title": "HappyCoffee Bootcamp",
      "category": "HQ Phaya Thai, Samut Sakhon, Phitsanulok",
      "date": "2025-12-16",
      "time": "09:00:00",
      "lecturer": "Mr. Max Naphawit",
      "location": "Head office",
      "event_name": "HappyCoffee Bootcamp",
      "details": "Tips for making money from Happy coffee",
      "image_url": "https://example.com/images/event2.jpg",
      "is_active": true,
      "created_at": "2025-01-14T10:00:00.000Z",
      "updated_at": "2025-01-14T10:00:00.000Z"
    }
  ],
  "pagination": {
    "total": 25,
    "limit": 10,
    "offset": 0,
    "has_more": true
  }
}
```

---

## 📝 รายละเอียดฟิลด์ข้อมูล

| ฟิลด์ | ประเภท | คำอธิบาย | ตัวอย่าง |
|-------|--------|----------|---------|
| `id` | number | ID ของกิจกรรม | `1` |
| `title` | string | ชื่อกิจกรรม | `"Better Business"` |
| `category` | string | ประเภทกิจกรรม | `"MEET CEO"` |
| `date` | string | วันที่จัดกิจกรรม (YYYY-MM-DD) | `"2025-12-15"` |
| `time` | string | เวลาจัดกิจกรรม (HH:MM:SS) | `"13:00:00"` |
| `lecturer` | string | ชื่อวิทยากร | `"Mr. Mick Napawat"` |
| `location` | string | สถานที่จัด | `"Hat Yai Branch"` |
| `event_name` | string | ชื่อกิจกรรม (อาจเหมือน title) | `"Better Business"` |
| `details` | string | รายละเอียดกิจกรรม | `"MLM entrepreneurship ideas..."` |
| `image_url` | string | URL รูปภาพกิจกรรม | `"https://..."` |
| `is_active` | boolean | สถานะการแสดงผล | `true` |
| `created_at` | string | วันที่สร้าง (ISO 8601) | `"2025-01-14T10:00:00.000Z"` |
| `updated_at` | string | วันที่แก้ไขล่าสุด (ISO 8601) | `"2025-01-14T10:00:00.000Z"` |

---

## 🔄 ขั้นตอนการทำงานแบบละเอียด

### ตัวอย่าง: เว็บไซต์อื่นต้องการดึงข้อมูลกิจกรรม

**1. เว็บไซต์อื่นส่ง Request:**
```http
GET /api/calendar/events?category=MEET CEO&limit=5
Headers:
  Authorization: Bearer YOUR_API_KEY
```

**2. API Server ตรวจสอบและประมวลผล:**
- ✅ ตรวจสอบ API Key (ถ้ามีการตั้งค่า)
- ✅ ตรวจสอบ CORS (อนุญาต domain หรือไม่)
- ✅ สร้าง SQL Query ตาม Parameters ที่ส่งมา

**3. Query ฐานข้อมูล:**
```sql
SELECT * 
FROM calendar_events 
WHERE category = 'MEET CEO' 
  AND is_active = true 
ORDER BY date ASC, time ASC 
LIMIT 5;
```

**4. ฐานข้อมูลส่งข้อมูลกลับ:**
- รายการกิจกรรมที่ตรงตามเงื่อนไข

**5. API Server จัดรูปแบบข้อมูล:**
- แปลงเป็น JSON Format
- เพิ่ม Metadata (pagination, success status)

**6. ส่ง Response กลับไปยังเว็บไซต์:**
```json
{
  "success": true,
  "data": [...],
  "pagination": {...}
}
```

---

## 🌐 ตัวอย่างการเรียกใช้จากเว็บไซต์อื่น

### JavaScript (Frontend)

```javascript
// เรียก API เพื่อดึงข้อมูลกิจกรรม
async function getEvents() {
  const SUPABASE_URL = 'https://YOUR_PROJECT_ID.supabase.co';
  const SUPABASE_KEY = 'YOUR_ANON_KEY';
  
  const response = await fetch(
    `${SUPABASE_URL}/rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc&limit=10`,
    {
      headers: {
        'apikey': SUPABASE_KEY,
        'Authorization': `Bearer ${SUPABASE_KEY}`,
        'Content-Type': 'application/json'
      }
    }
  );
  
  // ได้ข้อมูลกลับมาเป็น Array
  const events = await response.json();
  
  // events = [
  //   { id: 1, title: "Better Business", date: "2025-12-15", ... },
  //   { id: 2, title: "HappyCoffee Bootcamp", date: "2025-12-16", ... },
  //   ...
  // ]
  
  return events;
}

// ใช้งาน
getEvents().then(events => {
  console.log('กิจกรรมทั้งหมด:', events);
  events.forEach(event => {
    console.log(`- ${event.title} (${event.date} ${event.time})`);
  });
});
```

---

## 📊 ข้อมูลที่ส่งออกในแต่ละ Endpoint

### 1. `GET /api/calendar/events`
**ส่งออก:** Array ของ Events ทั้งหมด (พร้อม pagination)

```json
{
  "success": true,
  "data": [
    { "id": 1, "title": "...", ... },
    { "id": 2, "title": "...", ... }
  ],
  "pagination": {
    "total": 100,
    "limit": 10,
    "offset": 0,
    "has_more": true
  }
}
```

### 2. `GET /api/calendar/events/:id`
**ส่งออก:** Event เดี่ยว

```json
{
  "success": true,
  "data": {
    "id": 1,
    "title": "Better Business",
    "category": "MEET CEO",
    "date": "2025-12-15",
    ...
  }
}
```

### 3. `GET /api/calendar/events/date/:date`
**ส่งออก:** Array ของ Events ในวันนั้น

```json
{
  "success": true,
  "data": [
    { "id": 1, "date": "2025-12-15", ... },
    { "id": 5, "date": "2025-12-15", ... }
  ]
}
```

### 4. `GET /api/calendar/events/month/:year/:month`
**ส่งออก:** Array ของ Events ในเดือนนั้น

```json
{
  "success": true,
  "data": [
    { "id": 1, "date": "2025-12-01", ... },
    { "id": 2, "date": "2025-12-05", ... },
    ...
  ]
}
```

### 5. `GET /api/calendar/events/upcoming`
**ส่งออก:** Array ของ Events ที่กำลังจะมาถึง

```json
{
  "success": true,
  "data": [
    { "id": 10, "date": "2025-12-20", ... },
    { "id": 11, "date": "2025-12-25", ... }
  ]
}
```

### 6. `GET /api/calendar/categories`
**ส่งออก:** Array ของ Categories

```json
{
  "success": true,
  "data": [
    "MEET CEO",
    "HappyCoffee Bootcamp",
    "B-Vision"
  ]
}
```

### 7. `GET /api/calendar/stats`
**ส่งออก:** สถิติ Events

```json
{
  "success": true,
  "data": {
    "total": 50,
    "active": 45,
    "upcoming": 20,
    "past": 25,
    "by_category": {
      "MEET CEO": 15,
      "HappyCoffee Bootcamp": 10,
      "B-Vision": 20
    }
  }
}
```

---

## 🔍 ตัวอย่าง Request & Response จริง

### Request:
```bash
curl -X GET \
  'https://YOUR_PROJECT_ID.supabase.co/rest/v1/calendar_events?select=*&category=eq.MEET CEO&is_active=eq.true&limit=3' \
  -H 'apikey: YOUR_ANON_KEY' \
  -H 'Authorization: Bearer YOUR_ANON_KEY'
```

### Response:
```json
[
  {
    "id": 1,
    "title": "Better Business",
    "category": "MEET CEO",
    "date": "2025-12-15",
    "time": "13:00:00",
    "lecturer": "Mr. Mick Napawat",
    "location": "Hat Yai Branch",
    "event_name": "Better Business",
    "details": "MLM entrepreneurship ideas in the AI era",
    "image_url": null,
    "is_active": true,
    "created_at": "2025-01-14T10:00:00.000Z",
    "updated_at": "2025-01-14T10:00:00.000Z"
  },
  {
    "id": 3,
    "title": "B-Vision",
    "category": "MEET CEO",
    "date": "2025-12-18",
    "time": "13:00:00",
    "lecturer": "Mr. Mick Napawat",
    "location": "Trang Province",
    "event_name": "B-Vision",
    "details": "Build business trust",
    "image_url": null,
    "is_active": true,
    "created_at": "2025-01-14T10:00:00.000Z",
    "updated_at": "2025-01-14T10:00:00.000Z"
  }
]
```

---

## ⚙️ Filter Parameters ที่สามารถใช้ได้

### Query Parameters สำหรับ Filter:

| Parameter | คำอธิบาย | ตัวอย่าง |
|-----------|----------|---------|
| `category` | Filter ตามประเภท | `?category=MEET CEO` |
| `start_date` | Events ตั้งแต่วันที่ | `?start_date=2025-12-01` |
| `end_date` | Events จนถึงวันที่ | `?end_date=2025-12-31` |
| `active_only` | เฉพาะที่เปิดใช้งาน | `?active_only=true` |
| `limit` | จำนวนผลลัพธ์ | `?limit=10` |
| `offset` | ข้ามจำนวน | `?offset=0` |

### ตัวอย่างการ Filter:

```javascript
// ดึง Events ของเดือนธันวาคม 2025
GET /api/calendar/events?start_date=2025-12-01&end_date=2025-12-31

// ดึง Events ประเภท "MEET CEO" เพียง 5 รายการแรก
GET /api/calendar/events?category=MEET CEO&limit=5

// ดึง Events ที่กำลังจะมาถึง (ตั้งแต่วันนี้)
GET /api/calendar/events/upcoming?limit=10
```

---

## ✅ สรุป

**API ส่งออกอะไร:**
1. ✅ ข้อมูลกิจกรรม (Events) ทั้งหมดในรูปแบบ JSON
2. ✅ รายละเอียดครบถ้วน: ชื่อ, วันที่, เวลา, สถานที่, วิทยากร, รูปภาพ
3. ✅ Metadata เพิ่มเติม: Pagination, Stats, Categories
4. ✅ รองรับ Filter: ตาม Category, วันที่, สถานะ

**วิธีใช้งาน:**
1. 📡 เว็บไซต์อื่นส่ง HTTP Request มาที่ API
2. 🔍 API Query ข้อมูลจากฐานข้อมูล
3. 📦 API ส่งข้อมูลกลับเป็น JSON Format
4. 🎨 เว็บไซต์อื่นแสดงผลข้อมูลตามต้องการ

**ข้อมูลที่ได้:**
- Array ของ Events แต่ละรายการมีฟิลด์ครบถ้วนตามที่อธิบายไว้ข้างต้น

