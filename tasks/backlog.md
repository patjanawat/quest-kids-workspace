# Backlog — little-heroes

> อัปเดต: 2026-04-06  
> Priority reference: `docs/priority-modules.md`

---

## Module Build Status

| # | Module | Breakdown | Phase 1.5 | Phase 2 | Phase 3a | Phase 3b | Done |
|---|---|---|---|---|---|---|---|
| 1 | Auth (PIN) | 2026-04-06 | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 2 | Kid Home | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 3 | Quest Management | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 4 | Timer | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 5 | Parent Dashboard | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 6 | Quest Request | — | ⬜ | ⬜ | ⬜ | ⬜ | — |

> Phase 1.5 = UI Tester เขียน test cases | Phase 2 = Coder (TDD) | Phase 3a = Reviewer | Phase 3b = UI Tester verify

---

## Module 1 — Auth (PIN)

> Breakdown: 2026-04-06 | Spec: `docs/specs/01-auth.md`  
> Priority: 🔴 Critical — gate ของ parent area ทุก screen

### Tasks

#### Phase 1.5 — UI Tester: Test Cases
- [x] เขียน test cases checklist → `docs/specs/01-auth-test-cases.md`
  - Happy path: กรอก PIN ถูก 4 หลัก → navigate เข้า parent
  - Error: กรอก PIN ผิด → shake + error message
  - Lockout: ผิด 3 ครั้ง → overlay + countdown
  - Edge: lockout ค้างจาก session ก่อน → reset อัตโนมัติเมื่อเวลาผ่าน
  - Edge: กลับมา parent area → ต้องกรอก PIN ใหม่ (local state ไม่ persist)

#### Phase 2 — Coder: Implement (TDD)
- [ ] สร้าง branch `feat/auth-pin`
- [ ] เขียน unit test ก่อน (fail) — `hooks/usePinAuth.test.ts`
  - [ ] test: กรอก digit ทีละตัว → entered สะสมถูกต้อง
  - [ ] test: กรอกครบ 4 หลักถูก → unlocked = true, resetPinFails() ถูกเรียก
  - [ ] test: กรอกผิด → recordPinFail() ถูกเรียก, entered ล้างหลัง 600ms
  - [ ] test: pinFailCount >= 3 → isLockedOut = true
  - [ ] test: lockout หมดเวลา → isLockedOut = false
  - [ ] test: กด delete → ลบตัวสุดท้าย, disabled เมื่อว่าง
- [ ] implement `store/questStore.ts` — เพิ่ม `recordPinFail()`, `resetPinFails()`
- [ ] implement `hooks/usePinAuth.ts`
  - input: ไม่มี (อ่าน store โดยตรง)
  - output: `entered`, `error`, `unlocked`, `isLockedOut`, `lockoutMinutesLeft`, `handleDigit()`, `handleDelete()`
- [ ] implement `components/PinDot.tsx` — dot แสดงสถานะ filled/empty
- [ ] implement `components/PinKeypad.tsx` — numpad 0-9, delete, disabled state
- [ ] implement `app/parent/_layout.tsx` — PIN gate: render PinScreen หรือ `<Slot />`
- [ ] unit test ผ่านทั้งหมด (`npx jest`) ก่อน commit
- [ ] commit: `feat: implement PIN gate with lockout`

#### Phase 3a — Reviewer
- [ ] รัน `npx tsc --noEmit` + `npx jest`
- [ ] ตรวจ architecture (screen ไม่มี logic, hook มี logic ครบ)
- [ ] ตรวจ code quality (ไม่มี any, ไม่ hardcode สี, magic number)
- [ ] ตรวจ spec alignment (acceptance criteria ครบ)
- [ ] ออก Reviewer Report → PASS / FAIL

#### Phase 3b — UI Tester: Verify
- [ ] รัน test cases checklist จาก Phase 1.5
- [ ] ตรวจ mockup alignment (blue header `#185FA5`, dots, keypad layout)
- [ ] ตรวจ regression (store persistence, daily reset ยังทำงาน)
- [ ] ออก UI Tester Report → PASS / FAIL

### Acceptance Criteria (จาก spec)
- [ ] กรอก PIN 4 หลักถูก → เข้า parent dashboard ทันที (ไม่ต้องกด OK)
- [ ] PIN ผิด → shake animation + `"PIN ไม่ถูกต้อง ลองอีกครั้ง"`
- [ ] ผิด 3 ครั้ง → lockout overlay + countdown นาที
- [ ] countdown อัปเดตทุกวินาที
- [ ] หมด lockout → overlay หาย + กรอกได้ใหม่
- [ ] Delete ลบทีละหลัก, disabled เมื่อว่าง
- [ ] Hint `"รหัสทดสอบ: 1234"` แสดงเมื่อไม่มี error

### Files ที่ต้องสร้าง/แก้ไข
| Action | File |
|---|---|
| แก้ไข | `store/questStore.ts` — เพิ่ม `recordPinFail`, `resetPinFails` |
| สร้าง | `hooks/usePinAuth.ts` |
| สร้าง | `hooks/usePinAuth.test.ts` |
| สร้าง | `components/PinDot.tsx` |
| สร้าง | `components/PinKeypad.tsx` |
| แก้ไข | `app/parent/_layout.tsx` — PIN gate logic |
| สร้าง | `docs/specs/01-auth-test-cases.md` |

---

## Module 2–6 — Pending Breakdown

> จะ breakdown เมื่อ Module 1 เสร็จ (Phase 3b PASS)

- **Kid Home** — spec พร้อม: `docs/specs/04-kid-home.md`
- **Quest Management** — spec พร้อม: `docs/specs/03-quest-management.md`
- **Timer** — spec พร้อม: `docs/specs/06-timer.md`
- **Parent Dashboard** — spec พร้อม: `docs/specs/02-parent-dashboard.md`
- **Quest Request** — spec พร้อม: `docs/specs/05-quest-request.md`

---

## Infrastructure Fixes (ทำก่อน build เริ่ม)

- [ ] เพิ่ม SafeAreaProvider ใน `_layout.tsx`
- [ ] เพิ่ม Reanimated plugin ใน `babel.config.js`

---

## Future / Icebox

- [ ] First-run onboarding (ตั้งชื่อเด็ก + PIN)
- [ ] Background timer handling (pause เมื่อ app ออก)
- [ ] Haptic feedback (expo-haptics)
- [ ] Thai font — Kanit via expo-font
- [ ] Force PIN change จาก default 1234

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
- [x] Module 1 Auth — task breakdown (2026-04-06)
