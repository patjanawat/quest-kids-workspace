# CLAUDE.md — quest-kids-workspace

## Repos

| ชื่อ | Path |
|---|---|
| App (Expo React Native) | `./kit/` (submodule → quest-kids repo) |

> ⚠️ ห้ามแก้ code ใน workspace — แก้ได้เฉพาะใน `kit/` (quest-kids repo) เท่านั้น

## App Structure

```
kit/
├── app/              ← Expo Router screens
│   ├── _layout.tsx
│   ├── index.tsx     ← Kid home
│   ├── rewards.tsx   ← Timer screen
│   └── parent.tsx    ← Parent dashboard (PIN protected)
├── components/       ← Reusable UI components
├── constants/        ← Theme (colors, spacing, typography)
├── hooks/            ← useDailyReset, useTimer
├── store/            ← Zustand store + AsyncStorage
├── types/            ← TypeScript interfaces
└── assets/
```

## Tech Stack

- React Native + Expo SDK 54
- expo-router (file-based routing)
- Zustand 5 + AsyncStorage (state + persistence)
- react-native-paper (UI)
- react-native-reanimated (animations)
- TypeScript strict mode

## Design

- Primary: `#EA6C1A` (orange)
- Success: `#3B6D11` (green)
- Target: kids age 6–12, large touch targets, Thai language UI

## Working Protocol

1. Read spec in `docs/specs/` before building
2. Write code only in `kit/` (quest-kids repo)
3. Commit from `kit/` directory
4. Update `tasks/` after completing features

## Run App

```bash
cd kit
npx expo start
```
