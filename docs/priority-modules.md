# Priority Modules — little-heroes

> อัปเดต: 2026-04-06

## สถานะปัจจุบัน

App มี 4 ไฟล์เก่า (`index.tsx`, `parent.tsx`, `rewards.tsx`, `_layout.tsx`) + 2 hooks + 1 store  
**ยังไม่ตรงกับ mockup** — ต้อง rebuild ทุก module ตาม spec

---

## 6 Modules — Priority Order

| # | Module | Spec | Priority | เหตุผล |
|---|---|---|---|---|
| 1 | **Auth (PIN)** | `docs/specs/01-auth.md` | 🔴 Critical | Gate ของ Parent ทุก screen — ถ้าไม่มี parent เข้าไม่ได้เลย |
| 2 | **Kid Home** | `docs/specs/04-kid-home.md` | 🔴 Critical | หน้าหลักที่ลูกเห็นทุกวัน — quest list, XP, unlock button |
| 3 | **Quest Management** | `docs/specs/03-quest-management.md` | 🟠 High | Parent ต้องตั้งภารกิจก่อนลูกจะทำอะไรได้ |
| 4 | **Timer** | `docs/specs/06-timer.md` | 🟠 High | Core feature — ลูกกดทำภารกิจ → timer วิ่ง → ได้นาที |
| 5 | **Parent Dashboard** | `docs/specs/02-parent-dashboard.md` | 🟡 Medium | Approval + stats — ต้องมี quest + timer ก่อน |
| 6 | **Quest Request** | `docs/specs/05-quest-request.md` | 🟡 Medium | Nice-to-have — ลูกขอภารกิจเพิ่มจาก parent |

---

## Dependency Chain

```
Auth (PIN)
    ↓
Quest Management  ←→  Kid Home
                           ↓
                        Timer
                           ↓
                    Parent Dashboard
                           ↓
                     Quest Request
```

**Build order**: Auth → Kid Home + Quest Management (parallel) → Timer → Parent Dashboard → Quest Request

---

## Build Status

> รายละเอียด tasks แต่ละ module → `tasks/backlog.md`

| # | Module | Breakdown | Phase 1.5 | Phase 2 | Phase 3a | Phase 3b | Done |
|---|---|---|---|---|---|---|---|
| 1 | Auth (PIN) | 2026-04-06 | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 2 | Kid Home | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 3 | Quest Management | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 4 | Timer | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 5 | Parent Dashboard | — | ⬜ | ⬜ | ⬜ | ⬜ | — |
| 6 | Quest Request | — | ⬜ | ⬜ | ⬜ | ⬜ | — |

> Phase 1.5 = UI Tester เขียน test cases checklist  
> Phase 2 = Coder implement (TDD)  
> Phase 3a = Reviewer  
> Phase 3b = UI Tester verify  

---

## สิ่งที่พร้อมแล้ว

- ✅ Specs ครบทั้ง 6 modules (`docs/specs/`)
- ✅ Architecture design (`docs/specs/architecture.md`)
- ✅ Agents พร้อมทุกตัว (orchestrator, architect, spec-writer, coder, reviewer, ui-tester)
- ✅ Mockup เป็น source of truth (`docs/mockup/little_heroes_full_v2.html`)
