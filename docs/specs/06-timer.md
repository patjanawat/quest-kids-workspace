# Spec: Timer

> version: 1.0 | สถานะ: Draft
> mockup reference: Timer Screen (Dark Theme)

## Overview
Timer screen แบบ dark theme สำหรับนับเวลาขณะทำภารกิจ (stopwatch นับขึ้น ไม่ใช่ countdown) — มีปุ่ม Play/Pause/Stop/Reset, quest context, และปุ่ม "ทำเสร็จแล้ว" เมื่อ stop

## Screens / Components ที่เกี่ยวข้อง
- `app/timer.tsx` — Timer screen (รับ params: `questId?`)
- `components/DigitalClock.tsx` — clock display `MM:SS`
- `components/TimerControls.tsx` — 3 ปุ่มควบคุม (Reset / Play-Pause / Stop)
- `components/TimerProgressBar.tsx` — thin bar แสดง elapsed progress
- `components/QuestContext.tsx` — แสดง quest icon + title ด้านบน

## Data & State
### อ่านจาก store
- `useQuestStore(s => s.timerActive)` — boolean (timer กำลัง run)
- `useQuestStore(s => s.timerPaused)` — boolean (NEW — timer pause อยู่)
- `useQuestStore(s => s.timerRemainingSeconds)` — seconds elapsed (repurposed: เก็บ elapsed ไม่ใช่ remaining สำหรับ stopwatch)
- `useQuestStore(s => s.activeQuestId)` — `string | null`
- `useQuestStore(s => s.quests)` — เพื่อ lookup quest จาก activeQuestId

### เรียก actions
- `startQuestTimer(questId)` — เมื่อ mount (ถ้า param questId มี และ timer ยังไม่ run)
- `pauseTimer()` — เมื่อกดปุ่ม Pause
- `resumeTimer()` — เมื่อกดปุ่ม Play (ขณะ paused)
- `stopTimer()` — เมื่อกดปุ่ม Stop
- `resetTimer()` — เมื่อกดปุ่ม Reset
- `completeQuest(questId)` — เมื่อกดปุ่ม "✓ ทำเสร็จแล้ว!" หลัง stop

### Route Params
```ts
// app/timer.tsx
import { useLocalSearchParams } from 'expo-router';
const { questId } = useLocalSearchParams<{ questId?: string }>();
```

### Local State
```ts
const [elapsedSeconds, setElapsedSeconds] = useState<number>(0);
const [timerStatus, setTimerStatus] = useState<'idle' | 'running' | 'paused' | 'stopped'>('idle');
const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);
```

**Timer logic (local interval, sync กับ store):**
```ts
// เมื่อ status = 'running': setInterval ทุก 1000ms → setElapsedSeconds(prev => prev + 1)
// เมื่อ pause/stop: clearInterval
// เมื่อ reset: setElapsedSeconds(0)
```

### Types ที่ต้องใช้
```ts
// ใช้ timerPaused (NEW field ใน AppState)
// timerStatus enum ใน local state
type TimerStatus = 'idle' | 'running' | 'paused' | 'stopped';
```

---

## UI Spec — ทีละ section

### Screen Layout
- bg: `#1a1a2e` (full screen dark)
- StatusBar: hidden หรือ light-content
- SafeAreaView ครอบทั้งหมด
- Layout: Column, flex 1, justify-content space-between, padding horizontal 24px

### Section: Top Bar
- Layout: Row, align-items center, justify-content space-between, padding top 16px
- **ปุ่ม "← กลับ":**
  - Text: `"← กลับ"` — font size 16, color `rgba(255,255,255,0.6)`
  - On press: confirm dialog (ถ้า timer running/paused) → `router.back()`
  - **ถ้า running/paused:** Alert `"ออกจากหน้านี้จะหยุด timer ชั่วคราว ต้องการออกหรือไม่?"` → OK: `pauseTimer()` → `router.back()`
  - **ถ้า stopped/idle:** `router.back()` ทันที
- **Label "จับเวลา":**
  - Text: `"จับเวลา"` — font size 16, color `rgba(255,255,255,0.4)`, text-align center, flex 1

### Section: Quest Context (`<QuestContext />`)
- Container: align-items center, margin top 24px
- **Quest Icon:**
  - Outer circle: size 80×80px, border-radius 40, bg `rgba(0,255,135,0.1)`, border 1.5px solid `rgba(0,255,135,0.3)`
  - Text: quest.icon (emoji), font size 40
  - **ถ้าไม่มี quest (no questId):** แสดง `"⏱"` (generic timer)
- **Quest Title:**
  - Text: `"{quest.title}"` — font size 16, color `rgba(255,255,255,0.7)`, text-align center, margin top 10px
  - **ถ้าไม่มี quest:** `"จับเวลาอิสระ"` — font size 16, color `rgba(255,255,255,0.5)`
- **Quest Description:**
  - Text: `"{quest.description}"` — font size 12, color `rgba(255,255,255,0.4)`, text-align center, margin top 4px, numberOfLines 1

### Section: Digital Clock (`<DigitalClock />`)
- Container: align-items center, margin top 32px
- **Main clock:**
  - Format: `MM:SS` (minutes:seconds)
  - เช่น 0 → `"00:00"`, 65s → `"01:05"`, 3600s → `"60:00"` (ไม่ต้องแสดง hours)
  - Font: monospace (fontFamily `'Courier New'` หรือ system monospace)
  - Font size: 80px, font weight bold
  - Color: `#00FF87` (neon green)
  - Text shadow / glow effect: `textShadowColor: '#00FF87', textShadowRadius: 20, textShadowOffset: {width:0, height:0}`
  - Letter spacing: 4px
- **Milliseconds display:**
  - Format: แสดง `"XX"` (2 หลัก — เฉพาะ tenths ไม่ต้องแม่นยำ ms จริง, อัปเดตทุก 100ms)
  - Font size: 28px, font weight 400
  - Color: `rgba(0,255,135,0.4)` (dim neon)
  - Margin top: 4px

### Section: Status Label
- Text ตาม timerStatus:
  - `'idle'` → `"พร้อมเริ่ม"` — color `rgba(255,255,255,0.5)`
  - `'running'` → `"กำลังจับเวลา"` — color `#00FF87`
  - `'paused'` → `"หยุดชั่วคราว"` — color `rgba(255,200,80,0.9)` (warm amber)
  - `'stopped'` → `"หมดเวลา"` — color `rgba(255,255,255,0.6)`
- Font size: 14px, text-align center, margin top 12px, letter-spacing 1px

### Section: Progress Bar (`<TimerProgressBar />`)
- Container: full width, margin top 24px, height 4px, bg `rgba(255,255,255,0.1)`, border-radius 2px
- Fill: bg `#00FF87`, height 4px, border-radius 2px
- **Progress formula (stopwatch):**
  - ใช้ reference duration = `quest.rewardMinutes * 60` (target duration สำหรับ quest นั้น)
  - `progress = Math.min(elapsedSeconds / referenceDuration, 1.0)`
  - ถ้าไม่มี quest หรือ rewardMinutes = 0: progress = 0 (bar ไม่แสดง fill)
- **เมื่อ elapsed เกิน reference duration:** fill เต็ม 100% (ไม่ overflow), bar เปลี่ยนสี `#FFD700` (gold)

### Section: Control Buttons (`<TimerControls />`)
- Container: Row, justify-content center, align-items center, gap 24px, margin top 40px
- ปุ่มทั้ง 3 ทำ circular button

#### ปุ่ม Reset 🔄 (ซ้าย)
- Size: 60×60px, border-radius 30
- bg: `rgba(255,255,255,0.15)`
- Icon: reset/replay icon หรือ text `"🔄"`, font size 24, color `#FFFFFF`
- On press: Alert confirm `"รีเซ็ต timer ใช่ไหม?"` → OK: `resetTimer()` + `setElapsedSeconds(0)` + `setTimerStatus('idle')`
- **Disabled เมื่อ:** `timerStatus === 'idle'`
- Disabled style: opacity 0.4

#### ปุ่ม Play/Pause ▶️/⏸️ (กลาง)
- Size: 80×80px, border-radius 40 (ใหญ่กว่าปุ่มอื่น)
- **State: idle / paused → Play button**
  - bg: `#00FF87`
  - Icon: `"▶"` play triangle, font size 32, color `#1a1a2e` (dark — contrast บน neon bg)
  - On press (idle): `startQuestTimer(questId)` (หรือ `startTimer()` ถ้าไม่มี questId) + `setTimerStatus('running')`
  - On press (paused): `resumeTimer()` + `setTimerStatus('running')`
- **State: running → Pause button**
  - bg: `#00FF87`
  - Icon: `"⏸"` pause bars, font size 32, color `#1a1a2e`
  - On press: `pauseTimer()` + `setTimerStatus('paused')`
- **State: stopped → Play (disabled)**
  - bg: `rgba(0,255,135,0.3)`, opacity 0.6, disabled = true

#### ปุ่ม Stop ⏹️ (ขวา)
- Size: 60×60px, border-radius 30
- bg: `#FF6B6B` (red)
- Icon: square stop icon หรือ text `"⏹"`, font size 24, color `#FFFFFF`
- On press: `stopTimer()` + `setTimerStatus('stopped')`
- **Disabled เมื่อ:** `timerStatus === 'idle'`
- Disabled style: bg `rgba(255,107,107,0.3)`, opacity 0.4

### Section: "ทำเสร็จแล้ว" Button
- **แสดงเมื่อ:** `timerStatus === 'stopped'`
- **ซ่อน/disabled เมื่อ:** `timerStatus !== 'stopped'`
- Container: margin top 24px, padding horizontal 24px
- ปุ่ม:
  - bg `#00FF87`, border-radius 16px, padding vertical 18px, full width
  - Text: `"✓ ทำเสร็จแล้ว!"` — color `#1a1a2e`, font size 18, font weight bold, text-align center
  - On press:
    1. ถ้ามี `questId`: `completeQuest(questId)` + `updateProgress()`
    2. navigate `router.back()` → กลับ Kid Home (quest จะมี done overlay แล้ว)
- Animate: fade-in เมื่อ timerStatus เปลี่ยนเป็น 'stopped' (opacity 0 → 1, duration 300ms)

---

## Behavior & Logic

| Condition | Behavior |
|---|---|
| Mount พร้อม `questId` | ตรวจสอบ `activeQuestId` ใน store — ถ้า match → sync elapsed จาก store; ถ้าไม่ → `setTimerStatus('idle')` |
| กด Play (idle) | `startQuestTimer(questId)` → `setTimerStatus('running')` → เริ่ม interval |
| กด Pause (running) | clearInterval → `pauseTimer()` → `setTimerStatus('paused')` |
| กด Play (paused) | `resumeTimer()` → เริ่ม interval ใหม่ → `setTimerStatus('running')` |
| กด Stop | clearInterval → `stopTimer()` → `setTimerStatus('stopped')` → แสดงปุ่ม "ทำเสร็จแล้ว" |
| กด Reset → confirm | clearInterval → `resetTimer()` → `setElapsedSeconds(0)` → `setTimerStatus('idle')` |
| กด "ทำเสร็จแล้ว" | `completeQuest(questId)` → `updateProgress()` → `router.back()` |
| กด "← กลับ" (running) | Alert confirm → OK: `pauseTimer()` → `router.back()` |
| กด "← กลับ" (stopped) | `router.back()` ทันที |
| `elapsedSeconds` เกิน `rewardMinutes * 60` | progress bar เต็ม + เปลี่ยนสี gold — timer ยังนับต่อ |
| ไม่มี questId (global timer) | Quest Context แสดง generic, ปุ่ม "ทำเสร็จแล้ว" navigate กลับโดยไม่ `completeQuest` |
| `timerStatus === 'idle'` | ปุ่ม Reset + Stop disabled, ปุ่ม Play enabled |
| `timerStatus === 'running'` | ปุ่ม Pause enabled, ปุ่ม Stop enabled, ปุ่ม Reset enabled (ต้อง confirm) |
| `timerStatus === 'paused'` | ปุ่ม Play (resume) enabled, ปุ่ม Stop enabled, ปุ่ม Reset enabled |
| `timerStatus === 'stopped'` | ทุกปุ่ม control disabled, แสดงปุ่ม "ทำเสร็จแล้ว" |

---

## Edge Cases

| Case | Expected behavior |
|---|---|
| `questId` ใน params ไม่ match quest ใน store | Quest Context แสดง generic (`"⏱"` + `"จับเวลาอิสระ"`) |
| app ถูก background ขณะ timer running | timer หยุด (ไม่มี background execution) — เมื่อกลับมา status ยังเป็น 'running' แต่ elapsed ไม่ถูกต้อง → ต้องใช้ `AppState` API บันทึก backgroundAt แล้วบวกเมื่อ foreground |
| กด Play ซ้ำเร็วๆ | ป้องกัน: disable ปุ่มระหว่าง state transition (50ms debounce) |
| `rewardMinutes = 0` ใน quest | progress bar ไม่แสดง fill (ไม่ crash) |
| elapsed = 0 | clock แสดง `"00:00"` ถูกต้อง |
| elapsed ≥ 3600s (1 ชั่วโมง) | clock แสดง `"60:00"` → `"61:00"` ... (ไม่ reset เมื่อถึง 60 นาที) |
| navigate กลับโดยไม่กด Stop | timer pause (ผ่าน confirm dialog) — ไม่ auto-complete quest |
| กด "ทำเสร็จแล้ว" เร็วๆ (double tap) | disabled ทันทีหลัง press ครั้งแรก |

---

## Acceptance Criteria
- [ ] Screen bg `#1a1a2e` เต็มหน้าจอ (full dark)
- [ ] Quest icon และ title แสดงจาก questId ที่รับมา
- [ ] Clock แสดง `MM:SS` format ถูกต้อง, font monospace, color `#00FF87`
- [ ] Clock neon glow effect มองเห็นได้
- [ ] Milliseconds digit อัปเดตทุก 100ms
- [ ] Status label เปลี่ยนตาม timerStatus ทั้ง 4 states
- [ ] กด Play → timer เริ่มนับขึ้น
- [ ] กด Pause → timer หยุด (elapsed ค้างไว้), กด Play ต่อ → นับต่อจากเดิม
- [ ] กด Stop → timer หยุด + ปุ่ม "ทำเสร็จแล้ว" fade-in
- [ ] กด Reset → confirm dialog → clock กลับเป็น `"00:00"` + status `"พร้อมเริ่ม"`
- [ ] Progress bar fill ตาม `elapsed / (rewardMinutes * 60)`
- [ ] Progress bar เปลี่ยนสี gold เมื่อ elapsed เกิน target duration
- [ ] กด "ทำเสร็จแล้ว" → `completeQuest` เรียก → navigate กลับ Kid Home
- [ ] กด "← กลับ" ขณะ running → Alert confirm
- [ ] ปุ่ม Reset/Stop disabled เมื่อ idle

## Out of Scope
- Background timer (timer ทำงานเมื่อ app ถูก minimize) — sprint ถัดไป
- Circular progress animation (แทน linear progress bar) — sprint ถัดไป
- Sound effects / haptic feedback
- Timer history / statistics
