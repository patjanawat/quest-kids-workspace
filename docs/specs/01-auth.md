# Spec: PIN Gate (Auth)

> version: 1.0 | สถานะ: Draft
> mockup reference: Parent PIN Screen

## Overview
หน้าจอ PIN ทำหน้าที่เป็น guard สำหรับ parent area ทั้งหมด ผู้ใช้ต้องกรอก 4-digit PIN ก่อนเข้าถึง parent dashboard, manage, library, settings — รองรับ lockout 5 นาทีหลังผิด 3 ครั้ง

## Screens / Components ที่เกี่ยวข้อง
- `app/parent/_layout.tsx` — layout ที่ wrap parent group ทั้งหมด (PIN gate logic อยู่ที่นี่)
- `components/PinDot.tsx` — dots 4 ดวงแสดงสถานะกรอก PIN
- `components/PinKeypad.tsx` — numpad 0-9 + delete button
- `components/PinScreen.tsx` — full screen PIN UI (render เมื่อ !unlocked)

## Data & State
### อ่านจาก store
- `useQuestStore(s => s.settings.pin)` — PIN ที่ถูกต้อง (string 4 หลัก)
- `useQuestStore(s => s.pinFailCount)` — จำนวนครั้งที่ผิดสะสม
- `useQuestStore(s => s.pinLockoutUntil)` — ISO date string หรือ null; ถ้า `Date.now() < Date.parse(pinLockoutUntil)` → ล็อก

### เรียก actions
- `recordPinFail()` — เมื่อ PIN ผิด (ทุกครั้ง)
- `resetPinFails()` — เมื่อ PIN ถูกต้อง (ก่อน navigate)

### Local State (ใน component)
```ts
const [entered, setEntered] = useState<string>('');  // PIN ที่กำลังกรอก (max 4 หลัก)
const [error, setError] = useState<string>('');       // ข้อความ error
const [unlocked, setUnlocked] = useState<boolean>(false); // guard flag
```

### Types ที่ต้องใช้
```ts
// ใช้จาก store โดยตรง — ไม่ต้องเพิ่ม type ใหม่
// pinLockoutUntil: string | null (มีอยู่ใน AppState)
// pinFailCount: number (มีอยู่ใน AppState)
```

---

## UI Spec — ทีละ section

### Layout หลัก
- Screen แบ่งเป็น 2 ส่วน:
  - **ส่วนบน** (40% screen height): bg `#185FA5` — header area
  - **ส่วนล่าง** (60% screen height): bg `#FFFFFF` — keypad area
- SafeAreaView ครอบทั้งหมด

### Section: Lock Icon
- Container: วงกลม bg `rgba(255,255,255,0.2)`, size 80×80, border-radius 40
- Icon: emoji หรือ icon `🔒` ขนาด 36px, สีขาว
- Position: กลางบน ห่างจาก top safe area 24px

### Section: Title
- Text: `"โซนพ่อแม่"` — font size 22, font weight bold, color `#FFFFFF`, text-align center
- Margin top: 12px จาก lock icon

### Section: PIN Dots (4 ดวง)
- Component: `<PinDot filled={i < entered.length} />`
- Layout: Row, justify center, gap 16px, margin top 20px
- Dot spec:
  - Size: 16×16 px, border-radius 8
  - Empty: border 2px solid `rgba(255,255,255,0.6)`, bg transparent
  - Filled: bg `#FFFFFF`

### Section: Error / Hint Text Area
- Container: min height 24px, margin top 12px, padding horizontal 32px
- **Hint (default, ไม่มี error):**
  - Text: `"รหัสทดสอบ: 1234"` — font size 12, color `rgba(255,255,255,0.6)`, text-align center
- **Error (หลังผิด):**
  - Text: `"PIN ไม่ถูกต้อง ลองอีกครั้ง"` — font size 13, color `#FFD4A8`, text-align center
  - เมื่อ error แสดงขึ้น → ล้าง hint
- **Lockout:**
  - Text: `"ลองใหม่อีก X นาที"` — X = ceil((pinLockoutUntil - Date.now()) / 60000)
  - Color: `#FFD4A8`, font size 13

### Section: Shake Animation
- เมื่อ PIN ผิด → PIN dots container เล่น shake animation
- Keyframe: translate X `[-10, 10, -10, 10, 0]` ใน 400ms
- ใช้ `Animated.sequence` หรือ `useAnimatedStyle` (Reanimated)

### Section: Keypad (ส่วนล่าง bg ขาว)
- Layout: Grid 3 คอลัมน์, 4 แถว
- แต่ละปุ่มตัวเลข:
  - Size: ขนาดเท่ากัน flex 1, height 72px
  - Text หลัก: ตัวเลข 0-9, font size 24, font weight 600, color `#2C2C2A`
  - Text รอง (sub-label ใต้ตัวเลข): ตัวอักษรกำกับ (ไม่บังคับ — ถ้ามีใน mockup ให้ใส่)
  - bg: `#FFFFFF`, on press ripple: `#EEF5FC`
  - Border: ไม่มี border (ใช้ spacing แทน)
- Layout ปุ่ม:
  ```
  [1][2][3]
  [4][5][6]
  [7][8][9]
  [ ][0][⌫]
  ```
- ปุ่มซ้ายล่าง (position `[ ]`): ว่าง (disabled, invisible)
- ปุ่ม Delete `⌫`:
  - Icon: backspace icon หรือ text `"⌫"`
  - On press: ลบตัวอักษรสุดท้ายออกจาก `entered`
  - Disabled เมื่อ `entered.length === 0`

### Section: Lockout Overlay
- แสดงเมื่อ `pinLockoutUntil !== null && Date.now() < Date.parse(pinLockoutUntil)`
- Overlay ครอบ keypad ทั้งหมด: bg `rgba(255,255,255,0.9)`, absolute fill
- ข้อความ: `"ลองใหม่อีก X นาที"` — font size 18, color `#185FA5`, text-align center
- Sub text: `"กรุณารอสักครู่"` — font size 13, color `#888780`
- Countdown อัปเดตทุก 1 วินาที (ใช้ `setInterval` ใน useEffect, cleanup ใน return)

---

## Behavior & Logic

| Condition | Behavior |
|---|---|
| กดตัวเลข + `entered.length < 4` | append digit → `setEntered(prev => prev + digit)` |
| กดตัวเลข + `entered.length === 4` | ไม่ทำอะไร (ปุ่มทุกตัวต้อง disabled เมื่อครบ 4) |
| `entered.length === 4` | trigger PIN check อัตโนมัติ (useEffect ที่ watch `entered`) |
| PIN ถูกต้อง (`entered === settings.pin`) | `resetPinFails()` → `setUnlocked(true)` → render children (parent screens) |
| PIN ผิด | `recordPinFail()` → shake animation → `setError("PIN ไม่ถูกต้อง ลองอีกครั้ง")` → `setEntered('')` หลัง 600ms |
| `pinFailCount >= 3` (หลัง recordPinFail) | store set `pinLockoutUntil` อัตโนมัติ → UI detect lockout → แสดง overlay + countdown |
| lockout หมดเวลา | countdown ถึง 0 → ซ่อน overlay → `resetPinFails()` ถูกเรียกอัตโนมัติเมื่อ lockoutUntil เลยไปแล้ว |
| กด delete | `setEntered(prev => prev.slice(0, -1))` → ล้าง error ถ้ามี |
| `unlocked === true` | render `<Slot />` (expo-router: children ของ layout) |

---

## Edge Cases

| Case | Expected behavior |
|---|---|
| เปิด app ครั้งแรก (pin = '1234') | Hint แสดง `"รหัสทดสอบ: 1234"` |
| กลับมาที่ parent area หลัง navigate ออก | `unlocked` reset เป็น false → ต้องกรอก PIN ใหม่ทุกครั้ง (local state ไม่ persist) |
| `pinLockoutUntil` ยังค้างอยู่ใน store จากครั้งก่อน | ตรวจสอบเมื่อ mount → ถ้าเลยไปแล้ว `resetPinFails()` ทันที |
| กด back ระหว่างกรอก PIN | `entered` ลดลง 1 หลัก (ถ้า hardware back → navigate ออก) |
| PIN ถูกต้องแต่ยังอยู่ใน lockout window | ไม่ควรเกิด (UI disabled ระหว่าง lockout) — guard ด้วย `if (!isLockedOut)` ก่อน check PIN |
| `settings.pin` เป็น empty string | ไม่ควรเกิด แต่ถ้าเกิด → treat as '1234' (fallback) |

---

## Acceptance Criteria
- [ ] กรอก PIN 4 หลักถูกต้อง → navigate เข้า parent dashboard ได้ทันที (ไม่ต้องกดปุ่ม OK)
- [ ] กรอก PIN ผิด 1 ครั้ง → แสดง `"PIN ไม่ถูกต้อง ลองอีกครั้ง"` + shake animation
- [ ] กรอก PIN ผิด 3 ครั้ง → lockout overlay ปรากฏ + countdown นาทีถูกต้อง
- [ ] countdown อัปเดตทุกวินาที (ไม่หยุดนิ่ง)
- [ ] หมด lockout → overlay หาย + กรอก PIN ได้ใหม่
- [ ] ปุ่ม delete ลบทีละหลัก, disabled เมื่อ `entered` ว่าง
- [ ] Hint `"รหัสทดสอบ: 1234"` แสดงเมื่อไม่มี error
- [ ] Hint หายเมื่อมี error message
- [ ] ระหว่าง lockout ปุ่มทุกตัวใน keypad disabled (กดไม่ได้)

## Out of Scope
- Biometric authentication (Face ID / Fingerprint)
- PIN change flow (อยู่ใน settings.tsx)
- Multi-kid profiles (sprint นี้ยังไม่ทำ)
