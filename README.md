# Management System Customer Data Dashboard

หน้าเว็บระบบจัดการข้อมูลลูกค้าที่เผยแพร่ด้วย GitHub Pages และเชื่อม Google Sheet ผ่าน Google Apps Script Web App

- เว็บไซต์: https://pasakornudomm-hash.github.io/-Management-System-Customer-Data-Dashboard/
- Apps Script Web App: https://script.google.com/macros/s/AKfycbybY5j57KHPks8Grhm713wCc4f84pm9E9bPWehXHnT-mUf9TmRmM7uZ3FDQO98cDA/exec

## โครงสร้าง

- `index.html` มีหน้าตา ฟังก์ชัน และโค้ดเชื่อม API ทั้งหมด
- Google Apps Script เป็น backend สำหรับอ่าน/เพิ่ม/แก้ไข/ลบข้อมูลและประวัติใน Google Sheet
- หน้าเว็บใช้ข้อมูล cache ในเครื่องแสดงก่อน แล้วโหลดข้อมูลล่าสุดแบบไม่บล็อกหน้า

## Deploy Google Apps Script

1. เปิดโปรเจกต์ Google Apps Script ที่เชื่อมกับ Google Sheet ของระบบ
2. ตรวจว่า `doPost(e)` รับ JSON แบบ `{ "action": "...", ...payload }` และส่งผลลัพธ์ JSON ที่มี `ok: true` หรือ `ok: false`
3. ไปที่ **Deploy > Manage deployments**
4. กดไอคอนดินสอของ Web app เดิม เลือก **New version** แล้วกด **Deploy**
5. ตั้ง **Execute as: Me** และ **Who has access: Anyone**
6. ใช้ URL ที่ลงท้าย `/exec` เท่านั้น หาก URL เปลี่ยน ให้อัปเดตค่า `GAS_WEB_APP_URL` ใน `index.html`

ไม่ต้องใช้ JSONP สำหรับ deployment นี้ ปลายทาง POST ตอบ JSON, รองรับ CORS และ redirect ไป `script.googleusercontent.com` โดยอัตโนมัติ หน้าเว็บใช้ `Content-Type: text/plain;charset=utf-8` เพื่อไม่ให้เกิด preflight ที่ไม่จำเป็น

## Publish GitHub Pages

1. อัปเดต `index.html` บน branch `main`
2. เปิด **Settings > Pages** ของ repository
3. เลือก **Deploy from a branch**, branch `main`, folder `/ (root)`
4. รอ workflow Pages สำเร็จ แล้วเปิด URL เว็บไซต์ด้านบนและกดรีเฟรชแบบไม่ใช้ cache หากยังเห็นไฟล์เก่า

## การทำงานด้านความเร็วและความเสถียร

- รวมคำขออ่านที่กำลังทำงานซ้ำกัน และ cache ผลอ่านระยะสั้น
- โหลดข้อมูลหลักและประวัติแบบขนานเมื่อปลอดภัย
- timeout และ retry เฉพาะคำขออ่าน เพื่อไม่ให้สร้างข้อมูลซ้ำ
- debounce ช่องค้นหาและตัวกรองข้อความ
- แสดงตารางและประวัติเป็นช่วงย่อยผ่าน animation frame เพื่อลดอาการหน้าเว็บค้าง
- เขียน cache ในเครื่องช่วงที่ browser ว่าง
- ล็อกปุ่มระหว่างโหลด บันทึก และลบ เพื่อป้องกันการกดซ้ำ
- ไม่มีช่องอัปโหลดไฟล์ในระบบเวอร์ชันปัจจุบัน จึงไม่มีไฟล์ภาพหรือไฟล์แนบให้บีบอัด

## ตรวจปัญหาเชื่อมต่อ

- ตรวจว่า Web App เปิดแบบ `Anyone` และ URL ลงท้าย `/exec`
- เปิด URL Apps Script โดยตรงเพื่อยืนยันว่า deployment ยังทำงาน
- ตรวจว่า POST ตอบ `application/json` และผลลัพธ์มี `ok`
- หากแก้ `Code.gs` ต้องสร้าง version ใหม่ใน deployment เดิม การกด Save อย่างเดียวไม่อัปเดต Web App
