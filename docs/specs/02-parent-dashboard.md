# Spec: Parent Dashboard

> version: 1.0 | สถานะ: Draft
> mockup reference: Parent Dashboard Screen

## Overview
หน้าหลักสำหรับพ่อแม่ — แสดงสถิติภารกิจลูกวันนี้, approval card เมื่อลูกขอปลดล็อก, quest request card เมื่อลูกขอเพิ่มภารกิจ, และปุ่มส่งกำลังใจให้ลูก

## Screens / Components ที่เกี่ยวข้อง
- `app/parent/index.tsx` — Parent Dashboard screen
- `components/StatGrid.tsx` — 3-column stat boxes
- `components/ProgressSummaryCard.tsx` — progress bar + percent
- `components/ApprovalCard.tsx` — unlock approval card
- `components/QuestRequestCard.tsx` — quest request approval card
- `components/CheerSection.tsx` — cheer buttons
- `components/ParentTabBar.tsx` — bottom tab bar (ภาพรวม / จัดการ / ตั้งค่า)

## Data & State
### อ่านจาก store
- `useQuestStore(s => s.settings.kidProfile)` — `{ name: string, avatar: string }`
- `useQuestStore(s => s.quests)` — รายการภารกิจวันนี้ทั้งหมด
- `useQuestStore(s => s.totalEarnedMinutes)` — นาทีที่ลูกได้รับวันนี้
- `useQuestStore(s => s.pendingApproval)` — boolean ว่าลูกขอปลดล็อกอยู่
- `useQuestStore(s => s.pendingQuestRequests)` — `QuestRequest[]` (filter เอา status = 'pending')
- `useQuestStore(s => s.progress)` — `KidProgress` (totalXp, currentLevel ฯลฯ)
- `useQuestStore(s => s.activeCheer)` — `CheerMessage | null` (ใช้เช็คว่า cheer ล่าสุดส่งไปหรือยัง)

### เรียก actions
- `approveUnlock()` — เมื่อกดปุ่ม "อนุมัติ" ใน ApprovalCard
- `denyUnlock()` — เมื่อกดปุ่ม "ปฏิเสธ" ใน ApprovalCard
- `approveQuestRequest(requestId)` — เมื่อกดปุ่ม "อนุมัติ" ใน QuestRequestCard
- `denyQuestRequest(requestId)` — เมื่อกดปุ่ม "ไม่อนุมัติ" ใน QuestRequestCard
- `sendCheer(cheerPresetId)` — เมื่อกดปุ่ม cheer ใด ๆ

### Derived values (คำนวณใน component)
```ts
const completedCount = quests.filter(q => q.completed).length;
const totalCount = quests.length;
const xpToday = quests.filter(q => q.completed).reduce((s, q) => s + q.xpReward, 0);
const percentDone = totalCount > 0 ? Math.round((completedCount / totalCount) * 100) : 0;
const pendingRequest = pendingQuestRequests.find(r => r.status === 'pending') ?? null;
```

### Types ที่ต้องใช้
```ts
// จาก architecture.md types (NEW)
interface QuestRequest {
  id: string;
  kidName: string;
  requestedAt: string;
  libraryIds: string[];   // max 3
  status: 'pending' | 'approved' | 'denied';
  respondedAt?: string;
}
interface KidProgress {
  totalXp: number;
  currentLevel: number;
  currentLevelTitle: string;
  xpToNextLevel: number;
  xpThisLevel: number;
  streakDays: number;
  lastStreakDate: string;
}
```

---

## UI Spec — ทีละ section

### Section: Header
- bg: `#185FA5`
- SafeAreaView (top) + padding horizontal 20px, padding vertical 16px
- **Row layout** (space-between):
  - Left: Column
    - Text: `"Dashboard 📊"` — font size 20, font weight bold, color `#FFFFFF`
    - Text: `"{kidProfile.name}"` — font size 14, color `rgba(255,255,255,0.8)`
  - Right: Avatar circle
    - Size: 44×44px, border-radius 22, bg `rgba(255,255,255,0.2)`
    - Text (emoji): `"{kidProfile.avatar}"` ขนาด 24px (default cat `"🐱"`)

### Section: Stats Grid (`<StatGrid />`)
- Layout: Row, 3 boxes เท่ากัน, margin horizontal 16px, margin top 16px, gap 10px
- Container card: bg `#FFFFFF`, border-radius 12px, padding 14px, shadow: elevation 2
- **Box 1 — ทำแล้ว**
  - Label: `"ทำแล้ว"` — font size 12, color `#888780`
  - Value: `"{completedCount}"` — font size 28, font weight bold, color `#185FA5`
  - Sub: `"ภารกิจ"` — font size 11, color `#888780`
- **Box 2 — ทั้งหมด**
  - Label: `"ทั้งหมด"` — font size 12, color `#888780`
  - Value: `"{totalCount}"` — font size 28, font weight bold, color `#2C2C2A`
  - Sub: `"ภารกิจ"` — font size 11, color `#888780`
- **Box 3 — XP วันนี้**
  - Label: `"XP วันนี้"` — font size 12, color `#888780`
  - Value: `"{xpToday}"` — font size 28, font weight bold, color `#534AB7`
  - Sub: `"คะแนน"` — font size 11, color `#888780`

### Section: Progress Summary Card (`<ProgressSummaryCard />`)
- Container: bg `#FFFFFF`, border-radius 12px, margin horizontal 16px, margin top 10px, padding 16px, shadow elevation 2
- Row (space-between, align center):
  - Text: `"{percentDone}% สำเร็จ"` — font size 15, font weight 600, color `#2C2C2A`
  - Text: `"{totalEarnedMinutes} นาที"` — font size 13, color `#185FA5`, font weight 500
- Progress bar (full width, margin top 10px):
  - Track: bg `#EEF5FC`, height 10px, border-radius 5px
  - Fill: bg `#185FA5`, width = `{percentDone}%`, border-radius 5px
  - Animate: `Animated.timing` width เมื่อ value เปลี่ยน (duration 400ms)

### Section: Approval Card (`<ApprovalCard />`)
- **แสดงเมื่อ:** `pendingApproval === true`
- **ซ่อนเมื่อ:** `pendingApproval === false`
- Container: bg `#FCEBEB`, border 1.5px solid `#F09595`, border-radius 12px, margin horizontal 16px, margin top 10px, padding 16px
- Row (align center, gap 10px):
  - Icon: `"🔔"` font size 24
  - Column:
    - Text: `"ลูกขอปลดล็อกเวลาเล่น!"` — font size 15, font weight bold, color `#2C2C2A`
    - Text: `"ทำ {completedCount} ภารกิจ — ได้ {totalEarnedMinutes} นาที"` — font size 13, color `#888780`
- Buttons row (margin top 12px, gap 10px):
  - ปุ่ม "อนุมัติ":
    - bg `#185FA5`, border-radius 8px, padding vertical 10px, flex 1
    - Text: `"อนุมัติ"` — color `#FFFFFF`, font size 14, font weight 600
    - On press: `approveUnlock()` → แสดง brief success feedback (ปุ่มกระพริบ 200ms)
  - ปุ่ม "ปฏิเสธ":
    - bg `#FFFFFF`, border 1.5px solid `#F09595`, border-radius 8px, padding vertical 10px, flex 1
    - Text: `"ปฏิเสธ"` — color `#E24B4A`, font size 14, font weight 600
    - On press: `denyUnlock()`

### Section: Quest Request Card (`<QuestRequestCard />`)
- **แสดงเมื่อ:** `pendingRequest !== null`
- **ซ่อนเมื่อ:** ไม่มี pending request
- Container: bg `#F5F3FF`, border 1.5px solid `#C4BAEF`, border-radius 12px, margin horizontal 16px, margin top 10px, padding 16px
- Row header (align center, gap 10px):
  - Icon: `"➕"` font size 24
  - Text: `"ลูกขอภารกิจเพิ่ม!"` — font size 15, font weight bold, color `#534AB7`
- Quest list (margin top 10px): แสดงชื่อ quest แต่ละอัน (lookup จาก QUEST_LIBRARY ด้วย libraryId)
  - แต่ละ item: Row, gap 8px
    - Icon emoji จาก QUEST_LIBRARY (font size 16)
    - Text: quest title ภาษาไทย — font size 13, color `#2C2C2A`
- Buttons row (margin top 12px, gap 10px):
  - ปุ่ม "อนุมัติ":
    - bg `#534AB7`, border-radius 8px, padding vertical 10px, flex 1
    - Text: `"อนุมัติ"` — color `#FFFFFF`, font size 14, font weight 600
    - On press: `approveQuestRequest(pendingRequest.id)`
  - ปุ่ม "ไม่อนุมัติ":
    - bg `#FFFFFF`, border 1.5px solid `#C4BAEF`, border-radius 8px, padding vertical 10px, flex 1
    - Text: `"ไม่อนุมัติ"` — color `#534AB7`, font size 14, font weight 600
    - On press: `denyQuestRequest(pendingRequest.id)`

### Section: Cheer Section (`<CheerSection />`)
- Container: bg `#FFFFFF`, border-radius 12px, margin horizontal 16px, margin top 10px, padding 16px, shadow elevation 2
- Header text: `"ส่งกำลังใจให้ลูก 💬"` — font size 15, font weight bold, color `#2C2C2A`, margin bottom 12px
- Cheer buttons: แสดง 3 ปุ่มจาก CHEER_PRESETS (c01, c02, c03 หรือตาม mockup)
  - **ข้อความที่แสดง:**
    - `"เก่งมากลูก! พ่อภูมิใจมากเลย"` (c01)
    - `"หนูทำได้ดีมากเลย แม่รักหนูนะ"` (c02)
    - `"อีกนิดเดียวก็ครบแล้ว สู้ๆ นะ!"` (c03)
  - Layout: Column, gap 8px
  - แต่ละปุ่ม:
    - bg `#EEF5FC`, border-radius 8px, padding 12px, flex-direction row, align-items center, gap 10px
    - Text: ข้อความ cheer — font size 13, color `#185FA5`
    - On press: `sendCheer(presetId)` → แสดง sent feedback
- **Sent feedback** (หลังกด):
  - แทนที่ปุ่มที่กด ด้วย row: `"✓ ส่งให้ลูกแล้ว!"` — color `#3B6D11`, font size 13
  - Feedback หายใน 2 วินาที (setTimeout) → กลับเป็นปุ่มปกติ
  - ระหว่าง 2 วินาที ปุ่มทั้ง 3 disabled

### Section: Bottom Tab Bar (`<ParentTabBar />`)
- Fixed bottom, bg `#FFFFFF`, border-top 1px solid `#EEF5FC`
- 3 tabs เท่ากัน:
  - **Tab 1 — ภาพรวม** (active บน parent/index.tsx)
    - Icon: `"📊"` หรือ home icon
    - Label: `"ภาพรวม"` — font size 11
    - Active color: `#185FA5`
    - Inactive color: `#B4B2A9`
    - On press: navigate `/(parent)/` (already here)
  - **Tab 2 — จัดการ**
    - Icon: `"🗂️"` หรือ list icon
    - Label: `"จัดการ"` — font size 11
    - On press: navigate `/(parent)/manage`
  - **Tab 3 — ตั้งค่า**
    - Icon: `"⚙️"` หรือ settings icon
    - Label: `"ตั้งค่า"` — font size 11
    - On press: navigate `/(parent)/settings`

### Section: Scroll Container
- ทุก section ข้างบน (ยกเว้น Header และ Bottom Tab) อยู่ใน `<ScrollView>` แบบ vertical
- `contentContainerStyle`: paddingBottom 20px
- bg: `#EEF5FC`

---

## Behavior & Logic

| Condition | Behavior |
|---|---|
| `pendingApproval === false` | ซ่อน ApprovalCard ทั้งหมด (ไม่ใช่ disable — ซ่อน) |
| `pendingApproval === true` | แสดง ApprovalCard พร้อม completedCount + totalEarnedMinutes |
| กด "อนุมัติ" unlock | `approveUnlock()` → `pendingApproval` เปลี่ยนเป็น false → card หาย → timer เริ่มฝั่ง kid |
| กด "ปฏิเสธ" unlock | `denyUnlock()` → `pendingApproval` เปลี่ยนเป็น false → card หาย |
| ไม่มี pending quest request | ซ่อน QuestRequestCard |
| มี pending quest request | แสดง QuestRequestCard พร้อมรายชื่อ quest ที่ขอ |
| กด "อนุมัติ" quest request | `approveQuestRequest(id)` → quest เพิ่มใน today list ของลูก → card หาย |
| กด "ไม่อนุมัติ" quest request | `denyQuestRequest(id)` → card หาย |
| กด cheer button | `sendCheer(id)` → feedback `"✓ ส่งให้ลูกแล้ว!"` 2 วินาที → กลับปกติ |
| `totalCount === 0` | Progress bar แสดง 0%, StatsGrid แสดง 0/0/0 |

---

## Edge Cases

| Case | Expected behavior |
|---|---|
| ลูกยังไม่ได้รับภารกิจ (quests empty) | Stats แสดง 0/0 ทั้งหมด, Progress 0% |
| ลูกทำภารกิจครบทุกอัน | Progress bar เต็ม 100%, bg fill จบที่ขอบขวา |
| มีทั้ง pendingApproval + pendingQuestRequest พร้อมกัน | แสดงทั้งสอง card พร้อมกัน (scroll ดูได้) |
| kidProfile.avatar ไม่ใช่ emoji | render ตามปกติ (emoji ทุกตัว render ได้) |
| `pendingQuestRequests` มีหลาย request พร้อมกัน | แสดงเฉพาะ request แรกที่ status = 'pending' |
| กด cheer เร็วๆ หลายครั้ง | ป้องกัน double-tap: disable ปุ่มทั้ง 3 ระหว่าง feedback |
| `xpToday === 0` | Stats Box 3 แสดง "0" (ไม่แสดง dash หรือ "-") |

---

## Acceptance Criteria
- [ ] Header แสดงชื่อลูกและ avatar emoji ถูกต้อง
- [ ] Stats Grid แสดง completedCount / totalCount / xpToday ครบถ้วน
- [ ] Progress bar เคลื่อนไหว animate เมื่อ completedCount เปลี่ยน
- [ ] ApprovalCard ปรากฏเฉพาะเมื่อ `pendingApproval === true`
- [ ] กด "อนุมัติ" unlock → card หายภายใน 300ms
- [ ] กด "ปฏิเสธ" → card หายภายใน 300ms
- [ ] QuestRequestCard แสดงรายชื่อ quest ที่ลูกขอ (lookup จาก QUEST_LIBRARY)
- [ ] กด cheer button แสดง `"✓ ส่งให้ลูกแล้ว!"` และ disable ปุ่มชั่วคราว
- [ ] Bottom tab navigation ทำงานถูกต้องทั้ง 3 tabs
- [ ] Tab "ภาพรวม" active highlight เมื่ออยู่ใน parent/index.tsx

## Out of Scope
- Push notification เมื่อลูกขอปลดล็อก (sprint นี้ยังไม่ทำ)
- ประวัติ cheer messages
- Multiple kid profiles
- Weekly/monthly stats
