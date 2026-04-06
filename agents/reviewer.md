# Reviewer Agent — little-heroes

## Working Directory
`D:/2026/kids/quest-kids/`

## Checklist

### TypeScript
- [ ] รัน `npx tsc --noEmit` — ต้องไม่มี error
- [ ] ไม่มี `any` type
- [ ] imports ถูก path

### UI / Mockup Alignment
- [ ] สีตรงกับ mockup (parent = blue `#185FA5`, kid = orange `#FF8C42`)
- [ ] Text ภาษาไทยทั้งหมด
- [ ] Touch targets ≥ 48x48
- [ ] `SafeAreaView` จาก `react-native-safe-area-context`

### State
- [ ] ใช้ actions จาก store (ห้าม setState โดยตรง)
- [ ] ไม่มี hardcode data ใน component (ดึงจาก store)

### Edge Cases
- [ ] Empty state (ไม่มี quest)
- [ ] Timer = 0
- [ ] PIN ผิด 3 ครั้ง → lockout
- [ ] ไม่มี quest ที่ทำ → unlock button disabled

### Git
- [ ] ไม่มี commit ลง `main` โดยตรง — ต้องอยู่บน branch `feat/`, `fix/`, หรือ `chore/`
- [ ] Branch name ตรงกับ feature ที่ทำ

### Tests
- [ ] มี test file สำหรับทุก store action ที่เพิ่ม/แก้ไข
- [ ] มี test file สำหรับทุก hook ที่เพิ่ม/แก้ไข
- [ ] มี test file สำหรับทุก utility function
- [ ] รัน `npx jest` แล้วผ่านทั้งหมด — **ห้าม PASS ถ้า test fail แม้แต่ 1 case**
- [ ] test ครอบคลุม happy path + edge cases + error cases

## Output
รายงาน:
1. Issues found (ถ้ามี)
2. Fixes applied
3. TypeScript result
4. Test result: `X passed, Y failed` (รัน `npx jest` จริง)
5. Branch ที่ใช้
6. Status: PASS / FAIL
