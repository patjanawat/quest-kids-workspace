# Test Cases — Auth (PIN Gate)

> Phase 1.5 | อ้างอิง: `docs/specs/01-auth.md`  
> วันที่เขียน: 2026-04-19

---

## TC-01 Happy Path: กรอก PIN ถูกต้อง

**เงื่อนไขเริ่มต้น:** App เปิดหน้า parent area, `settings.pin = '1234'`, ไม่มี lockout

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | เปิดหน้า PIN | เห็น dots 4 ดวง (ว่างทั้งหมด), hint `"รหัสทดสอบ: 1234"` |
| 2 | กด 1 | dot แรก filled, entered = '1' |
| 3 | กด 2 | dot สองตัว filled, entered = '12' |
| 4 | กด 3 | dot สามตัว filled, entered = '123' |
| 5 | กด 4 | dot สี่ตัว filled → trigger PIN check อัตโนมัติ |
| 6 | PIN ถูก | navigate เข้า parent dashboard ทันที (ไม่ต้องกดปุ่ม OK) |

---

## TC-02 Error: กรอก PIN ผิด 1 ครั้ง

**เงื่อนไขเริ่มต้น:** `pinFailCount = 0`

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | กด 1, 2, 3, 5 (ผิด) | dots 4 ดวง filled → trigger check |
| 2 | PIN ผิด | shake animation เล่น (400ms, translate X [-10,10,-10,10,0]) |
| 3 | หลัง shake | error `"PIN ไม่ถูกต้อง ลองอีกครั้ง"` แสดงขึ้น (สี `#FFD4A8`) |
| 4 | หลัง 600ms | entered ล้างเป็น '' (dots ว่างทั้งหมด) |
| 5 | ตรวจ hint | hint `"รหัสทดสอบ: 1234"` ถูกซ่อน (error แสดงแทน) |
| 6 | ตรวจ store | `pinFailCount = 1` |

---

## TC-03 Lockout: กรอก PIN ผิด 3 ครั้ง

**เงื่อนไขเริ่มต้น:** `pinFailCount = 2` (ผิดมาแล้ว 2 ครั้ง)

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | กรอก PIN ผิดครั้งที่ 3 | `recordPinFail()` ถูกเรียก → `pinFailCount = 3` |
| 2 | store set lockout | `pinLockoutUntil` = เวลาปัจจุบัน + 5 นาที |
| 3 | UI detect lockout | lockout overlay ปรากฏทันที |
| 4 | overlay UI | ข้อความ `"ลองใหม่อีก X นาที"` (X = 5), sub-text `"กรุณารอสักครู่"` |
| 5 | keypad | ปุ่มทุกตัว disabled (กดไม่ได้ระหว่าง lockout) |
| 6 | หลัง 1 วินาที | countdown อัปเดต (ไม่หยุดนิ่ง) |

---

## TC-04 Lockout: countdown หมด → unlock

**เงื่อนไขเริ่มต้น:** อยู่ใน lockout state, `pinLockoutUntil` = อีก ~5 วินาที

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | รอ countdown ถึง 0 | overlay หาย |
| 2 | หลัง overlay หาย | กรอก PIN ได้ใหม่ (dots ว่าง, hint แสดง) |
| 3 | ตรวจ store | `pinFailCount = 0`, `pinLockoutUntil = null` |
| 4 | กรอก PIN ถูก | เข้า parent dashboard ได้ปกติ |

---

## TC-05 Delete Button

**เงื่อนไขเริ่มต้น:** หน้า PIN ปกติ

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | ก่อนกดตัวเลขใดเลย | ปุ่ม delete disabled (กดไม่ได้) |
| 2 | กด 1 | entered = '1', delete enabled |
| 3 | กด delete | entered = '', dot แรก empty, delete disabled อีกครั้ง |
| 4 | กด 1, 2, 3 | entered = '123' |
| 5 | กด delete | entered = '12', dot สามตัว empty |
| 6 | กด delete ขณะมี error | error ถูกล้าง, entered ลดลง 1 หลัก |

---

## TC-06 Edge: Lockout ค้างจาก session ก่อน (ยังไม่หมดเวลา)

**เงื่อนไขเริ่มต้น:** `pinLockoutUntil` = ในอนาคต (เช่น 3 นาทีข้างหน้า) ค้างอยู่ใน store

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | เปิด app / navigate มาที่ parent area | component mount → ตรวจ `pinLockoutUntil` ทันที |
| 2 | `Date.now() < Date.parse(pinLockoutUntil)` | lockout overlay แสดงทันที (ไม่รอให้กรอก PIN) |
| 3 | countdown | แสดงเวลาที่เหลือถูกต้อง (ไม่ reset เป็น 5 นาที) |

---

## TC-07 Edge: Lockout ค้างแต่เลยเวลาไปแล้ว

**เงื่อนไขเริ่มต้น:** `pinLockoutUntil` = เวลาในอดีต ค้างอยู่ใน store

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | เปิด app / navigate มาที่ parent area | component mount → ตรวจ `pinLockoutUntil` |
| 2 | `Date.now() >= Date.parse(pinLockoutUntil)` | `resetPinFails()` ถูกเรียกอัตโนมัติ |
| 3 | หน้า PIN | แสดงปกติ ไม่มี overlay, hint แสดง, กรอกได้เลย |

---

## TC-08 Edge: กลับมา parent area ใหม่ (local state ไม่ persist)

**เงื่อนไขเริ่มต้น:** เคยกรอก PIN ถูกและอยู่ใน parent dashboard แล้ว

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | กด back / navigate ออกจาก parent area | ออกไปหน้า kid home |
| 2 | navigate กลับมาที่ parent area | `unlocked` reset เป็น `false` (local state) |
| 3 | หน้าที่เห็น | PIN screen แสดงใหม่ ต้องกรอก PIN อีกครั้ง |

---

## TC-09 UI: Hint ซ่อนเมื่อมี error

**เงื่อนไขเริ่มต้น:** หน้า PIN ปกติ, hint แสดงอยู่

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | เริ่มต้น | hint `"รหัสทดสอบ: 1234"` แสดง |
| 2 | กรอก PIN ผิด | error `"PIN ไม่ถูกต้อง ลองอีกครั้ง"` แสดงแทน hint |
| 3 | ล้าง error (หลัง 600ms + entered ว่าง) | hint กลับมาแสดงอีกครั้ง |

---

## TC-10 UI: ปุ่มตัวเลข disabled เมื่อ entered ครบ 4 หลัก

**เงื่อนไขเริ่มต้น:** กำลังรอ PIN check (ช่วง 0–600ms หลัง entered = 4 หลัก)

| ขั้นตอน | Action | Expected |
|---|---|---|
| 1 | กด 1, 2, 3, 4 | ปุ่มทุกตัว disabled ทันทีที่ entered = 4 |
| 2 | พยายามกดเพิ่ม | ไม่มีผลใดๆ (entered ไม่เกิน 4) |

---

## TC-11 UI: Layout & Visual

**ตรวจด้วยตา (manual check)**

| รายการ | Expected |
|---|---|
| Header area | bg `#185FA5`, สูง ~40% screen |
| Lock icon | วงกลม `rgba(255,255,255,0.2)` ขนาด 80×80, emoji 🔒 ขนาด 36px |
| Title | `"โซนพ่อแม่"` สีขาว, bold, center |
| Dots | 4 ดวง, gap 16px, empty = border ขาว, filled = bg ขาว |
| Keypad area | bg `#FFFFFF`, สูง ~60% screen |
| Grid | 3 คอลัมน์, ปุ่มสูง 72px |
| ปุ่ม press | ripple `#EEF5FC` |
| Lockout overlay | bg `rgba(255,255,255,0.9)` ครอบ keypad |

---

## Regression Checklist

หลัง implement Auth ให้ตรวจว่า feature อื่นยังทำงานปกติ:

- [ ] Kid home screen โหลดได้ (ไม่ผ่าน PIN gate)
- [ ] Zustand store persist ข้ามครั้ง (reload app แล้ว `pinFailCount` ยังอยู่)
- [ ] `useDailyReset` hook ยังทำงาน (ไม่ถูก break โดยการแก้ store)
- [ ] `useTimer` hook ยังทำงาน
