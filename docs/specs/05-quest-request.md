# Spec: Quest Request (Kid)

> version: 1.0 | สถานะ: Draft
> mockup reference: Kid — ขอภารกิจเพิ่ม Screen

## Overview
หน้าสำหรับลูกเลือกภารกิจเพิ่มเติมจากคลัง (สูงสุด 3 รายการ) แล้วส่ง request ให้พ่อแม่อนุมัติ — แสดงเฉพาะภารกิจที่ยังไม่อยู่ใน today list

## Screens / Components ที่เกี่ยวข้อง
- `app/quest-request.tsx` — Quest Request screen
- `components/QuestRequestItem.tsx` — แต่ละ quest ใน list (selectable)
- `components/SelectionCounter.tsx` — "เลือก X/3" badge

## Data & State
### อ่านจาก store
- `useQuestStore(s => s.quests)` — `Quest[]` รายการที่มีใน today แล้ว (เพื่อ filter ออก)
- (ใช้ QUEST_LIBRARY constant โดยตรงสำหรับ available quests)

### เรียก actions
- `submitQuestRequest(libraryIds)` — เมื่อกดปุ่ม "ส่งให้พ่อแม่" พร้อม selected libraryIds

### Local State (ใน component)
```ts
const [selected, setSelected] = useState<string[]>([]);  // libraryIds ที่เลือก (max 3)
```

### Derived values
```ts
// Quests ที่แสดงให้เลือก = library ทั้งหมด - ที่อยู่ใน today แล้ว
const todayLibraryIds = quests
  .map(q => q.libraryId)
  .filter(Boolean) as string[];

const availableQuests = QUEST_LIBRARY.filter(
  item => !todayLibraryIds.includes(item.id)
);

const canSubmit = selected.length > 0;
const isMaxSelected = selected.length >= 3;
```

### Types ที่ต้องใช้
```ts
// QuestRequest (NEW — จาก architecture.md)
interface QuestRequest {
  id: string;
  kidName: string;
  requestedAt: string;           // ISO date string
  libraryIds: string[];          // max 3 items
  status: 'pending' | 'approved' | 'denied';
  respondedAt?: string;
}

// QuestLibraryItem (constant)
interface QuestLibraryItem {
  id: string;
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
- bg: `#534AB7`
- SafeAreaView (top), padding horizontal 20px, padding vertical 16px
- Row (align center, gap 12px):
  - ปุ่ม "← กลับ":
    - Icon: back arrow (`"←"` หรือ chevron-left), color `#FFFFFF`, font size 20
    - On press: `router.back()`
    - Touchable area: 40×40px
  - Column (flex 1):
    - Text: `"ขอภารกิจเพิ่ม ➕"` — font size 20, font weight bold, color `#FFFFFF`
    - Text (subtitle): `"เลือกภารกิจที่อยากทำเพิ่มวันนี้"` — font size 13, color `rgba(255,255,255,0.8)`
  - SelectionCounter (ชิดขวา, แสดงเมื่อ selected.length > 0):
    - bg `rgba(255,255,255,0.2)`, border-radius 16, padding horizontal 12, padding vertical 4
    - Text: `"เลือก {selected.length}/3"` — font size 13, color `#FFFFFF`, font weight 600

### Section: Info Bar
- bg `#3C3489` (darker purple), padding horizontal 20px, padding vertical 10px
- Text: `"เลือกได้สูงสุด 3 ข้อ · พ่อแม่จะอนุมัติก่อนนะ"` — font size 13, color `rgba(255,255,255,0.85)`, text-align center

### Section: Quest List (ScrollView)
- bg: `#F5F3FF`, padding horizontal 16px, padding top 12px, padding bottom 100px (เหนือปุ่ม)
- แต่ละ quest: `<QuestRequestItem />`
- Gap ระหว่าง item: 8px

#### `<QuestRequestItem />` spec

**Props:**
```ts
interface QuestRequestItemProps {
  item: QuestLibraryItem;
  selected: boolean;
  disabled: boolean;   // true เมื่อ isMaxSelected && !selected (ป้องกันเลือกเกิน 3)
  onToggle: (id: string) => void;
}
```

**Layout:** Pressable, flex-direction row, align-items center, gap 12px, padding 14px, border-radius 14px

**State: ไม่ได้เลือก (ปกติ)**
- bg `#FFFFFF`, border 1.5px solid transparent (ไม่มี border)
- shadow elevation 1

**State: เลือกแล้ว (selected)**
- bg `#FFF0E6`, border 2px solid `#FF8C42`
- shadow elevation 2

**State: Disabled (ไม่สามารถเลือกเพิ่มได้ เพราะครบ 3 แล้ว)**
- bg `#F5F3FF`, opacity 0.5
- (ไม่ interactive)

**Icon circle:**
- Size: 48×48px, border-radius 24
- bg: ตาม category (ดู Quest Category Colors ใน spec 03)
- Text: item.icon (emoji), font size 24

**Content (flex 1):**
- Title: `"{item.title}"` — font size 14, font weight 600, color `#2C2C2A`
- Sub: `"{item.description}"` — font size 12, color `#888780`, numberOfLines 2
- Badges row (margin top 4px):
  - XP: bg `#534AB7`, border-radius 10, padding horizontal 8, padding vertical 2
    - Text: `"+{item.defaultXpReward} XP"` — font size 11, color `#FFFFFF`, font weight 600
  - Minutes: bg `#EEF5FC`, border-radius 10, padding horizontal 8, padding vertical 2, margin left 6
    - Text: `"+{item.defaultRewardMinutes} นาที"` — font size 11, color `#185FA5`

**Checkmark (ชิดขวา):**
- แสดงเมื่อ selected: วงกลม bg `#FF8C42`, size 24×24, border-radius 12
  - Text: `"✓"` — color `#FFFFFF`, font size 14, font weight bold
- ไม่แสดงเมื่อ ไม่ได้เลือก (ไม่ใช่ empty circle — ซ่อนทั้งหมด)

**On press:** `onToggle(item.id)`

### Section: Empty State
- แสดงเมื่อ `availableQuests.length === 0`
- กลางหน้า: icon `"✨"` font size 56 + text `"ทำภารกิจครบทุกอันแล้ว!"` (font size 16, color `#888780`) + sub `"ไม่มีภารกิจเพิ่มเติมในคลัง"` (font size 13, color `#B4B2A9`)

### Section: Submit Button
- Fixed bottom, bg `#FFFFFF`, padding horizontal 16px, padding vertical 12px, border-top 1px solid `#E8E6F8`
- ปุ่ม "ส่งให้พ่อแม่":
  - **Enabled** (เมื่อ `selected.length > 0`):
    - bg `#534AB7`, border-radius 14px, padding vertical 16px, full width
    - Text: `"📨 ส่งให้พ่อแม่ ({selected.length} ภารกิจ)"` — color `#FFFFFF`, font size 16, font weight bold
  - **Disabled** (เมื่อ `selected.length === 0`):
    - bg `#C4BAEF`, border-radius 14px, padding vertical 16px, full width
    - Text: `"📨 ส่งให้พ่อแม่"` — color `rgba(255,255,255,0.6)`, font size 16, font weight bold
    - Disabled = true
  - **On press (enabled):** `submitQuestRequest(selected)` → navigate `router.back()` → (ไม่ต้องแสดง success alert ที่นี่ — kid home จะเห็น state เปลี่ยน)

---

## Behavior & Logic

| Condition | Behavior |
|---|---|
| tap quest ที่ยังไม่เลือก + `selected.length < 3` | เพิ่มเข้า selected, item เปลี่ยนเป็น selected state |
| tap quest ที่เลือกแล้ว | ยกเลิกการเลือก, item กลับ normal state |
| `selected.length === 3` | quest ที่ยังไม่เลือกทั้งหมด disabled (opacity 0.5, non-interactive) |
| tap quest ที่ selected ขณะ max | ยกเลิกได้ปกติ (selected → unselected) |
| กด "ส่งให้พ่อแม่" | `submitQuestRequest(selected)` → store สร้าง QuestRequest ใหม่ status 'pending' → `router.back()` |
| กลับหน้าก่อนโดยไม่ส่ง | ไม่เปลี่ยน store state ใด |
| quest ทุกอันใน library อยู่ใน today แล้ว | `availableQuests.length === 0` → แสดง empty state |

---

## Edge Cases

| Case | Expected behavior |
|---|---|
| `todayLibraryIds` มี quest ที่ไม่มี libraryId (custom quest) | filter ปกติ (libraryId = undefined ไม่ match ค่าใด) |
| กด submit เร็วๆ (double tap) | disable ปุ่มทันทีหลัง press ครั้งแรก (setSubmitting state) |
| ยังมี pending request อยู่ | `submitQuestRequest` สร้าง request ใหม่ (parent เห็น 2 request) — policy: allow multiple |
| `availableQuests` มีน้อยกว่า 3 | เลือกได้ทั้งหมดที่มี (max = min(3, availableQuests.length)) |
| QUEST_LIBRARY ว่าง | empty state แสดง (ไม่ crash) |
| navigate กลับ + กลับมาใหม่ | `selected` reset เป็น [] (local state ไม่ persist) |

---

## Acceptance Criteria
- [ ] Header bg `#534AB7`, แสดง title + subtitle + ปุ่มกลับ
- [ ] Info bar แสดง `"เลือกได้สูงสุด 3 ข้อ · พ่อแม่จะอนุมัติก่อนนะ"`
- [ ] List แสดงเฉพาะ quest ที่ไม่อยู่ใน today list
- [ ] tap quest → เปลี่ยน selected state (bg `#FFF0E6`, border `#FF8C42`)
- [ ] tap quest ที่ selected → ยกเลิก (กลับ normal)
- [ ] เลือกครบ 3 → quest ที่เหลือ disabled
- [ ] SelectionCounter แสดง `"เลือก X/3"` เมื่อ selected > 0
- [ ] ปุ่ม submit disabled เมื่อ selected = 0
- [ ] ปุ่ม submit enabled แสดง `"📨 ส่งให้พ่อแม่ (X ภารกิจ)"`
- [ ] กด submit → `submitQuestRequest` เรียก → navigate กลับ
- [ ] Empty state แสดงเมื่อ availableQuests = 0

## Out of Scope
- Parent approval flow (อยู่ใน spec 02-parent-dashboard.md)
- ประวัติ requests ที่ส่งไป
- Quest ที่พ่อแม่ปฏิเสธ → แสดง feedback ให้ลูก (sprint ถัดไป)
