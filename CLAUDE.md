# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build

Each slide deck builds independently from its `slides/` directory:

```bash
# Navigate to the topic's slides folder (2 or 3 levels deep)
cd functions/relations/slides   # subtopic (3 levels: functions/<subtopic>/slides/)
cd counting/slides              # flat topic (2 levels: <topic>/slides/)

# Compile with watch mode (auto-rebuilds on save)
latexmk -pvc main.tex

# Or single compilation
latexmk main.tex

# Direct XeLaTeX (if latexmk is unavailable)
xelatex -no-pdf -interaction=nonstopmode main.tex
xdvipdfmx main.xdv
```

**XeLaTeX is required** (not pdfLaTeX) because `fontspec` is used for Thai font support via TH Sarabun New. The font must be installed system-wide (`fc-list :lang=th` to verify).

**Important**: Each `main.tex` must start with `% !TEX program = xelatex` — this magic comment tells latexmk to use XeLaTeX. Without it (and without the `.latexmkrc` symlink), latexmk defaults to lualatex which fails on `\XeTeXlinebreaklocale{th}`.

**Important**: Each `slides/` directory has a symlink `.latexmkrc -> ../../.latexmkrc` (flat topics) or `../../../.latexmkrc` (subtopics). This is required because latexmk only searches the current directory and its immediate parent for config files. The symlink ensures `$pdf_mode = 5` (force XeLaTeX) is found.

Build artifacts (`.aux`, `.log`, `.xdv`, etc.) are generated in the current directory — clean with `latexmk -C`.

### Exercises

Each topic also has an `exercises/` directory with a three-file system:

```bash
cd functions/relations/exercises

# Compile the problem set (questions with blank space)
xelatex -no-pdf -interaction=nonstopmode exercise.tex
xdvipdfmx exercise.xdv

# Compile the answer key (answers only, no solutions)
xelatex -no-pdf -interaction=nonstopmode answer-key.tex
xdvipdfmx answer-key.xdv
```

**Exercise file structure**:
```
exercises/
├── problems.tex      # Problem definitions (\problem{question}{answer})
├── exercise.tex      # Renders problems + blank space for student work
└── answer-key.tex    # Renders just question numbers + answers
```

The key insight: `problems.tex` defines every problem once using `\problem{question}{answer}`. Both `exercise.tex` and `answer-key.tex` redefine `\problem` differently and then `\input{problems.tex}`:
- `exercise.tex` → shows question + `\vspace{4cm}` + `\hrule`
- `answer-key.tex` → shows only `\theqnum. answer`

The shared `exercise-preamble.tex` at the repo root provides A4/16pt formatting matching the real POSN exam paper style.

## Architecture

Multi-topic Beamer slide repository for POSN Computer qualification exam preparation. Each topic is a self-contained folder with its own `slides/` and `exercises/` subdirectories. All slide decks share a single `preamble.tex` at the repo root.

```
preamble.tex               — Shared beamer preamble (theme, fonts, colors, tcolorbox environments, custom commands)
exercise-preamble.tex      — Shared exercise styling (A4, 16pt, matching POSN exam paper)
fonts/THSarabunNew/        — 4 .ttf font files (Regular, Bold, Italic, BoldItalic)
.latexmkrc                 — Build config (XeLaTeX → xdvipdfmx, no PDF viewer)
.gitignore                 — Ignores LaTeX build artifacts

Flat topic:
  <topic>/
    slides/
      main.tex             — Slide deck (inputs ../../preamble.tex)
      .latexmkrc           — symlink → ../../.latexmkrc
      images/
    exercises/
      problems.tex         — Problem definitions (\problem{q}{a})
      exercise.tex         — Problem set with blank space (inputs ../../../exercise-preamble.tex)
      answer-key.tex       — Answer key, answers only (inputs ../../../exercise-preamble.tex)

Nested subtopics (e.g., functions/):
  <topic>/
    README.md              — Overview, prerequisites, learning order
    <subtopic>/
      README.md            — Per-subtopic prerequisites
      slides/
        main.tex           — Slide deck (inputs ../../../preamble.tex)
        .latexmkrc         — symlink → ../../../.latexmkrc
        images/
      exercises/
        problems.tex
        exercise.tex
        answer-key.tex
```

### How preambles and slides connect

Each `main.tex` loads the shared preamble via `\input{../../preamble.tex}` (relative path from `slides/` to root). The preamble sets up:
- **Beamer theme**: Madrid (light), custom blue accent colors
- **Font path**: `../../fonts/THSarabunNew/` (relative from `slides/` via `main.tex`)
- **Colored boxes** via `tcolorbox`

### Custom LaTeX environments (defined in preamble.tex)

| Environment | Color | Usage |
|---|---|---|
| `\begin{explanation}[title]...\end{explanation}` | Blue | Definitions, concepts, notes |
| `\begin{exambox}[title]...\end{exambox}` | Green | Worked examples, problems |
| `\begin{solbox}[title]...\end{solbox}` | Orange | Solutions, answers |
| `\begin{intuition}[title]...\end{intuition}` | Purple | Plain-language explanations, analogies |

Each environment takes an optional title: `\begin{explanation}[นิยาม]`. Boxes are breakable (can span slides).

### Custom LaTeX commands (defined in preamble.tex)

| Command | Purpose |
|---|---|
| `\highlight{text}` | Bold blue text for emphasis |
| `\R`, `\N`, `\Z`, `\Q` | Math blackboard bold shortcuts |

### Slide authoring conventions

When creating content slides, follow this structure:

```latex
% Definitions use the blue explanation box
\begin{frame}{Slide Title}
  \begin{explanation}[นิยาม: Concept Name]
    Definition and explanation text
  \end{explanation}

  \begin{intuition}[เข้าใจง่าย ๆ]
    Plain-language explanation or analogy — no formal math notation.
    "คิดง่าย ๆ เหมือน..."
  \end{intuition}
\end{frame}

% Examples use the green exambox
\begin{frame}{ตัวอย่าง: Topic Name}
  \begin{exambox}[โจทย์]
    Problem statement
  \end{exambox}

  \begin{solbox}[วิธีทำ]
    Step-by-step solution
  \end{solbox}
\end{frame}
```

### Beamer slide types available

- **Title slide**: `\begin{frame}[plain]\titlepage\end{frame}` — uses `\title`, `\subtitle`, `\author` set in preamble area. Do NOT use `\date` (omitted for clean look).
- **Outline**: `\begin{frame}{สารบัญ (Outline)}\tableofcontents\end{frame}` — auto-generated from `\section{}` and `\subsection{}` commands
- **Section header**: Auto-inserted by beamer when `\section{}` appears before a frame
- **Subsection header**: Auto-inserted via `\AtBeginSubsection` hook (defined in preamble.tex) — shows a clean centered title slide
- **Prerequisites slide**: Include a `ก่อนเรียน (Prerequisites)` frame after the outline listing what students should know
- **Content**: Standard `\begin{frame}{Title}...\end{frame}`
- **Two-column**: Use `\begin{columns}[T]...\column{0.45\textwidth}...\end{columns}`

### Adding a new topic (flat)

```bash
mkdir -p <topic>/slides/images <topic>/exercises
cp functions/relations/slides/main.tex <topic>/slides/main.tex
# Edit: title, author, adjust preamble path (../../../preamble.tex → ../../preamble.tex)
# Create .latexmkrc symlink: ln -sf ../../.latexmkrc <topic>/slides/.latexmkrc
cd <topic>/slides && latexmk -pvc main.tex
```

### Adding a new subtopic (nested under existing topic)

```bash
mkdir -p <topic>/<subtopic>/slides/images <topic>/<subtopic>/exercises
cp functions/polynomials/slides/main.tex <topic>/<subtopic>/slides/main.tex
# Edit: title, author, content
# Create .latexmkrc symlink: ln -sf ../../../.latexmkrc <topic>/<subtopic>/slides/.latexmkrc
cd <topic>/<subtopic>/slides && latexmk -pvc main.tex
```

### Adding exercises for a new topic

```bash
# Copy the exercise template files from another topic
cp -r functions/relations/exercises <topic>/exercises
# Edit problems.tex with your questions — ordered easy → hard
# Compile
cd <topic>/exercises
xelatex -no-pdf -interaction=nonstopmode exercise.tex && xdvipdfmx exercise.xdv
xelatex -no-pdf -interaction=nonstopmode answer-key.tex && xdvipdfmx answer-key.xdv
```

### Document settings (slides)

- 16:9 aspect ratio, 11pt base font
- TH Sarabun New at Scale=1.35
- Line spread 1.2 for Thai readability
- Page numbers in footer: "X / Y" format
- Footer: author name (left), short title (center), page numbers (right)
- Navigation symbols removed for clean look

### Document settings (exercises)

- A4 paper, 16pt base font (article class)
- TH Sarabun New at Scale=1.5
- Line spread 1.25 for Thai readability
- Page numbers in header, right-aligned
- Exercise footer: "แบบฝึกหัด สอวน. คอมฯ โดย ..."
