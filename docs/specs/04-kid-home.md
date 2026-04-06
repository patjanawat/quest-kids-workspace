# Spec: Kid Home

> version: 1.0 | สถานะ: Draft
> mockup reference: Kid Home Screen

## Overview
หน้าหลักของลูก — แสดงภารกิจวันนี้, ความก้าวหน้า XP/level, stats, ข้อความกำลังใจจากพ่อแม่, และปุ่มขอปลดล็อกเวลาเล่น/ขอเพิ่มภารกิจ

## Screens / Components ที่เกี่ยวข้อง
- `app/index.tsx` — Kid Home screen (root screen)
- `components/KidHeader.tsx` — header ส้ม + avatar + XP bar
- `components/XPBar.tsx` — level progress bar
- `components/KidStatGrid.tsx` — 3-column stat (นาที / เสร็จ / streak)
- `components/CheerBanner.tsx` — ข้อความกำลังใจจากพ่อแม่
- `components/QuestCard.tsx` — card ภารกิจแต่ละอัน (timer button / done overlay)
- `components/UnlockButton.tsx` — ปุ่มขอปลดล็อก / disabled state
- `components/KidTabBar.tsx` — bottom tab (ภารกิจ / Streak / รางวัล)

## Data & State
### อ่านจาก store
- `useQuestStore(s => s.settings.kidProfile)` — `{ name: string, avatar: string }`
- `useQuestStore(s => s.quests)` — `Quest[]` รายการภารกิจวันนี้
- `useQuestStore(s => s.totalEarnedMinutes)` — นาทีรวมที่ได้วันนี้
- `useQuestStore(s => s.progress)` — `KidProgress` (totalXp, currentLevel, currentLevelTitle, xpToNextLevel, xpThisLevel, streakDays)
- `useQuestStore(s => s.activeCheer)` — `CheerMessage | null`
- `useQuestStore(s => s.timerActive)` — boolean
- `useQuestStore(s => s.activeQuestId)` — `string | null` — quest ที่ timer กำลัง run
- `useQuestStore(s => s.pendingApproval)` — boolean (ใช้ disable unlock button ระหว่างรออนุมัติ)

### เรียก actions
- `requestApproval()` — เมื่อกดปุ่ม "ขอปลดล็อก"
- `markCheerRead()` — เมื่อลูกเห็น cheer banner (เรียกทันทีหลัง render CheerBanner)
- `startQuestTimer(questId)` — เมื่อกดปุ่ม timer บน quest card (navigate ไป timer.tsx ด้วย)

### Derived values (คำนวณใน component)
```ts
const completedQuests = quests.filter(q => q.completed);
const completedCount = completedQuests.length;
const totalCount = quests.length;
const hasCompleted = completedCount > 0;
const hasActiveTimer = timerActive && activeQuestId !== null;

// Greeting ตาม time of day
const hour = new Date().getHours();
const greeting =
  hour >= 5 && hour < 12  ? 'สวัสดีตอนเช้า!'   :
  hour >= 12 && hour < 18 ? 'สวัสดีตอนบ่าย!'   :
                             'สวัสดีตอนเย็น!';

// Level progress
const xpForCurrentLevel = progress.xpThisLevel;
const xpNeededForNext = progress.xpToNextLevel + progress.xpThisLevel; // total range ของ level นี้
const levelPercent = xpNeededForNext > 0
  ? Math.round((xpForCurrentLevel / xpNeededForNext) * 100)
  : 100;
```

### Types ที่ต้องใช้
```ts
interface KidProgress {
  totalXp: number;
  currentLevel: number;
  currentLevelTitle: string;
  xpToNextLevel: number;
  xpThisLevel: number;
  streakDays: number;
  lastStreakDate: string;
}
interface CheerMessage {
  id: string;
  text: string;
  emoji: string;
  sentAt: string;
  readAt?: string;
}
```

---

## UI Spec — ทีละ section

### Section: Kid Header (`<KidHeader />`)
- bg: `#FF8C42`
- SafeAreaView (top), padding horizontal 20px, padding top 16px, padding bottom 20px

#### Sub-section: Avatar + Greeting Row
- Layout: Row, align-items center, gap 14px
- **Avatar circle:**
  - Size: 56×56px, border-radius 28, bg `#FFD4A8`
  - Text: `"{kidProfile.avatar}"` (emoji, default `"🐱"`), font size 28
- **Column (flex 1):**
  - Greeting text: `"{greeting}"` — font size 13, color `rgba(255,255,255,0.85)`, margin bottom 2px
  - Name text: `"{kidProfile.name} 👋"` — font size 20, font weight bold, color `#FFFFFF`

#### Sub-section: XP Bar (`<XPBar />`)
- Container: margin top 16px, bg `rgba(0,0,0,0.15)`, border-radius 12px, padding 12px
- Row (space-between, align center):
  - Left: Column
    - Text: `"{progress.currentLevelTitle}"` — font size 12, color `rgba(255,255,255,0.85)`
    - Text: `"Lv.{progress.currentLevel}"` — font size 18, font weight bold, color `#FFFFFF`
  - Right: Column (align end)
    - Text: `"{completedCount}/{totalCount} quest"` — font size 12, color `rgba(255,255,255,0.85)`
    - Text: `"Lv.{progress.currentLevel + 1} ต้องการ {progress.xpToNextLevel} XP"` — font size 11, color `rgba(255,255,255,0.7)`
- Progress bar (margin top 8px):
  - Track: bg `rgba(0,0,0,0.2)`, height 8px, border-radius 4px, full width
  - Fill: bg `#FFFFFF`, width `{levelPercent}%`, border-radius 4px
  - Animate: Animated.timing เมื่อ xpThisLevel เปลี่ยน (duration 600ms)
- **Max level (Lv.6):** แสดง `"MAX"` แทน "Lv.7 ต้องการ..." และ fill 100%

### Section: Kid Stat Grid (`<KidStatGrid />`)
- Layout: Row, 3 boxes เท่ากัน, margin horizontal 16px, margin top 12px, gap 10px
- Container card: bg `#FFFFFF`, border-radius 12px, padding 12px, shadow elevation 2
- **Box 1 — นาทีวันนี้**
  - Icon: `"⏱️"` font size 18
  - Value: `"{totalEarnedMinutes}"` — font size 24, font weight bold, color `#FF8C42`
  - Label: `"นาที"` — font size 11, color `#888780`
- **Box 2 — เสร็จแล้ว**
  - Icon: `"✅"` font size 18
  - Value: `"{completedCount}"` — font size 24, font weight bold, color `#3B6D11`
  - Label: `"เสร็จแล้ว"` — font size 11, color `#888780`
- **Box 3 — streak**
  - Icon: `"🔥"` font size 18
  - Value: `"{progress.streakDays}"` — font size 24, font weight bold, color `#1D9E75`
  - Label: `"วันติดต่อกัน"` — font size 11, color `#888780`

### Section: Cheer Banner (`<CheerBanner />`)
- **แสดงเมื่อ:** `activeCheer !== null`
- **ซ่อนเมื่อ:** `activeCheer === null`
- Container: bg `#EAF3DE`, border 1.5px solid `#97C459`, border-radius 12px, margin horizontal 16px, margin top 10px, padding 14px
- Row (align center, gap 10px):
  - Icon: `"💬"` font size 22
  - Column (flex 1):
    - Label: `"ข้อความจากพ่อแม่"` — font size 11, color `#3B6D11`, font weight 600
    - Text: `"{activeCheer.text}"` — font size 14, color `#2C2C2A`
- **Behavior:** เรียก `markCheerRead()` ทันทีเมื่อ component mount (หลัง render ครั้งแรก)
  - ใช้ `useEffect(() => { if (activeCheer) markCheerRead(); }, [activeCheer?.id])`
  - Banner หายหลัง markCheerRead เพราะ `activeCheer` จะเป็น null ใน store

### Section: Quest List Header
- Layout: Row (space-between, align center), margin horizontal 16px, margin top 16px, margin bottom 8px
- Left text: `"🌟 ภารกิจวันนี้"` — font size 17, font weight bold, color `#2C2C2A`
- Right text: `"เสร็จ {completedCount}/{totalCount}"` — font size 13, color `#888780`

### Section: Quest List (ScrollView หลัก)
- แต่ละ quest แสดงด้วย `<QuestCard />`
- Gap ระหว่าง card: 8px
- Padding horizontal 16px

#### `<QuestCard />` spec

##### States ของ QuestCard
1. **ปกติ (ยังไม่ทำ, ไม่มี timer active)**
2. **Running** — quest นี้กำลัง timing (`activeQuestId === quest.id`)
3. **Locked** — มี timer active แต่เป็น quest อื่น (`timerActive && activeQuestId !== quest.id`)
4. **Done** — `quest.completed === true`

##### Layout card ทุก state
- bg `#FFFFFF`, border-radius 14px, padding 14px, flex-direction row, align-items center, gap 12px, shadow elevation 2

##### Icon circle
- Size: 50×50px, border-radius 25
- bg: ตาม category (ดู Quest Category Colors ใน spec 03)
- Text: quest.icon (emoji), font size 26

##### Content area (flex 1)
- Title: `"{quest.title}"` — font size 15, font weight 600, color `#2C2C2A`
- Sub: `"{quest.description}"` — font size 12, color `#888780`, numberOfLines 1
- XP badge row (margin top 4px): `"+{quest.xpReward} XP"` — bg `#534AB7`, border-radius 10, padding horizontal 8, padding vertical 2, font size 11, color `#FFFFFF`

##### Timer Button — State: ปกติ
- Circle size: 42×42px, border-radius 21, bg `#534AB7`
- Icon: `"▶"` play icon, color `#FFFFFF`, font size 18
- On press: `startQuestTimer(quest.id)` → navigate `router.push('/timer?questId=' + quest.id)`

##### Timer Button — State: Running (quest นี้)
- Circle bg: `#FF8C42` (ส้ม)
- Icon: `"⏱"` icon + pulsing animation (scale 1.0 → 1.1 → 1.0 loop 1s)
- Label ใต้ปุ่ม: `"กำลังทำ"` — font size 10, color `#FF8C42`
- On press: navigate ไป timer.tsx พร้อม questId เดิม (`/timer?questId={activeQuestId}`)

##### Timer Button — State: Locked (quest อื่น running)
- Circle bg: `#E0E0E0` (grey)
- Icon: `"🔒"` lock icon, color `#B4B2A9`
- Disabled (non-interactive)

##### Done Overlay — State: Done
- Overlay: position absolute, fill card, bg `rgba(234,243,222,0.9)`, border-radius 14
- Checkmark: `"✓"` ขนาด 32px, color `#3B6D11`, ตรงกลาง
- Border card: 1.5px solid `#97C459` (เปลี่ยนจาก default)
- ปุ่ม timer: ซ่อน (ไม่แสดง)
- Title/sub: ยังแสดงอยู่ใต้ overlay (สี muted ผ่าน overlay)

### Section: Empty State (quests.length === 0)
- แสดงกลางหน้า (หลัง Quest List Header)
- Icon: `"🔒"` font size 56
- Text: `"รอพ่อแม่ตั้งภารกิจก่อนนะ"` — font size 16, color `#888780`, text-align center
- Sub: `"พ่อแม่จะเพิ่มภารกิจให้เร็วๆ นี้"` — font size 13, color `#B4B2A9`, text-align center, margin top 8px

### Section: Bottom Action Buttons
- Fixed bottom (เหนือ bottom tab bar), bg `#FFF8F0`, padding horizontal 16px, padding vertical 12px, border-top 1px solid `#FFD4A8`
- Layout: Row, gap 10px

#### ปุ่ม "ขอปลดล็อก"
- **State: Disabled** (เมื่อ `!hasCompleted || hasActiveTimer || pendingApproval`)
  - bg `#E0E0E0`, border-radius 12px, flex 1, padding vertical 14px
  - Text: `"🔒 ทำภารกิจก่อนนะ!"` — color `#B4B2A9`, font size 14, font weight 600
  - Disabled = true (non-interactive)
- **State: Enabled** (เมื่อ `hasCompleted && !hasActiveTimer && !pendingApproval`)
  - bg `#FF8C42`, border-radius 12px, flex 1, padding vertical 14px
  - Text: `"🔓 ขอปลดล็อก ({totalEarnedMinutes} นาที)"` — color `#FFFFFF`, font size 14, font weight 600
  - On press: `requestApproval()` → ปุ่มเปลี่ยนเป็น disabled ทันที (pendingApproval = true)
- **State: Pending** (เมื่อ `pendingApproval === true`)
  - bg `#FFD4A8`, border-radius 12px, flex 1, padding vertical 14px
  - Text: `"⏳ รออนุมัติ..."` — color `#D85A30`, font size 14, font weight 600
  - Disabled = true

#### ปุ่ม "ขอเพิ่ม"
- bg `#534AB7`, border-radius 12px, padding horizontal 16px, padding vertical 14px
- Text: `"➕ ขอเพิ่ม"` — color `#FFFFFF`, font size 14, font weight 600
- On press: navigate `router.push('/quest-request')`
- **Disabled เมื่อ:** `hasActiveTimer === true` (ขณะมี timer running ไม่สามารถขอเพิ่มได้)
  - ถ้า disabled: bg `#C4BAEF`, text color `rgba(255,255,255,0.6)`

### Section: Bottom Tab Bar (`<KidTabBar />`)
- bg `#FFF8F0`, border-top 1px solid `#FFD4A8`
- 3 tabs:
  - **Tab 1 — ภารกิจ** (active ในหน้านี้)
    - Icon: `"🌟"`, Label: `"ภารกิจ"` — active color `#FF8C42`, inactive `#B4B2A9`
  - **Tab 2 — Streak**
    - Icon: `"🔥"`, Label: `"Streak"` — active color `#1D9E75`, inactive `#B4B2A9`
    - On press: แสดง Coming Soon screen (placeholder)
  - **Tab 3 — รางวัล**
    - Icon: `"🏆"`, Label: `"รางวัล"` — active color `#534AB7`, inactive `#B4B2A9`
    - On press: แสดง Coming Soon screen (placeholder)

#### Coming Soon Placeholder
- ตรงกลางหน้า: `"🚧"` font size 64 + text `"Coming Soon"` font size 20, color `#888780`

---

## Behavior & Logic

| Condition | Behavior |
|---|---|
| `quests.length === 0` | แสดง empty state, ปุ่ม "ขอปลดล็อก" disabled, ปุ่ม "ขอเพิ่ม" enabled |
| `completedCount === 0` | ปุ่ม "ขอปลดล็อก" แสดง disabled state `"🔒 ทำภารกิจก่อนนะ!"` |
| `completedCount > 0 && !timerActive` | ปุ่ม "ขอปลดล็อก" enabled `"🔓 ขอปลดล็อก (X นาที)"` |
| กด "ขอปลดล็อก" | `requestApproval()` → ปุ่มเปลี่ยนเป็น pending state |
| `pendingApproval === true` | ปุ่ม "ขอปลดล็อก" แสดง `"⏳ รออนุมัติ..."` + disabled |
| `timerActive === true` | quest ที่ activeQuestId running state, quest อื่น locked state, ปุ่ม "ขอเพิ่ม" disabled |
| กด timer button (ปกติ) | `startQuestTimer(questId)` + navigate `/timer?questId=xxx` |
| `activeCheer !== null` | แสดง CheerBanner + เรียก `markCheerRead()` ทันที |
| ชั่วโมง 05:00-11:59 | greeting = `"สวัสดีตอนเช้า!"` |
| ชั่วโมง 12:00-17:59 | greeting = `"สวัสดีตอนบ่าย!"` |
| ชั่วโมง 18:00-04:59 | greeting = `"สวัสดีตอนเย็น!"` |
| กลับจาก timer.tsx | quest ที่ complete แล้วแสดง done overlay |

---

## Edge Cases

| Case | Expected behavior |
|---|---|
| `progress` ยังไม่ได้คำนวณ (null/undefined) | แสดง Lv.1 / 0 XP / 0% (fallback default) |
| `totalEarnedMinutes = 0` แต่ `completedCount > 0` | ไม่ควรเกิด (xpReward ต้องมีเสมอ); แสดงตามค่าจริง |
| streak = 0 | Box 3 แสดง "0" ไม่ใช่ dash |
| `kidProfile.name` ยาวมาก | numberOfLines 1 + ellipsis |
| `activeCheer` เป็น null | CheerBanner ไม่ render (ไม่ใช่แค่ invisible) |
| ทุก quest completed | ปุ่ม "ขอปลดล็อก" enabled, XP bar เต็ม/เกือบเต็ม |
| กด "ขอปลดล็อก" ซ้ำ (double tap) | ป้องกัน: disabled ทันทีหลัง press ครั้งแรก |
| level 6 (MAX) | XP bar เต็ม 100%, ซ่อน "ต้องการ X XP", แสดง "MAX" แทน |
| navigate กลับจาก quest-request | ไม่มีการ reset state ใด |

---

## Acceptance Criteria
- [ ] Header bg `#FF8C42`, แสดงชื่อลูก + avatar + greeting ตามเวลา
- [ ] XP bar แสดง level title, level number, progress ถูกต้อง
- [ ] Stats Grid แสดง นาที / เสร็จแล้ว / streak ถูกต้อง
- [ ] CheerBanner แสดงเมื่อมี activeCheer และ markCheerRead เรียกอัตโนมัติ
- [ ] Quest card แสดง icon, title, description, XP badge ครบถ้วน
- [ ] Timer button ปกติ กด → navigate `/timer?questId=xxx`
- [ ] Timer button running state (orange + pulse) เมื่อ quest นั้น active
- [ ] Quest อื่นขณะ timer active แสดง locked state (grey, non-interactive)
- [ ] Done overlay แสดงเมื่อ `quest.completed === true`
- [ ] Empty state แสดงเมื่อ `quests.length === 0`
- [ ] ปุ่ม "ขอปลดล็อก" disabled → enabled → pending ตาม state
- [ ] ปุ่ม "ขอเพิ่ม" navigate ไป quest-request
- [ ] Bottom tab Coming Soon แสดงสำหรับ tab ที่ยังไม่ implement

## Out of Scope
- Streak calculation logic (อยู่ใน store action `checkStreak`)
- Timer screen UI (อยู่ใน spec 06-timer.md)
- Quest request flow (อยู่ใน spec 05-quest-request.md)
- Parent mode button / navigation ไป parent area
