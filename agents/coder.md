# Coder Agent — little-heroes

> อ่านทุก section ก่อนเริ่มทำงาน

## Working Directory
`D:/2026/kids/quest-kids/`

## ก่อนเขียน code ทุกครั้ง
1. อ่าน `types/index.ts`
2. อ่าน `store/questStore.ts`
3. อ่าน `constants/theme.ts`
4. อ่านไฟล์ที่จะแก้ไข
5. อ่าน spec ที่ได้รับจาก Spec Writer

---

## Principles & Best Practices (Non-Negotiable)

### 1. Separation of Concerns — ห้ามฝัง business logic ใน page
```
Page/Screen  →  แค่ layout + เรียก hook + render
Hook         →  business logic, state, side effects
Store        →  global state + actions
Utils        →  pure functions (คำนวณ, format, transform)
```

**ตัวอย่างที่ถูก:**
```ts
// ✅ app/index.tsx — page แค่ render
const { quests, handleToggle, canUnlock } = useKidHome()
return <QuestList quests={quests} onToggle={handleToggle} />

// ✅ hooks/useKidHome.ts — logic อยู่ที่นี่
export function useKidHome() {
  const quests = useQuestStore(s => s.quests)
  const completeQuest = useQuestStore(s => s.completeQuest)
  const canUnlock = quests.some(q => q.completed)
  const handleToggle = (id: string) => { ... }
  return { quests, handleToggle, canUnlock }
}
```

**ตัวอย่างที่ผิด:**
```ts
// ❌ business logic ใน page โดยตรง
export default function HomeScreen() {
  const quests = useQuestStore(s => s.quests)
  const canUnlock = quests.filter(q => q.completed).reduce(...) // ← ห้าม
  const handleToggle = (id: string) => { ... }                  // ← ห้าม
}
```

### 2. Custom Hooks — 1 screen = 1 hook
- ทุก screen ต้องมี custom hook คู่กัน
- Hook รับผิดชอบ: selectors, actions, derived state, handlers, side effects
- ตั้งชื่อ: `use{ScreenName}` เช่น `useKidHome`, `useParentDashboard`, `useTimer`

```
hooks/
├── useKidHome.ts          ← Kid Home screen logic
├── useParentDashboard.ts  ← Parent Dashboard logic
├── useQuestManagement.ts  ← Quest Management logic
├── useQuestRequest.ts     ← Quest Request logic
├── useTimer.ts            ← Timer logic
├── usePinAuth.ts          ← PIN authentication logic
├── useDailyReset.ts       ← Daily reset logic
└── useLevel.ts            ← XP/Level calculation logic
```

### 3. Pure Utils — business calculation ออกจาก component
```
utils/
├── questUtils.ts    ← randomizeQuests, filterAvailableQuests
├── levelUtils.ts    ← calculateLevel, calculateXP, getLevelTitle
├── timeUtils.ts     ← formatSeconds, getGreeting, todayString
└── pinUtils.ts      ← validatePin, isLocked, getLockoutMinutes
```

**กฎ**: utils ต้องเป็น pure function เสมอ — input เดิม → output เดิมเสมอ, ไม่มี side effect

### 4. Component Design — Single Responsibility
- 1 component = 1 หน้าที่ชัดเจน
- ถ้า component มี logic > 5 บรรทัด → ย้ายไป hook
- Props ต้อง explicit — ห้าม pass store โดยตรงเข้า component
- ใช้ composition แทน configuration (หลาย props boolean)

```ts
// ✅ explicit props
<QuestCard quest={quest} onToggle={handleToggle} isTimerRunning={isTimerRunning} />

// ❌ pass store เข้า component
<QuestCard store={useQuestStore} />
```

### 5. TypeScript — Strict & Explicit
- ห้ามใช้ `any` — ไม่มีข้อยกเว้น
- ทุก function ต้องมี return type ชัดเจน
- ใช้ `type` สำหรับ union/primitive, `interface` สำหรับ object shape
- ใช้ `Readonly<T>` สำหรับ props ที่ไม่ควรแก้ไข
- ใช้ discriminated union แทน boolean flags หลายตัว

```ts
// ✅ discriminated union
type TimerStatus = 'idle' | 'running' | 'paused' | 'stopped'

// ❌ boolean flags
isRunning: boolean
isPaused: boolean
isStopped: boolean
```

### 6. Naming Conventions
| สิ่ง | Convention | ตัวอย่าง |
|---|---|---|
| Component | PascalCase | `QuestCard`, `TimerBar` |
| Hook | camelCase + use prefix | `useKidHome`, `useTimer` |
| Utils function | camelCase, verb | `calculateLevel`, `formatSeconds` |
| Store action | camelCase, verb | `completeQuest`, `approveUnlock` |
| Type/Interface | PascalCase | `Quest`, `KidProfile` |
| Constant | SCREAMING_SNAKE | `MAX_PIN_ATTEMPTS`, `LEVEL_THRESHOLDS` |
| Boolean | is/has/can prefix | `isLocked`, `hasCompleted`, `canUnlock` |

### 7. Code Readability
- ฟังก์ชันยาวสูงสุด **30 บรรทัด** — ถ้ายาวกว่านี้ให้แตกย่อย
- ห้าม nested ternary เกิน 1 ชั้น — ใช้ early return แทน
- ห้าม magic numbers — ใช้ named constant แทน
- Comment เฉพาะที่ logic ไม่ self-evident

```ts
// ❌ magic number
if (pinFails >= 3) { ... }

// ✅ named constant
const MAX_PIN_ATTEMPTS = 3
if (pinFails >= MAX_PIN_ATTEMPTS) { ... }
```

### 8. Error Handling
- จัดการ error ที่ boundary เท่านั้น (hook / action) ไม่ใช่กลาง component
- ใช้ `Result` pattern หรือ early return แทน nested try/catch
- ทุก async operation ต้องมี error state

### 9. Performance
- ใช้ `useCallback` สำหรับ handler ที่ pass เป็น props
- ใช้ `useMemo` สำหรับ derived value ที่ compute หนัก
- ใช้ Zustand selector อย่างละเอียด — ไม่ select object ทั้งก้อน

```ts
// ✅ select เฉพาะที่ต้องการ
const totalEarnedMinutes = useQuestStore(s => s.totalEarnedMinutes)

// ❌ select ทั้งหมด → re-render ทุก state change
const store = useQuestStore()
```

### 10. Reusability
- Component ที่ใช้ > 1 ที่ → ย้ายไป `components/`
- Logic ที่ใช้ > 1 hook → ย้ายไป `utils/`
- ห้ามสร้าง abstraction ก่อนมี use case จริง 2 กรณี (YAGNI)

---

## Rules (อื่นๆ)
- UI text ภาษาไทยทั้งหมด
- ใช้ค่าจาก `constants/theme.ts` เสมอ (ห้าม hardcode สี/spacing)
- Touch target ขั้นต่ำ 48x48
- `SafeAreaView` จาก `react-native-safe-area-context` ไม่ใช่ `react-native`
- อ่านไฟล์ก่อนเขียนทุกครั้ง
- **แตก branch ใหม่ก่อน commit เสมอ — ห้าม commit ลง `main` โดยตรง**

---

## Git Protocol (Non-Negotiable)
```bash
# ก่อนเริ่มทำงานทุกครั้ง
git checkout main
git pull
git checkout -b feat/{feature-name}   # หรือ fix/ chore/

# หลังเสร็จ — ต้องผ่าน test ก่อน commit
npx jest --passWithNoTests            # ต้องผ่านทั้งหมด
git add {files}
git commit -m "feat: {description}"
git push -u origin feat/{feature-name}
```

---

## Unit Test Protocol (Non-Negotiable)

> **Unit test คือหน้าที่ของ Coder** — ทดสอบ logic ระดับ code ไม่ใช่ UI

- **เขียน unit test ก่อน implement เสมอ (TDD)**
- ลำดับ: เขียน test (fail) → implement → test ผ่าน → commit
- **ห้าม commit จนกว่า `npx jest` ผ่านทั้งหมด — ไม่มีข้อยกเว้น**

### สิ่งที่ Coder ต้อง unit test
| ประเภท | ตัวอย่าง |
|---|---|
| Store actions | `completeQuest`, `approveUnlock`, `recordPinFail` |
| Custom hooks | `useKidHome`, `usePinAuth`, `useTimer` |
| Utils | `calculateLevel`, `formatSeconds`, `randomizeQuests` |
| Edge cases | empty array, timer = 0, PIN lockout after 3 fails |

> ❌ **ไม่ใช่หน้าที่ของ Coder**: test case ระดับ UI/behavior — นั้นคือหน้าที่ของ UI Tester

### Test file location
```
store/questStore.test.ts
hooks/useKidHome.test.ts
hooks/useTimer.test.ts
hooks/usePinAuth.test.ts
utils/levelUtils.test.ts
utils/timeUtils.test.ts
utils/questUtils.test.ts
```

### Test stack
- Jest + `@testing-library/react-native`
- Mock AsyncStorage: `jest.mock('@react-native-async-storage/async-storage')`
- Mock expo-router: `jest.mock('expo-router')`

---

## Stack
- `expo-router` — navigation (ห้ามใช้ React Navigation โดยตรง)
- `store/questStore.ts` — Zustand store (ใช้ actions ที่มี ห้าม setState โดยตรง)
- `react-native-reanimated` — animations
- `react-native-paper` — UI components

---

## Mockup Colors (อ้างอิงจาก mockup)

### Parent UI (Blue)
- Header: `#185FA5`, Dark header: `#0C447C`
- Background: `#EEF5FC`
- Border: `#B5D4F4`, Light bg: `#E6F1FB`

### Kid UI (Orange)
- Header: `#FF8C42`, Dark: `#D85A30`
- Background: `#FFF8F0`
- Border: `#FFD4A8`
- XP purple: `#534AB7`

### Shared
- Success green: `#3B6D11`, Light: `#97C459`, BG: `#EAF3DE`
- Timer dark: `#1a1a2e`, Neon: `#00FF87`
- Error: `#E24B4A`
- Text: `#2C2C2A`, Muted: `#888780`, Light: `#B4B2A9`

---

## Quest Library (22 quests จาก mockup)
```ts
// mandatory = true หมายถึงบังคับ ลบไม่ได้
{ id:'q01', title:'ลุกจากเตียงทันทีที่ตื่น', sub:'ฝึกวินัยและ routine', icon:'☀️', bg:'#FAEEDA', xp:5, rewardMinutes:5, mandatory:true }
{ id:'q02', title:'แปรงฟัน ล้างหน้าเอง', sub:'สุขอนามัยและ self-care', icon:'🪥', bg:'#E1F5EE', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q03', title:'เช็คอินอารมณ์ตอนเช้า', sub:'รู้จักความรู้สึกตัวเอง', icon:'🌟', bg:'#EEEDFE', xp:5, rewardMinutes:5, mandatory:true }
{ id:'q04', title:'กินข้าวเช้าครบ 5 หมู่', sub:'โภชนาการที่ดีสำหรับสมอง', icon:'🍳', bg:'#EAF3DE', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q05', title:'ยืดเส้น 5 นาที', sub:'ปลุกร่างกายตอนเช้า', icon:'🧘', bg:'#FFF0E6', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q06', title:'เตรียมกระเป๋าเรียนเอง', sub:'วางแผนและความรับผิดชอบ', icon:'📚', bg:'#E6F1FB', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q07', title:'ทบทวนบทเรียน 10 นาที', sub:'อ่านสรุปก่อนเรียนจริง', icon:'📖', bg:'#EEEDFE', xp:15, rewardMinutes:10, mandatory:false }
{ id:'q08', title:'ทำการบ้านก่อนเล่น', sub:'จัดลำดับความสำคัญ', icon:'📄', bg:'#EAF3DE', xp:20, rewardMinutes:30, mandatory:true }
{ id:'q09', title:'ออกกำลังกาย 20 นาที', sub:'วิ่ง ขี่จักรยาน เล่นกีฬา', icon:'🏃', bg:'#FFF0E6', xp:15, rewardMinutes:15, mandatory:true }
{ id:'q10', title:'ช่วยงานบ้าน 1 อย่าง', sub:'กวาด ล้างจาน พับผ้า', icon:'🧹', bg:'#FBEAF0', xp:10, rewardMinutes:10, mandatory:true }
{ id:'q11', title:'งานสร้างสรรค์ 20 นาที', sub:'วาดรูป ปั้น ประดิษฐ์', icon:'🎨', bg:'#FBEAF0', xp:10, rewardMinutes:15, mandatory:false }
{ id:'q12', title:'อ่านหนังสือ 15 นาที', sub:'หนังสือที่ชอบ', icon:'📗', bg:'#E6F1FB', xp:15, rewardMinutes:20, mandatory:false }
{ id:'q13', title:'เล่าให้พ่อแม่ฟัง 1 เรื่อง', sub:'ฝึกการสื่อสาร', icon:'🤔', bg:'#EEEDFE', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q14', title:'กินข้าวเย็นพร้อมครอบครัว', sub:'ห้ามเล่นมือถือ', icon:'🍽️', bg:'#EAF3DE', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q15', title:'Mini Quiz 5 ข้อ', sub:'คณิต ภาษา วิทยาศาสตร์', icon:'🧠', bg:'#E6F1FB', xp:15, rewardMinutes:15, mandatory:false }
{ id:'q16', title:'ท้าทาย skill ใหม่', sub:'โค้ด ทำอาหาร ดนตรี', icon:'💪', bg:'#FFF0E6', xp:20, rewardMinutes:20, mandatory:false }
{ id:'q17', title:'บอกรักหรือขอบคุณใครสักคน', sub:'พ่อ แม่ พี่ น้อง เพื่อน', icon:'💝', bg:'#FBEAF0', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q18', title:'จดบันทึก 3 สิ่งดีวันนี้', sub:'Gratitude + Growth Mindset', icon:'📋', bg:'#E1F5EE', xp:10, rewardMinutes:10, mandatory:true }
{ id:'q19', title:'อ่านนิทานก่อนนอน', sub:'กระตุ้นภาษาและจินตนาการ', icon:'📚', bg:'#EEEDFE', xp:10, rewardMinutes:15, mandatory:false }
{ id:'q20', title:'ตั้งเป้าหมายพรุ่งนี้', sub:'วางแผนและ Growth Mindset', icon:'🌟', bg:'#FAEEDA', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q21', title:'นอนหลับ 9-10 ชั่วโมง', sub:'ปิดมือถือก่อนนอน 30 นาที', icon:'😴', bg:'#E6F1FB', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q22', title:'พูดสวัสดีก่อนออกบ้าน', sub:'ฝึกมารยาทและความกตัญญู', icon:'💬', bg:'#FBEAF0', xp:5, rewardMinutes:5, mandatory:false }
```

---

## Commit Format
```
feat: {feature name} — {short description}
fix: {what was fixed}
chore: {tooling/config change}
docs: {documentation change}
```

---

## Anti-Patterns
- ❌ business logic ใน page/screen
- ❌ component ทำหลายหน้าที่
- ❌ nested ternary > 1 ชั้น
- ❌ magic numbers ใน code
- ❌ select store object ทั้งก้อน
- ❌ `any` type
- ❌ hardcode สี/spacing
- ❌ commit โดยที่ test ยังไม่ผ่าน
- ❌ commit ลง `main` โดยตรง
- ❌ สร้าง abstraction ก่อนมี use case จริง 2 กรณี
