# Current Sprint — Fixes & Polish

## Goal
แก้ปัญหาพื้นฐานก่อน run app ได้จริง

## Tasks

- [x] **FIX-01** เพิ่ม SafeAreaProvider ใน `app/_layout.tsx`
- [x] **FIX-02** เพิ่ม `react-native-reanimated/plugin` ใน `babel.config.js`
- [x] **FIX-03** แก้ `index.ts` ให้ใช้ expo-router entry point
- [x] **FIX-04** สร้าง `metro.config.js` แก้ zustand ESM/import.meta error บน web
- [x] **FIX-05** แก้ react-dom version mismatch ด้วย npm overrides

✅ App รันบน web ได้แล้ว (branch: `fix/infrastructure-setup`)

## Next Sprint
- Module 1 — Auth (PIN): Phase 1.5 → เขียน test cases
- Onboarding flow
- Background timer
- Thai font
