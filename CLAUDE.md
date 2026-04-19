# CLAUDE.md — quest-kids-workspace

## Repos

| ชื่อ | Path |
|---|---|
| App (Expo React Native) | `./kit/little-heroes/` (submodule → quest-kids repo) |

> ⚠️ ห้ามแก้ code ใน workspace root — แก้ได้เฉพาะใน `kit/little-heroes/` เท่านั้น
> ⚠️ `kit/` เป็น submodule — commit ต้องรันจาก `kit/little-heroes/` ไม่ใช่ workspace root

## App Structure

```
kit/little-heroes/
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
2. Edit files in `kit/little-heroes/` (= `D:\2026\kids\quest-kids\`)
3. Create branch first: `git checkout -b <branch>` from `kit/little-heroes/`
4. Commit from `kit/little-heroes/` directory
5. Update `tasks/` after completing features

## Run App

```bash
cd kit/little-heroes
npx expo start --clear
```
