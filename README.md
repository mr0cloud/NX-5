# NX-5
> A first-of-its-kind visual encoding algorithm that uses **spatial containment** and **edge geometry** to represent text as nested squares

Unlike every encoding system before it — Braille, QR codes, Morse — NX-5 does not use a flat grid of binary states. It encodes meaning through **topology**: the relationship between shapes, their edges and the space they contain.

---

## Table of Contents

  

- [Concept](#concept)

- [How It Works](#how-it-works)

  - [The Base Unit](#the-base-unit)

  - [The Notch System](#the-notch-system)

  - [Side Encoding](#side-encoding)

  - [Word Encoding — Nesting](#word-encoding--nesting)

  - [Sentence Encoding — Clusters](#sentence-encoding--clusters)

- [Full Character Map](#full-character-map)

  - [Letters](#letters)

  - [Digits](#digits)

  - [Symbols](#symbols)

- [Visual Color Guide](#visual-color-guide)

- [Installation](#installation)

- [Usage](#usage)

  - [As a Python Library](#as-a-python-library)

  - [CLI — Encoder](#cli--encoder)

  - [CLI — Decoder](#cli--decoder)

- [Configuration](#configuration)

- [Project Structure](#project-structure)

- [Design Philosophy](#design-philosophy)

- [What Makes NX-5 Different](#what-makes-nx-5-different)


---


## Concept

  

NX-5 was born from iterative geometric art, the observation that a shape drawn inside itself, repeated, creates a natural rhythm. The insight was simple: **what if each iteration represented a letter?**

The result is an encoding system where:

- A **word** is a single cluster of nested squares

- Each **square layer** is one character

- The **edge of the square** carries the encoded data as a notch pattern

- Reading goes from the **inside out**, the innermost square is the first character

Encoded text looks like abstract architectural diagrams or arcane sigils, nothing about the output hints at its content to an untrained eye.

---

## How It Works

### The Base Unit

Every character is a **square** with a notch pattern cut into one of its sides. The square has four sides but only three are used in NX-5:


| Side   | Encodes               |
|--------|-----------------------|
| Left   | Capital letters A–Z   |
| Right  | Lowercase letters a–z |
| Top    | Digits 0–9 + symbols  |
| Bottom | Reserved              |


The side tells you **what type** of character it is, the notch pattern tells you **which** character.

---

### The Notch System


Each active side is divided into **5 equal segments**. Each segment is either:


- **Notched**: sticks outward from the square edge => `1`

- **Flat**: stays flush with the square edge => `0`

5 segments × 2 states = **2⁵ = 32 unique codes**.


```

Side of square (5 segments, top to bottom):

  ┌─ seg 0 ─┐
  ├─ seg 1 ─┤        ◄── flat   (0)
  ├─ seg 2 ──────┐   ◄── notched (1)  ← one full notch, one bit
  ├─ seg 3 ─┤    │   ◄── flat   (0)
  ├─ seg 4 ─┤    │   ◄── flat   (0)
  └─────────┘    │
                 └── notch sticks outward

  Code read top → bottom: 0 0 1 0 0 = 00100

```

  

Segments are read **top to bottom** for left/right sides, and **left to right** for the top side.

  

---

  

### Side Encoding

  

The side the notch appears on encodes the character class:

  

```

LEFT side  (blue)   → Capital letter    e.g. A, B, Z

RIGHT side (red)    → Lowercase letter  e.g. a, b, z

TOP side   (green)  → Digit             e.g. 0, 1, 9

TOP side   (purple) → Symbol            e.g. (, !, @, +

```

  

So `A` and `a` have the **same notch pattern** — they differ only by which side the notch is on.

  

---

  

### Word Encoding - Nesting

  

Characters within a single word are stacked as **concentric nested squares**:

- **Innermost** square → **first** character

- **Outermost** square → **last** character

- Each layer expands outward by a fixed gap

  

```

Word: "abc"

  

┌─────────────────────────────────┐  ← c  (outermost, 3rd)

│                                 │

│  ┌───────────────────────────┐  │

│  │                           ├─┐│  ← b (middle, 2nd)
|
│  │  ┌─────────────────────┐  │ ││
|
│  │  │                     ├─┐│ ││  ← a (innermost, 1st)
|
│  │  │                     │ ││ ││
|
│  │  └─────────────────────┘ ││ ││
|
│  │                          ├─┘│
|
│  └──────────────────────────┘  │
|
│                                │
|
└────────────────────────────────┘

```

  

Reading order: **inside → out**, which mirrors how the shapes are drawn.

  

---

  

### Sentence Encoding — Clusters

  

Each **word** is its own independent nested cluster. Words are placed **left to right** with a gap between them — exactly like normal writing.

  

```

"hello world"

  

[  nested cluster  ]     [  nested cluster  ]

      hello                    world

```

  

There is no special character for spaces — the physical gap between clusters is the separator.

  

---

  

## Full Character Map

  

### Letters

  

Letters A–Z are mapped to binary codes `00001`–`11010` (skipping `00000`).

The same code appears on the **left** for capitals and **right** for lowercase.

  

| Letter | Code  | Letter | Code  | Letter | Code  |
|--------|-------|--------|-------|--------|-------|
| A / a  | 00001 | J / j  | 01010 | S / s  | 10011 |
| B / b  | 00010 | K / k  | 01011 | T / t  | 10100 |
| C / c  | 00011 | L / l  | 01100 | U / u  | 10101 |
| D / d  | 00100 | M / m  | 01101 | V / v  | 10110 |
| E / e  | 00101 | N / n  | 01110 | W / w  | 10111 |
| F / f  | 00110 | O / o  | 01111 | X / x  | 11000 |
| G / g  | 00111 | P / p  | 10000 | Y / y  | 11001 |
| H / h  | 01000 | Q / q  | 10001 | Z / z  | 11010 |
| I / i  | 01001 | R / r  | 10010 |        |       |

  

### Digits

  

Digits use the **top side** (green). `0` is assigned `00000` — the only character that uses the all-flat code.

  

| Digit | Code  | Digit | Code  |
|-------|-------|-------|-------|
| 0     | 00000 | 5     | 00101 |
| 1     | 00001 | 6     | 00110 |
| 2     | 00010 | 7     | 00111 |
| 3     | 00011 | 8     | 01000 |
| 4     | 00100 | 9     | 01001 |

  

### Symbols

  

Symbols use the **top side** (purple). Paired brackets are assigned consecutive codes.

  

| Symbol | Code  | Symbol | Code  |
|--------|-------|--------|-------|
| `(`    | 01010 | `)`    | 01011 |
| `[`    | 01100 | `]`    | 01101 |
| `{`    | 01110 | `}`    | 01111 |
| `<`    | 10000 | `>`    | 10001 |
| `!`    | 10010 | `@`    | 10011 |
| `#`    | 10100 | `$`    | 10101 |
| `%`    | 10110 | `/`    | 10111 |
| `&`    | 11000 | `*`    | 11001 |
| `'`    | 11010 | `"`    | 11011 |
| `-`    | 11100 | `_`    | 11101 |
| `=`    | 11110 | `+`    | 11111 |

  

> The bottom side is reserved for future expansion — punctuation, extended character sets, or alternative scripts.

  

---

  

## Visual Color Guide

  

Colors are the only non-geometric hint in the output. An observer needs to know the color mapping to begin decoding.

  

| Color  | Hex       | Encodes               |
|--------|-----------|-----------------------|
| Blue   | `#2a6db5` | Capital letters A–Z   |
| Red    | `#c0392b` | Lowercase letters a–z |
| Green  | `#27ae60` | Digits 0–9            |
| Purple | `#9b59b6` | Symbols               |

  

---

  

## Installation

  

```bash

pip install NX-5

```

  

**Requirements:** Python 3.8+, Pillow (for PNG export)

  

---

  

## Usage

  

### As a Python Library

  

```python

from NX_5 import encode_text, decode_svg

  

# Encode text to SVG

encode_text("Hello World", "output.svg")

  

# Encode and also export PNG

encode_text("Hello World", "output.svg", export_png=True)

  

# Decode an SVG back to text

text = decode_svg("output.svg")

print(text)  # → "Hello World"

```

  

### CLI — Encoder

  

```bash

# Interactive mode

python -m NX_5.encoder

  

# One-shot

python -m NX_5.encoder Hello World

  

# With PNG export

python -m NX_5.encoder Hello World --png

```

  

```

        ||============================================||

                      NX-5 Encoder v1.0            

          left=CAP - right=lower - top=0-9+symbols  

        ||============================================||

  

Enter text to encode: Hello World

  Also export PNG? (y/N): y

Encoded 'Hello World' → Hello_World.svg

  PNG: Hello_World.png

  Words : 2

  Canvas: 3218 × 1660 px

```

  

### CLI — Decoder

  

```bash

# Interactive mode

python -m NX_5.decoder

  

# One-shot

python -m NX_5.decoder Hello_World.svg

```

  

```
        ||============================================||

                       NX-5 Decoder v1.0            

              reads svg files from the encoder      

        ||============================================||
Decoding: Hello_World.svg

  

  Word 1: 'Hello'

  Word 2: 'World'

  

Decoded: 'Hello World'

```

  

---

  

## Configuration

  

All visual parameters live in `NX_5/constants.py`. Tune them to control the output appearance:

  

| Constant       | Default | Effect                                                            |
|----------------|---------|-------------------------------------------------------------------|
| `INNER_W`      | 160     | Width of the innermost square (minimum size for first letter)     |
| `INNER_H`      | 160     | Height of the innermost square                                    |
| `LAYER_GAP`    | 150     | Space between each nested layer — increase for more breathing room |
| `NOTCH_DEPTH`  | 40      | How far notches protrude from the square edge                     |
| `SEGMENTS`     | 5       | Number of notch segments per side — do not change                 |
| `PADDING`      | 50      | Outer padding around each word cluster                            |
| `WORD_GAP`     | 50      | Horizontal gap between word clusters                              |
| `BG_PAD`       | 80      | Canvas margin around the entire output                            |
| `STROKE_WIDTH` | 7       | Line thickness of the squares                                     |

  

> **Rule of thumb:** `LAYER_GAP` should be at least `NOTCH_DEPTH × 3` to keep notches from looking cramped between layers.

  

---

  

## Project Structure

  

```

NX-5/

├── NX_5/

│   ├── __init__.py      ← exposes encode_text, decode_svg

│   ├── constants.py     ← all mappings, colors, layout values

│   ├── encoder.py       ← SVG + PNG generation

│   └── decoder.py       ← geometry-based SVG decoding

├── tests/

│   └── test_encoder.py  ← roundtrip encode → decode tests

├── pyproject.toml

├── LICENSE

└── README.md

```

  

---

  

## Design Philosophy

  

**Topology over grid.** Every encoding system before NX-5 encodes data in a flat matrix of binary states - dots on a grid, pits on a disc, pixels in a QR code. NX-5 encodes data through *containment* and *edge features*. The meaning lives in the relationship between shapes, not in a flat matrix.


**Reading order is spatial.** You read from the inside out, which mirrors the natural visual pull of nested shapes. The eye is drawn to the center first.


**Case is positional.** There is no separate symbol for uppercase versus lowercase. The same notch pattern on the left means capital; on the right means lowercase. One rule, zero ambiguity.
  

**The output is art.** A rendered NX-5 message looks like a technical diagram or a geometric sigil. It does not look like ciphertext. This is intentional — the best encoding is one that doesn't announce itself.



**Zero metadata.** The SVG output contains nothing but raw path coordinates and stroke colors. No labels, no attributes, no hints. Decoding requires knowing the system.

  

---

  

## What Makes NX-5 Different

  

| Property                 | QR Code | Braille | Morse | NX-5 |
|--------------------------|---------|---------|-------|------|
| Grid-based               | ✓       | ✓       | ✗     | ✗    |
| Requires special reader  | ✓       | ✗       | ✗     | ✗    |
| Encodes case             | ✓       | ✓       | ✗     | ✓    |
| Visually artistic        | ✗       | ✗       | ✗     | ✓    |
| Topology-based           | ✗       | ✗       | ✗     | ✓    |
| Readable without tools   | ✗       | ✗       | ✓     | ✓*   |
| Scalable vector output   | ✗       | ✗       | ✗     | ✓    |

  

*with knowledge of the system

  

---

  

## License

  

MIT — see [LICENSE](LICENSE)
