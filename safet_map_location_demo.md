# สรุปการทำ Map + Location จำลองสำหรับเว็บ SafeT

## แนวคิดหลัก

ส่วน Map + Location ของเว็บจำลอง SafeT จะใช้ตำแหน่งจากเครื่องที่เปิดเว็บอยู่ เช่น MacBook ที่ใช้เปิด Dashboard โดยไม่ต้องต่อ GPS Module, ESP32 หรืออุปกรณ์จริง

ระบบจะใช้ Browser Geolocation API เพื่ออ่านตำแหน่งปัจจุบันของผู้ใช้ แล้วนำค่าพิกัด latitude และ longitude ไปแสดงบนแผนที่ด้วย Leaflet.js และ OpenStreetMap

โครงสร้างโดยรวม:

```text
MacBook ที่เปิดเว็บ
↓
Browser ขอสิทธิ์เข้าถึงตำแหน่ง
↓
เว็บอ่านค่า latitude / longitude
↓
Leaflet.js แสดงแผนที่
↓
แสดงหมุดตำแหน่ง SafeT บน Dashboard
```

---

## เทคโนโลยีที่ใช้

| ส่วนที่ต้องการ | เทคโนโลยีที่ใช้ | ค่าใช้จ่าย |
|---|---|---|
| อ่านตำแหน่งจาก Mac | Browser Geolocation API | ฟรี |
| แสดงแผนที่ | Leaflet.js | ฟรี |
| พื้นหลังแผนที่ | OpenStreetMap | ฟรีตามเงื่อนไขการใช้งาน |
| ฝากเว็บออนไลน์ | GitHub Pages | ฟรี |
| สถานะอุปกรณ์ | ข้อมูลจำลองใน JavaScript | ฟรี |

---

## ความสามารถที่ต้องการในหน้าเว็บ

หน้า Dashboard ควรมีความสามารถดังนี้

1. แสดงแผนที่จริง
2. อ่านตำแหน่งปัจจุบันจาก Mac ที่เปิดเว็บ
3. แสดงหมุดตำแหน่งของ SafeT 01 บนแผนที่
4. แสดงตำแหน่งจำลองของ SafeT 02 ให้อยู่ใกล้กับ SafeT 01
5. แสดงข้อมูล latitude และ longitude
6. แสดงค่าความแม่นยำของตำแหน่ง
7. แสดงเวลาที่อัปเดตตำแหน่งล่าสุด
8. มีปุ่มอัปเดตตำแหน่ง
9. มีปุ่มจำลองสถานะ SOS
10. แสดงสถานะอุปกรณ์ เช่น Online, SOS, สถานะจำลอง

---

## ข้อจำกัดที่ควรระบุในการนำเสนอ

MacBook ส่วนใหญ่ไม่มี GPS จริงเหมือนโทรศัพท์มือถือ ดังนั้นตำแหน่งที่ได้จากเว็บอาจอ้างอิงจาก Wi-Fi, IP Address และบริการระบุตำแหน่งของระบบ macOS

ดังนั้นความแม่นยำอาจคลาดเคลื่อนได้ แต่เพียงพอสำหรับการทำเว็บจำลองและการนำเสนอแนวคิดระบบ SafeT

หากต้องการตำแหน่งที่แม่นยำขึ้น สามารถเปิดเว็บบนสมาร์ตโฟนแทน MacBook ได้ เพราะสมาร์ตโฟนมักมี GPS ในตัว

---

## เงื่อนไขสำคัญในการใช้งาน

Browser Geolocation API ต้องใช้ภายใต้เงื่อนไขต่อไปนี้

1. ผู้ใช้ต้องกดอนุญาตให้เว็บเข้าถึงตำแหน่ง
2. ควรเปิดผ่าน `localhost` หรือ `https`
3. ไม่ควรเปิดไฟล์ด้วยการ double click แบบ `file://`
4. หากใช้บน GitHub Pages จะใช้งานได้ เพราะ GitHub Pages เป็น HTTPS
5. หากใช้ใน VS Code ควรเปิดผ่าน Live Server

ตัวอย่าง URL ที่เหมาะสม:

```text
http://localhost:5500
```

หรือ

```text
https://username.github.io/project-name
```

---

## โครงสร้างข้อมูลจำลอง

ตัวอย่างข้อมูลที่แสดงใน Dashboard:

```json
{
  "devices": {
    "safet_01": {
      "name": "SafeT 01",
      "user": "คุณตา",
      "locationSource": "Mac Geolocation",
      "lat": "ค่าจาก Browser",
      "lng": "ค่าจาก Browser",
      "accuracy": "ค่าจาก Browser",
      "status": "Online",
      "sos": false,
      "lastUpdate": "เวลาปัจจุบัน"
    },
    "safet_02": {
      "name": "SafeT 02",
      "user": "หลานชาย",
      "locationSource": "Simulated Location",
      "lat": "ตำแหน่งจำลองใกล้ SafeT 01",
      "lng": "ตำแหน่งจำลองใกล้ SafeT 01",
      "accuracy": "จำลอง",
      "status": "Demo",
      "sos": false,
      "lastUpdate": "เวลาปัจจุบัน"
    }
  }
}
```

---

## แนวทางการทำงานของ SafeT 01

SafeT 01 ใช้ตำแหน่งจริงจากเครื่องที่เปิดเว็บอยู่

ขั้นตอนการทำงาน:

1. เมื่อเปิดหน้า Dashboard ให้เว็บเรียก `navigator.geolocation.getCurrentPosition()`
2. Browser แสดงกล่องขออนุญาตเข้าถึงตำแหน่ง
3. เมื่อผู้ใช้กด Allow เว็บจะได้ข้อมูลตำแหน่ง
4. เว็บนำค่า latitude และ longitude ไปวางหมุดบนแผนที่
5. เว็บแสดงค่าพิกัดใน Card ของ SafeT 01
6. เว็บแสดงค่าความแม่นยำของตำแหน่ง
7. เว็บบันทึกเวลาการอัปเดตล่าสุด

ข้อมูลที่อ่านได้จาก Browser:

```text
position.coords.latitude
position.coords.longitude
position.coords.accuracy
```

---

## แนวทางการทำงานของ SafeT 02

SafeT 02 เป็นตำแหน่งจำลอง เพื่อใช้แสดงว่า Dashboard รองรับอุปกรณ์หลายตัว

ตำแหน่งของ SafeT 02 สามารถคำนวณจาก SafeT 01 โดยขยับพิกัดเล็กน้อย เช่น

```javascript
const lat2 = lat + 0.001;
const lng2 = lng + 0.001;
```

เมื่อแสดงบนแผนที่ SafeT 02 จะอยู่ใกล้ SafeT 01 ทำให้การนำเสนอดูเหมือนมีอุปกรณ์ 2 ตัวในระบบเดียวกัน

---

## ไลบรารีแผนที่

ใช้ Leaflet.js และ OpenStreetMap

ส่วนที่ต้องใส่ใน `<head>`:

```html
<link
  rel="stylesheet"
  href="https://unpkg.com/leaflet/dist/leaflet.css"
/>
```

ส่วนที่ต้องใส่ก่อนปิด `</body>`:

```html
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
```

---

## การสร้างแผนที่เริ่มต้น

```javascript
let map = L.map("map").setView([13.7563, 100.5018], 13);

L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: "© OpenStreetMap contributors"
}).addTo(map);
```

ค่า `[13.7563, 100.5018]` เป็นตำแหน่งเริ่มต้นของกรุงเทพฯ ใช้เป็นค่า default ก่อนที่เว็บจะอ่านตำแหน่งจริงได้

---

## การอ่านตำแหน่งจาก Browser

```javascript
navigator.geolocation.getCurrentPosition(
  function(position) {
    const lat = position.coords.latitude;
    const lng = position.coords.longitude;
    const accuracy = position.coords.accuracy;

    console.log(lat, lng, accuracy);
  },
  function(error) {
    console.log(error);
  },
  {
    enableHighAccuracy: true,
    timeout: 10000,
    maximumAge: 0
  }
);
```

---

## การวางหมุดบนแผนที่

```javascript
let markerSafeT01 = L.marker([lat, lng]).addTo(map);

markerSafeT01.bindPopup(`
  <b>SafeT 01</b><br>
  ผู้ใช้งาน: คุณตา<br>
  สถานะ: Online
`).openPopup();

map.setView([lat, lng], 17);
```

---

## การแสดงวงรัศมีความแม่นยำ

```javascript
let accuracyCircle = L.circle([lat, lng], {
  radius: accuracy
}).addTo(map);
```

วงกลมนี้ช่วยให้ผู้ชมเข้าใจว่าตำแหน่งที่ได้อาจมีความคลาดเคลื่อนตามค่าความแม่นยำที่ Browser ส่งมา

---

## การอัปเดตตำแหน่งเดิม ไม่สร้างหมุดซ้ำ

ควรตรวจสอบก่อนว่ามีหมุดอยู่แล้วหรือไม่

```javascript
if (markerSafeT01) {
  markerSafeT01.setLatLng([lat, lng]);
} else {
  markerSafeT01 = L.marker([lat, lng]).addTo(map);
}
```

---

## ปุ่มจำลองสถานะ SOS

สามารถทำปุ่ม SOS โดยใช้ตัวแปรจำลอง เช่น

```javascript
let sosActive = false;

function toggleSOS() {
  sosActive = !sosActive;

  if (sosActive) {
    statusText = "SOS";
  } else {
    statusText = "Online";
  }
}
```

เมื่อกดปุ่ม SOS ให้เปลี่ยนสถานะใน Card และ Popup บนแผนที่

---

## การออกแบบ UI ที่แนะนำ

หน้า Map + Location ควรแบ่งเป็น 2 ส่วน

### 1. Device Status Panel

แสดงข้อมูลของอุปกรณ์

ตัวอย่างข้อมูล:

```text
SafeT 01
ผู้ใช้งาน: คุณตา
สถานะ: Online
Latitude: 6.xxxxxx
Longitude: 101.xxxxxx
Accuracy: 50 เมตร
Last Update: 28/6/2569 08:30
[อัปเดตตำแหน่ง]
[จำลองปุ่ม SOS]
```

```text
SafeT 02
ผู้ใช้งาน: หลานชาย
สถานะ: Demo
ตำแหน่งจำลองใกล้ SafeT 01
```

### 2. Map Panel

แสดงแผนที่พร้อมหมุด

ควรมีองค์ประกอบดังนี้

- หมุด SafeT 01
- หมุด SafeT 02
- Popup แสดงชื่ออุปกรณ์และผู้ใช้งาน
- วงกลมแสดงความแม่นยำของตำแหน่ง
- แผนที่สามารถ zoom และ drag ได้

---

## ข้อความอธิบายสำหรับใส่ในเว็บ

สามารถใส่ข้อความสั้น ๆ ในหน้า Dashboard ได้ว่า

```text
ตำแหน่งที่แสดงในหน้านี้เป็นตำแหน่งจากเครื่องที่เปิดเว็บอยู่ ใช้สำหรับจำลองการทำงานของระบบ SafeT Dashboard โดยยังไม่ได้เชื่อมต่อกับอุปกรณ์ IoT จริง
```

หรือ

```text
Demo Mode: ระบบใช้ตำแหน่งจาก Browser ของผู้ใช้งาน เพื่อจำลองการติดตามตำแหน่งของอุปกรณ์ SafeT
```

---

## สรุปการใช้งานสำหรับโปรเจกต์ SafeT

สำหรับเว็บจำลอง SafeT ให้ใช้แนวทางนี้:

```text
Geolocation API
+
Leaflet.js
+
OpenStreetMap
+
ข้อมูลสถานะจำลอง SafeT 01 / SafeT 02
```

ข้อดีของแนวทางนี้:

1. ฟรี
2. ทำง่าย
3. ไม่ต้องใช้อุปกรณ์ GPS จริง
4. ไม่ต้องใช้ Firebase
5. ไม่ต้องใช้ ESP32
6. แสดงแผนที่จริงได้
7. ใช้ตำแหน่งจริงจากเครื่องที่เปิดเว็บได้
8. เหมาะสำหรับเว็บต้นแบบและการนำเสนอ

ข้อจำกัด:

1. ตำแหน่งจาก Mac อาจไม่แม่นเท่า GPS จริง
2. ต้องขอสิทธิ์เข้าถึงตำแหน่งจากผู้ใช้
3. ต้องเปิดผ่าน HTTPS หรือ localhost
4. SafeT 02 เป็นตำแหน่งจำลอง ไม่ใช่อุปกรณ์จริง

---

## คำสั่งสำหรับ Codex

ให้เพิ่มหน้า Map + Location ในเว็บ SafeT Dashboard โดยใช้ Browser Geolocation API เพื่ออ่านตำแหน่งจากเครื่องที่เปิดเว็บ และใช้ Leaflet.js ร่วมกับ OpenStreetMap เพื่อแสดงแผนที่

รายละเอียดที่ต้องทำ:

1. เพิ่มแผนที่ด้วย Leaflet.js
2. อ่านตำแหน่งปัจจุบันด้วย `navigator.geolocation.getCurrentPosition()`
3. แสดงหมุด SafeT 01 จากตำแหน่งจริงของเครื่องที่เปิดเว็บ
4. แสดงหมุด SafeT 02 เป็นตำแหน่งจำลองใกล้ SafeT 01
5. แสดง Card ข้อมูลอุปกรณ์ SafeT 01 และ SafeT 02
6. SafeT 01 แสดง latitude, longitude, accuracy และ last update
7. SafeT 02 แสดงเป็น demo simulated location
8. เพิ่มปุ่มอัปเดตตำแหน่ง
9. เพิ่มปุ่มจำลอง SOS
10. เพิ่ม popup บนหมุดแผนที่
11. เพิ่มวงกลม accuracy รอบตำแหน่ง SafeT 01
12. รองรับการเปิดใช้งานผ่าน localhost หรือ GitHub Pages
13. หากผู้ใช้ไม่อนุญาตตำแหน่ง ให้แสดงข้อความแจ้งเตือนอย่างสุภาพ
