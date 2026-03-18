# รายงานตรวจสอบระบบ SEO / Search Engine (Happy MPM Web)

ตรวจสอบเมื่อ: ตามที่รัน  
ขอบเขต: หน้าบ้าน (Public) เท่านั้น (ไม่รวมหลังบ้าน Admin)

---

## สรุปภาพรวม

| หมวด | สถานะ | หมายเหตุ |
|------|--------|----------|
| Meta พื้นฐาน (title, description) | ⚠️ มีแต่เป็นค่าเริ่มต้น CRA | ยังไม่ปรับเป็นแบรนด์/เนื้อหาจริง |
| Open Graph (og:) | ❌ ไม่มี | แชร์ลิงก์ในโซเชียลจะได้ข้อความ/รูปไม่ตรง |
| Twitter Card | ❌ ไม่มี | แชร์บน Twitter/X ไม่มี card พิเศษ |
| เปลี่ยน title/description ตามหน้า | ❌ ไม่มี | ทุก URL ได้ title เดียวกัน |
| robots.txt | ✅ มี | อนุญาตทุก bot (ไม่มี Disallow) |
| Sitemap (sitemap.xml) | ❌ ไม่มี | Search engine ต้อง crawl เอง |
| Canonical URL | ❌ ไม่มี | เสี่ยง duplicate content ถ้ามีหลาย URL ชี้หน้าเดียวกัน |
| hreflang (หลายภาษา) | ❌ ไม่มี | Google อาจไม่รู้ว่าเป็นคนละภาษาของหน้าเดียวกัน |
| Structured Data (JSON-LD) | ❌ ไม่มี | ไม่มี rich result ในผลค้นหา |
| manifest.json (PWA) | ⚠️ มีแต่เป็นค่า CRA | short_name / name ยังไม่ใช่ Happy MPM |
| Favicon / Apple touch icon | ✅ มี | มีใน index.html |

---

## 1. สิ่งที่มีอยู่แล้ว

### 1.1 ไฟล์ `public/robots.txt`

```
User-agent: *
Disallow:
```

- อนุญาตให้ทุก bot เข้ามา crawl ได้ (ไม่มี Disallow)
- ไม่ได้บล็อก path ใด
- **ข้อเสนอ:** ถ้ามี path ที่ไม่ต้องการให้ index (เช่น `/admin` ถ้าเข้าหน้าบ้านจากโดเมนเดียว) สามารถเพิ่ม `Disallow: /admin` ได้

### 1.2 Meta ใน `public/index.html`

- `charset="utf-8"`  
- `viewport` (width, initial-scale, maximum-scale, user-scalable)  
- `theme-color`  
- `description` → ค่าตอนนี้คือ **"Web site created using create-react-app"** (ยังไม่เหมาะกับ SEO)  
- `title` → ค่าตอนนี้คือ **"React App"** (ยังไม่เหมาะกับ SEO)

### 1.3 Favicon และ PWA

- มี `favicon.ico`, `logo192.png`, `logo512.png`
- มี `apple-touch-icon` ชี้ไปที่ logo192
- มี `manifest.json` แต่ `name` / `short_name` ยังเป็น "Create React App Sample" / "React App"

---

## 2. สิ่งที่ยังไม่มี / ไม่เพียงพอ

### 2.1 Title และ Description ตามหน้า (Dynamic)

- ตอนนี้ทั้งไซต์ใช้ **title และ description เดียวจาก index.html**
- หน้าเช่น `/history`, `/awards`, `/news-article/123` ไม่ได้ตั้ง:
  - `<title>` เฉพาะหน้า (เช่น "เกียรติรางวัล | Happy MPM")
  - `<meta name="description">` เฉพาะหน้า
- ผล: ในผลค้นหาและแท็บเบราว์เซอร์ทุกหน้าจะแสดงเหมือนกัน และไม่ดึงข้อความจากเนื้อหาจริง

**แนวทางแก้:** ใช้ `react-helmet` หรือ `react-helmet-async` ในแต่ละ route/หน้า เพื่อตั้ง `document.title` และ `<meta name="description">` ตามเนื้อหาจริง (และถ้าต้องการ og/twitter ด้วย ให้ตั้งใน Helmet เดียวกัน)

### 2.2 Open Graph (og:)

- ไม่มี meta ชุด `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- เวลาแชร์ลิงก์ใน Facebook, Line, LinkedIn ฯลฯ จะดึงข้อความ/รูปจากค่า default หรือไม่สวย
- **ควรมีอย่างน้อย:** `og:title`, `og:description`, `og:image`, `og:url`, `og:type` (และ `og:locale` / `og:image:width` / `og:image:height` ถ้าต้องการละเอียด)

### 2.3 Twitter Card

- ไม่มี `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- แชร์บน Twitter/X จะไม่มี card พิเศษ

### 2.4 Sitemap (sitemap.xml)

- ไม่มีไฟล์ `sitemap.xml` ใน `public/`
- Search engine ต้องค้นหา URL จากการ crawl และลิงก์ภายในเท่านั้น
- **ข้อเสนอ:** สร้าง sitemap (แบบ static หรือ generate จาก routes + CMS) ใส่ URL หลัก เช่น หน้าแรก, ประวัติ, วิสัยทัศน์, รางวัล, ข่าว, บทความ (ถ้ามี slug จาก CMS)

### 2.5 Canonical URL

- ไม่มี `<link rel="canonical" href="...">`
- ถ้ามีหลาย URL ชี้หน้าเดียวกัน (เช่น query string, trailing slash) อาจเกิด duplicate content
- **ข้อเสนอ:** ใส่ canonical ในทุกหน้า ให้ชี้ไปที่ URL หลักที่ต้องการให้ index

### 2.6 หลายภาษา (hreflang)

- รองรับหลายภาษา (th, en, id, ms, ar) แต่ไม่มี `<link rel="alternate" hreflang="th" href="...">` ฯลฯ
- Google อาจไม่จับคู่หน้าภาษาไทย/อังกฤษเป็น alternate ของกันและกัน
- **ข้อเสนอ:** ใส่ hreflang สำหรับหน้าที่มีเนื้อหาคนละภาษา (หรืออย่างน้อย th/en)

### 2.7 Structured Data (JSON-LD)

- ไม่มี Schema.org (Organization, WebSite, Article, BreadcrumbList ฯลฯ)
- ผลค้นหาจะไม่มี rich result (เช่น breadcrumb, site links)
- **ข้อเสนอ:** เพิ่ม JSON-LD สำหรับ Organization + WebSite ในหน้าแรก และ Article ในหน้าข่าว/บทความ

### 2.8 การตั้งค่า Title/Description จากโค้ด

- ไม่มีใช้ `react-helmet` / `react-helmet-async` ในโปรเจกต์
- ไม่มีการเรียก `document.title` ตาม route หรือตามข้อมูลจาก API/CMS (ยกเว้นใน iframe preview ของ ArticleDetailEditor ซึ่งไม่เกี่ยวกับหน้าจริง)
- ดังนั้น **ยังไม่มีการตั้งค่า SEO ตามหน้าจากโค้ด**

---

## 3. โครงสร้าง URL และการ index

- ใช้ React Router (path แบบ SPA): หน้าหลักใช้ `/`, หน้าอื่นใช้ path ชัดเจน เช่น `/history`, `/awards`, `/news-article/:id`
- ถ้า deploy แบบ SPA (ทุก path fallback ไป index.html) แล้วตั้ง rewrite ถูกต้อง (เช่นใน Vercel/Netlify) bot จะเข้าถึง URL ต่างๆ ได้
- **หมายเหตุ:** Search engine ต้องรัน JavaScript เพื่อเห็นเนื้อหา (SPA) ดังนั้นการมี meta ที่ถูกต้องใน HTML (หรือที่ inject จาก Helmet ตาม route) จะช่วยให้ bot ที่ render ได้เห็น title/description ที่ตรงกับหน้า

---

## 4. สรุป Checklist

| รายการ | มีแล้ว | ควรทำ |
|--------|--------|--------|
| ตั้ง title/description หลักใน index.html เป็น Happy MPM | ❌ | ✅ |
| เปลี่ยน title/description ตามหน้า (dynamic) | ❌ | ✅ |
| Open Graph (og:title, og:description, og:image, og:url) | ❌ | ✅ |
| Twitter Card | ❌ | ✅ (ถ้าใช้แชร์บน Twitter) |
| robots.txt | ✅ | พิจารณา Disallow /admin ถ้าจำเป็น |
| sitemap.xml | ❌ | ✅ |
| Canonical URL | ❌ | ✅ |
| hreflang (th, en, …) | ❌ | ✅ (อย่างน้อย th, en) |
| JSON-LD (Organization, WebSite, Article) | ❌ | ✅ (อย่างน้อย Organization + WebSite) |
| ปรับ manifest.json (name, short_name) | ❌ | ✅ |

---

## 5. แนวทางดำเนินการ (ลำดับที่แนะนำ)

1. **เร็วและกระทบทุกหน้า**
   - แก้ `public/index.html`: ตั้ง `<title>` และ `<meta name="description">` เป็นค่าเริ่มต้นของแบรนด์ (เช่น "Happy MPM | เราพัฒนาชีวิต บนธุรกิจที่ดีกว่า")
   - แก้ `public/manifest.json`: ตั้ง `name` / `short_name` เป็น Happy MPM

2. **ต่อด้วย dynamic ตามหน้า**
   - ติดตั้ง `react-helmet-async`
   - สร้าง component หรือ hook สำหรับตั้ง title + description (และถ้าต้องการ og/twitter) ตาม route และตามข้อมูลจาก CMS (ข่าว, รางวัล ฯลฯ)
   - ใส่ `<Helmet>` ใน layout หรือในแต่ละหน้า/route

3. **ช่วยการ index และการแชร์**
   - สร้าง `public/sitemap.xml` (หรือระบบ generate sitemap จาก routes + ข้อมูลจาก CMS)
   - ใส่ canonical ในแต่ละหน้า (ผ่าน Helmet)
   - ใส่ meta Open Graph และ Twitter Card (ผ่าน Helmet, อาจดึงรูปจาก CMS สำหรับหน้ารายละเอียด)

4. **หลายภาษาและ rich result**
   - ใส่ hreflang ในหน้าที่มี alternate หลายภาษา
   - เพิ่ม JSON-LD สำหรับ Organization, WebSite และถ้ามีหน้าข่าว/บทความให้เพิ่ม Article

ถ้าต้องการให้ช่วยลงมือทำตามข้อ 1–4 เป็นขั้นตอน (เช่น เริ่มจากแก้ index.html + manifest แล้วค่อยเพิ่ม Helmet + sitemap) บอกได้เลยว่าจะเริ่มจากข้อไหน
