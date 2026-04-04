# Backlog — little-heroes

## ต้องทำ

- [ ] Fix: เพิ่ม SafeAreaProvider ใน `_layout.tsx`
- [ ] Fix: เพิ่ม Reanimated plugin ใน `babel.config.js`
- [ ] Feature: First-run onboarding (ตั้งชื่อเด็ก + PIN)
- [ ] Feature: Background timer handling (pause เมื่อ app ออก)
- [ ] Feature: Haptic feedback (expo-haptics)
- [ ] Feature: Thai font (Kanit via expo-font)
- [ ] Improvement: Force PIN change จาก default 1234

## เสร็จแล้ว

- [x] Init project (Expo SDK 54, expo-router, Zustand)
- [x] Theme system (colors, spacing, typography)
- [x] TypeScript types (Quest, KidProfile, AppState)
- [x] Zustand store + AsyncStorage persistence
- [x] useDailyReset hook
- [x] useTimer hook
- [x] QuestCard component (animated)
- [x] TimerBar component
- [x] PinInput component (shake + lockout)
- [x] RewardBadge component
- [x] QuestForm component
- [x] Kid home screen
- [x] Timer/rewards screen
- [x] Parent dashboard (PIN gate + 3 tabs)
