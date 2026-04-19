# Backlog — little-heroes

> อัปเดต: 2026-04-19  
> Priority reference: `docs/priority-modules.md`

---

## Module Build Status

| # | Module | Breakdown | Phase 1.5 | Phase 2 | Phase 3a | Phase 3b | Done |
|---|---|---|---|---|---|---|---|
| 1 | Auth (PIN) | ✅ | ✅ | ✅ | — | — | ✅ 2026-04-19 |
| 2 | Kid Home | ✅ | — | ✅ | — | — | ✅ 2026-04-19 |
| 3 | Quest Management | ✅ | — | ✅ | — | — | ✅ 2026-04-19 |
| 4 | Timer | ✅ | — | ✅ | — | — | ✅ 2026-04-19 |
| 5 | Parent Dashboard | ✅ | — | ✅ | — | — | ✅ 2026-04-19 |
| 6 | Quest Request | ✅ | — | ✅ | — | — | ✅ 2026-04-19 |

> Phase 1.5 = UI Tester เขียน test cases | Phase 2 = Coder (TDD) | Phase 3a = Reviewer | Phase 3b = UI Tester verify

---

## Module 4–6 — Pending Breakdown

- **Timer** — spec พร้อม: `docs/specs/06-timer.md`
- **Parent Dashboard** — spec พร้อม: `docs/specs/02-parent-dashboard.md`
- **Quest Request** — spec พร้อม: `docs/specs/05-quest-request.md`

---

## Infrastructure Fixes (ทำก่อน build เริ่ม)

- [x] เพิ่ม SafeAreaProvider ใน `_layout.tsx`
- [x] เพิ่ม Reanimated plugin ใน `babel.config.js`

---

## Future / Icebox

- [x] First-run onboarding — 3-step wizard, kidName + avatar + PIN (commit b3b2c1e)
- [x] Background timer handling — useBackgroundTimer hook, AppState diff (commit 35c708e)
- [x] Haptic feedback — PinInput, TimerControls, UnlockButton (commit 0bbefe8)
- [x] Thai font — Kanit 400/600/700 via @expo-google-fonts/kanit (commit 0bbefe8)
- [x] Force PIN change จาก default 1234 — onboarding บังคับตั้ง PIN ใหม่แล้ว (ห้าม 1234)
- [x] Session timeout — auto-lock parent screen on foreground (commit 0bbefe8)
- [x] ลบ PIN hint "รหัสเริ่มต้น: 1234" ออกจาก parent screen (commit 0bbefe8)

---

## Done

- [x] Init project (Expo SDK 54, expo-router, Zustand)
- [x] Theme system (colors, spacing, typography)
- [x] TypeScript types (Quest, KidProfile, AppState)
- [x] Zustand store + AsyncStorage persistence
- [x] useDailyReset hook
- [x] useTimer hook
- [x] Workspace agent system (orchestrator, architect, spec-writer, coder, reviewer, ui-tester)
- [x] Specs ครบ 6 modules (`docs/specs/`)
- [x] Priority & build order doc (`docs/priority-modules.md`)
- [x] Infrastructure fixes (SafeAreaProvider, Reanimated, metro config, react-dom)
- [x] **Module 1 — Auth (PIN)** — PinInput, lockout 3 ครั้ง/5 นาที, recordPinFail, resetPinFails (commit 5a4ccd3)
- [x] **Module 2 — Kid Home** — KidHeader + XP/level, KidStatGrid, CheerBanner, QuestCard 4 states, UnlockButton, KidTabBar (commit 9dd0038)
- [x] **Module 3 — Quest Management** — QUEST_LIBRARY 22 items, QuestManageCard, ParentTabBar, manage screen, library picker (commit f8157af)
- [x] **Module 4 — Timer** — dark theme stopwatch, DigitalClock neon glow, QuestContext, TimerProgressBar, TimerControls, pause/resume/reset (commit 26d1d38)
- [x] **Module 5 — Parent Dashboard** — StatGrid, ProgressSummaryCard, ApprovalCard, QuestRequestCard, CheerSection, Settings screen (commit 434cd47)
- [x] **Module 6 — Quest Request** — QuestRequestItem (3 states), quest filter, max 3 select, submit to parent (commit 37bbde7)
