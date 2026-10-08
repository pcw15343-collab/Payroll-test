# ระบบเงินเดือน (Payroll) บน Google Apps Script

ระบบเงินเดือนพนักงานรายชั่วโมง/Part-time: ทะเบียนพนักงาน/ตำแหน่ง · ใบลงเวลา (กรอกเอง/ให้ AI อ่านรูป) · คำนวณค่าแรง + OT · สลิป PDF · ทะเบียนเงินได้
ขอบเขต: ยังไม่คำนวณประกันสังคมและภาษีขั้นบันได (ภาษีหัก ณ ที่จ่าย กรอกเองในเมนู "ปรับเงินเดือน / โบนัส")

ไม่มีชื่อบริษัทหรือโลโก้ฝังในโค้ด — ตั้งเองในเมนู **ตั้งค่าระบบ** หลังติดตั้ง

## ไฟล์
| ไฟล์ | คืออะไร |
|---|---|
| `Code.gs` | ฝั่งเซิร์ฟเวอร์: ชีต/โฟลเดอร์, API หน้าเว็บ, คำนวณเงิน, สลิป, โลโก้ |
| `TimesheetOcr.gs` | AI (Gemini) อ่านรูปใบลงเวลาฟอร์ม `PAY-TS` |
| `Index.html` | หน้าเว็บทั้งระบบ |
| `appsscript.json` · `.claspignore` | manifest · อัปขึ้น Google เฉพาะ 4 ไฟล์ข้างบน |
| `tools/make-preview.js` | สร้าง `preview.html` เปิดดูหน้าเว็บด้วยข้อมูลสมมติ ไม่ต้อง deploy |
| `tests/` | ชุดทดสอบ (ไม่ขึ้น Google) |
| `branding/logo_IDC.webp` | โลโก้สำหรับอัปโหลดในหน้าตั้งค่า |

## ติดตั้ง
1. สร้างโปรเจกต์ Apps Script ใหม่ด้วยบัญชี Google ที่จะเป็นเจ้าของข้อมูลเงินเดือน แล้ว push:
   ```bash
   npx clasp login
   cp appsscript.json appsscript.json.bak            # create-script เขียนทับ manifest
   npx clasp create-script --type standalone --title "ระบบเงินเดือน"
   cp appsscript.json.bak appsscript.json
   npx clasp push -f                                 # ต้องเห็น "Pushed 4 files"
   npx clasp create-deployment -d "ติดตั้งครั้งแรก"     # เก็บ deploymentId ไว้ redeploy ทับ URL เดิม
   ```
   ไม่ใช้ clasp: script.google.com → New project → วาง `Code.gs` · สร้างไฟล์ script `TimesheetOcr` · สร้างไฟล์ HTML `Index` · แก้ manifest ตาม `appsscript.json`
2. เปิดเอดิเตอร์ → Run `setup` → กดอนุญาตสิทธิ์ (ก่อนขั้นนี้ URL /exec จะตอบ 403)
3. เปิด URL /exec → ระบบพาไปหน้า **ตั้งค่าระบบ**: อัปโลโก้ (`branding/logo_IDC.webp`) · กรอกข้อมูลบริษัท · ตั้งกติกาคำนวณ (ชม.ปกติ/วัน, ตัวคูณ OT, เวลาพัก, OT ขั้นต่ำ)
4. เมนู ตำแหน่งงาน → พนักงาน → พิมพ์ใบลงเวลา
5. (ถ้าใช้ AI อ่านใบลงเวลา) สร้างคีย์ที่ https://aistudio.google.com/apikey ในโปรเจกต์ใหม่ → Project Settings → Script Properties → `GEMINI_API_KEY` → หน้าตั้งค่ากด **ทดสอบคีย์ AI** ต้องขึ้น ✓

อัปเดตเว็บเดิม: `npx clasp push -f` (นับ "Pushed 4 files") แล้ว `npx clasp deploy -i <deploymentId> -d "..."` — ไม่ใส่ `-i` จะได้ URL ใหม่

## ทดสอบ
```bash
node tests/test-core.js Code.gs                          # สูตรคำนวณ 24 เคส
node tests/test-ocr-errors.js TimesheetOcr.gs Code.gs    # การเรียก Gemini 19 เคส (ไม่ยิงเน็ตจริง)
node tests/check-template-ui.js .                        # หน้าเว็บ 24 จุดด้วย Playwright
```
`check-template-ui.js` ต้องมี playwright-core — ถ้าไม่ได้ติดตั้งแบบปกติให้ตั้ง `PLAYWRIGHT_CORE=<path playwright-core>` และ `CHROME=<path chrome>`
