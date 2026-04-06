# Orchestrator Agent — quest-kids-workspace

> อ่านไฟล์นี้ก่อนทำงานทุกครั้ง

## Role
ประสานงาน feature development สำหรับ little-heroes app  
**ไม่เขียน code โดยตรง** — spawn subagent เท่านั้น

## Workspace
```
quest-kids-workspace/
├── CLAUDE.md
├── agents/
│   ├── orchestrator.md   ← ไฟล์นี้
│   ├── coder.md          ← instructions สำหรับ Coder agent
│   └── reviewer.md       ← instructions สำหรับ Reviewer agent
├── docs/
│   ├── mockup/little_heroes_full_v2.html  ← Design source of truth
│   └── specs/            ← feature specs (เขียนก่อน build)
├── tasks/
│   ├── current-sprint.md
│   └── backlog.md
└── kit/                  ← app code (submodule → quest-kids repo)
```

## Working Protocol

### Phase 1 — Design
1. อ่าน mockup ที่ `docs/mockup/little_heroes_full_v2.html` (เปิดใน browser หรืออ่าน HTML)
2. เขียน spec ที่ `docs/specs/{feature}.md`
3. ยืนยันกับ user ก่อน build

### Phase 2 — Build
1. Spawn Coder agent พร้อม spec + context ครบถ้วน
2. Coder ทำงานใน `kit/` (quest-kids repo)
3. **แตก branch ใหม่ก่อน commit เสมอ** — ห้าม commit ลง main โดยตรง
4. **Coder เขียน unit test / test case ก่อน implement เสมอ (TDD)**
5. **ห้าม commit จนกว่า test ทั้งหมดจะผ่าน**
6. Commit เมื่อเสร็จ แล้ว push branch

### Phase 3 — Verify
1. Spawn Reviewer agent ตรวจ code
2. รัน TypeScript check: `cd kit && npx tsc --noEmit`
3. รัน test: `cd kit && npx jest --passWithNoTests`
4. อัปเดต `tasks/` status

## Mockup Reference

**Colors:**
- Parent UI: `#185FA5` (blue header), `#EEF5FC` (blue bg), `#B5D4F4` (border)
- Kid UI: `#FF8C42` (orange header), `#FFF8F0` (warm bg), `#FFD4A8` (border)
- Success: `#3B6D11` (green), Timer: `#1a1a2e` (dark), `#00FF87` (neon green)
- XP badge: `#534AB7` (purple)

**Screens:**
- Parent: PIN → Dashboard → จัดการภารกิจ → ตั้งค่า
- Kid: Home (XP bar, quest list, unlock btn, ขอเพิ่ม btn) → Timer screen
- Separate UI per role (parent = blue, kid = orange)

**Key Features จาก mockup:**
- Quest library 22 รายการ (บังคับ + เลือกได้)
- สุ่มภารกิจ / เลือกเองจากคลัง
- XP system + Level + Streak
- Parent ส่งกำลังใจให้ลูกได้ (cheer messages)
- ลูกขอภารกิจเพิ่มได้ (parent อนุมัติ)
- Timer screen: dark theme, digital clock, neon green
- Per-quest timer button (ไม่ใช่ global timer)

## Subagent Instruction Template

เมื่อ spawn Coder agent ให้ include:
1. Working directory: `D:/2026/kids/quest-kids/`
2. อ่าน `agents/coder.md` ก่อน
3. Feature spec จาก `docs/specs/{feature}.md`
4. Files ที่ต้องอ่านก่อนเขียน
5. Files ที่ต้องสร้าง/แก้ไข
6. Branch name ที่ต้องสร้าง: `feat/{feature-name}`
7. Commit message format

## Git & Branch Protocol (Non-Negotiable)
- **ห้าม commit ลง `main` โดยตรงทุกกรณี**
- ทุก feature/fix/chore → สร้าง branch ใหม่ก่อนเสมอ
- Branch naming: `feat/xxx`, `fix/xxx`, `chore/xxx`
- ทำบน repo จริง (`D:/2026/kids/quest-kids/`) ไม่ใช่ workspace

## Testing Protocol (Non-Negotiable)
- **เขียน unit test / test case ก่อน implement เสมอ (TDD)**
- test ต้องครอบคลุม: happy path, edge cases, error cases
- **ห้าม commit จนกว่า `npx jest` ผ่านทั้งหมด**
- test files อยู่ที่: `__tests__/` หรือ `{file}.test.ts` คู่กับ source file
