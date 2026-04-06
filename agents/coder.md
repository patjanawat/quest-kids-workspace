# Coder Agent — little-heroes

## Working Directory
`D:/2026/kids/quest-kids/`

## ก่อนเขียน code ทุกครั้ง
1. อ่าน `types/index.ts`
2. อ่าน `store/questStore.ts`
3. อ่าน `constants/theme.ts`
4. อ่านไฟล์ที่จะแก้ไข

## Rules
- TypeScript strict — ห้ามใช้ `any`
- UI text ภาษาไทยทั้งหมด
- ใช้ค่าจาก `constants/theme.ts` เสมอ (ห้าม hardcode สี/spacing)
- Touch target ขั้นต่ำ 48x48
- `SafeAreaView` จาก `react-native-safe-area-context` ไม่ใช่ `react-native`
- อ่านไฟล์ก่อนเขียนทุกครั้ง
- **แตก branch ใหม่ก่อน commit เสมอ — ห้าม commit ลง `main` โดยตรง**

## Git Protocol (Non-Negotiable)
```bash
# ก่อนเริ่มทำงานทุกครั้ง
git checkout main
git pull
git checkout -b feat/{feature-name}   # หรือ fix/ chore/

# หลังเสร็จ
git add {files}
git commit -m "feat: {description}"
git push -u origin feat/{feature-name}
```

## Stack
- `expo-router` — navigation (ห้ามใช้ React Navigation โดยตรง)
- `store/questStore.ts` — Zustand store (ใช้ actions ที่มี ห้าม setState โดยตรง)
- `react-native-reanimated` — animations
- `react-native-paper` — UI components

## Mockup Colors (อ้างอิงจาก mockup)

### Parent UI (Blue)
- Header: `#185FA5`, Dark header: `#0C447C`
- Background: `#EEF5FC`
- Border: `#B5D4F4`, Light bg: `#E6F1FB`

### Kid UI (Orange)
- Header: `#FF8C42`, Dark: `#D85A30`
- Background: `#FFF8F0`
- Border: `#FFD4A8`
- XP purple: `#534AB7`

### Shared
- Success green: `#3B6D11`, Light: `#97C459`, BG: `#EAF3DE`
- Timer dark: `#1a1a2e`, Neon: `#00FF87`
- Error: `#E24B4A`
- Text: `#2C2C2A`, Muted: `#888780`, Light: `#B4B2A9`

## Quest Library (22 quests จาก mockup)
```ts
// mandatory = true หมายถึงบังคับ ลบไม่ได้
{ id:'q01', title:'ลุกจากเตียงทันทีที่ตื่น', sub:'ฝึกวินัยและ routine', icon:'☀️', bg:'#FAEEDA', xp:5, rewardMinutes:5, mandatory:true }
{ id:'q02', title:'แปรงฟัน ล้างหน้าเอง', sub:'สุขอนามัยและ self-care', icon:'🪥', bg:'#E1F5EE', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q03', title:'เช็คอินอารมณ์ตอนเช้า', sub:'รู้จักความรู้สึกตัวเอง', icon:'🌟', bg:'#EEEDFE', xp:5, rewardMinutes:5, mandatory:true }
{ id:'q04', title:'กินข้าวเช้าครบ 5 หมู่', sub:'โภชนาการที่ดีสำหรับสมอง', icon:'🍳', bg:'#EAF3DE', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q05', title:'ยืดเส้น 5 นาที', sub:'ปลุกร่างกายตอนเช้า', icon:'🧘', bg:'#FFF0E6', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q06', title:'เตรียมกระเป๋าเรียนเอง', sub:'วางแผนและความรับผิดชอบ', icon:'📚', bg:'#E6F1FB', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q07', title:'ทบทวนบทเรียน 10 นาที', sub:'อ่านสรุปก่อนเรียนจริง', icon:'📖', bg:'#EEEDFE', xp:15, rewardMinutes:10, mandatory:false }
{ id:'q08', title:'ทำการบ้านก่อนเล่น', sub:'จัดลำดับความสำคัญ', icon:'📄', bg:'#EAF3DE', xp:20, rewardMinutes:30, mandatory:true }
{ id:'q09', title:'ออกกำลังกาย 20 นาที', sub:'วิ่ง ขี่จักรยาน เล่นกีฬา', icon:'🏃', bg:'#FFF0E6', xp:15, rewardMinutes:15, mandatory:true }
{ id:'q10', title:'ช่วยงานบ้าน 1 อย่าง', sub:'กวาด ล้างจาน พับผ้า', icon:'🧹', bg:'#FBEAF0', xp:10, rewardMinutes:10, mandatory:true }
{ id:'q11', title:'งานสร้างสรรค์ 20 นาที', sub:'วาดรูป ปั้น ประดิษฐ์', icon:'🎨', bg:'#FBEAF0', xp:10, rewardMinutes:15, mandatory:false }
{ id:'q12', title:'อ่านหนังสือ 15 นาที', sub:'หนังสือที่ชอบ', icon:'📗', bg:'#E6F1FB', xp:15, rewardMinutes:20, mandatory:false }
{ id:'q13', title:'เล่าให้พ่อแม่ฟัง 1 เรื่อง', sub:'ฝึกการสื่อสาร', icon:'🤔', bg:'#EEEDFE', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q14', title:'กินข้าวเย็นพร้อมครอบครัว', sub:'ห้ามเล่นมือถือ', icon:'🍽️', bg:'#EAF3DE', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q15', title:'Mini Quiz 5 ข้อ', sub:'คณิต ภาษา วิทยาศาสตร์', icon:'🧠', bg:'#E6F1FB', xp:15, rewardMinutes:15, mandatory:false }
{ id:'q16', title:'ท้าทาย skill ใหม่', sub:'โค้ด ทำอาหาร ดนตรี', icon:'💪', bg:'#FFF0E6', xp:20, rewardMinutes:20, mandatory:false }
{ id:'q17', title:'บอกรักหรือขอบคุณใครสักคน', sub:'พ่อ แม่ พี่ น้อง เพื่อน', icon:'💝', bg:'#FBEAF0', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q18', title:'จดบันทึก 3 สิ่งดีวันนี้', sub:'Gratitude + Growth Mindset', icon:'📋', bg:'#E1F5EE', xp:10, rewardMinutes:10, mandatory:true }
{ id:'q19', title:'อ่านนิทานก่อนนอน', sub:'กระตุ้นภาษาและจินตนาการ', icon:'📚', bg:'#EEEDFE', xp:10, rewardMinutes:15, mandatory:false }
{ id:'q20', title:'ตั้งเป้าหมายพรุ่งนี้', sub:'วางแผนและ Growth Mindset', icon:'🌟', bg:'#FAEEDA', xp:5, rewardMinutes:5, mandatory:false }
{ id:'q21', title:'นอนหลับ 9-10 ชั่วโมง', sub:'ปิดมือถือก่อนนอน 30 นาที', icon:'😴', bg:'#E6F1FB', xp:10, rewardMinutes:10, mandatory:false }
{ id:'q22', title:'พูดสวัสดีก่อนออกบ้าน', sub:'ฝึกมารยาทและความกตัญญู', icon:'💬', bg:'#FBEAF0', xp:5, rewardMinutes:5, mandatory:false }
```

## Commit Format
```
feat: {feature name} — {short description}
fix: {what was fixed}
chore: {tooling/config change}
docs: {documentation change}
```
