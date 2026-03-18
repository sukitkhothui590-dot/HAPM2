# 🌐 ระบบ Translations จาก CMS

ระบบนี้ช่วยให้คุณสามารถจัดการ translations ผ่านหลังบ้าน (CMS) โดยไม่ต้องแก้ไขโค้ด

## 📋 โครงสร้างระบบ

### 1. **Database Structure**
Translations ถูกเก็บในตาราง `site_content` โดยใช้ `section_id = 'translations'`

**โครงสร้างข้อมูล:**
```
section_id: 'translations'
content_key: 'common' | 'nav' | 'promotions' | 'joinBusiness' | ...
content_value_th: JSON string (Thai translations)
content_value_en: JSON string (English translations)
content_value_id: JSON string (Indonesian translations)
content_value_ms: JSON string (Malay translations)
content_value_ar: JSON string (Arabic translations)
content_type: 'json'
is_active: true
```

### 2. **การทำงาน**

1. **Hook: `useTranslations`** (`src/hooks/useTranslations.js`)
   - ดึง translations จาก Supabase CMS
   - Merge กับ static translations.js (เป็น fallback)
   - Subscribe realtime updates อัตโนมัติ
   - Cache ข้อมูล 5 นาที เพื่อประสิทธิภาพ

2. **Fallback System**
   - ถ้า CMS มีข้อมูล → ใช้ CMS (CMS ครอบทับ static)
   - ถ้า CMS ไม่มีข้อมูล → ใช้ static translations.js
   - ถ้า CMS มีบางส่วน → Merge กัน (CMS ครอบทับส่วนที่ซ้ำ)

### 3. **ตัวอย่างการใช้งาน**

#### ในหลังบ้าน (Admin Panel)

เพิ่ม translations ใหม่:

```sql
INSERT INTO site_content (
  section_id,
  content_key,
  content_value_th,
  content_value_en,
  content_value_id,
  content_value_ms,
  content_value_ar,
  content_type,
  is_active
) VALUES (
  'translations',
  'common',
  '{"viewDetails": "ดูรายละเอียด →", "backToHome": "กลับหน้าแรก"}',
  '{"viewDetails": "View Details →", "backToHome": "Back to Home"}',
  '{"viewDetails": "Lihat Detail →", "backToHome": "Kembali ke Beranda"}',
  '{"viewDetails": "Lihat Butiran →", "backToHome": "Kembali ke Laman Utama"}',
  '{"viewDetails": "عرض التفاصيل →", "backToHome": "العودة إلى الصفحة الرئيسية"}',
  'json',
  true
);
```

หรืออัปเดตผ่าน Admin Panel:
- ไปที่ Content Management
- เลือก Section: `translations`
- เลือก Content Key: เช่น `common`, `nav`, `promotions`
- แก้ไข translations ในแต่ละภาษา
- กดบันทึก

#### ใน Frontend Code

```javascript
// AppRoutes.jsx - ใช้ hook
import { useTranslations } from "./hooks/useTranslations";

const AppRoutes = () => {
  const [language, setLanguage] = useState('th');
  const { t } = useTranslations(language);
  
  // t จะอัปเดตอัตโนมัติเมื่อ:
  // 1. เปลี่ยนภาษา
  // 2. มีการอัปเดตใน CMS (realtime)
  
  return <App language={language} t={t} />;
};
```

### 4. **โครงสร้าง JSON ที่ถูกต้อง**

#### ตัวอย่าง: `common` translations

```json
{
  "viewDetails": "ดูรายละเอียด →",
  "backToHome": "กลับหน้าแรก",
  "search": "ค้นหา",
  "loading": "กำลังโหลด...",
  "noResults": "ไม่พบข้อมูล"
}
```

#### ตัวอย่าง: `promotions` translations (nested structure)

```json
{
  "categories": {
    "all": "ทั้งหมด",
    "package": "แพ็กเกจ",
    "bonus": "โบนัส"
  },
  "starterPackage": {
    "title": "แพ็กเกจเริ่มต้น",
    "description": "แพ็กเกจสำหรับผู้เริ่มต้น",
    "duration": "ปัจจุบัน - 31 ธ.ค. 2025"
  }
}
```

### 5. **การทำงานของ Realtime Updates**

เมื่อมีการเปลี่ยนแปลงใน `site_content` table (INSERT, UPDATE, DELETE):
1. Supabase Realtime จะส่ง notification
2. `useTranslations` hook จะ detect การเปลี่ยนแปลง
3. Clear cache และ reload translations อัตโนมัติ
4. Frontend จะอัปเดตทันที (ไม่ต้อง refresh หน้า)

### 6. **Priority Order**

1. **CMS Translations** (priority สูงสุด)
2. **Static Translations** (fallback)

ถ้า CMS มี `common.viewDetails = "ดูรายละเอียด →"` แต่ static มี `common.viewDetails = "ดู"` 
→ จะใช้ `"ดูรายละเอียด →"` จาก CMS

### 7. **การจัดการใน Admin Panel**

#### เพิ่ม Translation ใหม่:

1. ไปที่ Admin Panel → Content Management
2. เลือก Section: `translations`
3. กด "Add New Content"
4. ตั้งค่า:
   - **Content Key**: เช่น `newSection`
   - **Content Type**: `json`
   - **Thai**: `{"title": "หัวข้อ", "subtitle": "คำบรรยาย"}`
   - **English**: `{"title": "Title", "subtitle": "Subtitle"}`
   - ... (ภาษาอื่นๆ)
5. กด Save

#### แก้ไข Translation:

1. ไปที่ Admin Panel → Content Management
2. เลือก Section: `translations`
3. ค้นหา Content Key ที่ต้องการแก้
4. แก้ไข translations ในภาษาใดภาษาหนึ่ง
5. กด Save → Frontend จะอัปเดตทันที

### 8. **ข้อดีของระบบนี้**

✅ **ไม่ต้องแก้โค้ด** - เพิ่ม/แก้ translations ได้ผ่านหลังบ้าน  
✅ **Realtime Updates** - อัปเดตทันทีไม่ต้อง refresh  
✅ **Safe Fallback** - ถ้า CMS ลง ยังใช้ static translations ได้  
✅ **Performance** - มี cache 5 นาที  
✅ **Flexible** - สามารถ override static translations ได้บางส่วน  

### 9. **Migration Guide**

หากต้องการย้าย translations จาก `translations.js` ไป CMS:

1. แปลง translations.js เป็น JSON format
2. อัปโหลดผ่าน Admin Panel หรือ SQL script
3. Frontend จะใช้ CMS translations อัตโนมัติ

### 10. **ตัวอย่าง SQL Script**

```sql
-- ย้าย common translations ไป CMS
INSERT INTO site_content (
  section_id, content_key, content_type, is_active,
  content_value_th, content_value_en, content_value_id, content_value_ms, content_value_ar
) VALUES (
  'translations',
  'common',
  'json',
  true,
  '{"viewDetails":"ดูรายละเอียด →","backToHome":"กลับหน้าแรก"}',
  '{"viewDetails":"View Details →","backToHome":"Back to Home"}',
  '{"viewDetails":"Lihat Detail →","backToHome":"Kembali ke Beranda"}',
  '{"viewDetails":"Lihat Butiran →","backToHome":"Kembali ke Laman Utama"}',
  '{"viewDetails":"عرض التفاصيل →","backToHome":"العودة إلى الصفحة الرئيسية"}'
);
```

---

**หมายเหตุ:** ระบบนี้ทำงานควบคู่กับ static translations.js โดย CMS translations จะมี priority สูงกว่า หากต้องการใช้เฉพาะ CMS translations ลบ static translations.js ออกได้ (แต่ไม่แนะนำ เพราะจะไม่มี fallback)

