# Spec: Quest Management (Parent)

> version: 1.0 | สถานะ: Draft
> mockup reference: Parent — จัดการภารกิจ Screen

## Overview
หน้าสำหรับพ่อแม่จัดการภารกิจวันนี้ — สุ่มอัตโนมัติหรือเลือกจากคลัง, แก้ไขรายการ, แล้วส่งให้ลูก Quest ที่มี `isMandatory: true` ลบไม่ได้

## Screens / Components ที่เกี่ยวข้อง
- `app/parent/manage.tsx` — Quest Management screen
- `app/parent/library.tsx` — Quest Library picker (navigate มาจาก manage)
- `components/QuestManageCard.tsx` — card แสดงแต่ละ quest ใน today list พร้อมปุ่มลบ
- `components/XpBadge.tsx` — badge แสดง XP reward
- `components/ParentTabBar.tsx` — bottom tab bar (ใช้ร่วมกับ parent screens)

## Data & State
### อ่านจาก store
- `useQuestStore(s => s.quests)` — `Quest[]` รายการภารกิจวันนี้ที่ assign ให้ลูกแล้ว
- (ใช้ QUEST_LIBRARY constant โดยตรง — ไม่ได้ใน store)

### เรียก actions
- `randomizeQuests()` — เมื่อกดปุ่ม "สุ่มภารกิจ" (action ใหม่ ดูด้านล่าง)
- `removeQuest(id)` — เมื่อกดปุ่มลบ quest (ที่ไม่ใช่ mandatory)
- `saveQuestsForKid()` — เมื่อกดปุ่ม "บันทึกและส่งให้ลูก" (เรียก `sendQuestsToKid()` ใน store)
- `addQuestsFromLibrary(libraryIds)` — เรียกจาก library.tsx เมื่อ confirm selection

### New Actions ที่ต้องเพิ่มใน store
```ts
randomizeQuests: () => {
  // 1. เริ่มจาก mandatory quests ทั้งหมด (q01,q03,q08,q09,q10,q18) = 6 items
  // 2. random เลือกจาก optional quests จนรวมเป็น 9 items (เพิ่ม 3 optional)
  // 3. set quests = selected items (completed: false)
  // หมายเหตุ: reset completed state ทั้งหมดด้วย
}

saveQuestsForKid: () => {
  // alias ของ sendQuestsToKid() — mark quest list ว่า published
  // set questsLastSentAt = now
}
```

### Types ที่ต้องใช้
```ts
// Quest interface (NEW fields จาก architecture.md)
interface Quest {
  id: string;
  title: string;
  description: string;
  icon: string;             // emoji
  rewardMinutes: number;
  xpReward: number;         // NEW
  completed: boolean;
  completedAt?: string;
  isMandatory: boolean;     // NEW
  libraryId?: string;       // NEW
}

// QuestLibraryItem (constant — ไม่ใช่ store)
interface QuestLibraryItem {
  id: string;               // q01-q22
  title: string;
  description: string;
  icon: string;
  defaultRewardMinutes: number;
  defaultXpReward: number;
  isMandatory: boolean;
  category: QuestCategory;
}
```

---

## UI Spec — ทีละ section

### Section: Header
- bg: `#185FA5`, padding horizontal 20px, padding vertical 16px, SafeAreaView (top)
- Text: `"จัดการภารกิจ 🗂️"` — font size 20, font weight bold, color `#FFFFFF`
- Text (subtitle): `"สุ่ม / เลือกจากคลัง"` — font size 13, color `rgba(255,255,255,0.75)`, margin top 2px

### Section: Action Buttons Row
- Layout: Row, gap 10px, margin horizontal 16px, margin top 16px
- **ปุ่ม "สุ่มภารกิจ":**
  - bg `#185FA5`, border-radius 10px, flex 1, padding vertical 14px
  - Text: `"🔀 สุ่มภารกิจ"` — color `#FFFFFF`, font size 14, font weight 600, text-align center
  - On press: `randomizeQuests()` → today list อัปเดต → แสดง brief toast (ดู Behavior)
- **ปุ่ม "เลือกเอง":**
  - bg `#FFFFFF`, border 1.5px solid `#185FA5`, border-radius 10px, flex 1, padding vertical 14px
  - Text: `"📚 เลือกเอง"` — color `#185FA5`, font size 14, font weight 600, text-align center
  - On press: navigate `/(parent)/library`

### Section: Today Quest List Header
- Layout: Row (space-between, align center), margin horizontal 16px, margin top 20px, margin bottom 8px
- Left text: `"ภารกิจวันนี้"` — font size 16, font weight bold, color `#2C2C2A`
- Right text: `"{quests.length} รายการ"` — font size 13, color `#888780`
- ปุ่ม "ล้างทั้งหมด" (ชิดขวา):
  - Text: `"ล้างทั้งหมด"` — font size 12, color `#E24B4A`, underline
  - On press: Alert confirm → `"ต้องการล้างภารกิจทั้งหมดใช่ไหม?\n(mandatory จะยังคงอยู่)"` → กด OK → remove all non-mandatory quests

### Section: Today Quest List (ScrollView)
- ScrollView แบบ vertical, padding horizontal 16px, gap 8px
- แต่ละ quest แสดงด้วย `<QuestManageCard />`

#### `<QuestManageCard />` spec
- Container: bg `#FFFFFF`, border-radius 12px, padding 14px, flex-direction row, align-items center, gap 12px, shadow elevation 1
- **Icon circle:**
  - Size: 44×44px, border-radius 22
  - bg: ตาม category color (ดู Quest Category Colors ด้านล่าง)
  - Text: quest.icon (emoji), font size 22
- **Content (flex 1):**
  - Title: `"{quest.title}"` — font size 14, font weight 600, color `#2C2C2A`
  - Sub: `"{quest.description}"` — font size 12, color `#888780`, numberOfLines 1
  - Row (margin top 4px):
    - XP Badge: bg `#534AB7`, border-radius 10, padding horizontal 8px, padding vertical 2px
      - Text: `"+{quest.xpReward} XP"` — font size 11, color `#FFFFFF`, font weight 600
    - Minutes badge: bg `#EEF5FC`, border-radius 10, padding horizontal 8px, padding vertical 2px, margin left 6px
      - Text: `"+{quest.rewardMinutes} นาที"` — font size 11, color `#185FA5`
- **Mandatory Badge (ถ้า isMandatory):**
  - วงกลมสีแดง: size 8×8px, bg `#E24B4A`, border-radius 4, position absolute top 8 right 8
  - (ไม่มีปุ่มลบ)
- **Delete Button (ถ้าไม่ mandatory):**
  - Icon: `"✕"` หรือ trash icon, size 20×20px, color `#888780`
  - On press: `removeQuest(quest.id)` (ไม่มี confirm dialog — ลบทันที)
  - Touchable area: 40×40px (padding)

#### Quest Category Colors (icon circle bg)
```
education  → #EEF5FC (bg) / #185FA5 (icon)
exercise   → #D4F0E8 (bg) / #1D9E75 (icon)
chores     → #FFF8F0 (bg) / #FF8C42 (icon)
reading    → #F5F3FF (bg) / #534AB7 (icon)
social     → #FFF0E6 (bg) / #D85A30 (icon)
health     → #EAF3DE (bg) / #3B6D11 (icon)
creativity → #FCEBEB (bg) / #E24B4A (icon)
```

### Section: Empty State
- แสดงเมื่อ `quests.length === 0`
- กลาง ScrollView area: icon `"📋"` (font size 48) + text `"ยังไม่มีภารกิจ"` (font size 16, color `#888780`) + sub text `"กด 'สุ่มภารกิจ' หรือ 'เลือกเอง' เพื่อเพิ่มภารกิจ"` (font size 13, color `#B4B2A9`, text-align center)

### Section: Save Button
- Fixed bottom (ด้านบน bottom tab bar), bg `#FFFFFF`, padding horizontal 16px, padding vertical 12px, border-top 1px solid `#EEF5FC`
- ปุ่ม "บันทึกและส่งให้ลูก":
  - bg: `#185FA5` (enabled) / `#B4B2A9` (disabled)
  - border-radius 12px, padding vertical 16px, full width
  - Text: `"✓ บันทึกและส่งให้ลูก"` — color `#FFFFFF`, font size 16, font weight bold, text-align center
  - **Disabled เมื่อ:** `quests.length === 0`
  - On press (enabled): `saveQuestsForKid()` → navigate `/(parent)/` → Alert `"✓ ส่งภารกิจให้ลูกแล้ว! {quests.length} ภารกิจ"`

### Section: Bottom Tab Bar
- ใช้ `<ParentTabBar />` เดียวกับ Dashboard
- Tab "จัดการ" active highlight ในหน้านี้

---

## Behavior & Logic

| Condition | Behavior |
|---|---|
| กด "สุ่มภารกิจ" | mandatory 6 อัน + random optional 3 อัน = 9 quests รวม → set quests → show toast `"สุ่มภารกิจแล้ว! 9 ภารกิจ"` (2 วินาที) |
| กด "สุ่มภารกิจ" ซ้ำ | override quests เดิม (reset completed ทั้งหมด) |
| กด "เลือกเอง" | navigate ไป library.tsx |
| กลับจาก library.tsx | quests อัปเดตใน store → today list re-render |
| กดลบ quest ที่ไม่ mandatory | removeQuest(id) → หายจาก list ทันที |
| กดลบ quest ที่ mandatory | ไม่มีปุ่มลบ (ไม่สามารถลบได้) |
| กด "ล้างทั้งหมด" → confirm OK | ลบ quest ทุกอัน ยกเว้น mandatory → คงเหลือ mandatory 6 อัน |
| กด "ล้างทั้งหมด" → cancel | ไม่เปลี่ยนอะไร |
| กด "บันทึกและส่งให้ลูก" (quests > 0) | `saveQuestsForKid()` → navigate dashboard → Alert |
| กด "บันทึกและส่งให้ลูก" (quests = 0) | ปุ่ม disabled ไม่ตอบสนอง |
| quests ถูก complete บางส่วน | แสดงใน manage list ตามปกติ (completed state ไม่กระทบ UI ที่นี่) |

---

## Edge Cases

| Case | Expected behavior |
|---|---|
| optional quests มีน้อยกว่า 3 ใน library | สุ่มเท่าที่มี (ได้น้อยกว่า 9) |
| quest ที่เลือกจาก library มีอยู่ใน today แล้ว | `addQuestsFromLibrary` ไม่เพิ่มซ้ำ (skip duplicate libraryId) |
| กด "สุ่ม" ขณะลูกกำลังทำ quest อยู่ | ควร Alert warn `"การสุ่มจะ reset ภารกิจที่ทำไปแล้ว ต้องการดำเนินการต่อหรือไม่?"` |
| `quests.length > 9` (เพิ่มจาก library เกิน) | ไม่ cap — แสดงทุกอัน (no hard limit ใน manage screen) |
| mandatory quest ยังไม่อยู่ใน today | ถ้าไม่มี mandatory ใน quests → "ล้างทั้งหมด" จะเพิ่ม mandatory กลับ (reset to mandatory only) |
| navigate กลับมาจาก library โดยไม่เลือกอะไร | today list ไม่เปลี่ยน |

---

## Acceptance Criteria
- [ ] Header แสดง `"จัดการภารกิจ 🗂️"` และ subtitle ถูกต้อง, bg `#185FA5`
- [ ] กด "สุ่มภารกิจ" → today list มี mandatory 6 อัน + optional 3 อัน รวม 9 อัน
- [ ] กด "เลือกเอง" → navigate ไป library.tsx
- [ ] แต่ละ quest card แสดง icon, title, description, XP badge, minutes badge
- [ ] mandatory quest มีจุดแดงและไม่มีปุ่มลบ
- [ ] non-mandatory quest มีปุ่มลบและลบออกได้ทันที
- [ ] "ล้างทั้งหมด" เหลือเฉพาะ mandatory (6 อัน)
- [ ] ปุ่ม "บันทึกและส่งให้ลูก" disabled เมื่อ quests = 0
- [ ] กด "บันทึกและส่งให้ลูก" → navigate dashboard + Alert ถูกต้อง
- [ ] Empty state แสดงเมื่อ quests = 0
- [ ] Tab "จัดการ" active highlight ใน bottom tab

## Out of Scope
- Drag-to-reorder quests
- Custom quest creation (add custom title/description)
- Quest scheduling (assign วันใดวันหนึ่ง)
- Per-quest reward minute override
