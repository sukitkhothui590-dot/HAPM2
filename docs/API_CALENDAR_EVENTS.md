# 📅 Calendar Events API Documentation

## Overview

API สำหรับดึงข้อมูลตารางกิจกรรม (Calendar Events) จากระบบ Happy MPM ไปใช้งานในเว็บไซต์หรือแอปพลิเคชันภายนอก

---

## Base URL

```
https://[YOUR_SUPABASE_PROJECT_ID].supabase.co/rest/v1/calendar_events
```

**หมายเหตุ:** แทนที่ `[YOUR_SUPABASE_PROJECT_ID]` ด้วย Project ID ของ Supabase ที่คุณใช้

---

## Authentication

### Method 1: Public Access (ไม่ต้อง API Key)
```http
GET /rest/v1/calendar_events?select=*&is_active=eq.true
apikey: YOUR_SUPABASE_ANON_KEY
```

### Method 2: With Authorization (แนะนำ)
```http
GET /rest/v1/calendar_events?select=*&is_active=eq.true
apikey: YOUR_SUPABASE_ANON_KEY
Authorization: Bearer YOUR_SUPABASE_ANON_KEY
```

---

## Endpoints

### 1. ดึง Events ทั้งหมด

**Request:**
```http
GET /rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc,time.asc
Headers:
  apikey: YOUR_SUPABASE_ANON_KEY
  Authorization: Bearer YOUR_SUPABASE_ANON_KEY
```

**Response:**
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
    "image_url": "https://...",
    "is_active": true,
    "created_at": "2025-01-14T10:00:00Z",
    "updated_at": "2025-01-14T10:00:00Z"
  }
]
```

---

### 2. Filter ตาม Category

**Request:**
```http
GET /rest/v1/calendar_events?select=*&category=eq.MEET CEO&is_active=eq.true&order=date.asc
```

---

### 3. Filter ตามช่วงวันที่

**Request:**
```http
GET /rest/v1/calendar_events?select=*&date=gte.2025-12-01&date=lte.2025-12-31&is_active=eq.true
```

**Query Parameters:**
- `date=gte.YYYY-MM-DD` - Events ตั้งแต่วันที่ (greater than or equal)
- `date=lte.YYYY-MM-DD` - Events จนถึงวันที่ (less than or equal)

---

### 4. ดึง Event เดี่ยวตาม ID

**Request:**
```http
GET /rest/v1/calendar_events?id=eq.1&select=*
```

---

### 5. ดึง Events ที่กำลังจะมาถึง (Upcoming)

**Request:**
```http
GET /rest/v1/calendar_events?select=*&date=gte.2025-01-14&is_active=eq.true&order=date.asc,time.asc
```

**หมายเหตุ:** ใช้วันที่ปัจจุบันแทน `2025-01-14`

---

### 6. ดึง Events ของเดือนเฉพาะ

**Request:**
```http
GET /rest/v1/calendar_events?select=*&date=gte.2025-12-01&date=lt.2026-01-01&is_active=eq.true
```

---

## Query Operators

Supabase รองรับ PostgREST query operators ดังนี้:

- `eq` - เท่ากับ (`column=eq.value`)
- `neq` - ไม่เท่ากับ
- `gt` - มากกว่า
- `gte` - มากกว่าหรือเท่ากับ
- `lt` - น้อยกว่า
- `lte` - น้อยกว่าหรือเท่ากับ
- `like` - ค้นหาแบบ pattern (`column=like.*keyword*`)
- `ilike` - ค้นหาแบบ pattern (case-insensitive)
- `in` - อยู่ใน array (`column=in.(value1,value2)`)

---

## Response Format

### Success Response
```json
[
  {
    "id": 1,
    "title": "Event Title",
    "category": "MEET CEO",
    "date": "2025-12-15",
    "time": "13:00:00",
    "lecturer": "Speaker Name",
    "location": "Location Name",
    "event_name": "Event Name",
    "details": "Event details...",
    "image_url": "https://...",
    "is_active": true,
    "created_at": "2025-01-14T10:00:00Z",
    "updated_at": "2025-01-14T10:00:00Z"
  }
]
```

### Error Response
```json
{
  "message": "Error message",
  "code": "ERROR_CODE",
  "details": "..."
}
```

---

## Example Code

### JavaScript (Fetch API)
```javascript
async function getCalendarEvents() {
  const SUPABASE_URL = 'https://YOUR_PROJECT_ID.supabase.co';
  const SUPABASE_ANON_KEY = 'YOUR_ANON_KEY';
  
  try {
    const response = await fetch(
      `${SUPABASE_URL}/rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc,time.asc`,
      {
        headers: {
          'apikey': SUPABASE_ANON_KEY,
          'Authorization': `Bearer ${SUPABASE_ANON_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const events = await response.json();
    console.log('Events:', events);
    return events;
  } catch (error) {
    console.error('Error fetching events:', error);
    throw error;
  }
}

// ใช้งาน
getCalendarEvents();
```

### PHP
```php
<?php
function getCalendarEvents() {
    $supabaseUrl = 'https://YOUR_PROJECT_ID.supabase.co';
    $supabaseKey = 'YOUR_ANON_KEY';
    
    $url = $supabaseUrl . '/rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc,time.asc';
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'apikey: ' . $supabaseKey,
        'Authorization: Bearer ' . $supabaseKey,
        'Content-Type: application/json'
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}

$events = getCalendarEvents();
print_r($events);
?>
```

### Python
```python
import requests

def get_calendar_events():
    supabase_url = 'https://YOUR_PROJECT_ID.supabase.co'
    supabase_key = 'YOUR_ANON_KEY'
    
    url = f'{supabase_url}/rest/v1/calendar_events'
    headers = {
        'apikey': supabase_key,
        'Authorization': f'Bearer {supabase_key}',
        'Content-Type': 'application/json'
    }
    params = {
        'select': '*',
        'is_active': 'eq.true',
        'order': 'date.asc,time.asc'
    }
    
    response = requests.get(url, headers=headers, params=params)
    response.raise_for_status()
    
    return response.json()

events = get_calendar_events()
print(events)
```

### cURL
```bash
curl -X GET \
  'https://YOUR_PROJECT_ID.supabase.co/rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc,time.asc' \
  -H 'apikey: YOUR_ANON_KEY' \
  -H 'Authorization: Bearer YOUR_ANON_KEY' \
  -H 'Content-Type: application/json'
```

---

## Rate Limiting

- **Free Tier:** ~500 requests/minute
- **Pro Tier:** Higher limits

---

## Support

สำหรับคำถามหรือปัญหาการใช้งาน API กรุณาติดต่อทีมพัฒนาระบบ Happy MPM

---

## Changelog

### v1.0.0 (2025-01-14)
- Initial API release
- Support for filtering by category, date range
- Support for upcoming events query

