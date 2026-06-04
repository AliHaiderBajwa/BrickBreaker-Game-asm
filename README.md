# 🧱 Brick Breaker — x86 Assembly
### COAL Project · FAST-NUCES Islamabad · Spring 2026

> A fully-featured Brick Breaker game written entirely in **8086 x86 Assembly (MASM)**, running in **VGA Mode 13h** (320×200, 256 colours) under DOSBox. Built from scratch across three iterations — no high-level languages, no libraries.

---

##  Team
| Ali Haider Bajwa |
| Jehangir         |
| Taimoor          |
| Abdul Wadood     |

**Course:** Computer Organization and Assembly Language (COAL)

---

## Game Features

### Core Gameplay
- **Paddle** controlled via keyboard (A/D, arrow keys) **or mouse** — both work simultaneously
- **Ball physics** with wall, ceiling, and paddle bounce
- **4×8 brick grid** per level with collision detection and destruction
- **3 Lives** — miss the ball and lose a life; lose all three and it's Game Over

### Three Levels of Increasing Difficulty

| Level | Brick Layout | Ball Speed | Bricks |
|---|---|---|---|
| 1 | Full 4×8 grid | DX=±2, DY=−2 | 32 |
| 2 | Corner-gap layout | DX=±2, DY=−3 | 24 |
| 3 | Sparse checkerboard | DX=±3, DY=−3 | 20 |

Level-sensitive paddle bounce: the speed reflected off the paddle scales with the current level and the zone of the paddle struck (Far Left / Mid Left / Centre / Mid Right / Far Right).

### Power-Up / Bonus System
Bonuses spawn randomly (25% chance) on brick destruction, fall toward the paddle, and must be caught to activate. Only one bonus falls at a time. Types cycle in round-robin order so all four appear equally:

| Colour | Type | Effect |
|---|---|---|
| 🟢 Green | Slow Ball | Sets \|DX\|, \|DY\| = 2 |
| 🔴 Red | Fast Ball | Sets \|DX\|, \|DY\| = 3 |
| 🟡 Yellow | Extra Life | +1 life (max 9) |
| 🔵 Cyan | Wide Paddle | +20 px width (max 100 px) |

### Complete Game Flow
```
Splash Screen → Name Entry → Main Menu
                                 ├── Level Select → Level 1 → Level 2 → Level 3 → YOU WIN!
                                 ├── Instructions
                                 ├── High Scores
                                 └── Exit
                              (Game Over at any level → back to Menu)
```

### Bonus Features (Iteration 3)
- **Pause / Resume** — press `P` at any time; game state is fully preserved
- **Mouse support** — INT 33h polling maps cursor X directly to paddle position
- **Ball trail effect** — 3-frame motion trail (dark grey → light grey → white) for visual speed feedback
- **Persistent high score** — saved to `score.dat` via DOS INT 21h file I/O; survives between sessions
- **Level Select screen** — start from any level, not forced to begin at Level 1
- **HUD** — live score, lives, and current level displayed during gameplay

---

## 🗂 Repository Structure

```
brick-breaker-asm/
├── game.asm          # Full source — 2630 lines of x86 MASM assembly
├── score.dat         # Created at runtime; stores high score & player name
├── docs/
│   ├── COAL_Project_Statement.pdf   # Assignment brief
│   └── Iteration3_Documentation.pdf  # Technical report (level design, bonus logic, game flow)
└── README.md
```

---

## ⚙️ How to Run

### Requirements
- [DOSBox](https://www.dosbox.com/) (any recent version)
- [MASM 6.x](https://en.wikipedia.org/wiki/Microsoft_Macro_Assembler) (`MASM.EXE` + `LINK.EXE`)

### Steps

```bash
# 1. Mount your project folder in DOSBox
mount c C:\path\to\BrickBreaker-Game-asm
c:

# 2. Assemble
masm game.asm;

# 3. Link
link game.obj;

# 4. Run
game.exe
```

> **Note:** The game targets real-mode DOS. Run exclusively inside DOSBox — do not attempt to run the `.exe` natively on Windows/Linux.

---

## 🏗 Technical Architecture

### Graphics
All rendering is done via **direct VGA memory writes** to segment `A000h`. No BIOS graphics interrupts are used for pixel-level drawing. Rectangles, text glyphs, and the brick grid are all written byte-by-byte into video memory.

### Anti-Flicker Strategy
A `SavePreviousPositions` snapshot captures the paddle, ball, and bonus coordinates (including the 3-frame ball trail) each frame. `DrawGameFrame` erases only the previous bounding boxes with black fills before redrawing at new positions — avoiding full-screen redraws and eliminating flicker.

### Pseudorandom Number Generation
A lightweight seed (`randSeed`) is incremented every frame by `UpdateBall`. Bonus spawn uses `(randSeed AND 03h) = 0` as a gate (~25% probability), and `brickIndex + currentLevel` is mixed into the seed to prevent predictable patterns.

### File I/O
High score persistence uses **DOS INT 21h** (functions `3Ch` create, `3Dh` open, `3Fh` read, `40h` write, `3Eh` close) to read and write `score.dat` — demonstrating file handling entirely in assembly.

### Input
- **Keyboard:** INT 16h non-blocking poll (`AH=01h` check + `AH=00h` read)
- **Mouse:** INT 33h function 3 returns cursor position each frame; X is halved from hardware coordinates and clamped to playfield bounds

---

## 📐 Key Procedures

| Procedure | Purpose |
|---|---|
| `InitLevel` | Reset ball, paddle, bricks, trail, bonus for the current level |
| `ApplyLevelLayout` | Carve level-specific gaps into the brick grid |
| `SetLevelSpeed` | Assign `ballDX`/`ballDY` based on current level |
| `GameLoop` | Per-frame loop: input → update ball → update bonus → draw → delay |
| `CheckBrickCollision` | AABB test against all active bricks; triggers `TrySpawnBonus` |
| `TrySpawnBonus` | 25% random spawn gate; positions bonus at destroyed brick centre |
| `CheckBonusPaddle` | AABB test between bonus rectangle and paddle |
| `ApplyBonus` | Dispatches to `BonusSlowBall`, `BonusFastBall`, `BonusExtraLife`, `BonusWidePaddle` |
| `DrawBall` | Renders 3-shade trail + current ball position |
| `LoadScore` / `SaveScore` | DOS file I/O for persistent high score |
| `TogglePause` / `PauseLoop` | Blocks all updates until `P` is pressed again |

---

## 📄 License

This project was developed as an academic submission for the Computer Organization and Assembly Language course at FAST-NUCES Islamabad. Free to reference for educational purposes.
