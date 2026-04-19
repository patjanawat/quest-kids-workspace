# Spec: Onboarding

> version: 1.0 | สถานะ: Draft

## Overview
หน้า first-run setup — แสดงครั้งแรกเมื่อเปิดแอปและยังไม่เคยตั้งค่า  
3 ขั้นตอน: ตั้งชื่อลูก + เลือก avatar → ตั้ง PIN พ่อแม่ → เสร็จแล้ว!  
หลังจบ onboarding redirect ไปหน้าหลัก (`app/index.tsx`)

## Screens / Components ที่เกี่ยวข้อง
- `app/onboarding.tsx` — Onboarding wizard screen (single screen, 3 steps)
- `app/_layout.tsx` — redirect logic ตาม `hasOnboarded`

---

## Data & State

### เพิ่มใน store (`store/questStore.ts`)
```ts
// State
hasOnboarded: boolean  // default: false

// Action
completeOnboarding(kidName: string, avatar: string, pin: string): void
```

- `completeOnboarding` ทำ:
  1. `set({ settings: { ...state.settings, kidProfile: { name: kidName, avatar }, pin }, hasOnboarded: true })`
  2. ไม่ต้อง navigate — component onboarding ทำเอง

### อ่านจาก store (ใน `_layout.tsx`)
- `useQuestStore(s => s.hasOnboarded)` — ถ้า `false` → redirect `/onboarding`

### Local state ใน `onboarding.tsx`
```ts
const [step, setStep] = useState<1 | 2 | 3>(1);
const [kidName, setKidName] = useState('');
const [avatar, setAvatar] = useState('🐱');
const [pin, setPin] = useState('');
const [pinConfirm, setPinConfirm] = useState('');
const [pinError, setPinError] = useState('');
```

---

## Routing & Redirect Logic

### `app/_layout.tsx`
```tsx
const hasOnboarded = useQuestStore((s) => s.hasOnboarded);

// ใน Stack ใช้ redirect ผ่าน useEffect
useEffect(() => {
  if (!hasOnboarded) {
    router.replace('/onboarding');
  }
}, [hasOnboarded]);
```

- ใช้ `router.replace` (ไม่ใช่ `push`) เพื่อไม่ให้กด back กลับมาได้
- เพิ่ม `<Stack.Screen name="onboarding" />` ใน Stack

---

## UI Spec — Step by Step

### Layout ทั่วไป
- bg: `#F5F3FF` (light purple ต่อเนื่องจาก theme)
- SafeAreaView ครอบทั้งหน้า
- แสดง step indicator ด้านบน (3 dots)
- Animation: slide-in ซ้ายขวาเมื่อเปลี่ยน step (Animated.timing, duration 250ms)

### Step Indicator
- Row 3 dots, align center, margin top 24px
- Dot active: `#534AB7`, size 10×10px, border-radius 5
- Dot inactive: `rgba(83,74,183,0.25)`, size 8×8px, border-radius 4
- Gap ระหว่าง dot: 8px

---

### Step 1 — ตั้งชื่อลูก + เลือก Avatar

#### Header Section
- Emoji: `"👶"` font size 64, text-align center, margin top 40px
- Title: `"สวัสดี! มาตั้งชื่อกัน"` — font size 26, font weight bold, color `#2C2C2A`, text-align center, margin top 16px
- Subtitle: `"เราจะเรียกลูกว่าอะไรดีนะ?"` — font size 15, color `#888780`, text-align center, margin top 8px

#### ชื่อลูก
- Label: `"ชื่อลูก"` — font size 14, color `#534AB7`, font weight 600, margin bottom 6px
- TextInput:
  - bg `#FFFFFF`, border 1.5px solid `#C4BAEF`, border-radius 14px
  - padding: 14px horizontal, 16px vertical
  - font size 18, color `#2C2C2A`, text-align center
  - placeholder: `"เช่น น้องมะลิ"` — color `#C4BAEF`
  - maxLength: 20
  - **focused state:** border color `#534AB7`

#### เลือก Avatar
- Label: `"เลือก Avatar"` — font size 14, color `#534AB7`, font weight 600, margin top 20px, margin bottom 10px
- Grid: 5 คอลัมน์, wrap, gap 10px
- Avatar options (15 ตัว):
  ```
  🐱 🐶 🐸 🦊 🐻
  🐼 🦁 🐯 🦄 🐲
  🦸 🧙 👸 🤴 🚀
  ```
- Avatar item:
  - Size: 56×56px, border-radius 28, bg `#F0EEFF`
  - Text: emoji, font size 28
  - **Selected state:** border 2.5px solid `#534AB7`, bg `#E8E4FF`
  - **Unselected state:** border 2px solid transparent

#### Next Button
- bg `#534AB7`, border-radius 16px, padding vertical 18px, margin top 32px
- Text: `"ถัดไป →"` — color `#FFFFFF`, font size 17, font weight bold, text-align center
- **Disabled เมื่อ:** `kidName.trim().length === 0`
  - bg `#C4BAEF`

---

### Step 2 — ตั้ง PIN พ่อแม่

#### Header Section
- Emoji: `"🔐"` font size 64, text-align center, margin top 40px
- Title: `"ตั้ง PIN สำหรับพ่อแม่"` — font size 26, font weight bold, color `#2C2C2A`, text-align center, margin top 16px
- Subtitle: `"PIN 4 หลัก สำหรับเข้าหน้าจัดการของพ่อแม่"` — font size 15, color `#888780`, text-align center, margin top 8px

#### PIN Input (สร้างใหม่ใน onboarding ไม่ reuse PinInput component เก่า)
- Label (ครั้งแรก): `"ตั้ง PIN"` — font size 14, color `#534AB7`, font weight 600, margin bottom 12px
- Row 4 circles:
  - Circle: 56×56px, border-radius 28, border 2px solid `#C4BAEF`
  - **ว่าง:** bg `#FFFFFF`
  - **มีค่า:** bg `#534AB7`, border-color `#534AB7`
- TextInput hidden (keyboardType numeric, secureTextEntry, maxLength 4) — ซ่อนแล้วใช้ onChangeText

- Spacer 24px

- Label (ยืนยัน): `"ยืนยัน PIN"` — font size 14, color `#534AB7`, font weight 600, margin bottom 12px
- Row 4 circles เหมือนกัน สำหรับ `pinConfirm`

- **Error message** (แสดงเมื่อ `pinError !== ''`):
  - Text: `"{pinError}"` — font size 13, color `#D84040`, text-align center, margin top 12px
  - Error cases:
    - `pin.length < 4`: ไม่แสดง error จนกว่าจะกด Next
    - `pin !== pinConfirm`: `"PIN ไม่ตรงกัน กรุณาลองใหม่"`
    - `pin === '1234'`: `"PIN ง่ายเกินไป กรุณาตั้ง PIN อื่น"`

#### Next Button
- bg `#534AB7`, border-radius 16px, padding vertical 18px, margin top 32px
- Text: `"ถัดไป →"` — color `#FFFFFF`, font size 17, font weight bold
- **Disabled เมื่อ:** `pin.length < 4 || pinConfirm.length < 4`
  - bg `#C4BAEF`
- On press:
  1. validate: `pin !== pinConfirm` → set error, return
  2. validate: `pin === '1234'` → set error, return
  3. clear error → `setStep(3)`

#### Back Button
- Text: `"← ย้อนกลับ"` — font size 14, color `#888780`, text-align center, margin top 16px
- On press: `setStep(1)`

---

### Step 3 — เสร็จแล้ว!

#### Content (centered)
- Emoji: `"🎉"` font size 80, text-align center, margin top 60px
- Title: `"พร้อมแล้ว!"` — font size 32, font weight bold, color `#2C2C2A`, text-align center, margin top 20px
- Subtitle: `"ยินดีต้อนรับ {kidName}!\nมาเริ่มทำภารกิจกันเลย 🌟"` — font size 17, color `#534AB7`, text-align center, margin top 12px, line height 26px

#### Summary Card
- bg `#FFFFFF`, border-radius 16px, padding 20px, margin top 32px, shadow elevation 3
- Row items (gap 16px):
  - Row 1: Avatar emoji (32px) + Text `"ชื่อ: {kidName}"` — font size 15, color `#2C2C2A`, font weight 600
  - Row 2: `"🔐"` (32px) + Text `"PIN ตั้งค่าแล้ว"` — font size 15, color `#2C2C2A`

#### Start Button
- bg `#FF8C42`, border-radius 16px, padding vertical 20px, margin top 40px
- Text: `"🚀 เริ่มเลย!"` — color `#FFFFFF`, font size 20, font weight bold, text-align center
- On press:
  1. `completeOnboarding(kidName.trim(), avatar, pin)`
  2. `router.replace('/')` — redirect หน้าหลัก

---

## Behavior & Logic

| Condition | Behavior |
|---|---|
| `hasOnboarded === false` | `_layout.tsx` redirect `/onboarding` ทันที |
| `hasOnboarded === true` | ไม่แสดง onboarding, เข้าหน้าหลักปกติ |
| กด Next (step 1) | ตรวจ `kidName.trim()` — ถ้าว่างปุ่ม disabled |
| กด Next (step 2) | validate PIN → error หรือไป step 3 |
| กด "เริ่มเลย!" | `completeOnboarding()` + `router.replace('/')` |
| กด back ใน step 2 | กลับ step 1, ไม่ clear ชื่อ/avatar |
| กด hardware back (Android) ใน onboarding | ไม่ทำอะไร (disable back navigation) |

---

## Edge Cases

| Case | Expected behavior |
|---|---|
| `kidName` เป็น whitespace ทั้งหมด | `trim()` ก่อน validate → disabled state |
| `kidName` ยาว 20 ตัวอักษร | maxLength ป้องกันแล้ว |
| `pin === pinConfirm` แต่ === '1234' | error "PIN ง่ายเกินไป" |
| กดปุ่ม Start ซ้ำเร็ว (double tap) | ป้องกันด้วย `useState(false)` flag |
| เปิดแอปครั้งที่ 2 (hasOnboarded = true) | ไม่เห็น onboarding เลย |
| store persist แล้ว `hasOnboarded` miss migration | migration guard: `state.hasOnboarded ?? false` |

---

## Store Migration

เพิ่มใน migration callback:
```ts
if (state.hasOnboarded === undefined) state.hasOnboarded = false;
```

และเพิ่ม version เป็น **v4** (ปัจจุบัน v3)

---

## Acceptance Criteria
- [ ] เปิดแอปครั้งแรก (hasOnboarded = false) → redirect `/onboarding` อัตโนมัติ
- [ ] Step indicator 3 dots แสดงถูกต้องตาม step ปัจจุบัน
- [ ] Step 1: TextInput ตั้งชื่อ + grid avatar 15 ตัว, เลือกได้ 1 ตัว
- [ ] Step 1: ปุ่ม Next disabled เมื่อ `kidName` ว่าง
- [ ] Step 2: PIN input 4 หลัก + confirm, แสดง filled circles
- [ ] Step 2: error "PIN ไม่ตรงกัน" เมื่อ pin ≠ pinConfirm
- [ ] Step 2: error "PIN ง่ายเกินไป" เมื่อ pin === '1234'
- [ ] Step 2: ปุ่ม Back กลับ step 1 โดยไม่ clear ข้อมูล
- [ ] Step 3: แสดงชื่อและ avatar ที่เลือก
- [ ] กด "เริ่มเลย!" → `completeOnboarding()` → redirect `/`
- [ ] เปิดแอปครั้งที่ 2 (hasOnboarded = true) → ไม่เจอ onboarding
- [ ] `settings.kidProfile.name` และ `settings.pin` อัปเดตถูกต้องหลัง onboarding

## Out of Scope
- การเปลี่ยน PIN ภายหลัง (อยู่ใน Settings screen)
- การเปลี่ยนชื่อ/avatar ภายหลัง (อยู่ใน Settings screen)
- Onboarding tutorial/walkthrough ของ feature ต่างๆ
- Multiple kid profiles
