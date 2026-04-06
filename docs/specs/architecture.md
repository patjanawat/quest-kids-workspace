# Architecture Spec — Little Heroes

**Date:** 2026-04-05
**Status:** Draft — for Spec Writer

---

## 1. Gap Analysis

### Module 1 — Auth (PIN Gate)

| Feature | Code มีอยู่ | Mockup ต้องการ | Gap |
|---|---|---|---|
| PIN input UI | `PinInput.tsx` (basic keypad) | Full numeric keypad พร้อม sub-labels (ABC, DEF…), lockout UI | ออกแบบ keypad ใหม่ทั้งหมด |
| Lockout logic | ใน store: 3 fails → 5 min lockout | เหมือนกัน | OK |
| Parent area header | `#EA6C1A` (orange) ไม่ตรง | Header `#185FA5` (blue) | ต้องแยก theme parent/kid |
| PIN screen bg | generic | Top area `#185FA5`, keypad area `#fff` | ต้องแยก layout |
| Lock icon | ไม่มี | วงกลม semi-transparent icon ด้านบน | เพิ่ม visual element |

### Module 2 — Parent Dashboard

| Feature | Code มีอยู่ | Mockup ต้องการ | Gap |
|---|---|---|---|
| Stats overview | ไม่มี | 3-column stat grid (ภารกิจ/เวลา/เซสชัน) พร้อม progress bar | ต้องสร้าง `StatGrid` component |
| Approval card | มี (basic) | Card สีชมพู (`#FCEBEB`) พร้อม border `#F09595` | ต้องปรับสไตล์ |
| Cheer feature | ไม่มีเลย | Cheer buttons list → ส่ง message ไปหาลูก | ต้องสร้างใหม่ทั้งหมด |
| Quest request approval | ไม่มี | Card อนุมัติคำขอ quest จากลูก | ต้องสร้างใหม่ |
| Navigation | flat tabs (`approve/quests/settings`) | Tab bar ด้านล่าง (Dashboard/ภารกิจ/คลัง/ตั้งค่า) | เปลี่ยน navigation structure |
| Quest management | inline ใน parent.tsx | หน้าแยก `manage.tsx` + `library.tsx` | ต้อง refactor |

### Module 3 — Quest Management

| Feature | Code มีอยู่ | Mockup ต้องการ | Gap |
|---|---|---|---|
| Quest list | basic list ใน parent.tsx | List พร้อม mandatory badge (จุดแดง), ลบได้เฉพาะ non-mandatory | ต้องเพิ่ม `isMandatory` flag |
| Quest library | ไม่มี | 22-item library, เลือก multi-select, highlight ที่ active อยู่แล้ว | ต้องสร้าง library screen + constant |
| Random quest | ไม่มี | ปุ่มสุ่มภารกิจจาก library | ต้องเพิ่ม action |
| Send to kid | ไม่มี | ปุ่ม "บันทึกและส่งให้ลูก" | ต้องเพิ่ม action + state |
| XP per quest | `rewardMinutes` เดียว | แต่ละ quest มี `xpReward` แยกจาก `rewardMinutes` | ต้องเพิ่ม field |

### Module 4 — Kid Home

| Feature | Code มีอยู่ | Mockup ต้องการ | Gap |
|---|---|---|---|
| XP bar | `RewardBadge` (minutes) | XP bar พร้อม level, title ("นักสำรวจน้อย"), progress | ต้องสร้าง `XPBar` component |
| Level/Title | ไม่มี | Level number + title string | ต้องเพิ่มใน store/types |
| Streak | ไม่มี | Streak counter (วันติดต่อกัน) | ต้องเพิ่มทั้ง logic และ UI |
| Stat grid | ไม่มี | 3-column: XP/เวลา/Streak | ต้องสร้าง |
| Quest card | มี แต่ minimal | Timer button บน card, done-stripe overlay, XP badge | ต้องปรับ `QuestCard` |
| Cheer message | ไม่มี | Banner/toast แสดง cheer จากพ่อแม่ | ต้องสร้าง |
| Quest request button | ไม่มี | ปุ่ม "ขอภารกิจเพิ่ม" → flow ไปหน้า request | ต้องสร้าง |
| Header color | `#FFF8F2` bg, orange | Header `#FF8C42`, bg `#FFF8F0`, border `#FFD4A8` | ต้องปรับ theme |

### Module 5 — Quest Request

| Feature | Code มีอยู่ | Mockup ต้องการ | Gap |
|---|---|---|---|
| Quest request flow | ไม่มีเลย | Kid เลือก quest จาก library (max 3) → ส่ง request | ต้องสร้างใหม่ทั้งหมด |
| Request state | ไม่มี | `pendingQuestRequests: QuestRequest[]` | ต้องเพิ่มใน state |
| Parent approval | ไม่มี | Card อนุมัติ/ปฏิเสธ ใน parent dashboard | ต้องเพิ่ม |
| Selection UI | ไม่มี | Library grid พร้อม checkbox, counter "เลือก 2/3" | ต้องสร้าง |

### Module 6 — Timer

| Feature | Code มีอยู่ | Mockup ต้องการ | Gap |
|---|---|---|---|
| Screen name | `rewards.tsx` | `timer.tsx` | ต้อง rename/move |
| Theme | light background | Dark theme (`#1a1a2e` bg, `#00FF87` clock) | ต้องสร้าง dark theme สำหรับ timer |
| Play/Pause | มีแค่ Stop | Play / Pause / Stop / Reset buttons | ต้องเพิ่ม pause logic |
| Quest context | global timer เท่านั้น | Per-quest timer (รู้ว่ากำลัง time quest ไหน) | ต้องเพิ่ม `activeQuestId` |
| Timer bar | `TimerBar.tsx` (linear) | Circular progress clock | ต้องสร้าง `CircularTimer` component |
| Low-time warning | color เปลี่ยนเป็น red | เหมือนกัน แต่ animation เพิ่ม | เพิ่ม animation |

---

## 2. Types ที่ต้องเพิ่ม/แก้ไข

```ts
// types/index.ts — แก้ไข Quest interface
export interface Quest {
  id: string;
  title: string;
  description: string;
  icon: string;                  // emoji
  rewardMinutes: number;
  xpReward: number;              // NEW — XP ที่ได้รับเมื่อทำเสร็จ
  completed: boolean;
  completedAt?: string;          // ISO date string
  isMandatory: boolean;          // NEW — ลบไม่ได้ถ้า true
  libraryId?: string;            // NEW — อ้างอิง QuestLibraryItem.id (null = custom)
}

// NEW — Quest Library Item (constant ไม่ใช่ store)
export interface QuestLibraryItem {
  id: string;                    // q01 - q22
  title: string;
  description: string;
  icon: string;
  defaultRewardMinutes: number;
  defaultXpReward: number;
  isMandatory: boolean;          // q01,q03,q08,q09,q10,q18
  category: QuestCategory;
}

export type QuestCategory =
  | 'education'    // การเรียน
  | 'exercise'     // ออกกำลังกาย
  | 'chores'       // งานบ้าน
  | 'reading'      // อ่านหนังสือ
  | 'social'       // สังคม/ครอบครัว
  | 'health'       // สุขภาพ
  | 'creativity';  // ความคิดสร้างสรรค์

// NEW — Quest Request (ลูกขอเพิ่ม)
export interface QuestRequest {
  id: string;
  kidName: string;
  requestedAt: string;           // ISO date string
  libraryIds: string[];          // max 3 items
  status: 'pending' | 'approved' | 'denied';
  respondedAt?: string;
}

// NEW — Cheer Message
export interface CheerMessage {
  id: string;
  text: string;
  emoji: string;
  sentAt: string;                // ISO date string
  readAt?: string;               // ISO date string — ลูกเห็นแล้ว
}

// NEW — Kid Progress (XP/Level/Streak)
export interface KidProgress {
  totalXp: number;
  currentLevel: number;          // 1-based
  currentLevelTitle: string;     // "นักสำรวจน้อย" ฯลฯ
  xpToNextLevel: number;         // XP ที่ต้องการถึง level ถัดไป
  xpThisLevel: number;           // XP สะสมในเลเวลปัจจุบัน
  streakDays: number;            // วันติดต่อกัน
  lastStreakDate: string;        // YYYY-MM-DD
}

// แก้ไข KidProfile
export interface KidProfile {
  name: string;
  avatar: string;                // emoji
  age?: number;                  // NEW — optional, ใช้ใน UI
}

// แก้ไข TimerSession
export interface TimerSession {
  startedAt: string;
  durationMinutes: number;
  approvedBy: 'parent';
  questId?: string;              // NEW — per-quest context
  pausedAt?: string;             // NEW — ISO หรือ undefined
  totalPausedSeconds: number;    // NEW — สะสมเวลา pause
}

// แก้ไข AppState
export interface AppState {
  quests: Quest[];
  settings: ParentSettings;
  progress: KidProgress;         // NEW — แทน totalEarnedMinutes เดิม
  totalEarnedMinutes: number;    // คงไว้เพื่อ backward compat, derive จาก quests
  pendingApproval: boolean;
  pendingQuestRequests: QuestRequest[];  // NEW
  activeCheer: CheerMessage | null;      // NEW — cheer ที่ยังไม่ได้อ่าน
  cheerHistory: CheerMessage[];          // NEW
  timerActive: boolean;
  timerPaused: boolean;                  // NEW
  timerRemainingSeconds: number;
  timerTotalSeconds: number;             // NEW — เก็บ total ไว้คำนวณ progress
  activeQuestId: string | null;          // NEW — quest ที่ timer กำลัง run
  timerSessions: TimerSession[];
  lastResetDate: string;
  pinLockoutUntil: string | null;
  pinFailCount: number;
}

// แก้ไข QuestStoreActions
export interface QuestStoreActions {
  // เดิม (คงไว้)
  completeQuest: (id: string) => void;
  uncompleteQuest: (id: string) => void;
  requestApproval: () => void;
  approveUnlock: () => void;
  denyUnlock: () => void;
  addQuest: (quest: Omit<Quest, 'id' | 'completed' | 'completedAt'>) => void;
  removeQuest: (id: string) => void;
  startTimer: () => void;
  tickTimer: () => void;
  stopTimer: () => void;
  resetDaily: () => void;
  updateSettings: (settings: Partial<ParentSettings>) => void;
  recordPinFail: () => void;
  resetPinFails: () => void;

  // NEW actions (ดูหัวข้อ 3)
  pauseTimer: () => void;
  resumeTimer: () => void;
  resetTimer: () => void;
  startQuestTimer: (questId: string) => void;
  addQuestsFromLibrary: (libraryIds: string[]) => void;
  sendQuestsToKid: () => void;
  submitQuestRequest: (libraryIds: string[]) => void;
  approveQuestRequest: (requestId: string) => void;
  denyQuestRequest: (requestId: string) => void;
  sendCheer: (cheerPresetId: string) => void;
  markCheerRead: () => void;
  updateProgress: () => void;
  checkStreak: () => void;
}
```

---

## 3. Store Actions ที่ต้องเพิ่ม

```ts
// store/questStore.ts — actions ใหม่

pauseTimer: () => {
  // ตั้ง timerPaused = true, timerActive = false
  // บันทึก pausedAt ใน session ปัจจุบัน
},

resumeTimer: () => {
  // ตั้ง timerPaused = false, timerActive = true
  // บวก totalPausedSeconds ใน session
},

resetTimer: () => {
  // reset กลับเป็น timerTotalSeconds
  // ต้องการ parent approval ใหม่ (pendingApproval = true)
  // หรือ policy: reset ได้แค่ถ้า parent ทำ
},

startQuestTimer: (questId: string) => {
  // เหมือน startTimer แต่ set activeQuestId = questId
  // timerTotalSeconds = earnedMinutes * 60
},

addQuestsFromLibrary: (libraryIds: string[]) => {
  // เพิ่ม quest จาก QUEST_LIBRARY constant
  // ไม่เพิ่มถ้า quest ที่มี libraryId เดียวกันอยู่แล้ว
},

sendQuestsToKid: () => {
  // mark state ว่า quest list ถูก "publish" แล้ว
  // kid home จะ refresh quest list
  // เพิ่ม flag `questsLastSentAt: string` ใน state
},

submitQuestRequest: (libraryIds: string[]) => {
  // ลูกส่ง request (max 3 libraryIds)
  // สร้าง QuestRequest ใหม่ status = 'pending'
  // เพิ่มใน pendingQuestRequests
},

approveQuestRequest: (requestId: string) => {
  // หา request → status = 'approved'
  // เรียก addQuestsFromLibrary ด้วย libraryIds ของ request นั้น
},

denyQuestRequest: (requestId: string) => {
  // หา request → status = 'denied'
},

sendCheer: (cheerPresetId: string) => {
  // หา CheerPreset จาก CHEER_PRESETS constant
  // สร้าง CheerMessage ใหม่
  // set activeCheer = message
  // append cheerHistory
},

markCheerRead: () => {
  // set activeCheer.readAt = now
  // set activeCheer = null
},

updateProgress: () => {
  // คำนวณ totalXp จาก completed quests (sum xpReward)
  // คำนวณ currentLevel จาก LEVEL_THRESHOLDS
  // อัปเดต progress ใน state
  // เรียกหลัง completeQuest / uncompleteQuest / resetDaily
},

checkStreak: () => {
  // เรียกตอน useDailyReset
  // ถ้า lastStreakDate = yesterday AND ทำ quest ครบวันก่อน → streakDays++
  // ถ้าข้ามวัน → reset streak = 0
  // อัปเดต lastStreakDate = today
},
```

---

## 4. Navigation Structure

```
app/
├── _layout.tsx              ← Root layout (PaperProvider, Stack)
├── index.tsx                ← Kid Home (tab: ภารกิจ/stats/ขอ)
├── timer.tsx                ← Timer screen (dark theme, per-quest)
│                              รับ params: questId? (optional)
├── quest-request.tsx        ← Kid เลือก quest จาก library (max 3)
└── parent/
    ├── _layout.tsx          ← Parent layout (PIN gate guard)
    │                          ใช้ expo-router group layout
    │                          ถ้า !unlocked → render PIN screen
    ├── index.tsx            ← Dashboard (stats, approval, cheer, requests)
    ├── manage.tsx           ← Quest management (active quests + send to kid)
    ├── library.tsx          ← Quest library (22 items, multi-select, add)
    └── settings.tsx         ← Settings (PIN, kid profile, daily limit)
```

### Navigation Flow

```
Kid Home (index.tsx)
  ├── กด timer button บน quest card → timer.tsx?questId=xxx
  ├── กด "ขอปลดล็อก" → timer.tsx (global unlock flow)
  ├── กด "ขอภารกิจเพิ่ม" → quest-request.tsx
  └── กด "โหมดพ่อแม่" → parent/ (PIN gate)

parent/index.tsx (Dashboard)
  ├── Tab: ภารกิจ → parent/manage.tsx
  ├── Tab: คลัง → parent/library.tsx
  └── Tab: ตั้งค่า → parent/settings.tsx

quest-request.tsx
  └── ส่ง request → กลับ Kid Home (pendingQuestRequests updated)

timer.tsx
  ├── รับ params: questId? จาก expo-router
  ├── play/pause/stop/reset buttons
  └── หมดเวลา → modal → กลับ Kid Home
```

### Routing Params

```ts
// app/timer.tsx
// useLocalSearchParams() → { questId?: string }

// app/quest-request.tsx
// ไม่มี params (stateless navigation)
```

---

## 5. Constants ที่ต้องเพิ่ม

### 5.1 Quest Library (22 items)

```ts
// constants/questLibrary.ts

export const QUEST_LIBRARY: QuestLibraryItem[] = [
  // === MANDATORY (ลบไม่ได้) ===
  { id: 'q01', title: 'ทำการบ้านให้เสร็จ', description: 'ทำการบ้านทุกวิชาให้เสร็จสิ้น', icon: '📚', defaultRewardMinutes: 30, defaultXpReward: 50, isMandatory: true, category: 'education' },
  { id: 'q03', title: 'ออกกำลังกาย 20 นาที', description: 'วิ่ง กระโดด หรือเล่นกีฬา 20 นาที', icon: '🏃', defaultRewardMinutes: 15, defaultXpReward: 40, isMandatory: true, category: 'exercise' },
  { id: 'q08', title: 'แปรงฟันเช้า-เย็น', description: 'แปรงฟันครบทั้ง 2 ครั้ง', icon: '🪥', defaultRewardMinutes: 10, defaultXpReward: 20, isMandatory: true, category: 'health' },
  { id: 'q09', title: 'กินข้าวครบ 3 มื้อ', description: 'กินอาหารครบทุกมื้อ ไม่ข้ามมื้อ', icon: '🍽️', defaultRewardMinutes: 10, defaultXpReward: 20, isMandatory: true, category: 'health' },
  { id: 'q10', title: 'นอนตรงเวลา', description: 'เข้านอนก่อนเวลาที่ตกลงกัน', icon: '😴', defaultRewardMinutes: 20, defaultXpReward: 30, isMandatory: true, category: 'health' },
  { id: 'q18', title: 'พูดขอบคุณและขอโทษ', description: 'ใช้มารยาทที่ดีตลอดวัน', icon: '🙏', defaultRewardMinutes: 10, defaultXpReward: 25, isMandatory: true, category: 'social' },

  // === OPTIONAL ===
  { id: 'q02', title: 'อ่านหนังสือ 15 นาที', description: 'อ่านหนังสือที่ชอบอย่างน้อย 15 นาที', icon: '📖', defaultRewardMinutes: 20, defaultXpReward: 35, isMandatory: false, category: 'reading' },
  { id: 'q04', title: 'ช่วยงานบ้าน', description: 'กวาดบ้าน ล้างจาน หรืองานบ้านอื่น', icon: '🧹', defaultRewardMinutes: 10, defaultXpReward: 30, isMandatory: false, category: 'chores' },
  { id: 'q05', title: 'เก็บของเล่นให้เรียบร้อย', description: 'เก็บของเล่นกลับที่ทุกครั้ง', icon: '🧸', defaultRewardMinutes: 5, defaultXpReward: 15, isMandatory: false, category: 'chores' },
  { id: 'q06', title: 'รดน้ำต้นไม้', description: 'รดน้ำต้นไม้ในบ้านหรือสวน', icon: '🌱', defaultRewardMinutes: 5, defaultXpReward: 15, isMandatory: false, category: 'chores' },
  { id: 'q07', title: 'วาดรูปหรืองานศิลปะ', description: 'ทำกิจกรรมสร้างสรรค์ 15 นาที', icon: '🎨', defaultRewardMinutes: 15, defaultXpReward: 30, isMandatory: false, category: 'creativity' },
  { id: 'q11', title: 'เล่นกีฬากับเพื่อน', description: 'เล่นกีฬาหรือออกกำลังกายกับเพื่อน', icon: '⚽', defaultRewardMinutes: 20, defaultXpReward: 40, isMandatory: false, category: 'exercise' },
  { id: 'q12', title: 'ช่วยน้อง/พี่', description: 'ช่วยเหลือพี่น้องในสิ่งที่ทำได้', icon: '🤝', defaultRewardMinutes: 10, defaultXpReward: 25, isMandatory: false, category: 'social' },
  { id: 'q13', title: 'ไม่เล่นโทรศัพท์ระหว่างกินข้าว', description: 'วางโทรศัพท์ระหว่างมื้ออาหาร', icon: '📵', defaultRewardMinutes: 15, defaultXpReward: 35, isMandatory: false, category: 'social' },
  { id: 'q14', title: 'ฝึกเปียโน/ดนตรี', description: 'ฝึกซ้อมเครื่องดนตรี 15 นาที', icon: '🎹', defaultRewardMinutes: 15, defaultXpReward: 30, isMandatory: false, category: 'creativity' },
  { id: 'q15', title: 'ทบทวนบทเรียน', description: 'ทบทวนสิ่งที่เรียนมาวันนี้', icon: '✏️', defaultRewardMinutes: 20, defaultXpReward: 40, isMandatory: false, category: 'education' },
  { id: 'q16', title: 'ช่วยตั้งโต๊ะ/เก็บโต๊ะ', description: 'ช่วยเตรียมและเก็บโต๊ะอาหาร', icon: '🍴', defaultRewardMinutes: 5, defaultXpReward: 15, isMandatory: false, category: 'chores' },
  { id: 'q17', title: 'ดื่มน้ำ 6 แก้ว', description: 'ดื่มน้ำให้ครบตามที่ตั้งใจ', icon: '💧', defaultRewardMinutes: 10, defaultXpReward: 20, isMandatory: false, category: 'health' },
  { id: 'q19', title: 'เขียนไดอารี่', description: 'เขียนบันทึกสิ่งที่เกิดขึ้นวันนี้', icon: '📓', defaultRewardMinutes: 10, defaultXpReward: 25, isMandatory: false, category: 'creativity' },
  { id: 'q20', title: 'ทำสมาธิ 5 นาที', description: 'นั่งหลับตาสงบจิตใจ 5 นาที', icon: '🧘', defaultRewardMinutes: 10, defaultXpReward: 25, isMandatory: false, category: 'health' },
  { id: 'q21', title: 'ช่วยทำอาหาร', description: 'ช่วยพ่อแม่เตรียมหรือทำอาหาร', icon: '👨‍🍳', defaultRewardMinutes: 15, defaultXpReward: 35, isMandatory: false, category: 'chores' },
  { id: 'q22', title: 'อ่านนิทานก่อนนอน', description: 'อ่านหรือฟังนิทานก่อนเข้านอน', icon: '🌙', defaultRewardMinutes: 15, defaultXpReward: 30, isMandatory: false, category: 'reading' },
];

export const MANDATORY_QUEST_IDS = ['q01', 'q03', 'q08', 'q09', 'q10', 'q18'] as const;
```

### 5.2 Level Thresholds

```ts
// constants/levels.ts

export interface LevelConfig {
  level: number;
  title: string;           // Thai title
  titleEn: string;         // for reference
  xpRequired: number;      // XP สะสมตั้งแต่ต้น (cumulative)
  color: string;           // accent color สำหรับ badge
}

export const LEVEL_CONFIGS: LevelConfig[] = [
  { level: 1, title: 'นักสำรวจน้อย',   titleEn: 'Little Explorer',  xpRequired: 0,    color: '#EA6C1A' },
  { level: 2, title: 'นักผจญภัย',       titleEn: 'Adventurer',       xpRequired: 200,  color: '#3B6D11' },
  { level: 3, title: 'นักรบกล้าหาญ',   titleEn: 'Brave Warrior',    xpRequired: 500,  color: '#185FA5' },
  { level: 4, title: 'อัศวินตัวน้อย',  titleEn: 'Little Knight',    xpRequired: 900,  color: '#534AB7' },
  { level: 5, title: 'วีรบุรุษน้อย',   titleEn: 'Little Hero',      xpRequired: 1400, color: '#C4520E' },
  { level: 6, title: 'ตำนานนักสำรวจ',  titleEn: 'Legend Explorer',  xpRequired: 2000, color: '#D4A017' },
];

// เลือก level ปัจจุบัน:
// currentLevel = ค่า level สูงสุดที่ totalXp >= xpRequired
// xpThisLevel = totalXp - LEVEL_CONFIGS[currentLevel-1].xpRequired
// xpToNextLevel = LEVEL_CONFIGS[currentLevel].xpRequired - totalXp
//                 (ถ้า max level แล้ว = 0)

// Note: "Lv1→Lv2 ต้องการ 5 quests" จาก mockup
// 5 quests × 40 XP avg = 200 XP → ตรงกับ threshold ข้างบน
```

### 5.3 Cheer Presets

```ts
// constants/cheers.ts

export interface CheerPreset {
  id: string;
  emoji: string;
  text: string;      // ข้อความที่ส่งถึงลูก
}

export const CHEER_PRESETS: CheerPreset[] = [
  { id: 'c01', emoji: '⭐', text: 'เก่งมากเลย! พ่อแม่ภูมิใจในตัวลูกมากๆ' },
  { id: 'c02', emoji: '💪', text: 'สู้ต่อไปนะ! ลูกทำได้แน่นอน' },
  { id: 'c03', emoji: '🎉', text: 'เยี่ยมมาก! ทำได้ดีมากวันนี้' },
  { id: 'c04', emoji: '🏆', text: 'แชมป์เปี้ยน! ลูกเป็นนักผจญภัยตัวจริง' },
  { id: 'c05', emoji: '❤️', text: 'พ่อแม่รักลูกมากนะ ทำดีต่อไปเลย' },
  { id: 'c06', emoji: '🌟', text: 'ดาวส่องแสง! วันนี้ลูกทำได้ดีมาก' },
];
```

### 5.4 Colors เพิ่มเติม

```ts
// constants/theme.ts — เพิ่ม

export const Colors = {
  // ... เดิม ...

  // Parent theme
  parentPrimary: '#185FA5',
  parentPrimaryDark: '#0C447C',
  parentBackground: '#EEF5FC',
  parentBorder: '#B5D4F4',
  parentSurface: '#FFFFFF',

  // Kid theme
  kidPrimary: '#FF8C42',
  kidPrimaryDark: '#D85A30',
  kidBackground: '#FFF8F0',
  kidBorder: '#FFD4A8',

  // Timer (dark)
  timerBackground: '#1a1a2e',
  timerClock: '#00FF87',
  timerClockLow: '#FF4444',

  // XP/Request
  xpPurple: '#534AB7',
  xpPurpleLight: '#E8E6F8',

  // Streak
  streakGreen: '#1D9E75',
  streakGreenLight: '#D4F0E8',

  // Quest done
  questDoneBg: '#EAF3DE',
  questDoneBorder: '#97C459',

  // Approval
  approvalBg: '#FCEBEB',
  approvalBorder: '#F09595',
};
```

---

## 6. Components ใหม่ที่ต้องสร้าง

### 6.1 แก้ไข Components ที่มีอยู่

| Component | การแก้ไข |
|---|---|
| `QuestCard.tsx` | เพิ่ม timer button, XP badge, done-stripe overlay, `onTimerPress` prop |
| `RewardBadge.tsx` | แทนที่หรือ supplement ด้วย `XPBar` component |
| `PinInput.tsx` | Redesign ทั้งหมด: top area blue, keypad with sub-labels (ABC/DEF ฯลฯ), error message row |

### 6.2 Components ใหม่

#### Kid Home
```
components/kid/
├── XPBar.tsx               ← XP bar พร้อม level badge, title, progress
│   Props: { totalXp, currentLevel, currentLevelTitle, xpThisLevel, xpToNextLevel }
│
├── KidStatGrid.tsx         ← 3-col grid: earned minutes / streak / level
│   Props: { earnedMinutes, streakDays, currentLevel }
│
├── CheerBanner.tsx         ← Banner แสดง cheer message จากพ่อแม่
│   Props: { cheer: CheerMessage, onDismiss: () => void }
│   behavior: auto-dismiss หลัง 5 วินาที + กด dismiss ได้
│
└── QuestRequestButton.tsx  ← ปุ่ม "ขอภารกิจเพิ่ม" พร้อม badge count pending
    Props: { pendingCount: number, onPress: () => void }
```

#### Parent Dashboard
```
components/parent/
├── ParentStatGrid.tsx      ← 3-col grid: quests/time/sessions
│   Props: { completedCount, totalCount, earnedMinutes, sessionCount }
│
├── ApprovalCard.tsx        ← Unlock approval card (pink bg)
│   Props: { kidName, earnedMinutes, limitMinutes, onApprove, onDeny }
│
├── QuestRequestCard.tsx    ← Kid quest request approval card
│   Props: { request: QuestRequest, onApprove, onDeny }
│   shows: library item details, request time
│
└── CheerPanel.tsx          ← List of cheer buttons
    Props: { presets: CheerPreset[], onSend: (id: string) => void }
```

#### Quest Management
```
components/parent/
├── QuestLibraryList.tsx    ← Scrollable list ของ library items
│   Props: { items, selectedIds, activeIds, onToggle }
│   shows: mandatory dot, selected state, already-active state
│
└── ActiveQuestRow.tsx      ← Quest row ใน manage tab
    Props: { quest: Quest, onRemove, removable: boolean }
    shows: mandatory badge ถ้า isMandatory
```

#### Timer
```
components/
├── CircularTimer.tsx       ← Circular progress clock
│   Props: { remainingSeconds, totalSeconds, isLow: boolean }
│   uses: react-native-reanimated SVG animation
│   color: #00FF87 → #FF4444 เมื่อ < 60 วินาที
│
└── TimerControls.tsx       ← Play/Pause/Stop/Reset buttons
    Props: { isActive, isPaused, onPlay, onPause, onStop, onReset }
```

#### Shared
```
components/shared/
├── SectionHeader.tsx       ← Section title พร้อม optional badge
│   Props: { title, badgeCount?: number }
│
└── EmptyState.tsx          ← Empty state placeholder
    Props: { icon, title, subtitle }
```

---

## 7. Architecture Decisions

### 7.1 Navigation: Expo Router Groups สำหรับ Parent

**การตัดสินใจ:** ใช้ `app/parent/` directory group แทน single file `parent.tsx`

**เหตุผล:**
- Parent มี 4 sections (dashboard, manage, library, settings) ที่มีความซับซ้อนพอสมควร
- การแยกไฟล์ทำให้ code splitting ดีขึ้นและ maintainable มากขึ้น
- `parent/_layout.tsx` จัดการ PIN gate logic ได้ที่เดียว ทุก child route ได้รับ protection โดยอัตโนมัติ
- Trade-off: navigation ซับซ้อนขึ้นเล็กน้อย (ต้องใช้ nested Stack)

### 7.2 Quest Library เป็น Constant ไม่ใช่ Store

**การตัดสินใจ:** `QUEST_LIBRARY` อยู่ใน `constants/questLibrary.ts` ไม่ใช่ใน Zustand store

**เหตุผล:**
- Quest library ไม่เปลี่ยนแปลง (static data) ไม่ต้องการ persistence
- ลด bundle size ของ AsyncStorage
- Type safety ดีขึ้น (tuple type, `as const`)
- Active quests ใน store อ้างอิง `libraryId` กลับมา library constant
- Trade-off: ถ้าอนาคตต้องการ custom library ต้องย้าย logic

### 7.3 Per-Quest Timer ผ่าน Expo Router Params

**การตัดสินใจ:** Timer screen รับ `questId` ผ่าน query params (`/timer?questId=q01`)

**เหตุผล:**
- ไม่ต้องการ global "activeQuestId" ใน store ที่อาจ stale
- Deep linkable
- Store ยังเก็บ `activeQuestId` ไว้ใน `TimerSession` เพื่อ history

**Trade-off:** ถ้า navigate ออกจาก timer แล้วกลับมา ต้องมั่นใจว่า params ยังอยู่ (expo-router จัดการ stack ให้)

### 7.4 Cheer ไม่ใช้ Real-time Push

**การตัดสินใจ:** Cheer ส่งผ่าน Zustand store (local state) ไม่ใช่ push notification

**เหตุผล:**
- App scope คือ single-device (พ่อแม่กับลูกใช้เครื่องเดียวกัน หรือ app เดียวกัน)
- ไม่ต้องการ network layer
- ลด complexity อย่างมาก
- Trade-off: ถ้าอนาคตต้องการ multi-device ต้องเพิ่ม backend

### 7.5 XP และ Minutes แยกกัน

**การตัดสินใจ:** `xpReward` แยกจาก `rewardMinutes` บน Quest type

**เหตุผล:**
- XP เป็น "แต้มสะสม" ตลอดกาล (ไม่ reset รายวัน)
- Minutes เป็น "รางวัลวันนี้" (reset ทุกวัน)
- ทำให้ design level system อิสระจาก screen time policy
- Trade-off: ต้อง migrate data ของ users เดิม (defaultQuests ใน store)

### 7.6 KidProgress ใน Store แทน Computed

**การตัดสินใจ:** เก็บ `progress: KidProgress` เป็น state ใน store ไม่ใช่ compute ทุกครั้ง

**เหตุผล:**
- XP เป็น cumulative ข้ามวัน → ถ้า compute จาก `quests` เท่านั้นจะได้แค่วันนี้
- ต้องมี `totalXp` สะสมอยู่ใน persistence
- `updateProgress()` action เรียกหลัง completeQuest เพื่อ keep in sync
- Trade-off: ต้องระวัง consistency (ถ้า state corrupt ต้องมี recalculate function)

### 7.7 Timer Rename: rewards.tsx → timer.tsx

**การตัดสินใจ:** rename screen จาก `rewards` เป็ `timer`

**เหตุผล:**
- ชื่อ `rewards` สื่อสารไม่ตรงกับ purpose จริง (timer countdown)
- `timer.tsx` ตรงกับ mockup naming และ feature set ใหม่ (dark theme, per-quest)
- Deep link path `/timer` สื่อความหมายชัดเจน
- Trade-off: ต้อง update ทุกที่ที่ `router.push('/rewards')` อยู่ (2 จุดใน index.tsx)

### 7.8 Mandatory Quests Guard

**การตัดสินใจ:** Guard การลบ quest ด้วย `isMandatory` flag ทั้งใน UI (ซ่อนปุ่มลบ) และใน action (`removeQuest`)

**เหตุผล:**
- Defense in depth — UI guard อาจ bypass ได้ถ้า action ไม่ guard ด้วย
- `removeQuest` จะ early return ถ้า `quest.isMandatory === true`
- Mandatory quests ถูก inject ตอน `resetDaily` ถ้าหายไปจาก list

### 7.9 Quest Request: Max 3 Selections

**การตัดสินใจ:** enforce max 3 ที่ UI level (disable items ที่เกิน) และ validate ใน `submitQuestRequest` action

**เหตุผล:**
- UX: ลูกอายุ 6-12 ปี การ limit selection ทำให้ไม่ overwhelmed
- ป้องกัน parent ต้อง review request ยาว
- Action validate เพื่อ prevent programmatic abuse

### 7.10 Daily Reset ไม่ Reset KidProgress

**การตัดสินใจ:** `resetDaily()` reset quests/timer/sessions แต่ไม่ reset `progress.totalXp`, `progress.currentLevel`, `progress.streakDays`

**เหตุผล:**
- XP และ Level เป็น long-term motivation ไม่ควร reset รายวัน
- Streak นับวันติดต่อกัน ต้อง update (ไม่ reset) ด้วย `checkStreak()`
- `lastResetDate` ยังคงใช้เพื่อ detect วันใหม่

---

## 8. File Checklist สรุป

### ไฟล์ที่ต้องสร้างใหม่
```
app/timer.tsx
app/quest-request.tsx
app/parent/_layout.tsx
app/parent/index.tsx
app/parent/manage.tsx
app/parent/library.tsx
app/parent/settings.tsx
constants/questLibrary.ts
constants/levels.ts
constants/cheers.ts
components/kid/XPBar.tsx
components/kid/KidStatGrid.tsx
components/kid/CheerBanner.tsx
components/kid/QuestRequestButton.tsx
components/parent/ParentStatGrid.tsx
components/parent/ApprovalCard.tsx
components/parent/QuestRequestCard.tsx
components/parent/CheerPanel.tsx
components/parent/QuestLibraryList.tsx
components/parent/ActiveQuestRow.tsx
components/CircularTimer.tsx
components/TimerControls.tsx
components/shared/SectionHeader.tsx
components/shared/EmptyState.tsx
```

### ไฟล์ที่ต้องแก้ไข
```
types/index.ts           ← เพิ่ม types ทั้งหมด (Section 2)
store/questStore.ts      ← เพิ่ม actions ทั้งหมด (Section 3), migrate defaultQuests
constants/theme.ts       ← เพิ่ม colors (Section 5.4)
app/_layout.tsx          ← เพิ่ม parent/ Stack.Screen group
app/index.tsx            ← เพิ่ม XPBar, KidStatGrid, CheerBanner, QuestRequestButton;
                           ปรับ QuestCard onTimerPress; router.push('/rewards') → '/timer'
components/QuestCard.tsx ← เพิ่ม timer button, XP badge, done-stripe
components/PinInput.tsx  ← redesign UI (blue top + white keypad)
hooks/useDailyReset.ts   ← เรียก checkStreak() เพิ่ม
```

### ไฟล์ที่ต้องลบ/rename
```
app/rewards.tsx          ← rename เป็น app/timer.tsx
app/parent.tsx           ← ลบ (แทนที่ด้วย app/parent/ directory)
```
