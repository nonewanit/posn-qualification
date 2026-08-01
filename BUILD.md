# Build Guide — สำหรับผู้สอนและผู้พัฒนา

## Requirements

- **XeLaTeX** (TeX Live 2025+) — จำเป็นสำหรับ Thai font ผ่าน `fontspec`
- **latexmk** — build automation (มาพร้อม TeX Live)
- **TH Sarabun New** — ฟอนต์สำหรับแบบฝึกหัด (ติดตั้งใน `fonts/THSarabunNew/` แล้ว)
- **Noto Sans Thai** — ฟอนต์หลักของสไลด์ (ติดตั้งใน `fonts/Noto_Sans_Thai/`)
- **Courier New** — ฟอนต์โค้ดสำหรับหัวข้อ Programming (ติดตั้งใน `fonts/CourierNew/`)

ตรวจสอบว่าติดตั้ง XeLaTeX แล้ว:

```bash
xelatex --version
```

## การคอมไพล์สไลด์

```bash
# เข้าไปในโฟลเดอร์ slides ของหัวข้อที่ต้องการ
cd geometry/triangles/slides       # หัวข้อย่อย (ลึก 3 ระดับ)
cd counting/slides                 # หัวข้อเดี่ยว (ลึก 2 ระดับ)

# คอมไพล์อัตโนมัติ (watch mode — rebuild ทุกครั้งที่เซฟ)
latexmk -pvc main.tex

# หรือคอมไพล์ครั้งเดียว
latexmk main.tex
```

**หมายเหตุ**: `latexmk` จะตรวจจับ magic comment `% !TEX program = xelatex` ใน `main.tex` และใช้ XeLaTeX โดยอัตโนมัติ

## การคอมไพล์แบบฝึกหัด

```bash
cd geometry/triangles/exercises

# แบบฝึกหัด
xelatex -no-pdf -interaction=nonstopmode exercise.tex
xdvipdfmx exercise.xdv

# เฉลย 
xelatex -no-pdf -interaction=nonstopmode answer-key.tex
xdvipdfmx answer-key.xdv
```

## โครงสร้างไฟล์สำคัญ

| ไฟล์ | ที่อยู่ | คำอธิบาย |
|---|---|---|
| `preamble.tex` | root | Shared preamble สำหรับสไลด์ทั้งหมด — Beamer theme, fonts, colored boxes, custom commands |
| `exercise-preamble.tex` | root | Shared preamble สำหรับแบบฝึกหัด — A4, 16pt, headers, `\problem` command, `pseudocode` environment |
| `.latexmkrc` | root | Config ให้ latexmk ใช้ `$pdf_mode = 5` (XeLaTeX → xdvipdfmx) |
| `fonts/` | root | ฟอนต์ Noto Sans Thai, TH Sarabun New (4 `.ttf`) และ Courier New (`cour.ttf`) |

### ไฟล์ `.latexmkrc` symlink

ทุก `slides/` ต้องมี symlink `.latexmkrc` ไปยัง root `.latexmkrc`:

- หัวข้อเดี่ยว (เช่น `counting/slides/`): `ln -sf ../../.latexmkrc .latexmkrc`
- หัวข้อย่อย (เช่น `geometry/triangles/slides/`): `ln -sf ../../../.latexmkrc .latexmkrc`

## Shared Preamble

### `preamble.tex` (สำหรับสไลด์)

สไลด์ทุกหัวข้อใช้ preamble ร่วมกันที่ root:

```latex
\input{../../preamble.tex}       # หัวข้อเดี่ยว (counting, sets, ...)
\input{../../../preamble.tex}    # หัวข้อย่อย (geometry/angles/, functions/relations/, ...)
```

preamble นี้ setup:
- **Beamer**: Madrid theme, 16:9, 11pt, สีฟ้า accent, custom title page (dark background)
- **Fonts**: Noto Sans Thai (main — Light/Medium/Bold), Latin Modern Mono (code)
- **Box environments**: `explanation`, `exambox`, `solbox`, `intuition` (ดูรายละเอียดด้านล่าง)
- **Commands**: `\highlight{text}`, `\R`, `\N`, `\Z`, `\Q`
- **Section slides**: สร้างอัตโนมัติจาก `\section{}` — ถ้ามี `(English)` ในชื่อ section จะแสดงเป็น 2 บรรทัด (ไทยสีขาว + อังกฤษสีฟ้าอ่อน) โดยใช้ `xstring` แยกข้อความอัตโนมัติ

### `exercise-preamble.tex` (สำหรับแบบฝึกหัด)

```latex
\input{../../exercise-preamble.tex}        # หัวข้อเดี่ยว
\input{../../../exercise-preamble.tex}     # หัวข้อย่อย
```

preamble นี้ setup:
- **Page**: A4, 16pt, margins 2.2--2.5 ซม.
- **Font**: TH Sarabun New, Scale 1.5
- **Headers/footers**: เลขหน้า + footer text
- **Commands**: `\problem{question}{answer}`, `\exerciseheader{title}`, `\examsection{title}`
- **Pseudocode**: `\begin{pseudocode}...\end{pseudocode}` (framed box)

## รูปแบบ Section Header

`preamble.tex` ใช้ `xstring` เพื่อแยกชื่อ section เป็น Thai และ English โดยอัตโนมัติ:

```latex
\section{ลำดับเลขคณิต (Arithmetic Sequence)}   % ✅ มี (English) → แสดง 2 บรรทัด
\section{สรุป}                                   % ✅ ไม่มี (English) → แสดงบรรทัดเดียว
```

**การทำงาน**:
- ถ้าชื่อ section มีวงเล็บ `(...)` → `\AtBeginSection` จะแสดง:
  - บรรทัดที่ 1: ข้อความก่อน `(` — สีขาว, ฟอนต์ title
  - บรรทัดที่ 2: ข้อความใน `(...)` — สีฟ้าอ่อน (`boxBlue!60`), ฟอนต์ subtitle
- ถ้าไม่มีวงเล็บ → แสดงข้อความทั้งหมดเป็นบรรทัดเดียว (สีขาว, ฟอนต์ title)
- **TOC (สารบัญ)**: แสดงชื่อ section แบบเดิมทั้งบรรทัด ไม่มีการเปลี่ยนแปลง

**ข้อกำหนดในการเขียน `\section`**:
- ใส่ชื่ออังกฤษในวงเล็บ `(...)` ต่อท้ายชื่อไทยเสมอ
- ชื่อไทยห้ามมีวงเล็บ — `xstring` จะใช้ `(` แรกเป็นตัวแบ่ง
- ตัวอย่างที่ถูกต้อง: `\section{การหารลงตัว (Divisibility)}`
- ตัวอย่างที่ไม่ถูกต้อง: `\section{การหารลงตัว (Divisibility) (เพิ่มเติม)}`

## ระบบแบบฝึกหัด (3 ไฟล์)

แต่ละ `exercises/` มี 3 ไฟล์ที่ทำงานร่วมกัน:

```
exercises/
├── problems.tex      # \problem{คำถาม}{คำตอบ} — แหล่งข้อมูลเดียว
├── exercise.tex      # Redefine \problem → แสดงโจทย์ + blank space
└── answer-key.tex    # Redefine \problem → แสดงเฉพาะคำตอบ
```

**วิธีเพิ่มหัวข้อใหม่**: คัดลอก `exercise.tex` และ `answer-key.tex` จากหัวข้อที่มีอยู่
ปรับ preamble path และชื่อ header — จากนั้นเขียนโจทย์ใน `problems.tex`

## การสร้างหัวข้อใหม่

### หัวข้อเดี่ยว (flat)

```bash
mkdir -p <topic>/slides/images <topic>/exercises
cp counting/slides/main.tex <topic>/slides/main.tex
ln -sf ../../.latexmkrc <topic>/slides/.latexmkrc
# แก้ไข title, author, preamble path → ../../preamble.tex
cd <topic>/slides && latexmk -pvc main.tex
```

### หัวข้อย่อย (nested)

```bash
mkdir -p <topic>/<subtopic>/slides/images <topic>/<subtopic>/exercises
cp geometry/triangles/slides/main.tex <topic>/<subtopic>/slides/main.tex
ln -sf ../../../.latexmkrc <topic>/<subtopic>/slides/.latexmkrc
# แก้ไข title, author — preamble path ควรเป็น ../../../preamble.tex
cd <topic>/<subtopic>/slides && latexmk -pvc main.tex
```

## กล่องสี (Colored Box Environments)

มีให้ใช้ 4 แบบในสไลด์ (`preamble.tex`):

| Environment | สี | Title bar | การใช้งาน |
|---|---|---|---|
| `\begin{explanation}[title]...\end{explanation}` | ฟ้า (Blue) | ใช่ | นิยาม, แนวคิด, หมายเหตุ |
| `\begin{exambox}[title]...\end{exambox}` | เขียว (Green) | ใช่ | ตัวอย่าง, โจทย์ |
| `\begin{solbox}[title]...\end{solbox}` | ส้ม (Orange) | ใช่ | วิธีทำ, เฉลย |
| `\begin{intuition}[title]...\end{intuition}` | ม่วง (Purple) | ใช่ | คำอธิบายแบบเข้าใจง่าย |

ทุกกล่องเป็นแบบ breakable (ข้ามหน้าได้) และมี title bar เป็น optional

**ข้อควรระวัง**: ห้ามใช้เครื่องหมาย `,` (comma) และคำสั่งคณิตศาสตร์ใน title — จะทำให้ tcolorbox error

### กล่องเพิ่มเติม (Programming และ Algorithms)

- **Programming**: `\begin{codebox}[title]` (เทา+ฟ้า border, Courier New, `fragile` frame) สำหรับ code listings
- **Algorithms**: `\begin{pseudobox}[title]` (เขียว theme, TH Sarabun New) สำหรับ pseudo code

## Custom Commands

| Command | ที่มา | คำอธิบาย |
|---|---|---|
| `\highlight{text}` | `preamble.tex` | ตัวหนา + สีฟ้า |
| `\py{code}` | Programming `main.tex` | Inline monospace code |
| `\R`, `\N`, `\Z`, `\Q` | ทั้งสอง preamble | Math blackboard bold |
| `\problem{q}{a}` | `exercise-preamble.tex` | นิยามโจทย์ (redefined โดย exercise/answer-key) |
| `\exerciseheader{title}` | `exercise-preamble.tex` | หัวข้อใหญ่ของแบบฝึกหัด |
| `\examsection{title}` | `exercise-preamble.tex` | หัวข้อตอน (ตอนที่ 1, 2, 3) |

## การแก้ปัญหาเบื้องต้น

| ปัญหา | วิธีแก้ |
|---|---|
| `\XeTeXlinebreaklocale{th}` error | ใช้ `% !TEX program = xelatex` magic comment + `.latexmkrc` symlink |
| ฟอนต์ไทยไม่แสดง | ตรวจสอบว่ามี `TH Sarabun New` ใน `fonts/` และ `fc-list :lang=th` |
| tcolorbox comma error | ห้ามมี `,` ใน title — ใช้ช่องว่างหรือ `\quad` แทน |
| `\begin{verbatim}` ใน `\problem` | ใช้ `\begin{codeblock}` หรือ `\begin{pseudocode}` แทน |
| Page overflow | ลด `scale` ของ TikZ, ใช้ `columns` แบ่งเนื้อหา, ย่อข้อความ |
