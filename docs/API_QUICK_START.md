# 🚀 Quick Start Guide - Calendar Events API

## วิธีใช้งาน API ตารางกิจกรรม (2 ทางเลือก)

---

## ทางเลือกที่ 1: ใช้ Supabase REST API โดยตรง (แนะนำ ⭐)

**เหมาะสำหรับ:** ผู้ที่ต้องการใช้งานง่ายๆ โดยไม่ต้องตั้งค่า server เพิ่ม

### ขั้นตอนการใช้งาน:

1. **เตรียมข้อมูล:**
   - Supabase Project URL: `https://YOUR_PROJECT_ID.supabase.co`
   - Supabase Anon Key: หาได้จาก Supabase Dashboard > Settings > API

2. **เรียกใช้ API:**
```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT_ID.supabase.co';
const SUPABASE_KEY = 'YOUR_ANON_KEY';

// ดึง events ทั้งหมด
fetch(`${SUPABASE_URL}/rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc`, {
  headers: {
    'apikey': SUPABASE_KEY,
    'Authorization': `Bearer ${SUPABASE_KEY}`
  }
})
.then(res => res.json())
.then(data => console.log(data));
```

**ดูเอกสารเพิ่มเติม:** `docs/API_CALENDAR_EVENTS.md`

---

## ทางเลือกที่ 2: ใช้ Express.js API Server

**เหมาะสำหรับ:** ผู้ที่ต้องการควบคุม API เอง มี custom logic หรือต้องการเพิ่ม security

### ขั้นตอนการติดตั้ง:

1. **ติดตั้ง Dependencies:**
```bash
cd api
npm install
```

2. **ตั้งค่า Environment Variables:**
```bash
# สร้างไฟล์ .env
cp ENV_EXAMPLE.txt .env

# แก้ไขไฟล์ .env
# ใส่ Supabase URL และ Key ของคุณ
```

3. **รัน Server:**
```bash
# Development
npm run dev

# Production
npm start
```

4. **ใช้งาน API:**
```javascript
// Server จะรันที่ http://localhost:3002

// ดึง events ทั้งหมด
fetch('http://localhost:3002/api/calendar/events')
  .then(res => res.json())
  .then(data => console.log(data));
```

**ดูเอกสารเพิ่มเติม:** `api/README.md`

---

## ตัวอย่างการใช้งานจริง

### ตัวอย่าง 1: ดึง Events ของเดือนนี้ (JavaScript)

```javascript
async function getThisMonthEvents() {
  const today = new Date();
  const year = today.getFullYear();
  const month = today.getMonth() + 1;
  
  const SUPABASE_URL = 'https://YOUR_PROJECT_ID.supabase.co';
  const SUPABASE_KEY = 'YOUR_ANON_KEY';
  
  const startDate = `${year}-${String(month).padStart(2, '0')}-01`;
  const lastDay = new Date(year, month, 0).getDate();
  const endDate = `${year}-${String(month).padStart(2, '0')}-${String(lastDay).padStart(2, '0')}`;
  
  const response = await fetch(
    `${SUPABASE_URL}/rest/v1/calendar_events?select=*&date=gte.${startDate}&date=lte.${endDate}&is_active=eq.true&order=date.asc`,
    {
      headers: {
        'apikey': SUPABASE_KEY,
        'Authorization': `Bearer ${SUPABASE_KEY}`
      }
    }
  );
  
  const events = await response.json();
  return events;
}

// ใช้งาน
getThisMonthEvents().then(events => {
  console.log('This month events:', events);
});
```

### ตัวอย่าง 2: แสดง Events ในหน้าเว็บ (HTML + JavaScript)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Calendar Events</title>
</head>
<body>
  <div id="events-container"></div>

  <script>
    async function loadEvents() {
      const SUPABASE_URL = 'https://YOUR_PROJECT_ID.supabase.co';
      const SUPABASE_KEY = 'YOUR_ANON_KEY';
      
      try {
        const response = await fetch(
          `${SUPABASE_URL}/rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc&limit=10`,
          {
            headers: {
              'apikey': SUPABASE_KEY,
              'Authorization': `Bearer ${SUPABASE_KEY}`
            }
          }
        );
        
        const events = await response.json();
        displayEvents(events);
      } catch (error) {
        console.error('Error loading events:', error);
      }
    }

    function displayEvents(events) {
      const container = document.getElementById('events-container');
      container.innerHTML = events.map(event => `
        <div class="event-card">
          <h3>${event.title}</h3>
          <p><strong>วันที่:</strong> ${event.date}</p>
          <p><strong>เวลา:</strong> ${event.time}</p>
          <p><strong>สถานที่:</strong> ${event.location}</p>
          <p><strong>วิทยากร:</strong> ${event.lecturer}</p>
        </div>
      `).join('');
    }

    // โหลด events เมื่อหน้าเว็บโหลดเสร็จ
    loadEvents();
  </script>
</body>
</html>
```

### ตัวอย่าง 3: ใช้กับ WordPress (PHP)

```php
<?php
function get_calendar_events() {
    $supabase_url = 'https://YOUR_PROJECT_ID.supabase.co';
    $supabase_key = 'YOUR_ANON_KEY';
    
    $url = $supabase_url . '/rest/v1/calendar_events?select=*&is_active=eq.true&order=date.asc&limit=10';
    
    $args = array(
        'headers' => array(
            'apikey' => $supabase_key,
            'Authorization' => 'Bearer ' . $supabase_key,
            'Content-Type' => 'application/json'
        )
    );
    
    $response = wp_remote_get($url, $args);
    
    if (is_wp_error($response)) {
        return array();
    }
    
    $body = wp_remote_retrieve_body($response);
    return json_decode($body, true);
}

// ใช้งานใน template
$events = get_calendar_events();
foreach ($events as $event) {
    echo '<div>';
    echo '<h3>' . esc_html($event['title']) . '</h3>';
    echo '<p>วันที่: ' . esc_html($event['date']) . '</p>';
    echo '<p>สถานที่: ' . esc_html($event['location']) . '</p>';
    echo '</div>';
}
?>
```

---

## 🔐 Security Tips

1. **อย่าเผยแพร่ Anon Key ใน Client-side Code โดยตรง**
   - ใช้ Environment Variables
   - หรือใช้ Server-side Proxy

2. **ตั้งค่า CORS ใน Supabase Dashboard**
   - Settings > API > CORS Configuration
   - เพิ่ม domains ที่อนุญาต

3. **ใช้ RLS (Row Level Security) Policies**
   - ตั้งค่า Policies ใน Supabase เพื่อควบคุมการเข้าถึงข้อมูล

---

## 📞 Support

สำหรับคำถามหรือปัญหาการใช้งาน กรุณาดูเอกสารเพิ่มเติม:
- `docs/API_CALENDAR_EVENTS.md` - Supabase REST API Documentation
- `api/README.md` - Express.js API Server Documentation

